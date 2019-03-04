本篇文章，我们将从索引基础开始，介绍什么是索引以及索引的几种类型，然后学习如何创建索引以及索引设计的基本原则。

本篇文章中用于测试索引创建的user表的结构如下：

![mark](http://songwenjie.vip/blog/180802/H927ahbK4F.png?imageslim)



## 什么是索引

>  索引（在 MySQL 中也叫“键key”）是存储引擎快速找到记录的一种数据结构
>
>  ——《高性能MySQL》

我们需要知道索引其实是一种数据结构，其功能是帮助我们快速匹配查找到需要的数据行，是数据库性能优化最常用的工具之一。其作用相当于超市里的导购员、书本里的目录。



## 索引类型

可以使用`SHOW INDEX FROM table_name;`查看索引详情

![mark](http://songwenjie.vip/blog/180802/Eif26fJiEc.png?imageslim)

1. **主键索引 PRIMARY KEY**

   它是一种特殊的唯一索引，不允许有空值。一般是在建表的时候同时创建主键索引。

   注意：一个表只能有一个主键

   ![mark](http://songwenjie.vip/blog/180802/1c7D2F0f76.png?imageslim)

2. **唯一索引 UNIQUE**

   唯一索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一。

   可以通过`ALTER TABLE  table_name ADD UNIQUE (column);`创建唯一索引

   ![mark](http://songwenjie.vip/blog/180802/DBdFeKE8Fk.png?imageslim)

   ![mark](http://songwenjie.vip/blog/180802/L2jl91b6J6.png?imageslim)

   可以通过`ALTER TABLE  table_name ADD UNIQUE (column1,column2);`创建唯一组合索引

   ![mark](http://songwenjie.vip/blog/180802/mihd7Hm5i6.png?imageslim)

   ![mark](http://songwenjie.vip/blog/180802/bJbdFA9AcL.png?imageslim)

3. **普通索引 INDEX**

   最基本的索引，它没有任何限制。

   可以通过`ALTER TABLE  table_name ADD INDEX index_name (column);`创建普通索引

   ![mark](http://songwenjie.vip/blog/180802/17CmJIIJhD.png?imageslim)

   ![mark](http://songwenjie.vip/blog/180802/4fA7L6kBBm.png?imageslim)

4. **组合索引 INDEX**

   组合索引，即一个索引包含多个列。多用于避免回表查询。

   可以通过`ALTER TABLE  table_name  ADD INDEX index_name(column1, column2, column3);`创建组合索引

   ![mark](http://songwenjie.vip/blog/180802/CLGIKiAC6J.png?imageslim)

   ![mark](http://songwenjie.vip/blog/180802/295B9bGi67.png?imageslim)

5. **全文索引 FULLTEXT**

   全文索引（也称全文检索）是目前搜索引擎使用的一种关键技术。

   可以通过`ALTER TABLE table_name ADD FULLTEXT (column);`创建全文索引

   ![mark](http://songwenjie.vip/blog/180802/AjfLLkhdH1.png?imageslim)

   ![mark](http://songwenjie.vip/blog/180802/bA1a1m49cL.png?imageslim)


索引一经创建不能修改，如果要修改索引，只能删除重建。可以使用`DROP INDEX index_name ON table_name;`删除索引。



## 索引设计的原则

1. 适合索引的列是出现在where子句中的列，或者连接子句中指定的列

2. 基数较小的类，索引效果较差，没有必要在此列建立索引

3. 使用短索引，如果对长字符串列进行索引，应该指定一个前缀长度，这样能够节省大量索引空间

4. 不要过度索引。索引需要额外的磁盘空间，并降低写操作的性能。在修改表内容的时候，索引会进行更新甚至重构，索引列越多，这个时间就会越长。所以只保持需要的索引有利于查询即可。




## 参考

* 《深入浅出MySQL》

