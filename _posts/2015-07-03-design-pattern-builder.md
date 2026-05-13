---
title: "PHP设计模式笔记--生成器"
date: 2015-07-03 00:00:00
categories: ["Design Pattern"]
---

### 引言

本文首先简单介绍Builder模式，然后在此基础上提供该模式的一种PHP的简单实现。

### 意图

将一个复杂对象的构建与它的表示分离，使得同样的通过一步步的构建**过程**可以创建不同的表示。
由于在每一步的构建过程当中都可以引入参数，使得经过相同的步骤创建之后，所得到的对象的暂时是不一样的。

### 适用性

当满同时满足以下条件的时候可以使用Builder模式：

- 当创建复杂对象的算法应该独立于该对象的组成部分以及他们的装配方式

- 当构造过程必须允许构造的对象有不同的表示

### 结构

![Builder模式类图](/img/builder_1.png)

- Builder 为创建一个Product对象的各个部件指定抽象接口

- ConcreteBuilder 实现Builder的接口,(真正创建产品的内部表示，并且定义了产品的装配过程)

- Director 构造一个使用Builder接口的对象

- Product 最终要被构造的复杂对象

### 协作流程

![Builder模式序列图](/img/builder_2.png)

### 代码实现

```php
<?php
class Product{
    private $product_content;
    function __construct(){
        $this->product_content = '';
    }
    //组装过程中给产品添加部件
    public function AddPart($product_part){
        $this->product_content .= $product_part;
    }
    //返回组装好后的产品
    public function ShowProduct(){
        echo $this->product_content;
    }
}

interface Builder{
    //创建产品
    public function BuildPanel();
    //构建部件A
    public function BuildPartA();
    //构建部件B
    public function BuildPartB();
    //构建部件C
    public function BuildPartC();
}

//A类产品Builder
class ConcreateBuilderA implements Builder{

    private $productA;

    public function BuildPanel(){
        $this->productA = new Product();
        $this->productA->AddPart("BuildProductA\n");
    }

    public function BuildPartA(){
        $this->productA->AddPart("BuildPartA\n");
    }

    public function BuildPartB(){
        $this->productA->AddPart("BuildPartB\n");
    }

    public function BuildPartC(){
        $this->productA->AddPart("BuildPartC\n");
    }

    public function GetProduct(){
        return $this->productA;
    }
}

//B类产品Builder
class ConcreateBuilderB implements Builder{

    private $productB;

    public function BuildPanel(){
        $this->productB = new Product();
        $this->productB->AddPart("BuildProductB\n");
    }

    public function BuildPartA(){
        $this->productB->AddPart("BuildPartA\n");
    }

    public function BuildPartB(){
        $this->productB->AddPart("BuildPartB\n");
    }

    public function BuildPartC(){
        $this->productB->AddPart("BuildPartC\n");
    }

    public function GetProduct(){
        return $this->productB;
    }
}
//Director
//职责是根据传入的具体Builder来构建相应的产品
class Director{

    private $m_builder;
    
    function __construct($builder){
        $this->m_builder = $builder;
    }

    public function GetP(){
        //具体构建的过程
        $this->m_builder->BuildPanel();
        $this->m_builder->BuildPartA();
        $this->m_builder->BuildPartB();
        $this->m_builder->BuildPartC();

        return $this->m_builder->GetProduct();
    }
}

//客户端
class Client{

    public static function main(){
        //create product A
        $c_builder = new ConcreateBuilderA(); 
        $c_director = new Director($c_builder);
        $product = $c_director->GetP();
        $product->ShowProduct();

        //create product B
        $c_builder = new ConcreateBuilderB(); 
        $c_director = new Director($c_builder);
        $product = $c_director->GetP();
        $product->ShowProduct();
    }
}

Client::main();

//Output:
/**
BuildProductA
BuildPartA
BuildPartB
BuildPartC

BuildProductB
BuildPartA
BuildPartB
BuildPartC
*/
```

### 相关模式

Abstract Factory与Builder相似，因为它们都用于创建复杂对象。主要的区别在于，Builder模式着重于一步步构造一个复杂对象；Abstract Factory则侧重于多个系列的产品对象(简单的或复杂的)。Builder是最后一步返回产品，而Abstract Factory是立即返回产品。Composite通常是用Builder生成的。

### 参考资料

设计模式-可复用面向对象软件的基础  GoF

### 后记

> 

本文仅为个人学习笔记整理，如有纰漏之处，欢迎指正-:)

—EOF—