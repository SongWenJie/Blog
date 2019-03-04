## Myisam 和 InnoDB 的区别？

其实关于这两个最常用的存储引擎，无非就是看场景，其实没有绝对的好与坏，不要教条主义，适合自己业务的就是最好的。

- Myisam：表级锁，不支持事务，读性能更好，读写分离中做读（从）节点。老版本 MySQL 的默认存储引擎。表的存储分为三个文件：frm表格式，MYD/MYData 数据文件，myi 索引文件。
- InnoDB：有条件的行级锁，支持事务，更适合作为读写分离中的写（主）节点。新版本 MySQL（5.7开始）的默认存储引擎。



## 双机热备

Mysql双机互为热备方案。在进行讨论的时候一定要考虑到很多的因素，其中在各种环境下应用的时候需要格外的引起注意。按照上面说的复制模式，一般有2中方案。1，双机热备它的工作原理是使用两台服务器，一台作为主服务器（Master或者Active），运行应用系统来提供服务。另一台作为备机，安装完全一样的应用系统，但处于待机状态（Slave或者叫Standby）。当Master服务器出现故障时，通过软件诊测将Slave机器激活，保证应用在短时间内完成恢复正常使用。2，另外一种形式就是，双机互备方式则是在双机热备的基础上，两个相对独立的应用在两台机器同时运行，但彼此均设为备机，当某一台服务器出现故障时，另一台服务器可以在短时间内将故障服务器的应用接管过来，从而保证了应用的持续性，这种方式实际上是双机热备方案的一种应用。





# MySQL——通过EXPLAIN分析SQL的执行计划

![mark](http://songwenjie.vip/blog/180802/30IeblBc39.png?imageslim)在MySQL中，我们可以通过**EXPLAIN**命令获取MySQL如何执行SELECT语句的信息，包括在SELECT语句执行过程中表如何连接和连接的顺序。

![mark](http://songwenjie.vip/blog/180728/FA4eKFJgff.png?imageslim)

下面分别对**EXPLAIN**命令结果的每一列进行说明：

* **select_type**:表示SELECT的类型，常见的取值有：

  |   类型   |               说明                |
  | :------: | :-------------------------------: |
  |  SIMPLE  |   简单表，不使用表连接或子查询    |
  | PRIMARY  |       主查询，即外层的查询        |
  |  UNION   | UNION中的第二个或者后面的查询语句 |
  | SUBQUERY |         子查询中的第一个          |

* **table**:输出结果集的表（表别名）

* **type**:表示MySQL在表中找到所需行的方式，或者叫访问类型。常见访问类型如下，从上到下，性能由差到最好：

  |       ALL        |         全表扫描         |
  | :--------------: | :----------------------: |
  |    **index**     |      **索引全扫描**      |
  |    **range**     |     **索引范围扫描**     |
  |     **ref**      |    **非唯一索引扫描**    |
  |    **eq_ref**    |     **唯一索引扫描**     |
  | **const,system** | **单表最多有一个匹配行** |
  |     **NULL**     |   **不用扫描表或索引**   |

  1. **type=ALL，全表扫描，MySQL遍历全表来找到匹配行**

     一般是没有where条件或者where条件没有使用索引的查询语句

     `EXPLAIN SELECT * FROM customer WHERE active=0;`

     ![mark](http://songwenjie.vip/blog/180728/Ledh860lh0.png?imageslim)

  2. **type=index，索引全扫描，MySQL遍历整个索引来查询匹配行，并不会扫描表**

     一般是查询的字段都有索引的查询语句

     `EXPLAIN SELECT store_id FROM customer;`

     ![mark](http://songwenjie.vip/blog/180728/H1c5I109hJ.png?imageslim)

  3. **type=range，索引范围扫描，常用于<、<=、>、>=、between等操作**

     `EXPLAIN SELECT * FROM customer WHERE customer_id>=10 AND customer_id<=20;`

     ![mark](http://songwenjie.vip/blog/180728/AFjb9ai88a.png?imageslim)

     注意这种情况下比较的字段是需要加索引的，如果没有索引，则MySQL会进行全表扫描，如下面这种情况，create_date字段没有加索引：

     `EXPLAIN SELECT * FROM customer WHERE create_date>='2006-02-13' ;`

     ![mark](http://songwenjie.vip/blog/180728/EDDiFef0Ee.png?imageslim)

  4. **type=ref，使用非唯一索引或唯一索引的前缀扫描，返回匹配某个单独值的记录行**

     `store_id`字段存在普通索引（非唯一索引）

     `EXPLAIN SELECT * FROM customer WHERE store_id=10;`

     ![mark](http://songwenjie.vip/blog/180728/HFmh56bafe.png?imageslim)

     **ref**类型还经常会出现在join操作中：

     **customer**、**payment**表关联查询，关联字段`customer.customer_id`（主键），`payment.customer_id`（非唯一索引）。表关联查询时必定会有一张表进行全表扫描，此表一定是几张表中记录行数最少的表，然后再通过非唯一索引寻找其他关联表中的匹配行，以此达到表关联时扫描行数最少。

     ![mark](http://songwenjie.vip/blog/180728/C03A0GmHb8.png?imageslim)

     因为**customer**、**payment**两表中**customer**表的记录行数最少，所以**customer**表进行全表扫描，**payment**表通过非唯一索引寻找匹配行。

     `EXPLAIN SELECT * FROM customer customer INNER JOIN payment payment ON customer.customer_id = payment.customer_id;`

     ![mark](http://songwenjie.vip/blog/180728/7egBmA56C2.png?imageslim)

  5. **type=eq_ref，类似ref，区别在于使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配**

     **eq_ref**一般出现在多表连接时使用primary key或者unique index作为关联条件。

     **film、film_text**表关联查询和上一条所说的基本一致，只不过关联条件由非唯一索引变成了主键。

     `EXPLAIN SELECT * FROM film film INNER JOIN film_text film_text ON film.film_id = film_text.film_id;`

     ![mark](http://songwenjie.vip/blog/180728/4H03LBl4Lj.png?imageslim)

  6. **type=const/system，单表中最多有一条匹配行，查询起来非常迅速，所以这个匹配行的其他列的值可以被优化器在当前查询中当作常量来处理**

     **const/system**出现在根据主键primary key或者 唯一索引 unique index 进行的查询

     根据主键primary key进行的查询：

      `EXPLAIN SELECT * FROM customer WHERE customer_id =10;`

     ![mark](http://songwenjie.vip/blog/180728/6h95c9ag8G.png?imageslim)

     根据唯一索引unique index进行的查询：

     `EXPLAIN SELECT * FROM customer WHERE email ='MARY.SMITH@sakilacustomer.org';`

     ![mark](http://songwenjie.vip/blog/180728/J4klb0cd97.png?imageslim)

     ​

     ![mark](http://songwenjie.vip/blog/180728/h7I5f0JJJa.png?imageslim)

  7. **type=NULL，MySQL不用访问表或者索引，直接就能够得到结果**

     ![mark](http://songwenjie.vip/blog/180729/1F9gm2h950.png?imageslim)

* **possible_keys**: 表示查询可能使用的索引

* **key**: 实际使用的索引

* **key_len**: 使用索引字段的长度

* **ref**: 使用哪个列或常数与key一起从表中选择行。

* **rows**: 扫描行的数量

* **filtered**: 存储引擎返回的数据在server层过滤后，剩下多少满足查询的记录数量的比例(百分比)

* **Extra**: 执行情况的说明和描述，包含不适合在其他列中显示但是对执行计划非常重要的额外信息

  最主要的有一下三种：

  |        Using Index        |                表示索引覆盖，不会回表查询                 |
  | :-----------------------: | :-------------------------------------------------------: |
  |      **Using Where**      |                  **表示进行了回表查询**                   |
  | **Using Index Condition** |                   **表示进行了ICP优化**                   |
  |     **Using Flesort**     | **表示MySQL需额外排序操作, 不能通过索引顺序达到排序效果** |




## 什么是ICP？

MySQL5.6引入了**Index Condition Pushdown（ICP）**的特性，进一步优化了查询。Pushdown表示操作下放，某些情况下的条件过滤操作下放到存储引擎。

`EXPLAIN SELECT * FROM rental WHERE rental_date='2005-05-25' AND customer_id>=300 AND customer_id<=400;`

在5.6版本之前：

优化器首先使用复合索引idx_rental_date过滤出符合条件`rental_date='2005-05-25'`的记录，然后根据复合索引idx_rental_date回表获取记录，最终根据条件`customer_id>=300 AND customer_id<=400`过滤出最后的查询结果（在服务层完成）。

在5.6版本之后：

MySQL使用了ICP来进一步优化查询，在检索的时候，把条件`customer_id>=300 AND customer_id<=400`也推到存储引擎层完成过滤，这样能够降低不必要的IO访问。Extra为`Using index condition`就表示使用了ICP优化。

![mark](http://songwenjie.vip/blog/180729/FDGC3F3EJ1.png?imageslim)



## 参考

《深入浅出MySQL》