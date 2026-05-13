---
title: "PHP设计模式笔记--原型模式"
date: 2015-07-04 00:00:00
categories: ["Design Pattern"]
---

### 引言

本文首先简单介绍原型模式，然后尝试提供一种原型模式的PHP实现。

### 意图

用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。
原型模式主要用于对象的复制。

### 适用场景

当一个系统应该独立于它的产品创建、构成和表示时，要使用Prototype模式；以及

- 当要实例化的类是在运行时刻指定时；

- 当需要为了避免创建一个与产品类层次平行的工厂类层次时；

- 当一个类的实例只能有几个不同状态组合中的一种时，建立相应数目的原型并克隆它们可能比每次用合适的状态手工实例化更方便一些。

### 结构

![Prototype模式结构图](/img/prototype.png)

### 代码实现

```php
<?php
class Prototype{
    static $instances_cnt = 0;
    public $instance_cnt;
    //拷贝的过程当中，会执行__construct() 和 _clone()
    public function __construct(){
        $this->instance_cnt = ++self::$instances_cnt;
    }

    public function __clone(){
        $this->instance_cnt = ++self::$instances_cnt;
    }

    public function myclone(){

    }

    public function operation(){
       // echo 'instance_cnt '.$this->instance_cnt;
      //  echo 'instances_cnt '.self::$instances_cnt;
        echo "hi\n";
    }
}

class ConcretePrototype extends Prototype{
    private $instance;
    public function __construct(){
        $this->instance = new Prototype();
    }
    //clone 当前对象并且返回
    public function myclone(){
        return clone $this->instance;
    }

    public function operation(){
        echo "Hi\n";
    }
}

class Client{
    public static function main(){
        $c_prototype = new ConcretePrototype();
        $m_prototype = $c_prototype->myclone();
        $m_prototype->operation();
        $n_prototype = $c_prototype->myclone();
        $n_prototype->operation();
    }   
}

Client::main();

//output
/*
hi
hi

*/
```

### 相关模式

Prototype 模式通过复制原型而达到创建新对象的功能，而PHP原生的clone() 方法让这一特性得以很方便的实现。
实际上Prototype、Builder和Abstract Factory模式都是通过一个类来专门负责对象的创建。
只是区别在于：

- Builder模式侧重于复杂对象的一步步创建;

- Abstract Factory 模式侧重于生产队歌相互依赖类的对象;

- Prototype 模式侧重于从一个已有的对象，通过复制该对象来产生新的对象。

### 参考资料

设计模式-可复用面向对象软件的基础  GoF

### 后记

> 

本文仅为个人学习笔记整理，如有纰漏之处，欢迎指正-:)

—EOF—