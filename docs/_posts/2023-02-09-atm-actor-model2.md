---
layout: post
title:  "实现 ATM Actor 模型（二）"
date:   2023-02-09 11:35 +0800
categories: message_queue
---

# 基于消息队列实现 ATM Actor 模型（二）

 > 参考：C++ Concurrency In Action

## 定义消息类型

这些 ATM 消息将会以 `messaging::wrapped_message<MsgType>` 的形式被使用。

#### 为什么 atm_queue 是 mutable？

mutable 有两种用法：
1. 在 const 函数中改变某个变量，需要将这个变量声明成 mutable。
2. 在 lambda 函数中修改值传递[=]的变量，需要将这个变量声明成 mutable。

但下面消息里面的 `mutable messaging::sender atm_queue` 好像都不符合。

```cpp
struct withdraw
{
  std::string account;
  unsigned amount;
  mutable messaging::sender atm_queue;

  withdraw(std::string const& account_,
           unsigned amount_,
           messaging::sender atm_queue_):
    account(account_), amount(amount_), atm_queue(atm_queue_)
  {}
};

struct withdraw_ok
{};

struct withdraw_denied
{};

struct cancel_withdrawal
{
  std::string account;
  unsigned amount;
  cancel_withdrawal(std::string const& account_,
                    unsigned amount_):
    account(account_),amount(amount_)
  {}
};

struct withdrawal_processed
{
  std::string account;
  unsigned amount;
  withdrawal_processed(std::string const& account_,
                       unsigned amount_):
    account(account_),amount(amount_)
  {}
};

struct card_inserted
{
  std::string account;
  explicit card_inserted(std::string const& account_):
    account(account_)
  {}
};

struct digit_pressed
{
  char digit;
  explicit digit_pressed(char digit_):
    digit(digit_)
  {}
};

struct clear_last_pressed
{};

struct eject_card
{};

struct withdraw_pressed
{
  unsigned amount;
  explicit withdraw_pressed(unsigned amount_):
    amount(amount_)
  {}
};

struct cancel_pressed
{};

struct issue_money
{
  unsigned amount;
  issue_money(unsigned amount_):
    amount(amount_)
  {}
};

struct verify_pin
{
  std::string account;
  std::string pin;
  mutable messaging::sender atm_queue;

  verify_pin(std::string const& account_,
             std::string const& pin_,
             messaging::sender atm_queue_):
    account(account_), pin(pin_), atm_queue(atm_queue_)
  {}
};

struct pin_verified
{};

struct pin_incorrect
{};

struct display_enter_pin
{};

struct display_enter_card
{};

struct display_insufficient_funds
{};

struct display_withdrawal_cancelled
{};

struct display_pin_incorrect_message
{};

struct display_withdrawal_options
{};

struct get_balance
{
  std::string account;
  mutable messaging::sender atm_queue;

  get_balance(std::string const& account_,
              messaging::sender atm_queue_):
    account(account_), atm_queue(atm_queue_)
  {} 
};

struct balance
{
  unsigned amount;
  explicit balance(unsigned amount_):
    amount(amount_)
  {}
};

struct display_balance
{
  unsigned amount;
  explicit display_balance(unsigned amount_):
    amount(amount_)
  {}
};

struct balance_pressed
{};
```