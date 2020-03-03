---
title: C++ gtest
toc: true
date: 2018-08-01 18:06:34
---
# C++ gtest




## 可以补充进来的

- **要仔细总结一下 gtest 到底是怎么使用的**
- **除了在 make 中使用 ，在 VS 中可以使用吗？**



一直想看一下 C++的单元测试怎么写，但是不知道真的用什么，刚在看别人的设计模式的例子代码的时候，看到他用了 gtest，test文件是这么写的：



```cpp
#include "gtest/gtest.h"
#include "command.h"

class CommandFixture: public ::testing::Test {
protected:
  virtual void TearDown() {};
  virtual void SetUp() {};

public:
  CommandFixture(): Test() {
    barbecuer_ = new Barbecuer();
    bake_mutton_command1_ = new BakeMuttonCommand(barbecuer_);
    bake_mutton_command2_ = new BakeMuttonCommand(barbecuer_);
    bake_chicken_command_ = new BakeChickenCommand(barbecuer_);
    waiter_ = new Waiter();
    waiter_->SetOrder(bake_mutton_command1_);
    waiter_->SetOrder(bake_mutton_command2_);
    waiter_->SetOrder(bake_chicken_command_);
    waiter_->CancelOrder(bake_mutton_command2_);
    waiter_->Notify();
  }

  virtual ~CommandFixture() {
    delete barbecuer_;
    delete bake_mutton_command1_;
    delete bake_mutton_command2_;
    delete bake_chicken_command_;
    delete waiter_;
  }

  Barbecuer *barbecuer_;
  Command *bake_mutton_command1_;
  Command *bake_mutton_command2_;
  Command *bake_chicken_command_;
  Waiter *waiter_;
};

TEST_F(CommandFixture, command_test) {
}
```

然后他的路径是这样的：


![](http://images.iterate.site/blog/image/180727/7h21C6mc6c.png?imageslim){ width=55% }

他用的是 make 不是 vs ，看来 make 还是一定要掌握的，不然连一些东西都不好用。



# 相关

- [玩转 Google 开源 C++单元测试框架 Google Test系列(gtest)(总)](https://www.cnblogs.com/coderzh/archive/2009/04/06/1426755.html)
