---
layout: post
title: mysql的数据备份、还原、日志
category: mysql
tags: [mysql]
---

数据库必备技能

## 初始化数据库

### 1.登陆

```
$ mysql -u root -p
```

### 创建数据库

```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| qiu                |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> use qiu;
Database changed
mysql> create database test
    -> ^C
mysql> create database test;
Query OK, 1 row affected (0.00 sec)

mysql> use test;
Database changed
```

### 3.新建一张表，并新增一条数据

```
mysql> create table user(id int,user_name varchar(64),age varchar(4));
Query OK, 0 rows affected (0.01 sec)

mysql> insert user(id,user_name,age) values(1,'木由','22');
Query OK, 1 row affected (0.00 sec)

mysql> select * from user;
+------+-----------+------+
| id   | user_name | age  |
+------+-----------+------+
|    1 | 木由      | 22   |
+------+-----------+------+
1 row in set (0.00 sec)

```

## 备份

### 退出mysql

```
mysql> exit
Bye
```

### 创建备份文件

```
$ cd ~

$ touch ~/test_back_up.sql

```

### 备份

```
$ mysqldump -u root -p test > ~/test_back_up.sql

```

### 备份文件样本

这里就备份好了一个库的表和数据了。可以查看备份文件。

从下面的备份可以看出来，先会删除对应备份过的表，所以不能随便还原的哦；


```
-- MySQL dump 10.13  Distrib 8.0.15, for macos10.14 (x86_64)
--
-- Host: localhost    Database: test
-- ------------------------------------------------------
-- Server version	8.0.15

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
 SET NAMES utf8mb4 ;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

--
-- Table structure for table `user`
--

DROP TABLE IF EXISTS `user`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
 SET character_set_client = utf8mb4 ;
CREATE TABLE `user` (
  `id` int(11) DEFAULT NULL,
  `user_name` varchar(64) DEFAULT NULL,
  `age` varchar(4) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `user`
--

LOCK TABLES `user` WRITE;
/*!40000 ALTER TABLE `user` DISABLE KEYS */;
INSERT INTO `user` VALUES (1,'木由','22');
/*!40000 ALTER TABLE `user` ENABLE KEYS */;
UNLOCK TABLES;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

-- Dump completed on 2019-07-16  1:00:50

```


## 还原

### 先删除user表

```
mysql> show tables;
+----------------+
| Tables_in_test |
+----------------+
| user           |
+----------------+
1 row in set (0.00 sec)

mysql> drop table user;
Query OK, 0 rows affected (0.01 sec)

mysql> show tables;
Empty set (0.00 sec)
```

### 还原

```
mysql> exit
Bye

$ mysql -u root -p test < ~/test_back_up.sql
```

### 检查一下

```
mysql> use test

mysql> select * from user;
+------+-----------+------+
| id   | user_name | age  |
+------+-----------+------+
|    1 | 木由      | 22   |
+------+-----------+------+
1 row in set (0.00 sec)
```

## mysql 日志文件查看

查询安装路径，可根据环境变量
```
cat ~/.bash_profile
```

我的电脑是：PATH=$PATH:/usr/local/mysql/bin

进入目录查看文件

```
cd /usr/local/mysql

sudo ls data/
```



### 开启日志的命令

```
SET GLOBAL log_output = 'TABLE';SET GLOBAL general_log = 'ON';  //日志开启

SET GLOBAL log_output = 'TABLE'; SET GLOBAL general_log = 'OFF';  //日志关闭
```


### 日志介绍

xxxx.log ：记录mysql的活动

xxxx.err ：记录错误日志

xxxx-bin ：记录二进制日志，记录更新的所有语句

xxxx-slow.log ： 记录查询缓慢的日志
