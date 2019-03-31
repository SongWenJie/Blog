# Spring - 使用 Jedis 访问Redis

<a name="30fd99f0"></a>
## Spring 对 Redis 的支持
Spring 中与 Redis 相关都集成在Spring Data Redis 模块下，Spring 支持以下几种方式访问操作 Redis。
* 支持客户端 Jedis/Luttuce
* RedisTemplate
* Repository 支持
* CacheManager 

这几种方式都是殊途同归，可以根据项目选择使用。
* JedisClient 是 Java 实现的 Redis 客户端，同时也可以作为 RedisTemplate 的指定连接客户端
* RedisTemplate 是 Spring 提供的封装了一些操作方法的操作模板类。（与 JdbcTemplate 类似）
* CacheManager 是 Spring 提供的一套通过注解方式访问操作缓存的统一编程模型，CacheManager 关注抽象层面，而不关注实现层面。具体的实现由各个缓存自己实现，与 Redis 缓存相关的具体实现是 RedisCacheManager，RedisCacheManager 内部实现使用的是 RedisTemplate。（与 Spring 声明式事务类似）

<a name="a38bc49f"></a>
## 使用 Jedis 访问Redis
我们先来介绍如何使用 Jedis 访问 Redis。

<a name="c830a77b"></a>
### Jedis 和 Luttuce 的比较
在 Redis 众多的客户端中，Redis 和 Luttuce 是公认比较优秀的两款。<br />**相同点**<br />Lettuce 和 Jedis 的定位都是 Redis 的 client，可以直接连接redis server。<br />**不同点**<br />Jedis 在实现上是直接连接的redis server，如果在多线程环境下是非线程安全的，这个时候只有使用连接池，为每个Jedis 实例增加物理连接。<br />Lettuce 的连接是基于 Netty 的，连接实例（StatefulRedisConnection）可以在多个线程间并发访问，应为StatefulRedisConnection 是线程安全的，所以一个连接实例（StatefulRedisConnection）就可以满足多线程环境下的并发访问，当然这个也是可伸缩的设计，一个连接实例不够的情况也可以按需增加连接实例。

<a name="641cf522"></a>
### 环境依赖
修改 pom 文件，引入 jedis 依赖
```
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
```

<a name="53f2a95d"></a>
### Redis 连接信息配置
在 application.properties 中添加redis的连接信息配置
```java
# Redis数据库索引（默认为0）
spring.redis.database=0
# Redis服务器地址
spring.redis.host=127.0.0.1
# Redis服务器连接端口
spring.redis.port=6379
# Redis服务器连接密码（默认为空）
spring.redis.password=
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.jedis.pool.max-active=8
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.jedis.pool.max-wait=-1
# 连接池中的最大空闲连接
spring.redis.jedis.pool.max-idle=8
# 连接池中的最小空闲连接
spring.redis.jedis.pool.min-idle=0
# 连接超时时间（毫秒）
spring.redis.timeout=0
```

<a name="f8634c10"></a>
### 自定义 JedisPool
因为 Jedis 不是线程安全的，也就意味着多个线程不能共用一个实例， 对于 Jedis 实例这种需要频繁创建销毁的资源，使用线程池控制实例的创建和销毁是一个好的选择。

```java
@Configuration
public class RedisConfig {
    @Bean
    @ConfigurationProperties("spring.redis")
    public JedisPoolConfig jedisPoolConfig() {
        return new JedisPoolConfig();
    }

    @Bean(destroyMethod = "close")
    public JedisPool jedisPool(@Value("${spring.redis.host}") String host) {
        return new JedisPool(jedisPoolConfig(), host);
    }
}
```

<a name="a649f5af"></a>
### 定义实体类
借助 lombok 简化开发
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class Book implements Serializable {
    /**
     * 书籍主键
     */
    private long id;
    /**
     * 书籍名称
     */
    private String bookName;
    /**
     * 作者
     */
    private String author;
    /**
     * 价格
     */
    private double price;
}
```

<a name="93b824b5"></a>
### 单元测试

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class RedisConfigTest {

    @Autowired
    private JedisPool jedisPool;

    private Book fooBook(){
        Book fooBook = Book.builder()
                .id(1)
                .bookName("foo book")
                .author("foo author")
                .price(58.00)
                .build();
        return fooBook;
    }

    private Book barBook(){
        Book barBook = Book.builder()
                .id(2)
                .bookName("bar book")
                .author("bar author")
                .price(39.00)
                .build();
        return barBook;
    }

    //string
    @Test
    public void string(){
        try(Jedis jedis = jedisPool.getResource()){
            jedis.set("string",fooBook().toString());
        }
    }

    //list
    @Test
    public void list(){
        try(Jedis jedis = jedisPool.getResource()){
            jedis.lpush("list",fooBook().toString());
            jedis.rpush("list",barBook().toString());
        }
    }

    //set
    @Test
    public void set(){
        try(Jedis jedis = jedisPool.getResource()){
            jedis.sadd("set",fooBook().toString());
            jedis.sadd("set",barBook().toString());
        }
    }

    //hash
    @Test
    public void hash(){
        try(Jedis jedis = jedisPool.getResource()){
            jedis.hset("hash","foo",fooBook().toString());
            jedis.hset("hash","bar",barBook().toString());
        }
    }

    //zSet
    @Test
    public void zSet(){
        try(Jedis jedis = jedisPool.getResource()){
            Book fooBook = fooBook();
            Book barBook = barBook();
            jedis.zadd("zSet",fooBook.getPrice(),fooBook.toString());
            jedis.zadd("zSet",barBook.getPrice(),barBook().toString());
        }
    }
}
```

运行单元测试，查询 Redis :<br />**string<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1553501834618-d05589ec-aa4b-4927-8ee4-945f4d75bdd6.png#align=left&display=inline&height=38&name=image.png&originHeight=42&originWidth=528&size=4310&status=done&width=480)**<br />**<br />**list<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1553501931918-243b5995-6e45-45e8-83fb-ac77b2b2b166.png#align=left&display=inline&height=51&name=image.png&originHeight=56&originWidth=542&size=7492&status=done&width=493)**<br />**<br />**set<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1553501955357-43588885-7c1d-4863-99f0-e70d6eaebba0.png#align=left&display=inline&height=52&name=image.png&originHeight=57&originWidth=535&size=7329&status=done&width=486)**<br />**<br />**hash<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1553501980876-3c529a29-d706-456e-8646-075177643c97.png#align=left&display=inline&height=54&name=image.png&originHeight=59&originWidth=556&size=8045&status=done&width=505)**<br />**<br />**zSet<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1553502871014-9fe16866-c9e2-47a3-9229-80fd480f325c.png#align=left&display=inline&height=83&name=image.png&originHeight=91&originWidth=545&size=9419&status=done&width=495)**

<a name="25f9c7fa"></a>
### 总结
在本文中，我们介绍了 Spring 对 Redis 的支持方式，并且重点介绍了如何使用 Jedis 客户端的方式访问操作 Redis。需要重点注意 Jedis 不是线程安全的，也就意味着多个线程不能共用一个实例，解决方式是使用线程池。下一篇文章我们来了解如何使用 RedisTemplate 访问 Redis。
