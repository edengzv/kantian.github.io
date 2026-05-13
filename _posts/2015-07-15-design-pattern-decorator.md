---
title: "PHP设计模式笔记--装饰模式"
date: 2015-07-15 00:00:00
categories: ["Design Pattern"]
---

### 引言

本文首先简单介绍装饰(Decorator)模式，然后尝试提供一种装饰(Decorator)模式的PHP实现示例。

### 意图

动态地给一个对象添加一些额外职责。就新增加功能来说，Decorator模式相比生成子类更为灵活。

> 

Decorator模式不是通过继承实现扩展，而是通过组合来实现扩展。

### 适用场景

- 在不影响其他对象的情况下，以动态、透明的方式给单个对象添加职责。

- 处理那些可以撤销的职责

当不能采用生产子类的方法进行扩充时。

- 一种情况是，可能有大量独立的扩展，为支持每一种组合将产生大量的子类，使得子类的数目呈爆炸性增长。

- 另一种情况是，类的定义被隐藏，或类定义不能用于生成子类。

- …

### 结构

![Decorator模式结构图](/img/decorator.png)

- Component : 定义对象接口，可以给这些对象动态地添加职责

- ComcreteComponent : 定义一个对象，可以给这个对象添加职责

- Decorator : 维持一个指向Component对象的指针，并定义一个与Component接口一直的接口。

- ConcreteDecorator : 向组件添加职责

### 代码实现

```php
<?php
class Component{

    public function Operation(){

    }
}

class ConcreteComponent extends Component{
    public function Operation(){
        echo "ConcreteComponent Operation...\n";
    }
}

//Decorator 从 Component 继承的目的是为了保持跟Component的接口一致
class Decorator extends Component{
    public function __construct(&$com){
        $this->_component = $com;
    }

    public function Operation(){
        $this->_component->Operation();
    }

    protected $_component;
}
//为Component添加职责，并且通过公共接口实现职责
class ConcreteDecorator extends Decorator{
    public function Operation(){
        $this->_component->Operation();
        $this->AddedBehavior();
    }
    //添加职责
    public function AddedBehavior(){
        echo "ConcreteDecorator::AddedBehavior\n";
    }
}

class Client{
    public static function main(){
        $m_component = new ConcreteComponent();
        //指明对哪一个ConcreteComponent进行装饰
        $m_decorator = new ConcreteDecorator($m_component);
        //装饰完之后的对象有着跟Component一样的接口，
        //但是此时它的职责已经发生变化了
        $m_decorator->Operation();
    }
}

Client::main();

//Output
/*
$ php decorator.php
ConcreteComponent Operation...
ConcreteDecorator::AddedBehavior
*/
```

### 相关模式

- 咋一看，Decorator模式跟Composite模式有着相似之处，不过正如[上篇](/2015/07/13/design-pattern-composite/)所述,Composite侧重于通过多个子组件的组合，构造一个新类，而Decorator模式则侧重于，对一个已有类的修饰，在不通过生成子类的情况下，通过Wrapper的方式给类添加一些职责。

- 除此之外，Decorator模式跟[Proxy模式]()也有相似之处，他们都拥有一个指向其他对象的指针。即通过组合的方式来为对象提供更多的操作。区别在于，Proxy模式会提供使用其作为代理的对象一样的接口，使用代理类将其操作委托给Proxy直接进行。

> 

相比较通过生成子类来添加职责的方式，如果要保证父子类的接口一致，那么在给子类添加新职责的过程当中，就必然需要在父类中添加同样的接口，这样必然导致父类变得异常臃肿。而Decorator 模式则为这种情况提供了一种很好的解决方法，当需要添加一个操作（职责）的时候可以在保证父类接口不变的情况下，一步步添加职责。

### 参考资料

设计模式-可复用面向对象软件的基础  GoF
[设计模式精解－GoF 23种设计模式解析](http://www.mscenter.edu.cn/blog/k_eckel)

### 后记

> 

本文仅为个人学习笔记整理，如有纰漏之处，欢迎指正-:)

—EOF—