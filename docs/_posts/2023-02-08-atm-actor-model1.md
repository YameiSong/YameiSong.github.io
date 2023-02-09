---
layout: post
title:  "实现 ATM Actor 模型（一）"
date:   2023-02-08 11:22 +0800
categories: message_queue
---

# 基于消息队列实现 ATM Actor 模型（一）

 > 参考：C++ Concurrency In Action

## 实现消息队列

在 messaging 命名空间中定义。

### 基础设施

* message_base：消息基类
* wrapped_message：消息模板派生类。衍生出不同类型的消息。
* queue：线程安全的消息队列。

```cpp
#include <mutex>
#include <condition_variable>
#include <queue>
#include <memory>

namespace messaging
{
  struct message_base
  {
    virtual ~message_base() {}
  };

  template<typename Msg>
  struct wrapped_message: message_base
  {
    Msg contents;

    explicit wrapped_message(Msg const& contents_): contents(contents_) {}
  };

  class queue
  {
    std::mutex m;
    std::condition_variable c;
    std::queue<std::shared_ptr<message_base> > q;  // 存储指向message_base类对象的指针
  public:
    template<typename T>
    void push(T const& msg) // 存储消息
    {
      std::lock_guard<std::mutex> lk(m);
      q.push(std::make_shared<wrapped_message<T> >(msg)); // 构造 wrapped_message<T> 类型的消息，转换成 message_base类型，存储到队列中
      c.notify_all();
    }

    std::shared_ptr<message_base> wait_and_pop() // 读取消息
    {
      std::unique_lock<std::mutex> lk(m);
      c.wait(lk,[&]{return !q.empty();});  // 当队列为空时阻塞
      auto res=q.front();
      q.pop();
      return res;
    }
  };
}
```

### 上层建筑

* 发送端组件：sender
* 接收端组件：receiver, dispatcher（只能处理close_queue消息）, TemplateDispatcher（处理其他特定消息）

调用链：
1. sender.send()发送消息。
2. receiver.wait()创建dispatcher作为响应函数链的源点，处理close_queue消息。
3. 根据业务逻辑，在receiver.wait()后调用handle\<Msg, Func\>(Func&& f)来链接处理指定消息。因为可以通过handle的形参来推导Func的实际类型，所以调用handle的时候可以简写成handle\<Msg\>(Func&& f)。

#### sender 类

```cpp
namespace messaging
{
  class sender
  {
    queue *q;  // sender是一个队列指针的包装类
  public:
    sender(): q(nullptr) {} // sender无队列(默认构造函数)
    explicit sender(queue *q_): q(q_) {} // 从指向队列的指针进行构造

    template<typename Message>
    void send(Message const& msg)
    {
      if(q)
      {
        q->push(msg);  // 将要发送的信息推送给队列
      }
    }
  };
}
```

### receiver 类

```cpp
namespace messaging
{
  class receiver
  {
    queue q;  // 接受者拥有对应队列
  public:
    // 允许将类中队列隐式转化为一个sender队列，如
    // receiver x;
    // sender y {x}; 或 sender y = x;
    operator sender()
    {
      return sender(&q);
    }
    dispatcher wait()  // 等待对队列进行调度
    {
      return dispatcher(&q); // 返回临时变量。在该变量即将被销毁时，调用dispatcher的析构函数
    }
  };
}
```

### dispatcher 类

```cpp
namespace messaging
{
  class close_queue  // 用于关闭队列的消息
  {};

  class dispatcher
  {
    queue* q;
    bool chained;

    dispatcher(dispatcher const&)=delete;  // dispatcher实例不能被拷贝
    dispatcher& operator=(dispatcher const&)=delete;

    template<
      typename Dispatcher,
      typename Msg,
      typename Func>
    friend class TemplateDispatcher; // 允许TemplateDispatcher实例访问内部成员

    void wait_and_dispatch()
    {
      for(;;)  // 循环等待调度消息
      {
        auto msg=q->wait_and_pop();
        dispatch(msg);
      }
    }

    bool dispatch(std::shared_ptr<message_base> const& msg) // 检查close_queue消息
    {
      // dynamic_cast结果：true，msg是close_queue类型；false，msg不是该类型
      if(dynamic_cast<wrapped_message<close_queue>*>(msg.get())) 
      {
        throw close_queue(); // 抛出close_queue消息
      }
      return false; // 没有找到可以处理消息的dispatcher，消息被丢弃
    }
  public:
    dispatcher(dispatcher&& other):  // dispatcher实例可以移动
      q(other.q),chained(other.chained)
    {
      other.chained=true;  // 源不能等待消息
    }

    explicit dispatcher(queue* q_):
      q(q_),chained(false)
    {}

    // 使用TemplateDispatcher处理指定类型的消息
    // 模板偏特化，指定了dispatcher
    template<typename Message, typename Func>
    TemplateDispatcher<dispatcher, Message, Func> handle(Func&& f)
    {
      return TemplateDispatcher<dispatcher, Message, Func>(
        q, this, std::forward<Func>(f));
    }

    ~dispatcher() noexcept(false)  // 析构函数可能会抛出异常。默认情况下析构函数是noexcept(true)，抛出异常会导致程序终止
    {  
      if(!chained)
      {
        wait_and_dispatch();
      }
    }
  };
}
```

### TemplateDispatcher类模板

```cpp
namespace messaging
{
  template<typename PreviousDispatcher,typename Msg,typename Func>
  class TemplateDispatcher
  {
    queue* q;
    PreviousDispatcher* prev;
    Func f;
    bool chained;

    TemplateDispatcher(TemplateDispatcher const&)=delete;
    TemplateDispatcher& operator=(TemplateDispatcher const&)=delete;

    template<typename Dispatcher, typename OtherMsg, typename OtherFunc>
    friend class TemplateDispatcher;  // 所有特化的TemplateDispatcher类型实例都是友元类

    void wait_and_dispatch()
    {
      for(;;)
      {
        auto msg=q->wait_and_pop();
        if(dispatch(msg))  // 如果消息处理过后，会跳出循环
          break;
      }
    }

    bool dispatch(std::shared_ptr<message_base> const& msg)
    {
      if(wrapped_message<Msg>* wrapper=
         dynamic_cast<wrapped_message<Msg>*>(msg.get()))  // 检查消息类型，并且调用函数
      {
        f(wrapper->contents);
        return true;
      }
      else
      {
        return prev->dispatch(msg);  // 链接到之前的调度器上。不断沿着调用链向前寻找可以处理消息的dispatcher
      }
    }
  public:
    TemplateDispatcher(TemplateDispatcher&& other):
        q(other.q), prev(other.prev), f(std::move(other.f)),
        chained(other.chained)
    {
      other.chained=true;
    }
    TemplateDispatcher(queue* q_, PreviousDispatcher* prev_, Func&& f_):
        q(q_), prev(prev_), f(std::forward<Func>(f_)), chained(false)
    {
      prev_->chained=true;
    }

    template<typename OtherMsg,typename OtherFunc>
    TemplateDispatcher<TemplateDispatcher, OtherMsg, OtherFunc> handle(OtherFunc&& of)  // 4 可以链接其他处理器
    {
      return TemplateDispatcher<
          TemplateDispatcher, OtherMsg, OtherFunc>(
          q, this, std::forward<OtherFunc>(of));
    }

    ~TemplateDispatcher() noexcept(false)  // 5 这个析构函数也是noexcept(false)的
    {
      if(!chained)
      {
        wait_and_dispatch();
      }
    }
  };
}
```
