---
title: "PHP设计模式笔记--组合模式"
date: 2015-07-13 00:00:00
categories: ["Design Pattern"]
---

### 引言

本文首先简单介绍组合(Composite)模式，然后尝试提供一种组合(Composite)模式的PHP实现示例。

### 意图

将对象组合成**树形结构**以表示“部分-整体”的层次结构。
Composite模式使得用户对单个对象和组合对象的使用具有一致性。

### 适用场景

- 想表示对象的“部分-整体”的层次结构

- 希望用户忽略组合对象与单个对象的不同，用户将统一使用组合结构中的所有对象

- …

### 结构

![Composite模式结构图](/img/composite.png)

### 代码实现

```php
<?php
//提供用户统一接口，
//可以理解为（树形结构节点对外提供的统一接口，可以实现节点的公共缺省操作）
class Component{
    public function Operation(){

    }

    public function Add(&$component){

    }

    public function Remove(&$component){

    }

    public function GetChild($index){
        return 0;
    }
}
//负责管理组合对象的组件，可以理解为（树形结构的非叶子节点）
class Composite extends Component{

    public function Operation(){
        foreach ($this->component_array as $key => $item) {
            $item->Operation();
        }
    }

    public function Add(&$component){
        array_push($this->component_array,$component);
    }

    public function Remove(&$component){
        /*foreach($this->component_array as $key => $value){
            if($value === $component){
                unset($this->component_array[$key]);
                break;
            }
        }*/
        if(($key = array_search($component, $this->component_array)) !== FALSE){
            unset($this->component_array[$key]);
        }
    }

    public function GetChild($index){
        return $this->component_array[$index];
    }

    //对子节点Leaf的管理
    private $component_array = array();
}
//待组合的对象（树形结构的叶子节点）
class Leaf{
    public function Operation(){
        echo "Leaf Operation ...\n";
    }
}

class Client{
    public static function main(){
        $m_leaf = new Leaf();
        $m_leaf->Operation();

        $m_composite = new Composite();
        $m_composite->Add($m_leaf);
        $m_composite->Operation();

        $m_component = $m_composite->GetChild(0);
        $m_component->Operation();

        $m_composite->Remove($m_leaf);
        $m_composite->Operation();
    }
}

Client::main();

//Output
/*
$ php composite.php
Leaf Operation ...
Leaf Operation ...
Leaf Operation ...
*/
```

### 相关模式

> 

Composite模式通过和Decorator模式有着类似的结构图，但是Composite模式旨在构造类，而Decorator模式重在不生成子类即可给对象添加职责。Decorator模式重在修饰，而Composite模式重在表示。

### 参考资料

设计模式-可复用面向对象软件的基础  GoF
[设计模式精解－GoF 23种设计模式解析](http://www.mscenter.edu.cn/blog/k_eckel)

### 后记

> 

本文仅为个人学习笔记整理，如有纰漏之处，欢迎指正-:)

—EOF—