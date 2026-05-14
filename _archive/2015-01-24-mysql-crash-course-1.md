---
title: "MySQL 知识点整理一"
date: 2015-01-24 00:00:00
categories: ["MySQL"]
---

# 概述

本文梳理了MySQL的常用语法和操作，持续不断完善中…

# 基本操作

## 查看数据库详情

```text
SHOW DATABASES;
```

## 选择数据库

```text
USE DATABASE;
```

## 查看选定数据库中的表

```text
SHOW TABLES;
```

## 显示表中的列

```text
SHOW COLUMNS FROM tablename;
```

此外其它关于SHOW的用法

```text
* SHOW STATUS  显示服务器状态信息
* SHOW CREATE DATABASE  显示数据库创建语句
* SHOW CREATE TABLE 显示表创建语句
* SHOW GRANTS 显示授予用户的安全权限
* SHOW ERRORS 显示服务器错误信息
* SHOW WARNINGS 显示服务器警告信息
```

## 创建表

```text
CREATE TABLE customers 
(
    cust_id int NOT NULL AUTO_INCREMENT,
    cust_name char(50) NOT NULL,
    cust_address char(50) NOT NULL,
    cust_city char(50) NULL,
    cust_state char(5) NULL,
    cust_zip char(10) NULL,
    cust_country char(50) NULL,
    cust_contact char(50) NULL,
    cust_email char(255) NULL,
    PRIMARY KEY (cust_id)
)ENGINE = InnoDB;

1. 设置为 NOT NULL 将会阻止试图插入没有值的列
2. PRIMARY KEY(order_num, order_item) 创建由多个列组成的主键
  （这种情况下，order_num 和 order_item 的组合是唯一的,可以唯一确定一条记录）
3. 每个表只允许一个AUTO_INCREMENT 列，每增加一列的时候，该列的值自动增加1
```

## 几个常见的引擎

- InnoDB 是一个可靠的事务处理引擎，它不支持全文本搜索

- MEMORY 在功能上等同于MyISAM，但由于数据存储在内存中，速度很快，特别适合临时表

- MyISAM 是一个性能极高的引擎，它支持全文本搜索，但不支持实务处理

## ALTER 修改表

添加字段

```text
ALTER TABLE vendors
ADD vend_phone CHAR(20);
```

定义外键

```text
ALTER TABLE orderitems
ADD CONSTRAINT fk_orderitems_orders
FOREIGN KEY (order_num) REFERENCES orders (order_num);
```

对单张表实现多次修改
对单张表实现多次修改的时候，可以使用单条ALTER TABLE 语句，每个更改用逗号分隔

```text
ALTER TABLE user
ADD name varchar(20), 
ADD address varchar(255);
```

## DROP TABLE 删除表

```text
DROP TABLE customers2;
```

## RENAME TABLE 重命名表

```text
RENAME TABLE customers2 TO customers;

//一次性重命名多张表
RENAME TABLE backup_customers TO customers,
            backup_vendors TO vendors,
            backup_products TO products;
```

## SELECT 检索数据

#### 检索单列

```sql
SELECT prod_name 
FROM products;
```

#### 检索多列

```sql
SELECT prod_id, prod_name, prod_price 
FROM products;
```

#### 检索所有行

```sql
SELECT *
FROM products;  在并不需要获取全部列的时候，慎用通配符, 因为检索不需要的列通常会降低应用程序性能.
```

#### 检索不同的行

```sql
SELECT DISTINCT vend_id 
FROM products;
```

#### 限制查询条数

```text
SELECT prod_name 
FROM products LIMIT 5,10; 

从第五行后开始，检索10条结果(第6-第15条结果) 等价于

SELECT prod_name 
FROM products LIMIT 10 OFFSET 5;

(MySQL > 5)
```

#### 使用完全限定的表名

```sql
SELECT tablename.prod_name 
FROM databasename.products;
```

## ORDER BY 排序

#### 排序数据

```text
SELECT prod_name 
FROM products 
ORDER BY prod_name;

(并非必须要用当前检索的列进行排序,用非检索的列排序也是完全合法的）
```

#### 按多多个咧排序

```text
SELECT prod_id, prod_price, prod_name 
FROM products 
ORDER BY prod_price, prod_name;
```

#### 指定排序方向

```text
SELECT prod_id, prod_price, prod_name 
FROM products 
ORDER BY prod_price DESC, prod_name;
（默认是ASC的，如果要对每个列都进行降序排序，需要对每列都制定DESC）
```

#### ORDER BY 和 LIMIT 结合

```text
SELECT prod_price FROM products ORDER BY prod_price DESC LIMIT 5,10;
```

## WHERE 过滤

备注：数据过滤的工作可以在数据库查询的时候过滤，也可以在应用程序中进行过滤（查询出所有的行返回给应用程序），但是后者会极大的影响应用程序的性能，并且创建的应用不具备可伸缩性,同时如果应用程序和数据库通过网络连接交互的话，必然会增大网络传输的压力,造成网络带宽的浪费。

#### WHERE 字句操作符

操作符
说明        

 =
等于       

 <>
不等于      

 !=
不等于      

 <
小于       

 <=
小于等于    

 >
大于       

 >=
大于等于  

 BETWEEN
在指定的两个值之间

#### 检查单个值

```text
SELECT prod_name, prod_price 
FROM products 
WHERE prod_name = 'foses';
```

#### 不匹配检查

```text
SELECT vend_id, prod_name 
FROM products 
WHERE vend_id <> 1003;
```

#### 范围检查

```text
SELECT prod_name, prod_price 
FROM products 
WHERE prod_price BETWEEN 5 AND 10;
```

#### 空值检查

```text
SELECT Prod_name 
FROM products 
WHERE prod_price IS NULL;   

(IS NOT NULL)
```

## AND 操作符

```text
SELECT prod_id, prod_price, prod_name 
FROM products 
WHERE vend_id = 1003 AND prod_price <= 10;
```

## OR 操作符

```text
SELECT prod_name, prod_price 
FROM products 
WHERE vend_id = 1002 OR vend_id = 1003;
```

## AND 和OR 的优先级

```text
SELECT prod_name, prod_price
FROM products
WHERE (vend_id = 1002 OR vend_id = 1003) AND prod_price >= 10;

先执行AND 然后 执行OR，因此适当的添加括号是明智的选择!
```

## IN 操作符

```text
SELECT prod_name, prod_price
FROM products
WHERE vend_id IN (1002,1003)
ORDER BY prod_name;

等价于

SELECT prod_name, prod_price
FROM products
WHERE vend_id = 1002 OR vend_id = 1003
ORDER BY prod_name;

IN 子句里还可包含另外一个SELECT查询，把其查询结果作为IN() 的内容
```

## WHERE NOT 操作符

```text
SELECT prod_name, prod_price
FROM products
WHERE vend_id NOT IN (1002,1003)
ORDER BY prod_name;
```

## LIKE 通配符进行过滤

```text
SELECT prod_id, prod_name
FROM products
WHERE prod_name LIKE 'jet%' //jet 开始的

(WHERE prod_name LIKE '%jet%')  //包含jet的
(WHERE prod_name LIKE '%jet') //jet结尾的
(WHERE prod_name LIKE 's%e') 
(WHERE prod_name LIKE '_ ton anvil') // 1 ton anvil

总的来说： 
% 代表了在搜索模式中给定位置的0个，1个或多个任意合法字符
_ 代表了在搜索模式中给定位置一个字符

注意：
1. (WHERE prod_name LIKE '%';) 不能匹配 prod_name 值为 NULL 的行,但可以匹配其它任何行。
2. LIKE，用后跟搜索模式利用通配符匹配，而不是直接利用相等匹配进行比较,
换言之，LIKE 应该跟通配符结合使用。
```

## REGEXP 正则表达式进行搜索

#### 正则表达式基本字符匹配

```text
SELECT prod_name
FROM products
WHERE prod_name REGEXP '1000'
ORDER BY prod_name;
```

#### 正则表达式进行 OR 匹配

```text
SELECT prod_name
FROM products
WHERE prod_name REGEXP '1000 | 2000‘
ORDER BY prod_name;
```

#### 正则表达式匹配几个字符之一

```text
SELECT prod_name
FROM products
WHERE prod_name REGEXP '[123] Ton'
ORDER BY prod_name;

区别:
(WHERE prod_name REGEXP '1|2|3 Ton') 匹配1 或 2 或 3 ton ...
```

#### 正则表达式匹配范围

```text
SELECT prod_name
FROM products
WHERE prod_name REGEXP '[1-5] Ton'
ORDER BY prod_name;
```

#### 正则表达式匹配特殊字符

```text
SELECT vend_name 
FROM vendors
WHERE vend_name REGEXP '\\.'
ORDER BY vend_name;

匹配所有vend_name 列包含'.'字符的行

特殊字符需要转义，两个\\的解释：MySQL解析的时候一个，正则表达式引擎解析的时候一个
同理，如果要匹配\，则需要使用 REGEXP '\\\' 了
```

#### 正则表达式匹配多个实例

元字符
说明

*
0个或多个

+
1个或多个 {1,}

?
0个或1个 {0,1}

{n}
指定数目的匹配

{n,}
不少于指定数目的匹配

{n,m}
匹配指定范围 (m <= 255)

```text
SELECT prod_name 
FROM products
WHERE prod_name REGEXP '\\([0-9] sticks?\\)'
ORDER BY prod_name;

prod_name
TNT (1 stick)
TNT (5 sticks)
```

#### 正则表达式定位符

元字符
说明

^
文本开始

$
文本结尾

[[:<:]]
词的开始

[[:>:]]
词的结尾

```text
SELECT prod_name
FROM products
WHERE prod_name REGEXP ’^[0-9\\.]'
ORDER BY prod_name;

//匹配任意一.或数字开头的
注意：
^ 在集合中[],用来否定集合 [^ ]; 否则用来指串的开始处
```

#### Trick 在不适用数据库表的情况下，测试正则表达式

```text
SELECT 'hello' REGEXP '[0-9]';

+-------------------------+
| 'hello
' REGEXP '[0-9]' |
+-------------------------+
|                       0 |
+-------------------------+
1 row in set (0.00 sec)
```

## MySQL 处理函数

#### Concat — 拼接字符串

```text
SELECT Concat(RTrim(vend_name), '(', RTrim(vend_country),')')  AS vend_title
FROM vendors
ORDER BY vendor_name;

RTrim()  去掉右侧空格
LTRIM()  去掉左侧空格
```

#### 算术计算

```text
SELECT prod_id,quantity,item_price,
       quantity * item_price AS expanded_price
FROM orderitems
WHERE order_num = 20005;
```

算术运算符

操作符
说明

+
加

-
减

*
乘

/
除

SELECT 测试运算
```text
mysql> SELECT  3 * 2;
+-------+
| 3 * 2 |
+-------+
|     6 |
+-------+
1 row in set (0.00 sec)

mysql> SELECT NOW();
+---------------------+
| NOW()               |
+---------------------+
| 2015-01-15 14:13:31 |
+---------------------+
1 row in set (0.01 sec)
```

#### 文本处理函数

Upper() ,Left(),Length(),Locate(),Lower(),LTrim(),Right(),RTrim(),Soundex()
```text
SELECT vend_name, Upper(vend_name) AS vend_name_upcase
FROM vendors
ORDER BY vend_name;
```

#### 日期处理函数

函数
说明

AddDate()
增加一个日期（天，周等）

AddTime()
增加一个时间（时，分等）

curDate()
返回当前日期

CurTime()
返回当前时间

Date()
返回日期时间的日期部分

DateDiff()
计算两个日期之差

Date_Add()
高度灵活的日期运算函数

Date_Format()
返回一个格式化的日期活时间串

DayOfWeek()
对于一个日期，返回对应星期几

Hour()
返回一个时间的小时部分

Minute()
返回一个时间的分钟部分

Month()
返回一个日期的月部分

Now()
返回当前日期的时间

Second()
返回一个时间的秒部分

Time()
返回一个日期时间的时间部分

Year()
返回一个日期的年份部分

日期格式：order_date =  2005-09-01 11:30:05
Date(order_date) = 2005-09-01 // 仅获取日期部分 (MySQL >= 4.1.1)

```text
SELECT cust_id, order_num
FROM orders 
WHERE Date(order_date) BETWEEN '2005-09-01' AND '2005-09-30';

(WHERE Year(order_date) = 2005 AND Month(order_date) = 9;)
```

#### 数值处理函数

函数
说明

Abs()
返回一个数的绝对值

Cos()
返回一个角度的余弦

Exp()
返回一个数的指数值

Mod()
返回除操作的余数

Pi()
返回圆周率

Rand()
返回一个随机数

Sin()
返回一个角度的正弦

Sqrt()
返回一个数的平方根

Tan()
返回一个角度的正切

## 5个聚合函数

函数
说明

AVG()
返回某列的平均值

COUNT()
返回某列的行数

MAX()
返回某列的最大值

MIN()
返回某列的最小值

SUM()
返回某列值的和

```text
SELECT AVG(prod_price) AS avg_price
FROM products
WHERE vend_id = 1003;

SELECT COUNT(*) AS num_cust
FROM customers;

SELECT COUNT(cust_email) AS num_cust
FROM customers;

SELECT MAX(prod_price) AS max_price
FROM products;

SELECT MIN(prod_price) AS min_price
FROM products;

SELECT SUM(quantity) AS items_orderd
FROM orderitems
WHERE order_num = 20005;

SELECT SUM(item_price * quantity) AS total_price
FROM orderitems
WHERE order_num = 20005;

//聚合不同的值
SELECT AVG(DISTINCT prod_price) AS avg_price
FROM products
WHERE vend_id = 1003;

//组合聚合函数
SELECT COUNT(*) AS num_items,
        MIN(prod_price) AS price_min,
        MAX(prod_price) AS price_max,
        AVG(prod_price) AS price_avg
FROM products;
```

## 参考资料

- [MySQL 必知必会](http://book.douban.com/subject/3354490/)