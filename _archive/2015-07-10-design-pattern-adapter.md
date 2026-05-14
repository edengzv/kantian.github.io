---
title: "PHP设计模式笔记--适配器模式"
date: 2015-07-10 00:00:00
categories: ["Design Pattern"]
---

### 引言

本文首先简单介绍适配器(Adapter)模式，然后尝试提供一种适配器(Adapter)模式的PHP实现示例。

### 意图

将一个类的接口转换成客户希望的另外一个接口。
Adapter模式使得原本由于接口不兼容而不能工作的那些类可以一起工作。

`解决了由于接口不匹配而导致的接口不能复用的问题`

### 适用场景

以下情况使用Adapter模式

- 你想使用一个已经存在的类，而它的接口不符合你的需求。(最原始的哦需求)

- 你想创建一个可以复用的类，该类可以与其他不相关的类或不可预见的类（即那些接口可能不一定兼容的类）协同工作。

- 你想使用一些已经存在的子类，但是不可能对每一个都进行子类化以匹配它们的接口。对象适配器可以适配它的父类接口。

### 结构

![Adapter模式结构图](/img/adapter.png)

### 代码实现

```php
<?php

class Target{

    public function Request(){
        echo "Target::Request\n";
    }

}

class Adaptee{
    //待适配的接口
    public function SpecificRequest(){
        echo "Adaptee::SpecificRequest";
    }
}

//Adapter 继承自Target，但是通过Adaptee的对象对Request()方法进行了重新定义。
//使在Adapter中，第三方接口（Adaptee）跟本地接口（Target）得到了统一。
//相当于对Adaptee进行了包装(Wrapper),从而达到了接口适配和复用的目的。

class Adapter extends Target{
    private $_ade;
    public function __construct($m_adaptee){
        $this->_ade = $m_adaptee;
    }
    public function Request(){
        $this->_ade->SpecificRequest();
    }

}

class Client{
    public static function main(){
        $m_adaptee = new Adaptee();
        $m_adapter = new Adapter($m_adaptee);
        $m_adapter->Request();
    }
}

Client::main();

//Output:
/*
    Adaptee::SpecificRequest
*/
```

### 相关模式

> 

- Bridge模式的结构与对象适配器类似，但是Bridge模式的出发点不同：Bridge目的是将接口部分和实现部分分离，从而对它们可以较为容易也相对独立的加以改变。而Adapter则意味着改变一个已有对象的接口。

- Decorator模式增强了其他对象的功能而同时又不改变它的接口。因此Decorator对应用程序的透明性比适配器要好。结果是Decorator支持递归组合，而纯粹使用适配器是不可能实现这一点的。

- Proxy模式在不改变它的接口的条件下，为另一个对象定义了一个代理。

### 参考资料

设计模式-可复用面向对象软件的基础  GoF

### 后记

> 

本文仅为个人学习笔记整理，如有纰漏之处，欢迎指正-:)

—EOF—