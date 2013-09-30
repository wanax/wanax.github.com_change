---
layout: post
title: "6个有用的MySQL语句(转)"
date: 2012-06-28 01:55
comments: true
categories: Tec
---
###1. 计算年数
	
>你想通过生日来计算这个人有几岁了。

``` sql
	SELECT	
	DATE_FORMAT(FROM_DAYS(TO_DAYS(now()) - TO_DAYS(@dateofbirth)), '%Y')
	 + 0;
```	 
<!--more-->
	 
###2. 两个时间的差

>取得两个 datetime 值的差。假设 dt1 和 dt2 是 datetime 类型，其格式为 ‘yyyy-mm-dd hh:mm:ss’，那么它们之间所差的秒数为：

``` sql
	UNIX_TIMESTAMP(
	 dt2 ) - UNIX_TIMESTAMP( dt1 )
```
	 
除以60就是所差的分钟数，除以3600就是所差的小时数，再除以24就是所差的天数。

###3. 显示某一列出现过N次的值

``` sql
	SELECT id FROM tbl GROUP BY id HAVING COUNT(*) = N;
```
	
###4. 计算两个日子间的工作日

>所谓工作日就是除出周六周日和节假日。

``` sql
	SELECT COUNT(*) FROM calendar WHERE d BETWEEN Start AND Stop  AND DAYOFWEEK(d) NOT IN(1,7) AND holiday=0
```
	
###5. 查找表中的主键

``` sql
	SELECT k.column_name FROM information_schema.table_constraints t JOIN information_schema.key_column_usage k USING  (constraint_name,table_schema,table_name) WHERE t.constraint_type='PRIMARY
	 KEY' AND t.table_schema='db' AND t.table_name=tbl'
```
	 
###6. 查看你的数库有多大

``` sql
	SELECT table_schema AS 'Db Name',  Round(Sum( data_length + index_length ) / 1024 / 1024, 3 ) AS 'Db Size (MB)',  Round( Sum( data_free ) / 1024 / 1024, 3 ) AS 'Free Space (MB)' FROM information_schema.tables GROUP BY table_schema ;
```
	
转自[这里](http://coolshell.cn/articles/3433.html)