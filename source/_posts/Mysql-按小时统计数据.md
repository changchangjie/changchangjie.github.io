---
title: Mysql-按小时统计数据
date: 2018-10-11 17:00:00
categories: 
- Mysql
---

Mysql统计一天中每小时数据量sql如下
```sql
select created, count(created)
from (
       select DATE_FORMAT( created ,'%Y-%m-%d %H') created 
       from shop_check_result 
       where created > '2018-10-08 00:00:00' and created < '2018-10-09 00:00:00' 
     ) a
group by created;
```
<!--more-->