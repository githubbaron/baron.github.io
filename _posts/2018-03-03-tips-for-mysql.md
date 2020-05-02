---
layout: post
title: "Tips for MySQL"
date: 2018-03-03 06:43:45
image: 'http://res.cloudinary.com/degzyaemb/image/upload/c_scale,w_750/v1520061329/infernal_afairs1_1.png'
description: 关于mysql的一些凌乱的总结，后续会再修改。
category: 'sql'
tags:
- mysql
- sql
twitter_text:
introduction: 关于mysql的一些凌乱的总结，后续会再修改。
---

## 主码、候选码、超码
关于主码、候选码以及超码相信大家通过定义就能很好的理解了。
> **主码**：超码是一个或多个属性的集合,这些属性的组合可以使我们在一个实体集中唯一的标识一个实体。<br>
> **候选码**：若关系中的一个属性或属性组的值能够唯一地标识一个元组，且他的真子集不能唯一的标识一个元组，则称这个属性或属性组做候选码。<br>
> **超码**：主关键字(primary key)是表中的一个或多个字段，它的值用于唯一地标识表中的某一条记录。

而候选码与超码的不同就在于，在一个表中，候选码可能有很多个，但是主码只有一个。
比如说，在一个students表中，学生身份证可以作为一个候选码，学生姓名+年龄如果唯一的话，也可作为一个候选码。
但是你只能定义学生身份证为一个主码。

## 子查询与联结查询

下面将对于同一问题使用子查询以及联结查询两种方法来实现以便更好的理解两种方式。

1.假如你发现某物品（其ID为"DTNTR"）存在问题，因此希望查找生产该物品的供应商生产的其他物品。

子查询：
```sql
SELECT prod_id, prod_name FROM products WHERE vend_id = (
    SELECT vend_id FROM  products WHERE prod_id = "DTNTR");
```

联结查询：
```sql
SELECT p1.prod_id, p1.prod_name
    FROM products AS p1, products AS p2
    where p1.vend_id = p2.vend_id and p2.prod_id = "DTNTR";
```

2.需要显示customers表中每个客户的订单总数（cust_name,cust_state,orders）

子查询：
```sql
SELECT cust_name, cust_state,
    (SELECT COUNT(*) FROM orders WHERE order.id = customers.cust_id)
    FROM customers;
```

联结查询：(这里如果使用LEFT JOIN的话还会查出没有下过订单的顾客)
```sql
SELECT cust_name, cust_state, COUNT(orders.order_num) AS num_ord
    FROM customers INNER JOIN orders
    ON customers.id = orders.cust_id
    GROUP BY customers.id;
```

## 视图

> **视图**: 是指计算机数据库中的视图，是一个虚拟表，其内容由查询定义。
同真实的表一样，视图包含一系列带有名称的列和行数据。但是，视图并不在数据库中以存储的数据值集形式存在。
行和列数据来自由定义视图的查询所引用的表，并且在引用视图时动态生成。

简单来说，就是当你执行一个视图的时候，视图会尝试从基表中获取相应的数据并返回，
但是实际上他并不包括数据在其中。由于这种特性，视图可以应用到很多场景。

### 视图的使用范围

1. 简化复杂的SQL操作。其实就是将复杂的操作封装在视图中，然后利用简单调用视图的代码就可以实现相同的功能。
2. 保护数据。可以给用户授予表的特定部分的访问权限而不是整个表的访问权限。
3. 重用SQL语句。
4. 更改数据格式和表示。就是可以通过视图封装concat语句（将两个属性更改成你想要的形式），而不需要每次都进行装换。

### 视图的创建以及使用语句
```sql
CREATE VIEW view_name
    SELECT attribute as alias FROM table_name
    ORDER BY attribute1;

SELECT * FROM view_name;
```

## 存储过程

> 存储过程（Stored Procedure）是在大型数据库系统中，一组为了完成特定功能的SQL 语句集，存储在数据库中，
经过第一次编译后再次调用不需要再次编译，用户通过指定存储过程的名字并给出参数（如果该存储过程带有参数）来执行它。存储过程是数据库中的一个重要对象。

由于其已经编译过了，所以运行起来会比普通的操作更加的迅速。
可以将存储过程理解成为一个方法，其能够传入相应的参数。比如，当你需要计算某个订单的总价格（包括税收）时，
你可以将所有的判断和逻辑写成一个存储过程，在需要的时候进行调用存储过程就可以了。

```sql
CREATE PROCEDURE method_name (
    IN/OUT attribute type,
    ....
)
BEGIN
    SELECT * FROM table_name;
END;

CALL method_name();
```

## 数据库引擎
两个最常使用的引擎分别为MyISAM和InnoDB。
<table style="width: 500px;">
    <tr>
        <th>数据库引擎</th>
        <th>MyISAM</th>
        <th>InnoDB</th>
    </tr>
    <tr>
        <td>支持事务</td>
        <td>❌</td>
        <td>✔️</td>
    </tr>
    <tr>
        <td>支持表级锁</td>
        <td>✔️</td>
        <td>✔️</td>
    </tr>
    <tr>
        <td>支持行级锁</td>
        <td>❌️</td>
        <td>✔️</td>
    </tr>
    <tr>
        <td>支持外键</td>
        <td>❌️</td>
        <td>✔️</td>
    </tr>
    <tr>
        <td>支持全文搜索</td>
        <td>✔️</td>
        <td>❌</td>
    </tr>
    <tr>
        <td>移植、备份及恢复是否方便</td>
        <td>✔️</td>
        <td>❌</td>
    </tr>
</table>

*可以对不同的表使用不同的数据库引擎。*

## CURD语法

**SELECT**:
```sql
SELECT attribute1, attribute2 ...
    FROM table(table INNER/LEFT/RIGHT JOIN table)
        WHERE <expression> AND/OR <expression>
            GROUP BY attribute HAVING <expression>
                ORDER BY attribute3, attribute4 ...;
```

**CREATE**:
```sql
CREATE TABLE table(
    attribute1  type(int/char()/varchar()/text) NULL/NOT NULL (AUTO_INCREMENT),
    attribute2  type(int/char()/varchar()/text) NULL/NOT NULL DEFAULT value,
    attribute3  type(int/char()/varchar()/text) NULL/NOT NULL,
    PRIMARY KEY(attribute1, attribute2 ...),
    FOREIGN KEY(attribute2) REFERNCES table2(attribute)
) ENGINE = InnoDB/MyISAM;
```

**UPDATE**:
```sql
UPDATE table SET attribute = '',
    attribute1 = '' where <expressions>;
```

**DELETE**:
```sql
DELETE FROM table WHERE <expressions>;
```

## 数据库的性能优化

1. 一般来说，存储过程执行的比一条一条地执行其中的各条MySQL语句快。
2. 使用正确的数据类型。
3. 决不要检索比需求还要多的数据。
4. 必须索引数据库表以改善数据检索的性能。
5. 通过使用多条SELECT语句和连接他们的UNION语句来代替复杂的OR条件。
6. LIKE很慢。一般来说，最好是使用FULLTEXT。
对于数据库更多的优化可以看<a href="https://www.jianshu.com/p/01b9f028d9c7">MySQL优化原理2</a>

## Summary
1. 外键不能跨引擎。
