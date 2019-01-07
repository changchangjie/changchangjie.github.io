---
title: Mysql-引号对索引的影响
date: 2018-11-09 15:30:00
categories: 
- Mysql
---

在进行查询操作时，如果对于字符型字段进行where查询不加引号时不会走索引
<!--more-->

### 创建表
```sql
CREATE TABLE `student` (
  `id` int(11) AUTO_INCREMENT primary key ,
  `name` varchar(10) DEFAULT NULL,
  `cid` int(11) DEFAULT NULL,
  created timestamp
) ENGINE=InnoDB  DEFAULT CHARSET=utf8;
```
使用前文存储过程随机插入2000条数据

### 创建索引
```sql
alter table student add index index_name (name);
```
创建字符型name字段的单列索引

### 结果对比
![](https://static.changjie.me/img/WX20181109-153743@2x.png)
可见对于字符型name字段加了引号时走了索引，不加引号时不走索引

### 结论
1. 查询关键字要保证和查询字段类型一致，在类型不一致时MySQL需要进行隐式转换，转换成相同的类型才能进行比较。隐式转换可能会使索引失效，严重影响系统性能。尤其是字段为字符型但是查询关键字没有加引号的情况下，开销相当的巨大
2. 如果类型不一致，有一方为数值行，MySQL会优先转换成数值型
3. 数值类型的查询效率要比字符型高，字符型索引长度较大
4. 如果字段为字符型，查询关键字一定要加上引号
5. 总之要避免隐式转换，隐式转换本身会有系统开销，而且会造成不可预知的影响
