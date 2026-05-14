---
title: "PHP设计模式笔记--代理模式"
date: 2015-07-16 00:00:00
categories: ["Design Pattern"]
---

### 引言

本文首先简单介绍代理(Proxy)模式，然后尝试提供一种代理(Proxy)模式的PHP实现示例。

### 意图

为其他对象提供一种代理，以控制对这个对象的访问。

### 适用场景

- 保护代理(Protection Proxy) 控制对原始对象的访问

- 远程代理(Remote Proxy) 为一个对象在不同的地址空间提供局部代表

- 虚代理(Virtual Proxy) 根据需要创建开销很大的对象

- 智能指引(Smart Reference) 智能指针的实现，取代简单指针，在访问对象的时候执行一些附加操作

代理(Proxy)模式在访问对象时引入了一定程度的间接性。根据代理的类型，附加的间接性有多种用途：

- 远程代理(Remote Proxy) 可以隐藏一个对象存在于不同地址空间的事实

- 虚代理(Virtual Proxy) 可以进行最优化，例如根据要求创建对象

- 保护代理(Protection Proxies)和智能指引(Smart Reference)都允许在访问一个对象时有一些附加的内务处理(Housekeeping task)

### 结构

![Proxy模式结构图](/img/proxy.png)

### 代码实现

```php
<?php
class Subject{
    public function Request(){
    }
}

//真正执行的类
class ConcreteSubject extends Subject{
    public function Request(){
        echo "ConcreteSubject::Request()\n";
    }
}

class Proxy extends Subject{

    public function __construct($subject){
        $this->_subject = $subject;
    }
    //Proxy 接受Request请求，然后调用ConcreteRequest的Request去执行真正的操作
    //这里Proxy在真正调用ConcreteRequest的Request之前，还能够做一些其他的操作，
    //这部分工作也许正是代理的优势的真正体现之处了
    public function Request(){
        echo "Proxy::Request()\n";
        $this->_subject->Request();
    }
    
    private $_subject;
}

class Client{
    public static function main(){
        $m_concrete_subject = new ConcreteSubject();

        $m_proxy = new Proxy($m_concrete_subject);
        $m_proxy->Request();
    }
}

Client::main();

//Output:
/*
$ php proxy.php
Proxy::Request()
ConcreteSubject::Request()
*/
```

> 

Proxy模式最大的好处就是实现了逻辑和实现的彻底解耦。

### 相关模式

- Adapter模式为了所适配的对象提供了一个不同的接口，而代理模式为对应的实体提供完全相同的接口。然而，用于访问保护的代理可能会拒绝执行某些实体的操作，因此它的接口实际上可能是实体接口的一个子集。

- Decorator模式实现了部分Proxy模式的部分功能，但是两者的侧重点不一样，前者主要是为对象添加一个或多个功能(职责)，而后者则侧重于控制对象的访问。

### 参考资料

设计模式-可复用面向对象软件的基础  GoF
[设计模式精解－GoF 23种设计模式解析](http://www.mscenter.edu.cn/blog/k_eckel)

### 后记

> 

本文仅为个人学习笔记整理，如有纰漏之处，欢迎指正-:)

—EOF—