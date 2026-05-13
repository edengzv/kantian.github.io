---
title: "PHP设计模式笔记--单例模式"
date: 2015-07-08 00:00:00
categories: ["Design Pattern"]
---

### 引言

本文首先简单介绍单例模式，然后尝试提供一种单例模式的PHP实现示例。

### 意图

保证一个类仅有一个实例，并提供一个访问它的全局访问点。

### 适用场景

在很多时候保证一个类只有一个实例是非常有必要的，在以下情况下我们应当使用Singleton模式：

- 当类只能有一个实例而且客户端只能通过该类所提供的某个特定方法才能访问它时;

- 当这个唯一实例应该是通过子类化可扩展的，并且客户应该无需代码就能使用一个扩展的实例时;

### 结构

![Singleton模式结构图](/img/singleton.png)

### 代码实现

```php
<?php
class Singleton{
    private static $instance;

    //返回实例
    public static function getInstance(){
        if(NULL == self::$instance){
            self::$instance = new Singleton();
        }
        return self::$instance;
    }

    //构造函数声明为private
    //防止在类以外通过new关键字来创建对象
    private function __construct(){

    }

    //防止对象被拷贝
    private function __clone(){

    }
    
    //防止被序列化
    private function __wakeup(){

    }
}

class SingletonSub extends Singleton{

}

class Client{
    public static function main(){
        $singleton = Singleton::getInstance();
        /*
            == checks if equivalent (value only)
            === checks if the same (value && type) 
        */
        var_dump($singleton === Singleton::getInstance());

        $ano_singleton = SingletonSub::getInstance();
        var_dump($ano_singleton === SingletonSub::getInstance());

        var_dump($ano_singleton === Singleton::getInstance());
    }
}

Client::main();

//Output
/*
bool(true)
bool(true)
bool(true) ??
*/
```

> 

这里除了定义getInstance函数外，还需要特别注意**construct(),**clone(),以及__wakeup() 函数的声明。
因为只有严格控制类的访问，才能达到`对唯一实例的受控访问`目的。

### 相关模式

在实际应用中,Singleton会经常使用到，比如说数据库 PDO 的访问对象，通常会是设计成单例的。
比如在游戏中的主角，如笔者之前开发的类Flappy Bird游戏[EarlyBird](https://github.com/OiteBoys/Earlybird)中的小鸟类就是设计成了单例

### 参考资料

设计模式-可复用面向对象软件的基础  GoF

### 后记

> 

本文仅为个人学习笔记整理，如有纰漏之处，欢迎指正-:)

—EOF—