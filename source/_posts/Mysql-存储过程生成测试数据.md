---
title: Mysql-存储过程生成测试数据
date: 2018-11-09 15:00:00
categories: 
- Mysql
---

在进行查询操作的性能测试时，往往需要测试大数据量模式下的查询功能的性能
MySQL表数据量的大小，也会影响索引的选择
<!--more-->

### 随机函数
```sql
CREATE DEFINER=`root`@`%` FUNCTION `rand_string`(n INT) RETURNS varchar(255) CHARSET utf8mb4
    DETERMINISTIC
BEGIN
    DECLARE chars_str varchar(100) DEFAULT 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
    DECLARE return_str varchar(255) DEFAULT '' ;
    DECLARE i INT DEFAULT 0;
    WHILE i < n DO
        SET return_str = concat(return_str, substring(chars_str, FLOOR(1 + RAND() * 62), 1));
        SET i = i + 1;
    END WHILE;
    RETURN return_str;
END;
```
函数说明
rand()：用于生成[0,1)之间的随机数
substring(str,pos,len)：从字符串第pos个字符开始，截取len个字符
concat(str1,str2)：用于字符串拼接
可知上面函数是生成指定位数的字符串

### 存储过程
```sql
CREATE DEFINER=`root`@`%` PROCEDURE `insert_student`(IN n int)
BEGIN
    DECLARE i INT DEFAULT 1;
    WHILE (i <= n) DO
        INSERT INTO student (name, cid) VALUES (rand_string(8), FLOOR(RAND() * 100));
        SET i = i + 1;
    END WHILE;
END;
```
用rand_string函数给name字段生成8位随机字符串，cid字段为[0,100)的整数，随机插入指定条

### 运行存储过程
```
CALL insert_student(1000000);
```