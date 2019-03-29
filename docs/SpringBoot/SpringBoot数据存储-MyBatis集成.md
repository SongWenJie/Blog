# Spring Boot 数据存储 - MyBatis 集成

在上文中，我们讲解了如何在 Spring Boot 中集成 JPA，本文我们继续学习如何在 Spring Boot 中集成另外一款优秀的持久层框架——MyBatis，MyBatis 的特点是支持定制化 SQL、存储过程和高级映射。我们可以根据项目的复杂程度选择不同的持久层框架，如果项目数据库映射关系比较简单，我们可以选择 JPA / Hibernate 来替我们屏蔽掉 SQL 层的操作，简化开发；如果映射关系比较复杂，可以选择 MyBatis 定制化 SQL、存储过程和高级映射，这样也可以对我们所写的 SQL 有更强的把控程度。 

<a name="641cf522"></a>
## 环境依赖
在 Spring Boot 中使用 MyBatis，需要引入 mybatis-spring-boot-starter 依赖
```
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.0.0</version>
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

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class Book {
    private long id;
    private String bookName;
    private String author;
    private double price;
}
```

<a name="68634e6b"></a>
## 定义映射
MyBatis 中有两种定义映射的方式——XML与注解。
<a name="24ae87ae"></a>
### 注解方式
使用注解方式定义映射，需要在接口使用 @Mapper 注解。<br />在方法定义上分别使用 @Insert()、@Delete()、@Update()、@Select() 注解进行增删改查操作，使用 @Results 注解进行查询结果的映射。

```java
@Mapper
public interface BookMapper {

    @Insert("insert into t_book (book_name,author,price) " +
            "values (#{bookName},#{author},#{price})")
    Long save(Book book);


    @Select("select * from t_book where id = #{id}")
    @Results({
            @Result(column = "book_name",property = "bookName")
    })
    Book findById(long id);
}
```

使用注解方式，需要在 Spring Boot 启动类添加 @MapperScan 注解，用于配置 mapper 的扫描位置。mapper 接口位于 data.mybatis_demo.mapper 包下，所以配置扫描位置为 data.mybatis_demo.mapper。

```
@MapperScan(value = "data.mybatis_demo.mapper")
```

**单元测试**<br />使用单元测试进行测试，并且打印相关信息

```java
@Slf4j
@RunWith(SpringRunner.class)
@SpringBootTest
public class BookMapperTest {

    @Autowired
    private BookMapper bookMapper;

    @Test
    public void save(){
        Book book = Book.builder()
                .bookName("Spring Boot in action")
                .author("Craig Walls")
                .price(59.00)
                .build();
        bookMapper.save(book);
        log.info("save book: {}",book);
    }

    @Test
    public void findById(){
        Book book = bookMapper.findById(1);
        log.info("findById book: {}",book);
    }
}
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1553234679087-f6ec0944-1f15-4975-afea-41fed54cadc1.png#align=left&display=inline&height=84&name=image.png&originHeight=92&originWidth=1419&size=32319&status=done&width=1290)

<a name="89aa7c32"></a>
### XML 方式
使用 XML 配置方式，首先要确定 Mapper.xml 文件放置的路径，我们统一放置在 /src/main/resources/mapper 路径下。然后在 /src/main/resources/application.properties 文件中配置 mybatis.mapper-locations 属性，用于指定Mapper.xml 文件放置的路径。

```
mybatis.mapper-locations: classpath:/mapper/*.xml
```

定义 BookMapperByXml接口，使用 @Mapper 注解表明这是一个 mapper 接口。

```java
@Mapper
public interface BookMapperByXml {
    Long save(Book book);

    Book findById(long id);
}
```

在 /src/main/resources/mapper/BookMapper.xml 文件配置 SQL及映射信息, mapper 节点映射到BookMapperByXml 接口

```xml
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="data.mybatis_demo.mapper.BookMapperByXml">

    <resultMap id="bookMap" type="data.mybatis_demo.model.Book">
        <id property="id" column="id"/>
        <result property="bookName" column="book_name"/>
        <result property="author" column="author"/>
        <result property="price" column="price"/>
    </resultMap>

    <select id="findById" resultMap="bookMap">
      select * from t_book where id = #{id}
    </select>

    <insert id="save">
        insert into t_book (book_name,author,price) values (#{bookName},#{author},#{price})
    </insert>
</mapper>
```
 <br />**单元测试**<br />使用单元测试进行测试，并且打印相关信息

```java
@Slf4j
@RunWith(SpringRunner.class)
@SpringBootTest
public class BookMapperByXmlTest {
    @Autowired
    private BookMapperByXml bookMapper;

    @Test
    public void save(){
        Book book = Book.builder()
                .bookName("Spring Boot in action")
                .author("Craig Walls")
                .price(59.00)
                .build();
        bookMapper.save(book);
        log.info("save book: {}",book);
    }

    @Test
    public void findById(){
        Book book = bookMapper.findById(1);
        log.info("findById book: {}",book);
    }
}
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1553237892453-ca5d0318-4aa5-40a9-961a-5d6b6819eeca.png#align=left&display=inline&height=81&name=image.png&originHeight=89&originWidth=1416&size=31860&status=done&width=1287)

<a name="d17a0f0b"></a>
## 参考
geektime - 玩转 Spring 全家桶<br />[Spring Boot 揭秘与实战系列](http://blog.720ui.com/columns/springboot_all/)

<a name="81cb1f5d"></a>
## 源代码
**示例源代码** [Spring-family ](https://github.com/SongWenJie/Spring-family.git)
