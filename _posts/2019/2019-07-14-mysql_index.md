---
layout: post
title: mysql索引及合理使用
category: mysql
tags: [mysql]
---

数据库必备技能

## 索引的定义

MySQL官方对索引的定义为:索引(Index)是帮助MySQL高效获取数据的数据结构.可以得出索引的本质就是数据结构

## 优势

  类似大学图书馆建书目录索引,提高数据检索的效率,降低数据库的IO成本
  通过索引列对数据进行排序,降低数据排序成本,降低了CPU的消耗
  可以加速表和表之间的连接

## 劣势
  实际上索引也是一张表,该表保存了主键与索引字段,并指向实体表的记录,所以索引也是要占内存空间的

  虽然索引大大提高了查询速度,同时都会降低更新表的速度,如对表进行insert,update和delete因为更新表时,MySQL不仅要保存数据,还要保存一下索引文件每次更新添加了索引的字段,都会调整因为更新所带来的键值变化后的索引信息

  索引只是高效的一个因素,如果你的MySQL有大数据量的表,就需要花时间研究建立最优秀的索引, 或优化查询方法

## 索引的分类

一般来说索引本身很大,不适合全部存储在内存中,因此索引往往以索引文件的形式存储在磁盘上

我们平常所说的索引,如果没有特别指明,都是指B树(多路搜索树,并不一定是二叉的)结构组织的索引,

其中聚集索引,次要索引,覆盖索引,复合索引,前缀索引,唯一索引默认都是使用B+树索引,统称索引.当然,除了B+树这种类型的索引之外,还有哈稀索引(hash index)

索引的分类

* 单值索引:即一个索引只包含单个列,一个表不\可以有多个单列索引
* 唯一索引:索引列的值必须唯一,但允许有控制,例如手机号,银行卡号等值必须是唯一
* 复合索引:即一个索引包含多个列,例如手机号和银行卡号一起,如果一个表中的数据在查询时有多个字段总是同时出现则这些字段就可以作为复合索引

## 数据结构

B-Tree\B+Tree\HashTree。

-- todo

## 使用索引的理由
由于mysql在默认情况下，表中的数据记录是没有顺序可言的，也就是说在数据检索过程中，符合条件的数据存储在哪里，我们是完全不知情的，如果使用select语句进行查询，数据库会从第一条记录开始检索。也称为全表扫描。索引是为检索而存在的。

## 索引的分类与创建
MySQL 索引可以分为单列索引、复合索引、唯一索引、主键索引等。下面分别介绍

### 查看表的索引

```
show index from tableName;
```

###  删除索引

```
drop index index_name on tableName;
```

### 单列索引

```
-- 直接创建
CREATE INDEX index_name ON tbl_name(index_col_name);

-- 修改表结构
ALTER TABLE tbl_name ADD INDEX index_name ON (index_col_name);

-- 或者创建表的时候初始化
CREATE TABLE `table` (
`id` int(11) NOT NULL AUTO_INCREMENT ,
`name` varchar(32)  NOT NULL ,
PRIMARY KEY (`id`),
indexName (name(32))  -- 创建name字段索引
);

```

### 复合索引

复合索引：复合索引是在多个字段上创建的索引。复合索引遵守“最左前缀”原则。即在查询条件中使用了复合索引的第一个字段，索引才会被使用。因此，在复合索引中索引列的顺序至关重要

```
-- 直接创建
CREATE INDEX index_name ON tbl_name(index_col_name1,index_col_name2,...);

-- 修改表结构
ALTER TABLE tbl_name ADD INDEX index_name ON (index_col_name1,index_col_name2,...);


-- 创建表时直接指定
CREATE TABLE `table` (
`id` int(11) NOT NULL AUTO_INCREMENT ,
`name` varchar(32)  NOT NULL ,
'address' varchar(32) ,
......  -- 其他字段
PRIMARY KEY (`id`),
indexName (name(32),address(32))

```

### 唯一索引
唯一索引限制列的值必须唯一，但允许有空值。可以是单列也可以是多列组合

```
-- 创建唯一索引
CREATE UNIQUE INDEX index_name ON tbl_name(index_col_name[,...]);

-- 添加（通过修改表结构）
ALTER TABLE tbl_name ADD UNIQUE INDEX index_name ON (index_col_name[,...]);


-- 创建表时直接指定
CREATE TABLE `table` (
`id` int(11) NOT NULL AUTO_INCREMENT ,
`name` varchar(32)  NOT NULL ,
......  -- 其他字段
PRIMARY KEY (`id`),
UNIQUE indexName (name(32))
);

```

## 合理的使用索引提升性能
* where条件最适合做索引
* 不要给重复率高极高，例如性别（男/女），状态（0/1，y/n）。因为数据结构是树状的，无论怎么检索都会得出一大半的数据
* 复合索引查询sql要遵循“最左前缀法则”
* 不过建立过多的索引，每一次的更新，删除，插入都会维护该表的索引，更多的索引意味着占用更多的空间
* 使用InnoDB存储引擎时，记录(行)默认会按照一定的顺序存储，如果已定义主键，则按照主键顺序存储，由于普通索引都会保存主键的键值，因此主键应尽可能的选择较短的数据类型，以便节省存储空间
* 不要尝试在索引列上使用函数。
