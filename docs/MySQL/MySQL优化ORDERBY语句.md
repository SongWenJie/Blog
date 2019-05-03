![mark](http://songwenjie.vip/blog/180807/JcHglHgCgb.png?imageslim)

#  MySQL——优化ORDER BY语句

本篇文章我们将了解ORDER BY语句的优化，在此之前，你需要对索引有基本的了解，不了解的老少爷们可以先看一下我之前写过的索引相关文章。现在让我们开始吧。

链接：

## MySQL中的两种排序方式

1.**通过有序索引顺序扫描直接返回有序数据**

因为索引的结构是B+树，索引中的数据是按照一定顺序进行排列的，所以在排序查询中如果能利用索引，就能避免额外的排序操作。EXPLAIN分析查询时，Extra显示为Using index。

2.**Filesort排序，对返回的数据进行排序**

所有不是通过索引直接返回排序结果的操作都是Filesort排序，也就是说进行了额外的排序操作。EXPLAIN分析查询时，Extra显示为Using filesort。



## ORDER BY优化的核心原则

**尽量减少额外的排序，通过索引直接返回有序数据。**



## ORDER BY优化实战

用于实验的customer表的索引情况：

![mark](http://songwenjie.vip/blog/180804/0k0D7hlE1E.png?imageslim)

首先要注意：

**MySQL一次查询只能使用一个索引，如果要对多个字段使用索引，建立复合索引。**



### ORDER BY优化

**1.查询的字段，应该只包含此次查询使用的索引字段和主键，其余的非索引字段和索引字段作为查询字段则不会使用索引。**

只查询用于排序的索引字段，可以利用索引排序：

`explain select store_id,email from customer order by store_id,email;`

![mark](http://songwenjie.vip/blog/180804/LAgKe8fgF1.png?imageslim)

但是要注意，排序字段在多个索引中,无法使用索引排序,查询一次只能使用一个索引：

`explain select store_id,email,last_name from customer order by store_id,email,last_name;`

![mark](http://songwenjie.vip/blog/180804/JI10H8hJhh.png?imageslim)

只查询用于排序的索引字段和**主键**，可以利用索引排序：

<font color="FF7F00" face="楷体">画外音：MySQL默认的InnoDB引擎在物理上采用聚集索引这种方式，按主键进行搜索，所以InnoDB引擎要求表必须有主键，即使没有显式指定主键，InnoDB引擎也会生成唯一的隐式主键，也就是说索引中必定有主键。</font>

` explain select customer_id,store_id,email from customer order by store_id,email;`

![mark](http://songwenjie.vip/blog/180804/dA530dclB2.png?imageslim)

查询用于排序的索引字段和主键之外的字段，不会利用索引排序：

`explain select store_id,email,last_name from customer order by store_id,email;`

![mark](http://songwenjie.vip/blog/180804/dh2ILLFhbH.png?imageslim)

`explain select * from customer order by store_id,email;`

![mark](http://songwenjie.vip/blog/180804/IH78HmFcdJ.png?imageslim)



###  WHERE + ORDER BY 优化

1.**排序字段在多个索引中,无法利用索引排序**

排序字段在多个索引（不在同一个索引）中,无法利用索引排序：

`explain select * from customer where last_name='swj' order by last_name,store_id;`

![mark](http://songwenjie.vip/blog/180804/m7gJIFgC0c.png?imageslim)

<font color="FF7F00" face="楷体">画外音：当排序字段不在同一个索引时，无法满足在一颗B+树中完成排序，必须再进行一次额外的排序</font>

**排序字段在一个索引中,并且WHERE条件和ORDER BY使用相同的索引**,可以利用索引排序：

`explain select * from customer where last_name='swj' order by last_name;`

![mark](http://songwenjie.vip/blog/180804/b6FhDdhmD6.png?imageslim)

当然组合索引也可以利用索引排序：

注意字段store_id,email在一个组合索引中

`explain select * from customer where store_id = 5 order by store_id,email;`

![mark](http://songwenjie.vip/blog/180804/6d0Bf6JJ9D.png?imageslim)



2.**排序字段顺序与索引列顺序不一致,无法利用索引排序**

<font color="FF7F00" face="楷体">画外音：这条是针对组合索引而言的，我们都知道使用组合索引必要要遵循**最左原则**，WHERE子句必须有索引中第一列，虽然ORDER BY子句没有这个要求，但是也要求排序字段顺序和组合索引列顺序匹配。我们平常在使用组合索引的时候，一定要养成按照组合索引列顺序书写的好习惯。</font>

排序字段顺序与索引列顺序不一致,无法利用索引排序：

`explain select * from customer where store_id > 5 order by email,store_id;`

![mark](http://songwenjie.vip/blog/180804/ahlfcKhc8f.png?imageslim)

应该确保排序字段顺序与索引列顺序一致,这样可以利用索引排序：

`explain select * from customer where store_id > 5 order by store_id,email;`

![mark](http://songwenjie.vip/blog/180804/4408g3j3ch.png?imageslim)

ORDER BY子句不要求必须索引中第一列,没有仍然可以利用索引排序。但是有个前提条件，**只有在等值过滤时才可以，范围查询时不可以**：

`explain select * from customer where store_id =  5 order by email;`

![mark](http://songwenjie.vip/blog/180804/JeAiCi0HD4.png?imageslim)

`explain select * from customer where store_id >  5 order by email;`

![mark](http://songwenjie.vip/blog/180804/fD64hbLID0.png?imageslim)

<font color="FF7F00" face="楷体">画外音：![mark](http://songwenjie.vip/blog/180804/llBl1m5fhL.png?imageslim)

其原因其实也很简单，范围查询时，第一列a肯定是排序好的（默认是升序），而第二个字段b其实就不是排序的了。但是如果a字段有相同的值时，那么b字段就是排序的了。所以如果是范围查询，就只能对b做一次额外的排序。</font>



3.**升降序不一致,无法利用索引排序**

ORDER BY排序字段要么全部正序排序，要么全部倒序排序，否则无法利用索引排序。

`explain select * from customer where store_id > 5 order by store_id,email;`

![mark](http://songwenjie.vip/blog/180804/CCdIafEmKE.png?imageslim)

` explain select * from customer where store_id > 5 order by store_id desc,email desc;`

![mark](http://songwenjie.vip/blog/180804/8gekHiFb8h.png?imageslim)

`explain select * from customer where store_id > 5 order by store_id desc,email asc;`

![mark](http://songwenjie.vip/blog/180804/6gHB3E08g1.png?imageslim)



**总结：**

上面的优化其实可以汇总为：**WHERE条件和ORDER BY使用相同的索引，并且ORDER BY的顺序和索引顺序相同，并且ORDER BY的字段都是升序或者降序**。否则肯定需要额外的排序操作，就会出现Filesort。



### Filesort优化

通过创建合适的索引能够减少Filesort的出现，但是在某些情况下，无法完全让Filesort消失，此时只能想办法加快Filesort的操作。

**Filesort的两种排序算法：**

1.两次扫描算法

首先根据条件取出排序字段和行指针信息，之后在排序区sort buffer中排序。这种排序算法需要访问两次数据，第一次获取排序字段和行指针信息，第二次根据行指针获取记录，第二次读取操作可能会导致大量随即I/O操作。优点是排序的时候内存开销较小。

2.一次扫描算法

一次性取出满足条件的行的所有字段，然后在排序区sort buffer中排序后直接输出结果集。排序的时候内存开销比较大，但是排序效率比两次扫描算法要高。

根据两种排序算法的特性，**适当加大系统变量max_length_for_sort_data的值**，能够让MySQL选择更优化的Filesort排序算法。并且在书写SQL语句时，**只使用需要的字段，而不是SELECT * 所有的字段**，这样可以减少排序区的使用，提高SQL性能。



## 参考

《深入浅出MySQL》