---
title: "PHP设计模式笔记--命令模式"
date: 2015-07-19 00:00:00
categories: ["Design Pattern"]
---

### 引言

本文首先简单介绍命令(Command)模式，然后尝试提供一种命令(Command)模式的PHP实现示例。

### 意图

将一个请求封装为一个对象，从而使你可用不同的请求对客户进行参数化;对请求排队或记录请求日志，以及支持可撤销的操作。

### 适用场景

- 需要抽象出某个待执行的动作以参数化某个对象。如同回调函数机制一样，现在某处注册，然后在稍后某个需要的时候被调用。Command模式是回调机制的一个面向对象的替代品

- 需要在不同的时刻指定、排列和执行请求

- 需要支持撤销之前的操作

- 需要记录所有的操作日志，以便对系统操作进行回滚

- 需要提供对事务的支持

### 结构

![Command模式结构图](/img/command.png)

### 协作

![Command模式协作图](/img/command_2.png)

> 

如下代码实现提供了该模式的一种简单实现

### 代码实现

```php
<?php
//将请求封装成Command对象(ConcreteCommand)，该对象里面保存了该请求的处理者(Receiver)
//由Invoker来激活这个请求的处理过程,在处理的过程中会调用Receiver中的处理函数
//这里有回调意思：先预先设置处理对象，在合适的时候，激活调用该对象中的方法
class Command{
    public function Excute(){

    }
}
//ConcreteCommand 保存了请求的接收者的对象
class ConcreteCommand{
    public function __construct($rev){
        $this->_rev = $rev;
    }

    public function Excute(){
        $this->_rev->Action();
    }
    private $_rev;
}

//请求的最终接受和处理者
class Receiver{
    public function Action(){
        echo "Reciever::Action\n";
    }
}

//当请求到来时，Invoker发出Invoke消息，激活Command对象
//ConcreteCommand再进而将处理请求交的任务给Receiver对象进行处理
class Invoker{
    public function __construct($cmd){
        $this->_cmd = $cmd;
    }

    public function Invoke(){
        $this->_cmd->Excute();
    }

    private $_cmd;
}

class Client{
    public static function main(){
        $m_receiver = new Receiver();
        //创建ConcreteCommand对象，并且指定它的Receiver对象
        $m_command = new ConcreteCommand($m_receiver);
        //Invoker对象存储该ConcreteCommand对象
        $m_invoker = new Invoker($m_command);
        //Invoker通过调用Command对象的Execute操作来提交一个请求，如果该命令是可撤销的，
        //ConcreteCommand就在执行Execute操作之前存储当前状态，以用户取消该命令
        $m_invoker->Invoke();//ConcreteCommand对象调用它的Receiver的一些操作以执行该请求
    }
}

Client::main();

//Output
/*
$ php command.php
Reciever::Action
*/
```

### 相关模式

- Command 模式将调用操作的对象和具体执行该操作的对象进行了解耦。

- Command 模式通过继承自Command类产生子类ConcreteCommand的方式，可以很快的创建新的操作对象

- Command 模式跟Memento模式结合起来，支持取消操作。

### 参考资料

设计模式-可复用面向对象软件的基础  GoF
[设计模式精解－GoF 23种设计模式解析](http://www.mscenter.edu.cn/blog/k_eckel)

### 后记

> 

本文仅为个人学习笔记整理，如有纰漏之处，欢迎指正-:)

—EOF—