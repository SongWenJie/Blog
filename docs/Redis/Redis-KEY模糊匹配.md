# Redis key 模糊匹配

关系型数据库可以使用 LIKE 关键字进行模糊查询，对于 Redis 这样的 NoSQL 数据库要实现模糊查询应该怎么办呢？关于 Redis key 模糊匹配的场景，可以使用以下两种方式：

<a name="d9cdedee"></a>
## KEYS pattern
KEYS pattern 用于查找所有符合给定模式pattern（正则表达式）的 key 。时间复杂度为O(N)，N为数据库里面 key的数量。
<a name="f80daa26"></a>
### 优点：
速度非常快。例如，Redis在一个有1百万个 key 的数据库里面执行一次查询需要的时间是40毫秒 。
<a name="757bf1f8"></a>
### 缺点：
一次性返回所有匹配的key，所以，当Redis 中的 key 非常多时，可能会堵塞 Redis 服务器。
<a name="fa28890d"></a>
### 支持的正则表达模式：
创建 5 条测试数据<br />**![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1554711827153-2ff22656-5275-436e-8727-24931679b976.png#align=left&display=inline&height=99&name=image.png&originHeight=109&originWidth=243&size=3828&status=done&width=221)<br />**

1. **匹配一位 ?**

  ![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1554712181974-35f1179c-37e0-48c5-8f71-6f65140f2322.png#align=left&display=inline&height=62&name=image.png&originHeight=68&originWidth=257&size=3135&status=done&width=234)
1. **匹配多位 ***

  ![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1554712325323-d9d436a7-7574-42fe-9335-1069aa1c0df0.png#align=left&display=inline&height=114&name=image.png&originHeight=125&originWidth=240&size=4437&status=done&width=218)
1. **或值匹配 [ab]**

  ![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1554712586379-75482c0b-26d2-41ea-beb3-ee6e975123e1.png#align=left&display=inline&height=47&name=image.png&originHeight=52&originWidth=216&size=2672&status=done&width=196)
1. **非值匹配 [^a]**

  ![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1554712640087-31239437-aff1-4cf8-98a7-f82f36182f96.png#align=left&display=inline&height=49&name=image.png&originHeight=54&originWidth=234&size=2703&status=done&width=213)
1. **范围匹配 [a-i]**

  ![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1554712714095-c354db6c-4f9e-4e51-8626-511eafbca425.png#align=left&display=inline&height=63&name=image.png&originHeight=69&originWidth=227&size=3160&status=done&width=206)

<a name="00b2c17a"></a>
## SCAN cursor [MATCH pattern] [COUNT count]
<a name="f80daa26-1"></a>
### 优点：
SCAN 支持增量式迭代，它们每次执行都只会返回少量元素，所以这些命令可以用于生产环境，而不会出现像         KEYS 命令带来的可能会阻塞服务器的问题。
<a name="757bf1f8-1"></a>
### 缺点：
有可能在增量迭代过程中，集合元素被修改，对返回值无法提供完全准确的保证。可以使用 SET 结构去除重复。
* 从完整遍历开始直到完整遍历结束期间， 一直存在于数据集内的所有元素都会被完整遍历返回； 这意味着， 如果有一个元素， 它从遍历开始直到遍历结束期间都存在于被遍历的数据集当中， 那么 SCAN 命令总会在某次迭代中将这个元素返回给用户。
* 同样，如果一个元素在开始遍历之前被移出集合，并且在遍历开始直到遍历结束期间都没有再加入，那么在遍历返回的元素集中就不会出现该元素。
* 同一个元素可能会被返回多次。 处理重复元素的工作交由应用程序负责， 比如说， 可以考虑将迭代返回的元素仅仅用于可以安全地重复执行多次的操作上。(可以将返回的结果置于 SET 集合去重，Spring Data Redis 就是这样实现的)
* 如果一个元素是在迭代过程中被添加到数据集的， 又或者是在迭代过程中从数据集中被删除的， 那么这个元素可能会被返回， 也可能不会。



<a name="d448d98a"></a>
### SCAN 命令使用：
SCAN命令的返回值是一个包含两个元素的数组， 第一个数组元素是用于进行下一次迭代的新游标， 而第二个数组元素则是一个数组， 这个数组中包含了所有被迭代的元素。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1554713866616-a9c9e767-ee04-44b7-a952-a490aaec2104.png#align=left&display=inline&height=82&name=image.png&originHeight=90&originWidth=276&size=3841&status=done&width=251)
<a name="cursor"></a>
#### cursor
第一次迭代使用 0 作为游标， 表示开始一次新的迭代。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1554714194982-f35d8d92-f9b7-4aaf-9239-1a5ec687df9c.png#align=left&display=inline&height=197&name=image.png&originHeight=217&originWidth=227&size=6654&status=done&width=206)<br />第二次迭代使用的是第一次迭代时返回的游标 15，作为新的迭代参数 。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1554714247371-d15bd5a0-feac-4ec2-a8e5-d00546a69fc3.png#align=left&display=inline&height=47&name=image.png&originHeight=52&originWidth=229&size=2381&status=done&width=208)<br />调用 SCAN 命令时， 命令返回了游标 0 ， 这表示迭代已经结束， 整个数据集已经被完整遍历过了。<br /><br />
<a name="5dbbb505"></a>
#### COUNT 选项
COUNT 选项的作用就是让用户告知迭代命令， 在每次迭代中应该从数据集里返回多少元素。<br />COUNT 选项的默认值是 10，也可以指定大小。COUNT 要根据扫描数据量大小而定，Scan虽然无锁，但是也不能保证在超过百万数据量级别搜索效率；COUNT 不能太小，网络交互会变多，COUNT 要尽可能的大。在搜索结果集1万以内，建议直接设置为与所搜集大小相同或略大。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1554772720426-265a10fb-4d7c-403d-b2c2-450e3f189959.png#align=left&display=inline&height=81&name=image.png&originHeight=89&originWidth=242&size=3623&status=done&width=220) ![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1554772739849-b1d9cc7d-c1b1-4921-91f3-311055085f08.png#align=left&display=inline&height=112&name=image.png&originHeight=123&originWidth=252&size=4371&status=done&width=229)

<a name="3c534007"></a>
#### MATCH 选项
类似于KEYS 命令，SCAN 命令通过给定 MATCH 参数的方式实现了通过提供一个 glob 风格的模式参数， 让命令只返回和给定模式相匹配的元素。<br />SCAN 命令支持的正则表达模式与 KEYS 命令相同，SCAN 命令的 MATCH 选项对元素的模式匹配工作是在命令从数据集中取出元素后和向客户端返回元素前的这段时间内进行的， 所以如果被迭代的数据集中只有少量元素和模式相匹配， 那么迭代命令或许会在多次执行中都不返回任何元素（先取出，后匹配）。所以同时指定 MATCH 选项和 COUNT 选项时，数据集返回的元素个数并不一定和 COUNT 一致。<br />如下所示：<br />Redis 中一共存在 11 个 key<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1554774076319-47dd1340-e4de-4331-a6ee-c690e7b7f1ae.png#align=left&display=inline&height=196&name=image.png&originHeight=216&originWidth=197&size=6451&status=done&width=179)<br />我们指定MATCH 选项为 h?llo ，同时指定COUNT 选项为10。此时 SCAN 命令先取出前 10 个 key，然后匹配并且返回符合条件的 7 个，数据集返回的元素个数并不和 COUNT 一致。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1554774161611-5d5ce366-c29b-4e02-8346-0aab08053787.png#align=left&display=inline&height=148&name=image.png&originHeight=163&originWidth=313&size=6303&status=done&width=285)
