---
layout: '[post]'
title: SQL 温习
date: 2017-04-27 21:21:04
tags: SQL MySQL Terminal Learning
---


## 背景
公司搭建的接口文档平台 rap 使用了 MySQL 存储数据，后台同事对数据库表做了修改，作为 rap 测试的我在操作 rap 后需要去数据库看数据。
所以学习了一下 SQL 的基本操作。

## 本文内容
* Ubuntu 终端下连接远程 MySQL server。
* 复习 SQL 教程的笔记。

## 安装 client
因为只需要连接远程数据库并修改数据，不需要本地提供 mysql 服务，所以只安装 mysql 客户端。
```bash
$ sudo apt install mysql-client
```

## 连接到远程服务器
```
$ mysql -h 10.xx.xxx.xxx -u root -p -P 3306
```
其中：
`mysql`： 终端中的 MySQL 命令
`-h`：指定连接的 server host
`-u`：指定连接的用户名
`-p`：指定连接的密码，如果没有在命令行中指定，**会在该命令执行后询问用户输入**，这样可以避免黑客通过查看命令历史获取到 server 的连接密码。
直接输入密码的话：
 * `-pxxxx`，不用加空格，可能跟 MySQL 命令解析命令行参数的规则有关。
 * 或`--password=xxxx`，完整参数。

`-P`：指定 server 的 port，默认为 3306，所以可以省略

然后输入数据库密码，就可以连接到远程 server，此时终端会等待用户输入 SQL 语句。
```bash
Welcome to the MySQL monitor.  Commands end with ; or \g.
...

mysql>
```

## 数据库查询
* 注意：
 * **数据库语句后面需要加分号才会执行**。
 * **数据库的关键字不区分大小写**。

* 显示数据库
```bash
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| test_by_chenhanjie |
| bbb                |
| ccc                |
| ddd                |
+--------------------+
4 rows in set (0.00 sec)
```

* 选择使用的数据库
没有默认正在使用的数据库，所以需要先选择。
```bash
mysql> use test_by_chenhanjie;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A
```

* 表查询
```bash
mysql> show tables;
+------------------------------+
| Tables_in_test_by_chenhanjie |
+------------------------------+
| persons                      |
| scores                       |
| sss                          |
| test                         |
+------------------------------+
```

## 数据库操作
* 新建数据库
```bash
mysql> create database test_by_chenhanjie;
Query OK, 1 row affected (0.03 sec)
```

* 新建表
```bash
mysql> create table test ( id int, score int, info text);
```

**此处省略一万字。。。**

## 参考资料
参考资料不是放在最后的吗，怎么出了个大标题？
因为一个一个讲命令没有太大意义，**不如系统看下文档**。我看的是 w3schools 的教程，顺便强迫自己看英文。
> https://www.w3schools.com/sql/default.asp

## 应该关注的点
* 数据库组成
 一个数据库服务（RDBMS）包含多个数据库（Database）；一个数据库由多个表（Table）组成；一个表由多个记录（Record）组成，一个记录为一行（Row）；一个记录有多个字段（Field），一个表中的同一个字段为一列（Row）。

* SQL 语法
SQL 语法的关键在于理解关键字和关键字的用法，关键字是基础，对一个 SQL 语句的理解是思维的挑战和提示。

* DML 和 DDL
DML 对数据库中的数据进行**增删改查**操作；DDL 对数据库和表进行操作，用于**定义数据库**。

* 数据类型
数据类型是表中字段存储的数据类型，如`INT` `TEXT`等。

* SQL 函数
SQL 内置的函数可以用于数据的计算，如`COUNT` `NOW`等。

* 数据筛选
数据条件筛选是语句正确执行的关键。

* 操作符
包括算术操作符，位操作符，比较操作符，复合操作符（Compound Operators）和逻辑操作符。

* 难点
难点是学习中可能带来疑惑的地方，关键在于理解和实践。我总结的几个点：
 * 联合（Join）
 * 条件查询（Condition Query）
 * 别名（Aliases）
 * 主键（Primary Key）
 * 外键（Foreign Key）
 * 排序（Order）
 * 分组（Group）
 * 空（Null）
 * 操作符（Operator）
 * 索引（Index）
   索引可以提高查询速度，但是会减低修改（增删改）的效率，因为需要同时修改索引。


## 专业名词
SQL（Structured Query Language)
DML（Database Modify Language）
DDL （Database Define Language）
ANSI（American National Standards Institute）
RDBMS（Relational Database Management System）
ESC (escend)
DESC (descend)

## 后记
耗时两天，学到很多东西，欢迎批评指正。

## EOF
