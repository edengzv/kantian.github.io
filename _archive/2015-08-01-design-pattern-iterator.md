---
title: "PHP设计模式笔记--迭代器模式"
date: 2015-08-01 00:00:00
categories: ["Design Pattern"]
---

### 引言

本文首先简单介绍迭代器(Iterator)模式，然后尝试提供一种迭代器(Iterator)模式的PHP实现示例。

### 意图

提供一种方法顺序访问一个聚合对象中的各个元素，而又不需暴露该对象的内部表示。

> 

迭代器模式的关键思想是：将对列表的访问和遍历从列表对象中分离出来并放入一个迭代器(iterator)中

### 适用场景

- 访问一个聚合对象的内容而无需暴露它的内部结构

- 支持对聚合对象的多种遍历

- 为遍历不同的聚合结构提供一个统一的接口（即，支持多态迭代）

### 结构

![Iterator模式结构图](/img/iterator.png)

> 

如下代码实现提供了该模式的一种简单实现

### 代码实现

```php
<?php
class Aggregate{
    public function CreateIterator(){

    }

    public function GetItem($idx=0){

    }

    public function GetSize(){
        return $_size;
    }

    protected $_objs;
    protected $_size;
}

class ConcreteAggregate extends Aggregate{
    //创建列表元素
    public function __construct(){
        for($i=0;$i<10;$i++){
            $this->_objs[] = $i;
        }
    }
    //根据实际需要创建迭代器
    public function CreateIterator(){
        return new ConcreteIterator($this);
    }
    //获取列表中某一个指定下标的元素
    public function GetItem($idx=0){
        if($idx < $this->GetSize()){
            return $this->_objs[$idx];
        }else{
            return FALSE;
        }
    }
    //获取当前元素个数
    public function GetSize(){
        return count($this->_objs);
    }
}

class vIterator{
    public function First(){

    }

    public function Next(){

    }

    public function IsDone(){

    }

    public function CurrentItem(){

    }

    protected $_ag;
    protected $_idx;
}

//定义一个顺序迭代器
class ConcreteIteratorA extends vIterator{

    public function __construct($ag, $idx = 0){
        $this->_ag = $ag;
        $this->_idx = $idx;
    }

    public function First(){
        $this->_idx = 0;
    }

    public function Next(){
        if($this->_idx < $this->_ag->GetSize()){
            $this->_idx ++;
        }
    }

    public function IsDone(){
        if($this->_idx == $this->_ag->GetSize()){
            return TRUE;
        }else{
            return FALSE;
        }
    }

    public function CurrentItem(){
        return $this->_ag->GetItem($this->_idx);
    }
}

//定义一个逆序迭代器
class ConcreteIteratorB extends vIterator{

    public function __construct($ag,$idx=0){
        $this->_ag = $ag;
        $this->_idx = $idx;
    }

    public function First(){
        $this->_idx = $this->_ag->GetSize() - 1;
    }

    public function Next(){
        if($this->_idx >= 0){
            $this->_idx --;
        }
    }
    public function IsDone(){
        if($this->_idx < 0){
            return TRUE;
        }else{
            return FALSE;
        }
    }
    public function CurrentItem(){
        return $this->_ag->GetItem($this->_idx);
    }
}

class Client{
    public static function main(){
        $ag = new ConcreteAggregate();
        $ita = new ConcreteIteratorA($ag);
        $ita->First();
        for(;$ita->IsDone() == FALSE;$ita->Next()){
            echo $ita->CurrentItem().' ';
        }

        echo "\n";
        $itb = new ConcreteIteratorB($ag);
        $itb->First();
        for(;$itb->IsDone() == FALSE;$itb->Next()){
            echo $itb->CurrentItem().' ';
        }
    }
}

Client::main();

//Output
/*
$ php iterator.php
0 1 2 3 4 5 6 7 8 9
9 8 7 6 5 4 3 2 1 0
*/
```

### 相关模式

迭代器的使用在C++中非常普遍

- 迭代器模式常被应用于Composite模式这样的递归结构上

- 多态迭代器，通过Factory Method来实例化适当的迭代器子类

### 参考资料

设计模式-可复用面向对象软件的基础  GoF
[设计模式精解－GoF 23种设计模式解析](http://www.mscenter.edu.cn/blog/k_eckel)

### 后记

> 

本文仅为个人学习笔记整理，如有纰漏之处，欢迎指正-:)

—EOF—