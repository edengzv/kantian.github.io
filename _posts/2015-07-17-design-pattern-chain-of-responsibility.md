---
title: "PHP设计模式笔记--职责链模式"
date: 2015-07-17 00:00:00
categories: ["Design Pattern"]
---

### 引言

本文首先简单介绍职责链(Chain Of Responsibility)模式，然后尝试提供一种职责链(Chain Of Responsibility)模式的PHP实现示例。

### 意图

使多个对象都有机会处理请求，从而避免请求的发送者和接受者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理它为止。

### 适用场景

- 有多个对象可以处理一个请求，哪个对象处理该请求运行时刻自动确定。

- 想在不明确指定接收者的情况下，向多个对象中的一个提交一个请求。

- 可处理一个请求的对象集合应被动态指定。

### 结构

![Chain Of Responsibility模式结构图](/img/chainofresponsibility.png)

### 代码实现

```php
<?php
class Handle{
    public function __construct(){
    }
    //请求处理接口
    public function HandleRequest(){

    }
    //设置后继处理对象
    public function SetSuccessor($succ){
        $this->_succ = $succ;
    }
    //获取后继处理对象
    public function &GetSuccessor(){
        return $this->_succ;
    }

    protected $_succ;
}

class ConcreteHandleA extends Handle{
    public function __construct(){
        parent::__construct();
    }
    public function HandleRequest(){
        //判断决定是自己处理还是交给后继处理
        //此处简单判断是否有后继，如果有，就交给后继处理
        if($this->GetSuccessor() instanceof Handle){//交给后继处理当前请求
            echo "ConcreteHandleA's successor handle it\n";
            $this->GetSuccessor()->HandleRequest();
        }else{//自己处理当前请求
            echo "ConcreteHandleA handle it\n";
        }
    }
}

class ConcreteHandleB extends Handle{
    public function __construct(){
        parent::__construct();
    }

    public function HandleRequest(){
        //判断决定是自己处理还是交给后继处理
        //此处简单判断是否有后继，如果有，就交给后继处理
        if($this->GetSuccessor() instanceof Handle){//交给后继处理当前请求
            echo "ConcreteHandleB's successor handle it\n";
            $this->GetSuccessor()->HandleRequest();
        }else{//自己处理当前请求
            echo "ConcreteHandleB handle it\n";
        }
    }
}

class Client{
    public static function main(){
        $m_handle1 = new ConcreteHandleA();
        $m_handle2 = new ConcreteHandleB();
        //设置$m_handle2为$m_handle1的后继
        $m_handle1->SetSuccessor($m_handle2);
        //调用$m_handle1来处理，由于$m_handle1有后继，
        //它会沿职责链把请求移交给它的后继handle来处理
        $m_handle1->HandleRequest();
    }
}

Client::main();

//Output
/*
$ php chianofresponsibility.php
ConcreteHandleA's successor handle it
ConcreteHandleB handle it
*/
```

### 相关模式

- 职责链模式常与组合(Composite)模式一起使用。这种情况下，一个构件的父构件可作为它的后继。

职责链模式在很大程度上降低了系统的耦合性，请求的发送者和处理者并
不是相互绑定，一一对应的。请求的发送者完全不必知道该请求会被那个应答对象处理。

### 参考资料

设计模式-可复用面向对象软件的基础  GoF
[设计模式精解－GoF 23种设计模式解析](http://www.mscenter.edu.cn/blog/k_eckel)

### 后记

> 

本文仅为个人学习笔记整理，如有纰漏之处，欢迎指正-:)

—EOF—