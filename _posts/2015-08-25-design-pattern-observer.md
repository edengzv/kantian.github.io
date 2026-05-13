---
title: "PHP设计模式笔记--观察者模式"
date: 2015-08-25 00:00:00
categories: ["Design Pattern"]
---

### 引言

本文首先简单介绍观察者(Observer)模式，然后尝试提供观察者(Observe)模式的一种PHP实现示例。

### 意图

定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都可得到通知并被自动更新。

> 

建立并维护若干个观察者(Observer)和目标(Subject)之间的关系，然后在目标(Subject)的状态发生变化时，所有订阅了该目标(Subject)的观察者(Observer)都可以迅速得到通知，同时做出相应的动作响应这个通知。

### 适用场景

- 当一个抽象模型有两个方面，其中一个方面依赖于另一方面。将这两者封装在独立的对象中以使他们可以各自独立地改变和复用。

- 当对一个对象的改变需要同时改变其它对象，而不知道将来具体会改变多少个对象。

- 当一个对象必须通知其它对象，而它又不能假定其它对象是谁。换言之，你不希望这些对象是紧密耦合的。

### 结构

![Observer模式结构图](/img/observer.png)

- 目标(Subject)管理所有订阅了该(Subject)的观察者(Observer), 因此会有一个添加和删除的接口。

- 每一个观察者(Observer)在收到目标(Subject)的通知时都需要更新，因此需要有一个更新的接口。

> 

如下代码实现提供了该模式的一种简单实现

### 代码实现

```php
<?php
class Subject{

    public function Attach(&$observer){
        //添加观察者
        if(($key = array_search($observer,$this->_obvs)) === FALSE){
            $this->_obvs[] = $observer;
        }
        return TRUE;
    }
    //移除观察者
    public function Detach(&$observer){
        if(($key = array_search($observer, $this->_obvs))!== FALSE){
            unset($this->_obvs[$key]);
            array_values($this->_obvs);
            return TRUE;
        }else{
            return FALSE;
        }
    }
    //发送通知给所有观察该Subject的Observer
    //执行其Update操作
    public function Notify(){
        foreach ($this->_obvs as $key => $value) {
            $value->Update();
        }
    }

    private $_obvs = array();
}

class ConcreteSubject extends Subject{
    public function __construct(){
        $this->_stat = "\n";
    }
    //获取当前Subject的状态
    public function GetState(){
        return $this->_stat;
    }
    //设置当前Subject的状态
    public function SetState($stat){
        $this->_stat = $stat;
    }

    private $_stat;
}

class Observer{
    public function GetSubject(){
        return $this->_subject;
    }

    public function Update(){

    }

    public function PrintInfo(){

    }

    private $_subject;
}

class ConcreteObserverA extends Observer{

    public function __construct(&$subject){
        $this->_subject = $subject;//设置当前观察者Observer所观察的目标Subject
        $this->_subject->Attach($this);//对应Subject中添加该观察者Observer
    }

    public function ConcreteObserverA(&$subject){
        $this->__construct($subject);
    }

    public function __destruct(){
        //从当前观察的Subject列表中移除
        $this->_subject->Detach($this);
    }

    public function Update(){
        //处理当Subject状态发生变化是的一些动作,这是仅仅是打印出Subject的状态
        $this->PrintInfo();
    }

    public function PrintInfo(){
        //打印当前观察的Subject的状态信息
        echo "ConcreteObserverA_Observer:: ".$this->_subject->GetState()."\n";
    }

    public function operator(){
        //Subject的状态可以通过自身对象改变，也可以由某个观察者中的某个操作改变
        //随着Subject状态的改变，而后调用Notify()操作便会将这种状态的改变通知到所有
        //观察（监听）了这个Subject的观察者，执行其对应的Update()操作
        $this->_subject->SetState("OBSERVER_CHANGE_STATE");
    }
}

class ConcreteObserverB extends Observer{

    public function __construct(&$subject){
        $this->_subject = $subject;
        $this->_subject->Attach($this);
    }

    public function ConcreteObserverB(&$subject){
        $this->__construct($subject);
    }

    public function __destruct(){
        $this->_subject->Detach($this);
    }

    public function Update(){
        //处理当Subject状态发生变化是的一些动作,这是仅仅是打印出Subject的状态
        $this->PrintInfo();
    }

    public function PrintInfo(){
        echo "ConcreteObserverB_Observer:: ".$this->_subject->GetState()."\n";
    }
}

class Client{
    public static function main(){
        $sub = new ConcreteSubject();
        $oa = new ConcreteObserverA($sub);
        $ob = new ConcreteObserverB($sub);

        $sub->SetState("old");
        $sub->Notify();
        $sub->SetState("new");
        $sub->Notify();

        $oa->operator();
        $sub->Notify();
    }
}

Client::main();

/*
Output:
$ php observer.php
ConcreteObserverA_Observer:: old
ConcreteObserverB_Observer:: old
ConcreteObserverA_Observer:: new
ConcreteObserverB_Observer:: new
ConcreteObserverA_Observer:: OBSERVER_CHANGE_STATE
ConcreteObserverB_Observer:: OBSERVER_CHANGE_STATE
*/
```

### 相关模式

- 在MVC模式当中，Model担任Subject的角色，而View担任Observer的角色，当Model发生变化的时候，View可以及时收到Subject针对这种变化所产生的通知，响应这个通知，并且做出对应的操作。

### 参考资料

设计模式-可复用面向对象软件的基础  GoF
[设计模式精解－GoF 23种设计模式解析](http://www.mscenter.edu.cn/blog/k_eckel)

### 后记

> 

本文仅为个人学习笔记整理，如有纰漏之处，欢迎指正-:)

—EOF—