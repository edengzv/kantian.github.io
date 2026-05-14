---
title: "PHP设计模式笔记--桥接模式"
date: 2015-07-11 00:00:00
categories: ["Design Pattern"]
---

### 引言

本文首先简单介绍桥接(Bridge)模式，然后尝试提供一种桥接(Bridge)模式的PHP实现示例。

### 意图

将抽象部分与它的实现部分分离，使它们都可以独立地变化。

> 

这里的**实现**的含义不是指抽象基类的具体子类对抽象基类中的抽象函数（接口）进行实现，而是和继承结合在一起。
这里的**实现**指的是怎么去实现用户的需求，并且指的是通过组合（委托）的方式实现的，因此这里的**实现**不是指继承基类、实现基类接口，而是指通过对象组合实现用户的需求。

### 适用场景

- 不希望在抽象和它的实现部分之间有一个固定的绑定关系。

- 类的抽象以及它的实现都应该可以通过生成子类的方法加以扩充。通过使用Bridge模式对不同的抽象接口和实现部分进行组合，并分别对它们进行扩充。

- …

### 结构

![Bridge模式结构图](/img/bridge.png)

### 代码实现

```php
<?php
class Abstraction{
    public function Operation(){

    }
}

//可以认为RefinedAbstraction 起到了桥梁的作用，
//它把Operation的定义 和 Operation的具体实现连接了起来
class RefinedAbstraction extends Abstraction{
    private $_imp;
    public function __construct($abs_imp){
        $this->_imp = $abs_imp;
    }
    public function Operation(){
        $this->_imp->Operation();
    }
}

class AbstractionImp{
    public function Operation(){
        echo "AbstractionImp::Operation()\n";
    }
}

class ConcreteAbstractionImpA extends AbstractionImp{
    public function Operation(){
        echo "ConcreteAbstractionImpA::Operation()\n";
    }
}

class ConcreteAbstractionImpB extends AbstractionImp{
    public function Operation(){
        echo "ConcreteAbstractionImpB::Operation()\n";
    }
}

class Client{
    public static function main(){
        $c_A = new ConcreteAbstractionImpA();
        $r_bs = new RefinedAbstraction($c_A);
        $r_bs->Operation();

        $c_B = new ConcreteAbstractionImpB();
        $r_bs = new RefinedAbstraction($c_B);
        $r_bs->Operation();
    }
}

Client::main();

//Output
/**
$ php bridge.php
ConcreteAbstractionImpA::Operation()
ConcreteAbstractionImpB::Operation()
*/
```

### 相关模式

Bridge是设计模式中比较复杂和难理解的模式之一，也是OO开发与设计中经常会用到的模式之一。使用组合（委托）的方式将抽象和实现彻底地解耦，这样的好处是抽象和实现可以分别独立地变化，系统的耦合性也得到了很好的降低。

### 参考资料

设计模式-可复用面向对象软件的基础  GoF
[设计模式精解－GoF 23种设计模式解析](http://www.mscenter.edu.cn/blog/k_eckel)

### 后记

> 

本文仅为个人学习笔记整理，如有纰漏之处，欢迎指正-:)

—EOF—