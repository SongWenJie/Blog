# Spring Boot 数据存储 - JPA集成

<a name="ORM"></a>
## ORM
在开始讲解 Spring Data JPA 之前，我们先来了解一下 ORM(Object Relational Mapping)，ORM 的出现是为了解决面向对象编程和关系型数据库之间的 O/R阻抗失衡 (O/R Impedance Mismatch)，通过 ORM 就能使用面向对象编程，来操作关系型数据库。<br /><br />![](https://cdn.nlark.com/yuque/0/2019/png/291118/1553050134265-33fa6ee1-be99-4c98-a9b2-f92e97a719a9.png#align=left&display=inline&height=359&originHeight=359&originWidth=700&size=0&status=done&width=700)<br />![](https://cdn.nlark.com/yuque/0/2019/png/291118/1553046753681-f17543b9-2029-405f-addf-710ae0bbccf1.png#align=left&display=inline&height=407&originHeight=407&originWidth=523&status=done&width=523)

**数据库与对象之间的映射：**<br />**<br />![](https://cdn.nlark.com/yuque/0/2019/png/291118/1553050334703-0848b3b1-c0ac-40c8-8767-c6a8522864a3.png#align=left&display=inline&height=257&originHeight=257&originWidth=429&size=0&status=done&width=429)

<a name="JPA"></a>
## JPA
JPA 全称 Java Persistence API，为对象关系映射提供了一种基于 POJO 的持久化模型，屏蔽了不同持久化 API 的差异。
<a name="c63abe91"></a>
### JPA 和 ORM 的关系
JPA 是 ORM 的一种实现标准。
<a name="a9ec0ac5"></a>
### JPA 和 Hibernate 的关系
JPA 是一个持久化存储标准。Hibernate 是 JPA 的一个标准实现。
<a name="7bd97fde"></a>
### Hibernate 
Hibernate 是一款开源的对象关系映射(ORM)框架，屏蔽掉了底层数据库的各种细节，开发者可以使用操作对象的方式操作数据库，从常见的数据库持久化工作中解放出来。

<a name="18208c8a"></a>
## 依赖环境
在 Spring Boot 中使用 JPA，需要引入 spring-boot-starter-data-jpa 依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

使用 MySQL 数据库，引入 mysql-connector-java 依赖
```
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

引入 lombok 依赖
```
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

<a name="b98e47fb"></a>
## 数据库脚本初始化

```sql
CREATE DATABASE /*!32312 IF NOT EXISTS*/`springboot_db` /*!40100 DEFAULT CHARACTER SET utf8 */;
 
USE `springboot_db`;
 
DROP TABLE IF EXISTS `t_book`;
 
CREATE TABLE `t_book` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '书籍ID',
  `book_name` varchar(32) NOT NULL COMMENT '书籍名称',
  `author` varchar(32) NOT NULL COMMENT '作者',
  `price` DECIMAL NOT NULL COMMENT '书籍价格',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

<a name="321b4184"></a>
## 数据源配置
在/src/main/resources/application.properties 中配置数据源信息

```
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/springboot_db?serverTimezone=GMT
spring.datasource.username=root
spring.datasource.password=123456
```

<a name="bd6c2e59"></a>
## 定义实体
@Entity 注解表明 Book 类是一个实体类，@Table 注解表明与数据库中的哪张表映射。<br />@Id 注解表明字段是数据表主键，@GeneratedValue 注解表示主键的生成策略，JPA提供的四种标准用法为AUTO，IDENTITY,SEQUENCE,TABLE，默认值是AUTO。<br />AUTO：自动选择一个最适合底层数据库的主键生成策略，MySQL 数据库不支持；<br />IDENTITY：主键由数据库自动生成（主要是自动增长型），Oracle 数据库不支持；<br />SEQUENCE：根据底层数据库的序列来生成主键，条件是数据库支持序列，MySQL 数据库不支持；<br />TABLE：使用一个特定的数据库表格来保存主键。

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "t_book")
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

    @Column(name = "book_name")
    private String bookName;

    private String author;

    private double price;
}
```

<a name="3542cbb2"></a>
## 定义 Repository 接口
在 Spring Data JPA 中，只需要定义 Repository 接口即可，并不需要定义接口的实现。我们在定义 Repository 接口时，可以根据需要选择继承下面的三个接口之一，UML 类图如下图所示：<br />CrudRepository<T,ID> 接口定义了基本的增删改查操作；PagingAndSortingRepository<T, ID> 接口继承CrudRepository<T,ID> 接口，并且扩展了排序查询和分页查询；而JpaRepository<T,ID> 接口继承PagingAndSortingRepository<T, ID> 接口，扩展了批量操作功能。

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1553065431408-45111806-a432-4136-a425-280610bd9320.png#align=left&display=inline&height=292&name=image.png&originHeight=321&originWidth=360&size=11563&status=done&width=327)

我们这里只需要基本的 CRUD 操作，所以继承 CrudRepository<T,ID>接口即可。

```java
public interface BookRepository extends CrudRepository<Book,Long> {
}
```

<a name="69dbc1f0"></a>
## 开启 JPA 支持
使用 @EnableJpaRepositories 注解开启 JPA 支持
```java
@EnableJpaRepositories
@SpringBootApplication
public class JpaDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(JpaDemoApplication.class, args);
    }
}
```

<a name="93b824b5"></a>
## 单元测试
使用单元测试进行 CRUD 操作。

```java
@Slf4j
@RunWith(SpringRunner.class)
@SpringBootTest
public class BookRepositoryTest {

    @Autowired
    private BookRepository bookRepository;

    @Test
    public void save(){
        Book book = Book.builder()
                .bookName("Spring Boot in action")
                .author("Craig Walls")
                .price(59.00)
                .build();
        bookRepository.save(book);
        log.info("save: {}", book);
    }

    @Test
    public void deleteById(){
        bookRepository.deleteById(3L);
    }

    @Test
    public void update(){
        Book book = bookRepository.findById(1L).get();
        if(book != null){
            log.info("before update: {}", book);
            book.setPrice(69.00);
            bookRepository.save(book);
            log.info("after update: {}", book);
        }
    }

    @Test
    public void findAllById(){
        List<Long> ids = new ArrayList<>(1);
        ids.add(1L);

        Iterable<Book> books = bookRepository.findAllById(ids);

        books.forEach(book -> log.info("findAllById: {}", book));
    }

    @Test
    public void findAll(){
        Iterable<Book> books = bookRepository.findAll();
        books.forEach(book -> log.info("findAll: {}", book));
    }
}
```

<a name="d17a0f0b"></a>
## 参考
geektime - 玩转 Spring 全家桶<br />[Spring Boot 揭秘与实战系列](http://blog.720ui.com/columns/springboot_all/)<br />[ORM 实例教程](http://www.ruanyifeng.com/blog/2019/02/orm-tutorial.html)

<a name="81cb1f5d"></a>
## 源代码
**示例源代码** [Spring-family ](https://github.com/SongWenJie/Spring-family.git)
