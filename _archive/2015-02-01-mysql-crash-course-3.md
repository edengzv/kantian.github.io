---
title: "MySQL 知识点整理三"
date: 2015-02-01 00:00:00
categories: ["MySQL"]
---

## 概述

本文接上文[MySQL 知识点整理(上)](/2015/01/24/mysql-crash-course-1/),[MySQL 知识点整理(中)](/2015/01/26/mysql-crash-course-2/).
该部分主要梳理了MySQL的视图、存储过程、游标、触发器以及事务处理等。

## 视图

#### 视图简介

视图是虚拟的表。与包含数据的表不一样，视图只包含使用是动态检索数据的查询。
为什么使用视图：

- 简化并且重用SQL语句,一次性编写基础SQL语句

- 封装内部表结构，保护数据

- …

视图本身不包含数据，在针对视图进行SELECT的操作时，或者对多个视图进行联结时，都会执行视图所封装的查询。
因此如果用多个联结和过滤创建了一个复杂的视图时，可能会发现性能大幅下降。

#### 视图的规则和限制

- 视图名字唯一

- 可创建的视图数目没有限制

- 创建视图需要获取权限

- 视图可以嵌套

- 对视图进行SELECT时，如果用到了ORDER BY，那么会覆盖视图内部SELECT中的ORDER BY

- 视图可以和表一起使用

#### 创建视图

```text
CREATE VIEW productcustomers AS
SELECT cust_name, cust_contact, prod_id
FROM customers, orders, orderitems
WHERE customers.cust_id = orders.cust_id
    AND orderitems.order_num = orders.order_num;
```

#### 查看视图创建语句

```text
SHOW CREATE VIEW productcustomers;
```

#### 删除视图

```text
DROP VIEW productcustomers;
```

#### 修改视图

```text
1.
DROP VIEW viewname;
CREATE VIEW ...

2.
CREATE OR REPLACE VIEW ...
```

#### 利用视图进行查询

```text
SELECT cust_name, cust_contact
FROM productcustomers
WHERE prod_id = 'TNT2';
```

#### 创建视图，重新格式化查询数据

```text
CREATE VIEW vendorlocation AS
SELECT Concat(RTrim(vend_name),' (', RTrim(vend_country),')')
    AS vend_title
FROM vendors
ORDER BY vevnd_name;

SELECT *
FROM vendorlocations;
```

#### 视图查询，过滤数据

```text
//视图把cust_email = NULL 的行过滤掉了
//查询的时候就可以省去过滤的步骤
CREATE VIEW customeremaillist AS
SELECT cust_id, cust_name, cust_email
FROM customers
WHERE cust_email IS NOT NULL;

SELECT *
FROM customeremaillist;

//如果在查询视图的时候，以及视图实现的时候都用到了WHERE关键字
//那么，这两个WHERE 将会被合并
```

#### 更新视图(INSERT,UPDATE,DELETE)

原则：**视图的主要目的是为了简化和方便检索**，并不是所有所图都是可更新的。
简单来说，如果MySQL不能正确的确定被更新的基数据，则不允许更新。
如果视图有以下操作，则不能进行更新：

- GROUP BY 和 HAVING

- 联结 JOIN 

- 子查询

- 合并 UNION

- 聚合函数 MIN(),MAX(),AVG(),SUM(),COUNT()

- DISTINCT

- 包含计算

## 存储过程

#### 简介

存储过程可以认为是多条MySQL语句的集合，但是其又不仅仅只是MySQL语句的简单堆切，更形象一点的或许可以理解
为是MySQL中的批处理程序

#### 为什么要使用存储过程

- 简化应用层逻辑

- 安全,限制对基础数据的访问以减少数据出错

- 提高性能,存储过程比单独的SQL语句要快

#### 修改终止符

```text
DELIMITER //
...
//
DELIMITER;

除\ 符号外，任何字符都可以用做语句分隔符
```

#### 查看存储过程创建

```text
SHOW CREATE PROCEDURE ordertotal;
```

#### 变量传递

- 调用时,变量传递皆以 @ 开头

声明时,变量分为 IN 和 OUT 两种类型，分别表示传入和传出
```text
CREATE PROCEDURE ordertotal(
    IN onumber INT,
    IN taxable BOOLEAN,
    OUT ototal DECIMAL(8,2)
)COMMENT 'Obtain order total,optionally add tax'
```

#### 变量赋值

```text
SELECT Avg(prod_price) INTO pa FROM products;
```

#### 变量声明

```text
DECLARE total DECIMAL(8,2);
```

#### 注释

```text
--
```

#### 调用存储过程

```text
CALL ordertotal(20009,@total);
```

#### 一个完整的存储过程实例

```text
-- Name: ordertotal
-- Parameters: onumber = order number
--             taxable = 0 if not taxable ,1 if taxable
--             ototal = order total variable
CREATE PROCEDURE ordertotal(
    IN onumber INT,
    IN taxable BOOLEAN,
    OUT ototal DECIMAL(8,2)
)COMMENT 'Obtain order total,optionally add tax'
BEGIN
    --Declare variable fot total
    DECLARE total DECIMAL(8,2);

    --Declare tax percentage
    DECLARE taxrate INT DEFAULT 6;

    --GET the order total
    SELECT SUM(item_price * quantity)
    FROM orderitems
    WHERE order_num = onumber
    INTO total;

    --Is this taxable 
    IF taxable THEN
        --Yes,so add taxrate to the total
        SELELCT total + (total/100*taxrate) INTO total;
    END IF;

    --Finally,save to out variable
    SELECT total INTo ototal;
END;

//调用
CALL ordertotal(20005,0,@total);

//获取计算结果total
SELECT @total;
```

## 游标

#### 概述

游标（cursor）是一个存储在MySQL中的数据库查询，它不是一条SELECT语句，而是被该语句检索出来的结果即。通过游标，我们可以对结果即进行跟进一步的处理。
MySQL的游标只能被用于存储过程和函数

#### 声明游标

```text
CREATE PROCEDURE processorders()
BEGIN
    DECLARE ordernumbers CURSOR
    FOR 
    SELECT order_num FROm orders;
END;
```

#### 打开游标

```text
OPEN ordernumbers;
```

#### 关闭游标

```text
CLOSE ordernumbers;
```

#### REPEAT … FETCH … UNTIL () END REPEAT

```text
OPEN ordernumbers;
REPEAT
    FETCH ordernumbers INTO o
UNTIL done END REPEAT;
CLOSE ordernumbers;
```

#### DECLARE CONTINUE HANDLER FOR …

```text
//设置条件
//当SQLSTATE ‘02000’ 出现时，设置done = 1
DECLARE CONTINUE HANDLER FOR SQLSTATE '0200' SET done = 1;
```

#### 一个完整的游标例子

```text
CREATE PROCEDURE processorders()
BEGIN
    DECLARE done BOOLEAN DEFAULT 0;
    DECLARE o INT;
    DECLARE t DECIMAL(8,2);

    DECLARE ordernumbers CURSOR
    FOR
    SELECT order_num FROM orders;
    DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done = 1;

    CREATE TABLE IF NOT EXISTS ordertotals
        (order_num INT, total DECIMAL(8,2));

    OPEN ordernumbers;

    REPEAT
        FETCH ordernumbers INTO o;
        CALL ordertoal(o,1,t);

        INSERT INTO ordertotals(order_num,total)
        VALUES(o,t);
    UNTIL doen END REPEAT;
    
    CLOSE ordernumbers;
END;
```

## 触发器

#### 简介

通过创建触发器，设置在DELETE/UPDATE/INSERT的BEFORE/AFTER时触发一个事件。

#### 可以使用触发器的语句

- DELETE

- INSERT

- UPDATE

#### 创建触发器

```text
CREATE TABLE account(acct_num INT,amount DECIMAL(10,2));

CREATE TRIGGER ins_sum BEFORE INSER ON account
FOR EACH ROW SET @sum = @sum + NEW.amount;

SET @sum = 0;
INSERT INTO account VALUES(137,14.98),(141,1937.50),(97,-100.00);
SELECT @sum AS 'Total amount inserted';
+-----------------------+
| Total amount inserted |
+-----------------------+
| 1852.48               |
+-----------------------+
//在向products插入一条数据以后会显示‘';
```

#### 删除一个触发器

```text
DROP TRIGGER test.ins_sum;
```

#### Example

```text
delimiter //
CREATE TRIGGER upd_check BEFORE UPDATE ON account
FOR EACH ROW
BEGIN
   IF NEW.amount < 0 THEN
      SET NEW.amount = 0;
   ELSEIF NEW.amount > 100 THEN
      SET NEW.amount = 100;
   END IF;
END;//
delimiter ;
```

#### 注意点：

- 不能在同一个表上添加多个同类型的触发器（如：不能针对同一个表添加两个BEFORE UPDATE 触发器）

- 每个数据库中，触发器的名字必须是唯一的

## 用户权限管理

#### 查看用户

```text
USE mysql;
SELECT user FROM user;
```

#### 创建用户

```text
CREATE USER zz IDENTIFIED BY 'p@ssw0rd';

//IDENTIFIED BY 设置密码，最终保持的密码是经过加密的
```

#### 重命名用于

```text
RENAME USER zz TO zoth;
//需要有CREATE 权限
```

#### 删除账号

```text
DROP USER zoth;
```

#### 查看访问权限

```text
SHOW GRANTS FOR zoth;
```

#### 设置访问权限 GRANT … REVOKE …

```text
GRANT SELECT ON crashcourse.* TO zoth;
//赋予zoth对crashcourse数据库中所有表的读（select）权限
```

#### 取消权限

```text
REVOKE SELECT ON crashcourse.* TO zoth;
```

#### 权限控制层次

- 整个服务器 GRANT ALL/ REVOKE ALL

- 整个数据库 ON database.*

- 特定的表 ON database.table

- 特定的列

- 特定的存储过程

#### 权限列表

权限
说明

ALL
除GRANT OPTION 外的所有权限

ALTER
ALTER TABLE

ALTER ROUTINE
ALTER PROCEDURE / DROP PROCEDURE

CREATE
CREATE TABLE

CREATE ROUTINE
CREATE PROCEDURE

CREATE TEMPORARY TABLES
CREATE TEMPORARY TABLE

CREATE USER
CREATE USER/ DROP USER / RENAME USER /REVOKE ALL PRIVILEGES

CREATE VIEW
CREATE VIEW

DELETE
DELETE

DROP
DROP TABLE

EXECURE
CALL / 存储过程

FILE
SELECT INTO OUTFILE/ LOAD DATA INFILE

GRANT OPTION
GRANT / REVOKE

INDEX
CREATE INDEX / DROP INDEX

INSERT
INSERT

LOCK TABLES
LOCK TABLES

PROCESS
SHOW FULL PROCESSLIST

RELOAD
FLUSH

PRELICATION CLIENT
服务器位置访问

REPLICATION SLAVE
???

SELECT
SELECT

SHOW DATABASES
SHOW DATABASES

SHOW VIEW
SHOW VIEW

SHUTDOWN
mysqladmin shutdown

SUPER
CHANGE MASTER, KILL ,LOGS, PURGE, MASTER, SET GLOBAL

UPDATE
UPDATE

USAGE
无访问权限

## 参考资料

- [MySQL 必知必会](http://book.douban.com/subject/3354490)

- [MySQL Trigger Syntax and Examples](http://dev.mysql.com/doc/refman/5.5/en/trigger-syntax.html)