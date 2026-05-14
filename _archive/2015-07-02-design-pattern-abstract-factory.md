---
title: "PHP设计模式笔记--抽象工厂"
date: 2015-07-02 00:00:00
categories: ["Design Pattern"]
---

### 引言

本文首先简单介绍抽象工厂模式，然后尝试提供一种抽象工厂模式的PHP实现。

### 意图

> 

提供一个创建一系列相关或相互依赖的对象的接口，而无需指定它们具体的类。
抽象工厂模式可以认为是工厂方法模式的一种扩展
该模式描述了怎样在不直接实例化具体类的情况下创建一系列相关的产品对象。

### 适用场景

- 一个系统要独立与它的产品的创建、组合和表示时

- 一个系统要由多个产品系列中的一个来配置时

- 当你要强调一系列相关的产品对象的设计以便进行联合使用时

- 当你提供一个产品类库，而只想显示他们的接口而不是实现时

### 结构

![抽象工厂结构图](/img/abstract_factory.png)

从抽象工厂的结构图可以看出，该模式下，对工厂的扩展会非常方便，只要添加工厂类，并且实现该工厂类的相关产品创建即可，而不会破环原有的结构；然后如果说针对产品的扩展，就会变得相对麻烦，必须对每一个工厂都添加该产品的创建。

### 代码实现

```php
<?php
//A 系列产品
interface AbstractProductA{
    public function operation();
}
//A 系列产品1
class ProductA1 implements AbstractProductA{
    public function operation(){
        echo "in product A1\n";
    }
}
//A 系列产品2
class ProductA2 implements AbstractProductA{
    public function operation(){
        echo "in product A2\n";
    } 
}

//B系列产品
interface AbstractProductB{
    public function operation();
}
//B系列产品1
class ProductB1 implements AbstractProductB{
    public function operation(){
        echo "in product B1\n";
    }
} 
//B系列产品2
class ProductB2 implements AbstractProductB{
    public function operation(){
        echo "in product B2\n";
    }
}

//厂商
interface AbstractFactory{
    public function CreateProductA();
    public function CreateProductB();
}

//厂商1
class ConcreteFactory1 implements AbstractFactory{
    public function CreateProductA(){
        return new ProductA1();
    }

    public function CreateProductB(){
        return new ProductB1();
    }
}

//厂商2
class ConcreteFactory2 implements AbstractFactory{
    public function CreateProductA(){
        return new ProductA2();
    }

    public function CreateProductB(){
        return new ProductB2();
    }
}

class Client{
    public static function main(){
        $factory1 = new ConcreteFactory1();
        $factory2 = new ConcreteFactory2();
        //factory1 
        //create productA
        $producta1 = $factory1->CreateProductA();
        $producta1->operation();
        //create productB
        $productb1 = $factory1->CreateProductB();
        $productb1->operation();

        //factory2
        //create productA
        $producta2 = $factory2->CreateProductA();
        $producta2->operation();
        //create productB
        $productb2 = $factory2->CreateProductB();
        $productb2->operation();
    }
}

Client::main();

//Output:
/*
in product A1
in product B1
in product A2
in product B2
*/
```

### 参考资料

设计模式-可复用面向对象软件的基础  GoF

### 后记

> 

本文仅为个人学习笔记整理，如有纰漏之处，欢迎指正-:)

—EOF—