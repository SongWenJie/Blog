# Redis 简介

<a name="cc27f951"></a>
## Redis 是什么
Redis是一款开源的、支持多种数据结构的内存 KV 存储，它可以存储键（key）和 5 种不同类型的值（value）之间的映射。可以将存储在内存中的键值对数据持久到硬盘，可以使用复制特性扩展读性能，还可以使用客户端分片来扩展写。

<a name="6383c531"></a>
## Redis 可以做什么
主要使用Redis做分布式缓存，Session共享，缓存读多写少的数据（如网站配置项）以提升读的速度，使用消<br />息队列做项目间的数据通信（解耦）。

<a name="12972472"></a>
## Redis数据结构简介
Redis 可以存储键（key）和5种不同类型的值（value）之间的映射。这5种数据结构类型分别为STRING（字符串）、LIST（列表）、SET（集合）、HASH（散列）和ZSET（有序集合）。
<a name="0dc089af"></a>
### STRING（字符串）
Redis的字符串和其他编程语言或其他键值存储提供的字符串非常相似。除了能 GET （获取值）、 SET （设置值）<br />和 DEL （删除值）字符串值之外，Redis还提供了还提供了一些可以对字符串的其中一部分内容进行读取和写入的<br />命令，以及一些能对字符串存储的数值执行自增或者自减操作的命令。

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1553495760478-2c2b5b53-93b1-44ac-a981-f99b1494632d.png#align=left&display=inline&height=323&name=image.png&originHeight=355&originWidth=471&size=21486&status=done&width=428)

<a name="a63033e8"></a>
### LIST（列表）
一个列表结构可以有序地存储多个字符串。Redis列表可执行的操作和很多编程语言里面的列表操作非常相似。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1553495835744-c36796b8-ceeb-4e7e-84c4-97a2117a0599.png#align=left&display=inline&height=380&name=image.png&originHeight=418&originWidth=502&size=37091&status=done&width=456)

<a name="e2d8ecb7"></a>
### SET（集合）
Redis的集合和列表都可以存储多个字符串，它们之间的不同在于，列表可以存储多个相同的字符串，而集合则通<br />过使用散列表来保证自己存储的每个字符串都是各不相同的。<br />因为SET（集合）使用无序方式存储元素，所以不能像使用列表那样将推入集合的某一端，或者从集合的某一端弹<br />出元素。但是可以使用 SADD 命令将元素添加到集合。或者使用 SREM 命令从集合中移除元素。还可以通过 SISMEMBER 命令快速检查一个元素是否已经存在于集合中。使用 SMEMBERS 命令返回集合包含的所有元素。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1553495954703-f316c213-b1fb-41f0-b7af-d59ef06fa2f5.png#align=left&display=inline&height=398&name=image.png&originHeight=437&originWidth=532&size=39630&status=done&width=484)

<a name="d8659265"></a>
### HASH（散列）
Redis的散列可以存储多个键值对之间的映射。和字符串一样，散列存储的值既可以是字符串又可以使数字值，并<br />且用户同样可以对散列存储的数字值执行自增或自减操作。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1553496046567-6b711169-3351-402a-8e02-31e74c15aa62.png#align=left&display=inline&height=414&name=image.png&originHeight=455&originWidth=499&size=45253&status=done&width=454)

<a name="ec5f3c14"></a>
### ZSET（有序集合）
有序集合和散列一样，都用于存储键值对：有序集合的键称为成员，每个成员都是各不相同的；而有序集合的值则<br />被称为分值，分值必须为浮点数。有序集合是Redis中唯一一个既可以根据成员访问元素（和散列一样），又可以<br />根据分值以及分值的排列顺序来访问元素的结构。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1553496107546-67bbd810-d7d4-44d4-af3c-1d96ac3444ae.png#align=left&display=inline&height=407&name=image.png&originHeight=447&originWidth=467&size=46279&status=done&width=425)

