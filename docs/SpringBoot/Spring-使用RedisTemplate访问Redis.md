# Spring - 使用 RedisTemplate 访问Redis

与 JdbcTemplate 类似，RedisTemplate 是 Spring 提供的封装了一些操作方法的操作模板类，其内部实现依靠的是客户端 Jedis/Luttuce。简单点说就是，RedisTemplate 不具备连接 Redis 的能力，只是为了便于使用，对 Jedis /Luttuce 等客户端做了一层封装。

我们来看一下如何在 Spring Boot 中使用 RedisTemplate 访问 Redis 并进行数据操作。

<a name="393d9096"></a>
## 使用 RedisTemplate 访问Redis
<a name="641cf522"></a>
### 环境依赖
引入 redis 依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
引入 jackson 序列化依赖
```
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.module</groupId>
    <artifactId>jackson-module-jaxb-annotations</artifactId>
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

<a name="bad6a22c"></a>
### 自定义 RedisTemplate
Spring 中默认的 RedisTemplate 值类型为 Object 类型，如果需要的话，需要自己定义。我们这里使用 Book 类型，自定义一个 RedisTemplate<String,Book> 类型 。
```java
@Configuration
public class RedisConfig {
    @Bean
    public RedisTemplate<String,Book> redisTemplateofBook (RedisConnectionFactory redisConnectionFactory){
        RedisTemplate<String,Book> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        return redisTemplate;
    }
}
```

<a name="93b824b5"></a>
### 单元测试
这里需要强调一下，在使用 RedisTemplate 时，一定要设置缓存过期时间！！！否则缓存会常驻内存，导致堆栈溢出。
```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class RedisConfigTest {

    @Autowired
    private RedisTemplate<String,Book> redisTemplateofBook;

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

    /**
    * @Description: stringOperation
    * @Param: []
    * @return: void
    * @Author: songwenjie
    * @Date: 2019/3/25
    */
    @Test
    public void stringOperation(){
        ValueOperations<String,Book> valueOperations = redisTemplateofBook.opsForValue();
        valueOperations.set("string_book",fooBook());
        //设置一分钟过期时间
        redisTemplateofBook.expire("string_book", 1, TimeUnit.MINUTES);
    }

    /**
     * @Description: listOperation
     * @Param: []
     * @return: void
     * @Author: songwenjie
     * @Date: 2019/3/25
     */
    @Test
    public void listOperation(){
        ListOperations<String,Book> listOperations = redisTemplateofBook.opsForList();
        listOperations.leftPush("list_book", fooBook());
        listOperations.rightPush("list_book", barBook());
        redisTemplateofBook.expire("list_book", 1, TimeUnit.MINUTES);
    }

    /**
     * @Description: setOperation
     * @Param: []
     * @return: void
     * @Author: songwenjie
     * @Date: 2019/3/25
     */
    @Test
    public void setOperation(){
        SetOperations<String,Book> setOperations = redisTemplateofBook.opsForSet();
        setOperations.add("set_book",fooBook());
        setOperations.add("set_book",barBook());
        redisTemplateofBook.expire("set_book", 1, TimeUnit.MINUTES);
    }

    /**
     * @Description: hashOperation
     * @Param: []
     * @return: void
     * @Author: songwenjie
     * @Date: 2019/3/25
     */
    @Test
    public void hashOperation(){
        HashOperations<String,String,Book> hashOperations = redisTemplateofBook.opsForHash();
        hashOperations.put("hash_book", "foo", fooBook());
        hashOperations.put("hash_book", "bar", barBook());
        redisTemplateofBook.expire("hash_book", 1, TimeUnit.MINUTES);
    }

    /**
    * @Description: zSetOperation
    * @Param: []
    * @return: void
    * @Author: songwenjie
    * @Date: 2019/3/25
    */
    @Test
    public void zSetOperation(){
        ZSetOperations<String,Book> zSetOperations = redisTemplateofBook.opsForZSet();
        Book fooBook = fooBook();
        Book barBook = barBook();
        zSetOperations.add("zSet_book",fooBook,fooBook.getPrice());
        zSetOperations.add("zSet_book",barBook,barBook.getPrice());
        redisTemplateofBook.expire("zSet_book", 1, TimeUnit.MINUTES);
    }
}
```

当运行单元测试后，查询 Redis 会发现一个问题，redis 存储的是 "乱码"，这是由于RedisTemplate 默认的编码格式(JdkSerializationRedisSerializer) 导致的。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1553507704261-c43cfa98-7e13-414c-beb6-a012e3b3bbe6.png#align=left&display=inline&height=90&name=image.png&originHeight=99&originWidth=367&size=5888&status=done&width=334)<br />我们可以在自定义 RedisTemplate 时指定编码格式解决这个问题。一般来说，key 使用 StringRedisSerializer 编码，value 使用Jackson2JsonRedisSerializer 序列化编码。

```java
@Configuration
public class RedisConfig {
    @Bean
    public RedisTemplate<String,Book> redisTemplateofBook (RedisConnectionFactory redisConnectionFactory){
        RedisTemplate<String,Book> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);

        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        redisTemplate.setKeySerializer(redisSerializer);
        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
        redisTemplate.setHashKeySerializer(redisSerializer);
        redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);

        return redisTemplate;
    }
}
```

乱码的问题得到解决<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1553508123321-4782052d-d13b-4abb-8d53-b3a6e3738938.png#align=left&display=inline&height=101&name=image.png&originHeight=111&originWidth=209&size=4507&status=done&width=190)

具体的结果：<br />**string**<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1553508515847-0af3794b-f396-4338-bef4-32ec38ce4aae.png#align=left&display=inline&height=35&name=image.png&originHeight=39&originWidth=559&size=4548&status=done&width=508)

**list<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1553508549853-e24af51b-188b-415d-ac62-66e083d8b63e.png#align=left&display=inline&height=49&name=image.png&originHeight=54&originWidth=579&size=7608&status=done&width=526)**

**set<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1553508574496-9eaa4780-b708-448c-a33c-b75b1f849d87.png#align=left&display=inline&height=51&name=image.png&originHeight=56&originWidth=573&size=7410&status=done&width=521)**

**hash<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1553508603384-68d1d535-6540-4469-9f35-07499165e5e8.png#align=left&display=inline&height=49&name=image.png&originHeight=54&originWidth=577&size=8146&status=done&width=525)**<br />**<br />**zSet<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1553508625291-ce65d079-088f-4cab-b791-df89b56b652d.png#align=left&display=inline&height=85&name=image.png&originHeight=93&originWidth=580&size=9576&status=done&width=527)**<br />**
<a name="d31932be"></a>
## 指定 Redis 客户端
Spring Boot 在 2.0 版本前默认使用的是 Jedis 客户端，为了获得更高的性能，Spring Boot 在 2.0 版本后默认使用 Luttuce 客户端。这一点可以通过打印 RedisTemplate 的连接信息证明。

```java
@Slf4j
@SpringBootTest
@RunWith(SpringRunner.class)
public class RedisConfigTest {
    @Autowired
    private RedisTemplate<String,Book> redisTemplateofBook;

    @Test
    public void printRedisConnection(){
        RedisConnectionFactory redisConnectionFactory =  redisTemplateofBook.getConnectionFactory();
        log.info(redisConnectionFactory.toString());
    }
}
```

打印的信息说明是使用 LettuceConnectionFactory 创建的 lettuce 连接实例。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1553508899406-4ebd521b-72c4-462a-9779-09a190b5812d.png#align=left&display=inline&height=45&name=image.png&originHeight=50&originWidth=1396&size=14911&status=done&width=1269)

若是我们想要使用 Jedis 客户端，可以引入 jedis 的maven依赖，并且在配置类中自定义一个 JedisConnectionFactory 的 Bean。Spring Boot 启动时便不会加载默认的 RedisConnectionFactory（LettuceConnectionFactory），而是使用我们自定义的 JedisConnectionFactory 。

```java
//自定义JedisConnectionFactory
@Bean
public RedisConnectionFactory redisConnectionFactory(@Value("${spring.redis.host}") String host){
    RedisStandaloneConfiguration redisStandaloneConfiguration =
            new RedisStandaloneConfiguration(host);
    return new JedisConnectionFactory(redisStandaloneConfiguration);
}
```

此时打印连接信息：<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1553510835741-cf97deaa-d686-4a47-a41b-c2df335e8b62.png#align=left&display=inline&height=38&name=image.png&originHeight=42&originWidth=1391&size=14490&status=done&width=1265)

<a name="25f9c7fa"></a>
## 总结
本文介绍了如何在 Spring Boot 中使用 RedisTemplate 访问 Redis 。需要重点注意的是 RedisTemplate的编码问题、过期时间问题还有如何手动指定 Redis 客户端的问题。 
