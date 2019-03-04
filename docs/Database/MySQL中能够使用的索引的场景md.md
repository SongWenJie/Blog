### 匹配全值

**对索引中所有列都有指定具体值，即是对索引中的所有列都有等值匹配的条件**。

rental表的索引情况，其中idx_rental_date是复合索引：

![mark](http://songwenjie.vip/blog/180729/7a3GHJhAG8.png?imageslim)

` EXPLAIN SELECT * FROM rental WHERE rental_date='2005-05-25' AND inventory_id=373 AND customer_id=343;`

![mark](http://songwenjie.vip/blog/180729/cfG1j75Hkc.png?imageslim)

优化器选择了复合索引idx_rental_date



### 匹配值的范围查询

**对索引的值进行范围查找。**

` EXPLAIN SELECT * FROM rental WHERE customer_id>373 AND customer_id<400;`

![mark](http://songwenjie.vip/blog/180729/LI0HcJl9f2.png?imageslim)

优化器选择索引范围查询，使用索引idx_fk_customer_id加速访问。注意Extra列为`Using index condition`，表明MySQL使用了**ICP**进一步优化查询。关于**ICP**后面会单独介绍。



### 匹配最左前缀

**MySQL中B-Tree索引使用的首要原则——最左匹配原则**

对于复合索引而言，MySQL仅仅使用索引的最左列进行查找。比如在col1+col2+col3字段上的联合索引能够被包含col1、（col1+col2)、（col1+col2+col3）的等值查询利用到，但是不能够被col2、col3、（col2+col3）的等值查询利用到。

如果查询条件中包含了索引的第一列rental_date和第三列customer_id，从key中可以看出优化器使用了复合索引idx_rental_date进行条件过滤：

` EXPLAIN SELECT * FROM rental WHERE rental_date='2005-05-25' AND customer_id=343;`

![mark](http://songwenjie.vip/blog/180729/HbIlhCG9C1.png?imageslim)

如果查询条件中包含了索引的第二列inventory_id和第三列customer_id，从key中可以看出优化器使用了其各自的索引进行条件过滤，但是进行了索引合并优化（type=index_merge）：

`EXPLAIN SELECT * FROM rental WHERE  inventory_id=373 AND customer_id=343;`

![mark](http://songwenjie.vip/blog/180729/bAJch04gGK.png?imageslim)



### 仅仅对索引进行查询

**当查询的列都在索引的字段中时，查询的效率更高。**

`EXPLAIN SELECT customer_id FROM rental;`

![mark](http://songwenjie.vip/blog/180729/iaifmeLbB1.png?imageslim)

优化器选择了查询列的索引加速访问，并且Extra为`Using index`意味着直接访问索引就足够取到所需要的数据，不会再通过索引回表查询。



### 匹配列前缀

**使用只包含索引字段开头一部分进行数据查找**

为customer表建立前缀索引：

![mark](http://songwenjie.vip/blog/180729/ile7h05eJG.png?imageslim)

查看customer表的索引情况：

![mark](http://songwenjie.vip/blog/180729/igk78mI9dA.png?imageslim)

` EXPLAIN SELECT email  FROM customer WHERE email LIKE 'MARY%';`

![mark](http://songwenjie.vip/blog/180729/m5dAfFIh21.png?imageslim)

可以看到，前缀索引idx_email已经被利用上了，Extra为`Using where`表明优化器还需要通过索引回表查询数据，因为索引的只是email字段的前5个字符。



### 如果列名是索引，那么使用column_name is null 就会使用索引（区别于Oracle）

![mark](http://songwenjie.vip/blog/180729/69A8elfdf0.png?imageslim)

`EXPLAIN SELECT * FROM payment WHERE rental_id IS NULL;`

![mark](http://songwenjie.vip/blog/180729/dgAJBA6gH3.png?imageslim)



### ICP优化

MySQL5.6引入了Index Condition Pushdown（ICP）的特性，进一步优化了查询。Pushdown表示操作下放，某些情况下的条件过滤操作下放到存储引擎。

`EXPLAIN SELECT * FROM rental WHERE rental_date='2005-05-25' AND customer_id>=300 AND customer_id<=400;`

在5.6版本之前：

优化器首先使用复合索引idx_rental_date过滤出符合条件`rental_date='2005-05-25'`的记录，然后根据复合索引idx_rental_date回表获取记录，最终根据条件`customer_id>=300 AND customer_id<=400`过滤出最后的查询结果（在服务层完成）。

在5.6版本之后：

MySQL使用了ICP来进一步优化查询，在检索的时候，把条件`customer_id>=300 AND customer_id<=400`也推到存储引擎层完成过滤，这样能够降低不必要的IO访问。Extra为`Using index condition`就表示使用了ICP优化。

![mark](http://songwenjie.vip/blog/180729/FDGC3F3EJ1.png?imageslim)







