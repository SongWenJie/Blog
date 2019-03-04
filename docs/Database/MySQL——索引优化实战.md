# MySQL——索引优化实战

上篇文章中介绍了索引的基本内容，这篇文章我们继续介绍索引优化实战。在介绍索引优化实战之前，首先要介绍两个与索引相关的重要概念，这两个概念对于索引优化至关重要。

本篇文章用于测试的user表结构：

![mark](http://songwenjie.vip/blog/180801/aFGjcIDhDd.png?imageslim)



## 索引相关的重要概念

### 基数

单个列唯一键（distict_keys）的数量叫做基数。

`SELECT COUNT(DISTINCT name),COUNT(DISTINCT gender) FROM user;`

![mark](http://songwenjie.vip/blog/180801/8j3H8cKCc8.png?imageslim)

user表的总行数是5，gender 列的基数是 2，说明 gender 列里面有大量重复值，name 列的基数等于总行数，说明 name列没有重复值，相当于主键。

**返回数据的比例：**

user表中共有5条数据：

`SELECT * FROM user;`

![mark](http://songwenjie.vip/blog/180801/f0bL3ag3de.png?imageslim)

查询满足性别为0（男）的记录数：

![mark](http://songwenjie.vip/blog/180801/52biIe2jHC.png?imageslim)

那么返回记录的比例数是：

![mark](http://songwenjie.vip/blog/180801/LBD77bKeh4.png?imageslim)

同理，查询name为'swj'的记录数：

![mark](http://songwenjie.vip/blog/180801/830C5lceiK.png?imageslim)

返回记录的比例数是：

![mark](http://songwenjie.vip/blog/180801/jDBHjlb1F1.png?imageslim)

现在问题来了，假设name、gender列都有索引，那么`SELECT * FROM user WHERE gender = 0;` `SELECT * FROM user WHERE name = 'swj';`都能命中索引吗？

user表的索引详情：

![mark](http://songwenjie.vip/blog/180801/04m46bdJ99.png?imageslim)

`SELECT * FROM user WHERE gender = 0;`没有命中索引，注意filtered的值就是上面我们计算的返回记录的比例数。

![mark](http://songwenjie.vip/blog/180801/6LE6KlcDaK.png?imageslim)

`SELECT * FROM user WHERE name = 'swj';`命中了索引index_name，因为走索引直接就能找到要查询的记录，所以filtered的值为100

![mark](http://songwenjie.vip/blog/180801/G86HGaIJEA.png?imageslim)

**结论：**

返回表中 30% 内的数据会走索引，返回超过 30% 数据就使用全表扫描。当然这个结论太绝对了，也并不是绝对的30%，只是一个大概的范围。



### 回表

当对一个列创建索引之后，索引会包含该列的键值及键值对应行所在的 rowid。通过索引中记录的 rowid 访问表中的数据就叫回表。回表次数太多会严重影响 SQL 性能，如果回表次数太多，就不应该走索引扫描，应该直接走全表扫描。

EXPLAIN命令结果中的`Using Index`意味着不会回表，通过索引就可以获得主要的数据。`Using Where`则意味着需要回表取数据。



## 索引优化实战

有些时候虽然数据库有索引，但是并不被优化器选择使用。

我们可以通过`SHOW STATUS LIKE 'Handler_read%';`查看索引的使用情况：

![mark](http://songwenjie.vip/blog/180731/iCb54E2mDJ.png?imageslim)

**Handler_read_key**：如果索引正在工作，Handler_read_key的值将很高。

**Handler_read_rnd_next**：数据文件中读取下一行的请求数，如果正在进行大量的表扫描，值将较高，则说明索引利用不理想。



**索引优化规则**：

1. **如果MySQL估计使用索引比全表扫描还慢，则不会使用索引**

   返回数据的比例是重要的指标，比例越低越容易命中索引。记住这个范围值——30%，后面所讲的内容都是建立在返回数据的比例在30%以内的基础上。

2. **前导模糊查询查询不能命中索引**

   name列创建普通索引：

   ![mark](http://songwenjie.vip/blog/180801/CgdaaLjBdf.png?imageslim)

   前导模糊查询查询不能命中索引：

   ` EXPLAIN SELECT * FROM user WHERE name LIKE '%s%';`

   ![mark](http://songwenjie.vip/blog/180801/cK8Ga63ClG.png?imageslim)

   非前导模糊查询则可以使用索引，可优化为使用非前导模糊查询：

   `EXPLAIN SELECT * FROM user WHERE name LIKE 's%';`

   ![mark](http://songwenjie.vip/blog/180801/DElLC4He5g.png?imageslim)

2. **数据类型出现隐式转换的时候不会命中索引，特别是当列类型是字符串，一定要将字符常量值用引号引起来**

   `EXPLAIN SELECT * FROM user WHERE name=1;`

   ![mark](http://songwenjie.vip/blog/180801/G0cHbffDAe.png?imageslim)

   `EXPLAIN SELECT * FROM user WHERE name='1';`

   ![mark](http://songwenjie.vip/blog/180801/eajlBBFLGD.png?imageslim)

   ​

3. **复合索引的情况下，查询条件不包含索引列最左边部分（不满足最左原则），不会命中符合索引**

   name,age,status列创建复合索引：

   `ALTER TABLE user ADD INDEX index_name (name,age,status);`

   ![mark](http://songwenjie.vip/blog/180801/dA2IeE38ID.png?imageslim)

   user表索引详情：

   `SHOW INDEX FROM user;`

   ![mark](http://songwenjie.vip/blog/180801/H05Emk9mIF.png?imageslim)

   根据最左原则，可以命中复合索引index_name：

   `EXPLAIN  SELECT * FROM user WHERE name='swj' AND status=1;`

   ![mark](http://songwenjie.vip/blog/180801/mlJBJke9cj.png?imageslim)

   注意，最左原则并不是说是查询条件的顺序：

   `EXPLAIN SELECT * FROM user WHERE  status=1 AND name='swj';`

   ![mark](http://songwenjie.vip/blog/180801/F84m8aB6ak.png?imageslim)

   而是查询条件中是否包含索引最左列字段：

   `EXPLAIN SELECT * FROM user WHERE  status=2 ;`

   ![mark](http://songwenjie.vip/blog/180801/CFFdGfh663.png?imageslim)

4. **union、in、or 都能够命中索引，建议使用 in。**

   **union**:

   `EXPLAIN  SELECT * FROM user WHERE status = 1 `

   `UNION ALL `

   `SELECT * FROM user WHERE status = 2;`

   ![mark](http://songwenjie.vip/blog/180801/7H7Kj58gEa.png?imageslim)

   **in**:

   `EXPLAIN SELECT * FROM user WHERE status IN (1,2);`

   ![mark](http://songwenjie.vip/blog/180801/3JLjfIdDAG.png?imageslim)

   **or**:

   `EXPLAIN SELECT * FROM user WHERE status=1 OR status=2;`

   ![mark](http://songwenjie.vip/blog/180801/JedELeDciA.png?imageslim)

   **查询的CPU消耗：or > in >union**

5. **用or分割开的条件，如果or前的条件中列有索引，而后面的列中没有索引，那么涉及到的索引都不会被用到**

   ` EXPLAIN SELECT * FROM payment WHERE customer_id = 203 OR amount = 3.96;`

   ![mark](http://songwenjie.vip/blog/180731/d3e4AJJ66i.png?imageslim)

   因为or后面的条件列中没有索引，那么后面的查询肯定要走全表扫描，在存在全表扫描的情况下，就没有必要多一次索引扫描增加IO访问。

7. **负向条件查询不能使用索引，可以优化为 in 查询。**

   负向条件有：!=、<>、not in、not exists、not like 等。

   status列创建索引：

   `ALTER TABLE user ADD INDEX index_status (status);`

   ![mark](http://songwenjie.vip/blog/180801/mKEc4jL2E5.png?imageslim)

   user表索引详情：

   `SHOW INDEX FROM user;`

   ![mark](http://songwenjie.vip/blog/180801/2Jm10HB37a.png?imageslim)

   负向条件不能命中缓存：

   `EXPLAIN SELECT * FROM user WHERE status !=1 AND status != 2;`

   ![mark](http://songwenjie.vip/blog/180801/bKc3jJd23j.png?imageslim)

   可以优化为 in 查询，但是前提是区分度要高，返回数据的比例在30%以内：

   ` EXPLAIN SELECT * FROM user WHERE status IN (0,3,4);`

   ![mark](http://songwenjie.vip/blog/180801/e0hDkBA2c0.png?imageslim)

8. **范围条件查询可以命中索引**

   范围条件有：<、<=、>、>=、between等

   status,age列分别创建索引：

   `ALTER TABLE user ADD INDEX index_status (status);`

   ![mark](http://songwenjie.vip/blog/180801/mKEc4jL2E5.png?imageslim)

   `ALTER TABLE user ADD INDEX index_age (age);`

   ![mark](http://songwenjie.vip/blog/180801/18ffhKIhiG.png?imageslim)

   user表索引详情：

   `SHOW INDEX FROM user;`

   ![mark](http://songwenjie.vip/blog/180801/Di9EKJmBHL.png?imageslim)

   范围条件查询可以命中索引：

   `EXPLAIN SELECT * FROM user WHERE status>5;`

   ![mark](http://songwenjie.vip/blog/180801/eLF1h3mh6e.png?imageslim)

   范围列可以用到索引（联合索引必须是最左前缀），但是范围列后面的列无法用到索引，索引最多用于一个范围列，如果查询条件中有两个范围列则无法全用到索引：

   `EXPLAIN SELECT * FROM user WHERE status>5 AND age<24;`

   ![mark](http://songwenjie.vip/blog/180801/4h5hdElfJe.png?imageslim)

   如果是范围查询和等值查询同时存在，优先匹配等值查询列的索引：

   `EXPLAIN SELECT * FROM user WHERE status>5 AND age=24;`

   ![mark](http://songwenjie.vip/blog/180801/0eAf30hK4i.png?imageslim)

9. **数据库执行计算不会命中索引**

   `EXPLAIN SELECT * FROM user WHERE age > 24;`

   ![mark](http://songwenjie.vip/blog/180801/375FK1GeaI.png?imageslim)

   ` EXPLAIN SELECT * FROM user WHERE age+1 > 24;`

   ![mark](http://songwenjie.vip/blog/180801/fHGdle0jAG.png?imageslim)

   **计算逻辑应该尽量放到业务层处理，节省数据库的 CPU的同时最大限度的命中索引。**

10. **利用覆盖索引进行查询，避免回表**

   被查询的列，数据能从索引中取得，而不用通过行定位符 row-locator 再到 row 上获取，即“被查询列要被所建的索引覆盖”，这能够加速查询速度。

   user表的索引详情：

   ![mark](http://songwenjie.vip/blog/180801/hemkaiHgI7.png?imageslim)

   因为status字段是索引列，所以直接从索引中就可以获取值，不必回表查询：

   `Using Index`代表从索引中查询

   `EXPLAIN SELECT status FROM user where status=1;`

   ![mark](http://songwenjie.vip/blog/180801/5AJKihifj3.png?imageslim)

   当查询其他列时，就需要回表查询，这也是为什么要避免`SELECT *`的原因之一：

   `EXPLAIN SELECT * FROM user where status=1;`

   ![mark](http://songwenjie.vip/blog/180801/Kmig57FBHE.png?imageslim)

11. **建立索引的列，不允许为 null**

    单列索引不存 null 值，复合索引不存全为 null 的值，如果列允许为 null，可能会得到“不符合预期”的结果集，所以，请使用 not null 约束以及默认值。

    remark列建立索引：

    `ALTER TABLE user ADD INDEX index_remark (remark);`

    ![mark](http://songwenjie.vip/blog/180801/G77i9938id.png?imageslim)

    IS NULL可以命中索引：

    `EXPLAIN SELECT * FROM user WHERE remark IS NULL;`

    ![mark](http://songwenjie.vip/blog/180801/BDa7kH1E6c.png?imageslim)

    IS NOT NULL不能命中索引：

    `EXPLAIN SELECT * FROM user WHERE remark IS NOT NULL;`

    ![mark](http://songwenjie.vip/blog/180801/11LKdf4cEk.png?imageslim)

    **虽然IS NULL可以命中索引，但是NULL本身就不是一种好的数据库设计，应该使用NOT NULL 约束以及默认值**

12. **更新十分频繁的字段上不宜建立索引**

    因为更新操作会变更B+树，重建索引。这个过程是十分消耗数据库性能的。

13. **区分度不大的字段上不宜建立索引**

    类似于性别这种区分度不大的字段，建立索引的意义不大。因为不能有效过滤数据，性能和全表扫描相当。另外返回数据的比例在30%以外的情况下，优化器不会选择使用索引。

14. **业务上具有唯一特性的字段，即使是多个字段的组合，也必须建成唯一索引**

    虽然唯一索引会影响insert速度，但是对于查询的速度提升是非常明显的。另外，即使在应用层做了非常完善的校验控制，只要没有唯一索引，在并发的情况下，依然有脏数据产生。

15. **多表关联时，要保证关联字段上一定有索引**

16. **创建索引时避免以下错误观念**

    - 索引越多越好，认为一个查询就需要建一个索引。

    - 宁缺勿滥，认为索引会消耗空间、严重拖慢更新和新增速度。

    - 抵制唯一索引，认为业务的唯一性一律需要在应用层通过“先查后插”方式解决。

    - 过早优化，在不了解系统的情况下就开始优化。

 

## 总结

对于自己编写的SQL查询语句，要尽量使用EXPLAIN命令分析一下，做一个对SQL性能有追求的程序员。衡量一个程序员是否靠谱，SQL能力是一个重要的指标。作为后端程序员，深以为然。



## 参考

* 《深入浅出MySQL》