---
layout: post
title:  "实现 ATM Actor 模型"
date:   2023-02-08 11:22 +0800
categories: message_queue
---

# 基于消息队列实现 ATM Actor 模型

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
      return dispatcher(&q);
    }
  };
}
```