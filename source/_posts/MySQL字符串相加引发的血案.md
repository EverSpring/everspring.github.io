---
title: MySQL字符串相加引发的血案
date: 2022-02-16 12:57:32
author: EverSpring
top: true
toc: false
mathjax: false
categories: 数据库
tags:
  - MySQL
---
晚上接到同事电话，说有几十张表的数据全被删除了，当时懵逼了。
先说背景：环境是MySQL 5.7.16，数据是在运维过程中运行的DELETE语句导致。选择其中一条为例
```SQL
delete from fcexam where bussno in ('GBD000000016323'+'GBD000000016325')
```
fcexam的bussno结构为varchar，本意是删除GBD000000016323、GBD000000016325这两条数据，但把,写成了+。delete from改成select查询一次，发现数据全被查出来了，所以delete的所有数据。
**原因分析**
这跟MySQL中字符串加减乘除、字符串与0/1比较有关。

#### 1. 字符串加减乘除
> 在mysql当中,字符串类型间进行加减乘除运算的时候,会`截取字符串以数字开头的那一部分数字进行运算`,如果字符串前面没有数字,那么就只能截取的数值为0,那么进行加减的时候结果都是0,进行乘除的时候结果都是NULL
> 感谢链接作者：不姓马的小马哥
> 文章地址：https://www.jianshu.com/p/2ab2c0dc3cb5
```
select ('GBD000000016323'+'GBD000000016325'); --结果为0
select ('GBD000000016323'/'GBD000000016325'); --结果为null
select ('GBD000000016323'*'GBD000000016325'); --结果为0

select ('1GBD000000016323'+'GBD000000016325'); --结果为1
select ('1GBD000000016323'+'2GBD000000016325'); --结果为3
select ('1GBD000000016323'/'GBD000000016325'); --结果为null
```

#### 2. 字符串与0、1比较
MySQL boolean类型0代表false，1代表true。字符串开头没有数字时与0、1比较，如果是和0比较，得到的结果是1也就是true，和其他数字比较结果是0也就是false
```
select 'GBD000000016323'=0; --结果为1
select '1GBD000000016323'=1; --结果为1
```
所以在我们跑的脚本里面，因为bussno都是'GBD'开头，转换出来就是where 0 = 0，where条件结果为true就把所以数据都查询出来了。
```
select * from fcexam where bussno in ('GBD000000016323'+'GBD000000016325'); --等于是select * from fcexam where 0 in (0);
```
假如bussno有3条数据，'GBD000000016323'，'GBD000000016325'，'2GBD000000016325'，用上面的SQL只能查出前面2条，最后一条因为是2开头所以查询不出来

#### 总结
```SQL
delete from fcexam where bussno in ('GBD000000016323'+'GBD000000016325')
```
这条SQL相当于就是
```
delete from fcexam where 0 in (0)
```

参考资料：
1. https://www.jianshu.com/p/2ab2c0dc3cb5
2. https://stackoverflow.com/questions/22080382/mysql-why-comparing-a-string-to-0-gives-true