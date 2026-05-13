---
title: "PHP设计模式笔记--工厂方法"
date: 2015-06-30 00:00:00
categories: ["Design Pattern"]
---

> 

Each pattern describes a problem which occurs
over and over again in our environment, and then
describes the core of the solution to that problem,
in such a way that you can use this solution a
million times over, without ever doing the same
thing twice.
                       —— Christopher Alexander

### 引言

本文首先简单介绍Factory Method 模式，然后尝试提供Factory Method模式的一种PHP实现。

### 意图

工厂方法模式定义一个用于创建对象的接口，让子类决定实例化哪一个类。使得一个类的实例化过程延迟到了其子类。
许多类型的对象创建都需要一系列的步骤，比如说：

- 可能需要计算或取得对象的初始位置

- 可能需要选择生成哪个子对象的实例

- 可能需要在对象生成之前必须先生成一些辅助功能的对象

类似这种对象的创建，其实是一个“过程”，而不是一个简单的new操作。
把所有的这些对象创建的过程都封装起来，然后返回一个所需要的新类，这也许就是工厂方法的本质所在。
因此简单的来说工厂方法的职责就是“生产”对象。

### 适用场景

- 当一个类不知道它所必须创建的对象的类的时候

- 当一个类希望由它的子类来制定它所创建的对象的时候

- 当类将创建对象的职责委托给多个子类中的某一个，并且希望将哪一个子类是代理者这一信息局部化的时候。

### 结构

![工厂方法结构图](/img/factory_method.png)

### 简单实现

```php
<?php
//Creator依赖于它的子类来定义工厂方法，
//所以它返回一个适当的ConcretProduct实例
interface Creator {
    public function FactoryMethod();
}

class ConcreteCreator implements Creator {
    //实现工厂方法，来创建并返回具体的对象
    public function FactoryMethod(){
        return new ConcreteProduct();
    }
}

interface Product {
    public function operation();
}

//实现不同的产品类
class ConcreteProduct implements Product {
    public function operation(){
        echo "Product Created\n";
    }
}

class Client{
    public static function main(){
        $creator = new ConcreteCreator();
        $product = $creator->FactoryMethod();
        $product->operation();
    }
}

Client::main();

//Output:
//Product Created
```

### 参考资料

设计模式-可复用面向对象软件的基础  GoF

> 

本文仅为个人学习笔记整理，如有纰漏之处，欢迎指正-:)

—EOF—