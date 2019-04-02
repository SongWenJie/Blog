# Spring 缓存抽象

与声明式事务类似，Spring 也提供了一套基于注解的缓存操作统一编程模型。Spring 定义 org.springframework.cache.CacheManager 和 org.springframework.cache.Cache 接口用来统一不同的缓存技术。支持concurrentMap、Ehcache、JCache 等多种缓存实现，当然也支持 Redis。与 Redis 缓存相关的具体实现是 RedisCacheManager，RedisCacheManager 内部实现使用的是 RedisTemplate。
<a name="a0c89998"></a>
## Spring 缓存注解
![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1553561911297-d5b0506e-55d3-4759-a9d8-fe5a356865cd.png#align=left&display=inline&height=359&name=image.png&originHeight=396&originWidth=358&size=25680&status=done&width=325)
* **@EnableCaching** 开启 Spring 缓存注解
  * **@Cacheable**  先查看缓存中是否有数据，如果有数据，则直接返回缓存数据；若没有数据，调用方法并将方法返回值放进缓存
  * **@CachePut  **更新缓存，不管有没有缓存都会执行方法并将结果缓存起来
  * **@CacheEvict  **删除缓存
  * **@Caching **根据条件分别执行**@Cacheable/@CachePut/@CacheEvict **
  * **@CacheConfig **一个类中可能会有多个对同一个缓存的操作，这个时候可以使用**@CacheConfig**

<a name="641cf522"></a>
### 环境依赖
示例演示了缓存使用的常见流程：发起数据查询请求，先查询缓存，若缓存中存在，则直接返回数据；若缓存中不存在，则查询数据库并将数据放入缓存，返回数据。这一流程需要引入下列依赖：

```
//cache
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
//redis
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
//jpa
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
//mysql
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
//lombok
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

<a name="224e2ccd"></a>
### 配置
在 application.properties 中添加 数据源配置
```
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/springboot_db?serverTimezone=GMT
spring.datasource.username=root
spring.datasource.password=123456
```

在 application.properties 中添加 cache 配置和 redis 的连接配置
```
spring.cache.type=redis
#spring.cache.cache-names=book
spring.cache.redis.time-to-live=5000
spring.cache.redis.cache-null-values=false

spring.redis.host=localhost
```

<a name="Model"></a>
### Model

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "t_book")
public class Book implements Serializable {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

    @Column(name = "book_name")
    private String bookName;

    private String author;

    private double price;
}
```

<a name="Repository"></a>
### Repository

```java
public interface BookRepository extends CrudRepository<Book,Long> {
}
```

<a name="Service"></a>
### Service

```java
@Slf4j
@Service
public class BookService {
    @Autowired
    private BookRepository bookRepository;

    @Cacheable(value = "book")
    public Book getBook(long id){
        log.info("Reading from database");
        return bookRepository.findById(id).get();
    }

    @CachePut(value = "book")
    public Book update(long id){
        Book book = bookRepository.findById(id).get();
        book.setPrice(99.00);
        bookRepository.save(book);
        return book;
    }

    @CacheEvict(value = "book")
    public void delete(long id){
    }
}
```

因为都是在一个类里对同一个缓存的操作，也可以使用 **@CacheConfig **统一配置

```java
@CacheConfig(cacheNames = "book")
@Slf4j
@Service
public class BookService {
    @Autowired
    private BookRepository bookRepository;

    @Cacheable
    public Book getBook(long id){
        log.info("Reading from database");
        return bookRepository.findById(id).get();
    }

    @CachePut
    public Book update(long id){
        Book book = bookRepository.findById(id).get();
        book.setPrice(99.00);
        bookRepository.save(book);
        return book;
    }

    @CacheEvict
    public void delete(long id){
    }
}
```

<a name="0434c455"></a>
### 开启Cache
在启动类或者配置类上添加 @EnableCaching 注解开启 Spring 缓存

```java
@EnableCaching
@EnableJpaRepositories
@SpringBootApplication
public class CacheRedisDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(CacheRedisDemoApplication.class, args);
    }
}
```

<a name="93b824b5"></a>
### 单元测试

```java
@Slf4j
@RunWith(SpringRunner.class)
@SpringBootTest
public class BookServiceTest {
    @Autowired
    private BookService bookService;

    @Test
    public void run(){
        //读取缓存
        for (int i = 0; i < 5; i++) {
            log.info("getBook: {}",bookService.getBook(1));
        }

        //更新缓存
        log.info("updateBook: {}",bookService.update(1));
        log.info("Reading after refresh.");
        log.info("getBook: {}",bookService.getBook(1));

        //删除缓存
        log.info("deletBook");
        bookService.delete(1);
        log.info("Reading after delete.");
        log.info("getBook: {}",bookService.getBook(1));
    }
}
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1553567057725-fe0950e6-2e80-4941-98cc-572bebac115d.png#align=left&display=inline&height=423&name=image.png&originHeight=465&originWidth=1372&size=162627&status=done&width=1247)

我们在单元测试中，先读取 5 次数据；然后更新缓存，读取数据；最后删除缓存，读取数据。分析打印的结果，前 5 次读取数据，只有第 1 次是从数据库读取的，其余都是从缓存中读取的。这一点验证了 **@Cacheable**  先查看缓存中是否有数据，如果有数据，则直接返回缓存数据；若没有数据，调用方法并将方法返回值放进缓存。更新缓存后再读取数据，直接从缓存中读取出了更新后的数据。验证了 **@CachePut  **更新缓存，不管有没有缓存都会执行方法并将结果缓存起来。删除缓存后再读取数据，第一次读取是读取的数据库中的数据，验证了 **@CacheEvict  **删除缓存，也间接再次验证了**@Cacheable。**
