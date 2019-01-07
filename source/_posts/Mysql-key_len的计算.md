---
title: Mysql-key_len的计算
date: 2018-11-09 11:00:00
categories: 
- Mysql
---

当用Explain查看SQL的执行计划时，里面有列显示了 key_len 的值，根据这个值可以判断索引的长度，在组合索引里面可以更清楚的了解到了哪部分字段使用到了索引
<!--more-->

### 建表
```sql
CREATE TABLE student (
  id int(11) AUTO_INCREMENT primary key ,
  name varchar(10) DEFAULT NULL,
  cid int(11) DEFAULT NULL,
  created timestamp
) ENGINE=InnoDB  DEFAULT CHARSET=utf8;
```

### 索引
```sql
alter table student drop index index_name_cid_created;
alter table student add index index_name_cid_created (name, cid, created);
```

### 执行计划
```sql
EXPLAIN SELECT * FROM student WHERE name = 'aaaa' and cid = 1 and created > '2018-11-11 11:11:11';
```
### MySQL 字段类型占用空间
常用类型存储需求
[完整对应链接](https://blog.csdn.net/free_ant/article/details/52936722)

| 类型        | 存储空间   |
| --------   | -----:  |
| TINYINT     | 1个字节 |
| SMALLINT        |   2个字节   |
| INT, INTEGER        |    4个字节    |
| BIGINT        |   8个字节   |
| DATE        |   3个字节   |
| DATETIME        |   8个字节   |
| TIMESTAMP        |   4个字节   |
| CHAR(M)        |   M个字节，0 <= M <= 255   |
| VARCHAR(M)        |   2+3M，0 <= M <= 65535   |
> int（10）这里的10指的是数值的宽度，并不是字节数，如果使用了zerofill，当结果小于显示宽度时，左边用0补齐，当等于或超过显示宽度时正常显示，没啥用
> char(20)、varchar(20)，20指的是字符而不是字节（4.0版本以上，以下指的是字节）
> 字符和字节的转换要看字符集，utf-8下，1字符=3字节；gbk下，1字符=2字节

### 结果分析
![](https://static.changjie.me/img/WX20181109-111215.png)

此时索引长度是42个字节
由于表的字符集是utf8,  1char=3byte，10 * 3=30
由于varchar是变长类型，需要额外的2个字节，固定长度字段(char,int)不需要额外的字节
由于字段默认为null，所以需要1个字节的额外空间
(30 + 2 + 1) + (4 + 1) + 4 = 42

### 最左前缀原则
对于联合索引mysql会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配，比如a = 1 and b = 2 and c > 3 and d = 4 如果建立(a,b,c,d)顺序的索引，d是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整


