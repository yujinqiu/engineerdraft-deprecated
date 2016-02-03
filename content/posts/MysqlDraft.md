+++
date = "2014-12-04T13:53:18+08:00"
draft = false
title = "MysqlDraft"
description = "Mysql 手稿"

+++

## Mysql多列去重  
### 背景 
在项目中通常需要我们对数据进行去重, 第一反应应该是使用` distinct` 来进行去重, 假设数据如下

| id | name     | age  |
| --- | ------- | ---- |   
|  1 | monalisa |  100 |
|  2 | foo      |  100 |
|  3 | monalisa |   26 |
|  4 | foo      |  100 |
|  5 | bar      |  100 |
|  6 | test     |  100 |
|  7 | demo     |  100 |

<!--more-->

	select distinct name , age from driver; 但是很多时候我们需要保留 id
	
	mysql> select distinct name , age from driver;
	+----------+------+
	| name     | age  |
	+----------+------+
	| monalisa |  100 |
	| foo      |  100 |
	| monalisa |   26 |
	| bar      |  100 |
	| test     |  100 |
	| demo     |  100 |
	+----------+------+

	
采用 `select distinct id, name, age from driver` 无法实现想要的效果, 因为 distinct 看来是三元组 [id, name, age], 均不一样.  

可以采用 `select id, name, age from driver group by name, age ` 来实现. 


## Mysql 复制行
### 背景
在开发中, 经常需要初始数据, INSERT INTO <TABLE> values (....), (...) , 效率比较慢,  我们经常需要充以后的内容中复制几行.   
 

	mysql> select * from driver;
	+----+----------+------+
	| id | name     | age  |
	+----+----------+------+
	|  1 | monalisa |  100 |
	|  2 | foo      |  100 |
	|  3 | monalisa |   26 |
	|  4 | foo      |  100 |
	|  5 | bar      |  100 |
	|  6 | test     |  100 |
	|  7 | demo     |  100 |
	+----+----------+------+
	
假设我们需要复制 monalisa 行到新行里边, 然后就可以修改新行内容.   

	insert into driver (name, age) select name,  age from driver where name = 'monalisa' ; 
	
	
## Mysql 输出取消 table 内容
### 背景
我们在 Mysql 查询内容的时候, 为方便数据处理, 有时候我们不需要输出结果中带有 table 边框.  

	-s, --silent        Be more silent. Print results with a tab as separator,
                      each row on new line.

## Mysql 异常
### 1290 the mysql server is running with the read-only option 
1: 确认权限问题   
2: show variables like '%read_only%'; 

	Variable_name  Value
	read_only       On #这里应该为 OFF
3: 设置 read_only = off  

	set GLOBAL read_only = false;  