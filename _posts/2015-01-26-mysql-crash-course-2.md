---
title: "MySQL 知识点整理二"
date: 2015-01-26 00:00:00
categories: ["MySQL"]
---

## 概述

本文接上文[MySQL 知识点整理(上)](http://www.fengcode.com/2015/01/24/mysql-crash-course-1/)
本部分主要梳理了MySQL的分组查询、子查询、联结表、组合查询、全文搜索、以及插入/更新/删除数据等。

## GROUP BY 分组数据

```text
SELECT vend_id, COUNT(*) AS num_prods
FROM products
GROUP BY vend_id;
```

- 对分组结果进行过滤

```text
SELECT cust_id, COUNT(*) AS orders
FROM orders
GROUP BY cust_id
HAVING COUNT(*) >= 2;
```

HAVING 和 WHERE 的区别：
WHERE在数据分组之前进行过滤，HAVING在数据分组后进行过滤。
如果WHERE和HAVING同时使用，则需要注意，WHERE排除的行不包括在分组内。

```text
SELECT vend_id, COUNT(*) AS num_prods
FROM products
WHERE prod_price >= 10
GROUP BY vend_id
HAVING COUNT(*) >= 2;
```

分组和排序
```text
//一般GROUP BY以后 ，用ORDER BY 确保数据是正确排序的
SELECT order_num, SUM(quantity * item_price) AS ordertotal
FROM orderitems
GROUP BY order_num
HAVING SUM(quantity * item_price) >= 50
ORDER BY ordertotal;
```

## 子查询

一级子查询

```text
SELECT cust_id
FROM order
WHERE order_num IN(
    SELECT order_num 
    FROM orderitems
    WHERE prod_id = 'TNT2');
```

多级子查询

```text
SELECT cust_name, cust_contact
FROM customers
WHERE cust_id IN (SELECT cust_id
                    FROM orders
                    WHERE order_num IN(SELECT order_num
                                        FROM orderitems
                                        WHERE prod_id = 'TNT2'));
```

子查询作为结果的一部分

```text
SELECT cust_name,
       cust_state,
       (SELECT COUNT(*)
        FROM orders
        WHERE orders.cust_id = customers.cust_id) AS orders
FROM customers
ORDER BY cust_name;

//注意orders.cust_id = customers.cust_id 在引用的列可能出现二义性是，
必须用完全限定列名,用一个点分隔的表名和列名。
```

## 联结表

普通联结

```text
SELECT vend_name, prod_name, prod_price
FROM vendors, products
WHERE vendors.vend_id = products.vend_id
ORDER BY vend_name, prod_name;
```

笛卡尔积(第一个表的行*第二个表的行) 没有指明条件的情况下

```text
SELECT vend_name, prod_name, prod_price
FROM vendors, products
ORDER BY vend_name, prod_name;

//不要忘了WHERE
```

#### 内部联结

```text
//明确定义联结类型——内部联结
SELECT vend_name, prod_name, prod_price
FROM vendors INNER JOIN products
ON vendors.vend_id = products.vend_id;
```

#### 联结多个表

```text
SELECT prod_name, vend_name, prod_price, quantity
FROM orderitems, products, vendors
WHERE products.vend_id = vendors.vend_id
    AND orderitems.prod_id = products.prod_id
    AND order_num = 20005;
```

#### 使用表别名

```text
SELECT cust_name, cust_contact
FROM customers AS c, orders AS o, orderitems AS oi
WHERE c.cust_id = o.cust_id
    AND oi.order_num = o.order_num
    AND prod_id = 'TNT2';
```

#### 自联结

```text
SELECT prod_id, prod_name
FROM products
WHERE vend_id = (SELECT vend_id
                 FROM products
                 WHERE prod_id = 'DTNTR');
等价于

SELECT p1.prod_id, p1.prod_name
FROM products AS p1, products AS p2
WHERE p1.vend_id = p2.vend_id
    AND p2.prod_id = 'DTNTR'；
```

#### 自然联结

```text
//自然联结排除列出现多次的可能，使结果中每个列只返回一次

SELECT c.*, o.order_num, o.order_date,
        oi.prod_id, oi.quantity,oi.item_price
FROM customers AS c, orders AS o, orderitems AS oi
WHERE c.cust_id = o.cust_id
    AND oi.order_num = o.order_num
    AND oi.prod_id = 'FB';
```

#### 外部联结

LEFT OUTER JOIN … ON

```text
SELECT customers.cust_id, orders.order_num 
FROM customers LEFT OUTER JOIN orders
ON customers.cust_id = orders.cust_id;
```

RIGHT OUTER JOIN … ON

```text
SELECT customers.cust_id, orders.order_num
FROM customers RIGHT OUTER JOIN orders
ON orders.cust_id = customers.cust_id;
```

INNER JOIN … GROUP BY

```text
SELECT customers.cust_name,
        customers.cust_id,
        COUNT(orders.order_num) AS num_ord
FROM customers INNER JOIN orders
ON customers.cust_id = orders.cust_id
GROUP BY customers.cust_id;
```

LELFT OUTER JOIN … ON … GROUP BY

```text
SELECT customers.cust_name,
        customers.cust_id,
        COUNT(orders.order_num) AS num_ord
FROM customers LEFT OUTER JOIN orders
ON customers.cust_id = orders.cust_id
GROUP BY customers.cust_id;
```

## UNION 组合查询

UNION 规则：

- UNION 必须由两条或两条以上的SELECT语句组成，语句之间用UNION分隔

- UNION 中的每个查询必须包含相同的列，表达式或聚合函数（列的顺序不一定要相同）

- 列数据类型必须兼容:类型不必完全相同，但补习是DBMS可以隐含地转换的类型（有可能两种表中相同的列采用了不同的数据类型）

```text
//UNION ALL 可能包含重复的行（即满足prod_price <=5 又满足 vend_id IN (1001,1002）的行出现两次
//UNION 自动去除重复的行（其行为与单个SELECT语句，多个WHERE条件相同 WHERE prod_price <=5 OR vend_id IN (1001,1002);)
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <= 5
UNION ALL 
SELECT vend_id, prod_id, prod_price
FROM products
WHERE vend_id IN (1001,1002);
```

## 全文本搜索

- MyISAM 引擎支持全文本搜索

InnoDB 引擎不支持全文搜索

匹配note_text 列中包含anvils 关键字的行

```text
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('anvils');
```

使用查询扩展 （会匹配出一些不包含anvils但是相关联的行）

```text
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against ('anvils' WITH QUERY EXPANSION);
```

BOOL 模式

```text
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against ('heavy -rope*' IN BOOLEAN MODE);
排除任何包含rope开始的词的行
```

全文本布尔操作符

布尔操作符
说明

+
包含，词必须存在

-
排除，词必须不出现

>
包含，而且增加等级值

<
包含，而且减小等级值

()
把词组成子表达式（允许这些子表达式作为组被包含、排除、排列等)

~
取消一个词的排序值

*
词尾的通配符

“”
定义一个短语（与单个词的列表不一样，它匹配整个短语以便包含或排除这个短语)

搜索匹配包含词rabbit和bait的行

```text
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('+rabbit +bait' IN BOOLEAN MODE);
```

搜索匹配包含rabbit和bait中的至少一个的行

```text
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('rabbit bait' IN BOOLEAN MODE);
```

搜索匹配短语rabbit bait 而不是匹配两个词 rabbit 和 bait

```text
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('"rabbit bait"' IN BOOLEAN MODE);
```

搜索匹配包含rabbit和carrot至少一个的词，增加前者的等级，降低后者的等级

```text
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against ('>rabbit <carrot' IN BOOLEAN MODE);
```

搜索匹配词safe和combination，降低后者的等级

```text
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('+safe +(<combination)' IN BOOLEAN MODE);
```

## 参考资料

- [MySQL 必知必会](http://book.douban.com/subject/3354490/)