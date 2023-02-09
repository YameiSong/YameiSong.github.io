---
layout: post
title:  "实现 ATM Actor 模型（三）"
date:   2023-02-09 15：34 +0800
categories: message_queue
---

# 基于消息队列实现 ATM Actor 模型（三）

 > 参考：C++ Concurrency In Action

## 状态机

### ATM

主循环 `run`。从 `state=&atm::waiting_for_card;` 开始，等待消息，更新 `state`。一次操作以 `done_processing()` 结束，状态恢复为 `state=&atm::waiting_for_card;`。

```cpp
class atm
{
  messaging::receiver incoming;
  messaging::sender bank;
  messaging::sender interface_hardware;

  void (atm::*state)(); // state：指向下一个状态相应函数的指针

  std::string account;
  unsigned withdrawal_amount;
  std::string pin;

  void process_withdrawal() 
  {
    incoming.wait()
      .handle<withdraw_ok>(
       [&](withdraw_ok const& msg)
       {
         interface_hardware.send(
           issue_money(withdrawal_amount));

         bank.send(
           withdrawal_processed(account, withdrawal_amount));

         state=&atm::done_processing;
       })
      .handle<withdraw_denied>(
       [&](withdraw_denied const& msg)
       {
         interface_hardware.send(display_insufficient_funds());

         state=&atm::done_processing;
       })
      .handle<cancel_pressed>(
       [&](cancel_pressed const& msg)
       {
         bank.send(
           cancel_withdrawal(account, withdrawal_amount));

         interface_hardware.send(
           display_withdrawal_cancelled());

         state=&atm::done_processing;
       });
   }

  void process_balance()
  {
    incoming.wait()
      .handle<balance>(
       [&](balance const& msg)
       {
         interface_hardware.send(display_balance(msg.amount));

         state=&atm::wait_for_action;
       })
      .handle<cancel_pressed>(
       [&](cancel_pressed const& msg)
       {
         state=&atm::done_processing;
       });
  }

  void wait_for_action()
  {
    interface_hardware.send(display_withdrawal_options());

    incoming.wait()
      .handle<withdraw_pressed>(
       [&](withdraw_pressed const& msg)
       {
         withdrawal_amount=msg.amount;
         bank.send(withdraw(account, msg.amount, incoming));
         state=&atm::process_withdrawal;
       })
      .handle<balance_pressed>(
       [&](balance_pressed const& msg)
       {
         bank.send(get_balance(account, incoming));
         state=&atm::process_balance;
       })
      .handle<cancel_pressed>(
       [&](cancel_pressed const& msg)
       {
         state=&atm::done_processing;
       });
  }

  void verifying_pin()
  {
    incoming.wait()
      .handle<pin_verified>(
       [&](pin_verified const& msg)
       {
         state=&atm::wait_for_action;
       })
      .handle<pin_incorrect>(
       [&](pin_incorrect const& msg)
       {
         interface_hardware.send(
         display_pin_incorrect_message());
         state=&atm::done_processing;
       })
      .handle<cancel_pressed>(
       [&](cancel_pressed const& msg)
       {
         state=&atm::done_processing;
       });
  }

  void getting_pin()
  {
    incoming.wait()
      .handle<digit_pressed>(
       [&](digit_pressed const& msg)
       {
         unsigned const pin_length=4;
         pin+=msg.digit;

         if(pin.length()==pin_length)
         {
           bank.send(verify_pin(account,pin,incoming));
           state=&atm::verifying_pin;
         }
       })
      .handle<clear_last_pressed>(
       [&](clear_last_pressed const& msg)
       {
         if(!pin.empty())
         {
           pin.pop_back();
         }
       })
      .handle<cancel_pressed>(
       [&](cancel_pressed const& msg)
       {
         state=&atm::done_processing;
       });
  }

  void waiting_for_card()
  {
    interface_hardware.send(display_enter_card());

    incoming.wait()
      .handle<card_inserted>(
       [&](card_inserted const& msg)
       {
         account=msg.account;
         pin="";
         interface_hardware.send(display_enter_pin());
         state=&atm::getting_pin;
       });
  }

  void done_processing()
  {
    interface_hardware.send(eject_card());
    state=&atm::waiting_for_card;
  }

  atm(atm const&)=delete;
  atm& operator=(atm const&)=delete;
public:
  atm(messaging::sender bank_,
      messaging::sender interface_hardware_):
  bank(bank_),interface_hardware(interface_hardware_)
  {}

  void done()
  {
    get_sender().send(messaging::close_queue());
  }

  void run()
  {
    state=&atm::waiting_for_card;
    try
    {
      for(;;)
      {
        (this->*state)();
      }
    }
    catch(messaging::close_queue const&)
    {
    }
  }

  messaging::sender get_sender()
  {
    return incoming;
  } 
};
```

### 银行

银行状态机比较简单，只维护一个 `balance` 状态，并且该状态仅受当前消息影响。只要一直等待消息，更新 `balance`，然后回复消息就行了。

```cpp
class bank_machine
{
  messaging::receiver incoming;
  unsigned balance;
public:
  bank_machine(): balance(199)
  {}

  void done()
  {
    get_sender().send(messaging::close_queue());
  }

  void run()
  {
    try
    {
      for(;;)
      {
        incoming.wait()
          .handle<verify_pin>(
           [&](verify_pin const& msg)
           {
             if(msg.pin=="1937")
             {
               msg.atm_queue.send(pin_verified());
             }
             else
             {
               msg.atm_queue.send(pin_incorrect());
             }
           })
          .handle<withdraw>(
           [&](withdraw const& msg)
           {
             if(balance>=msg.amount)
             {
               msg.atm_queue.send(withdraw_ok());
               balance-=msg.amount;
             }
             else
             {
               msg.atm_queue.send(withdraw_denied());
             }
           })
          .handle<get_balance>(
           [&](get_balance const& msg)
           {
             // 这里有2个balance:
             // 1. 消息类型，全局变量
             // 2. bank_machine的成员变量
             // ::balance 表示全局作用域里的 balance
             msg.atm_queue.send(::balance(balance));
           })
          .handle<withdrawal_processed>(
           [&](withdrawal_processed const& msg)
           {
           })
          .handle<cancel_withdrawal>(
           [&](cancel_withdrawal const& msg)
           {
           });
      }
    }
    catch(messaging::close_queue const&)
    {
    }
  }

  messaging::sender get_sender()
  {
  return incoming;
  }
};
```

### 用户

用户状态机比较简单，只负责显示操作提示。只要一直等待消息，打印信息。

```cpp
class interface_machine
{
  messaging::receiver incoming;
public:
  void done()
  {
    get_sender().send(messaging::close_queue());
  }

  void run()
  {
    try
    {
      for(;;)
      {
        incoming.wait()
          .handle<issue_money>(
           [&](issue_money const& msg)
           {
             {
               std::lock_guard<std::mutex> lk(iom);
               std::cout<<"Issuing "<<msg.amount<<std::endl;
             }
           })
          .handle<display_insufficient_funds>(
           [&](display_insufficient_funds const& msg)
           {
             {
               std::lock_guard<std::mutex> lk(iom);
               std::cout<<"Insufficient funds"<<std::endl;
             }
           })
          .handle<display_enter_pin>(
           [&](display_enter_pin const& msg)
           {
             {
               std::lock_guard<std::mutex> lk(iom);
               std::cout<<"Please enter your PIN (0-9)"<<std::endl;
             }
           })
          .handle<display_enter_card>(
           [&](display_enter_card const& msg)
           {
             {
               std::lock_guard<std::mutex> lk(iom);
               std::cout<<"Please enter your card (I)"
                 <<std::endl;
             }
           })
          .handle<display_balance>(
           [&](display_balance const& msg)
           {
             {
               std::lock_guard<std::mutex> lk(iom);
               std::cout
                 <<"The balance of your account is "
                 <<msg.amount<<std::endl;
             }
           })
          .handle<display_withdrawal_options>(
           [&](display_withdrawal_options const& msg)
           {
             {
               std::lock_guard<std::mutex> lk(iom);
               std::cout<<"Withdraw 50? (w)"<<std::endl;
               std::cout<<"Display Balance? (b)"
                 <<std::endl;
               std::cout<<"Cancel? (c)"<<std::endl;
             }
           })
          .handle<display_withdrawal_cancelled>(
           [&](display_withdrawal_cancelled const& msg)
           {
             {
               std::lock_guard<std::mutex> lk(iom);
               std::cout<<"Withdrawal cancelled"
                 <<std::endl;
             }
           })
          .handle<display_pin_incorrect_message>(
           [&](display_pin_incorrect_message const& msg)
           {
             {
               std::lock_guard<std::mutex> lk(iom);
               std::cout<<"PIN incorrect"<<std::endl;
             }
           })
          .handle<eject_card>(
           [&](eject_card const& msg)
           {
             {
               std::lock_guard<std::mutex> lk(iom);
               std::cout<<"Ejecting card"<<std::endl;
             }
           });
      }
    }
    catch(messaging::close_queue&)
    {
    }
  }

  messaging::sender get_sender()
  {
    return incoming;
  }
};
```

## 驱动

```cpp
int main()
{
  bank_machine bank;
  interface_machine interface_hardware;

  atm machine(bank.get_sender(), interface_hardware.get_sender());

  std::thread bank_thread(&bank_machine::run, &bank);
  std::thread if_thread(&interface_machine::run, &interface_hardware);
  std::thread atm_thread(&atm::run, &machine);

  messaging::sender atmqueue(machine.get_sender());

  bool quit_pressed=false;

  while(!quit_pressed)
  {
    char c=getchar();
    switch(c)
    {
    case '0':
    case '1':
    case '2':
    case '3':
    case '4':
    case '5':
    case '6':
    case '7':
    case '8':
    case '9':
      atmqueue.send(digit_pressed(c));
      break;
    case 'b':
      atmqueue.send(balance_pressed());
      break;
    case 'w':
      atmqueue.send(withdraw_pressed(50));
      break;
    case 'c':
      atmqueue.send(cancel_pressed());
      break;
    case 'q':
      quit_pressed=true;
      break;
    case 'i':
      atmqueue.send(card_inserted("acc1234"));
      break;
    }
  }

  bank.done();
  machine.done();
  interface_hardware.done();

  atm_thread.join();
  bank_thread.join();
  if_thread.join();
}
```