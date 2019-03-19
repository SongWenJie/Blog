# 使用JdbcTemplate操作数据库

在上一篇文章中我们介绍了在 Spirng Boot 中如何配置数据源，数据源配置完成后，就可以使用 JDBC 操作数据库了。JDBC 提供了一个操作模板类 JdbcTemplate 用以操作数据库，其中包含下面几个方法：<br />query、 queryForObject、queryForList 用于从数据库中查询数据，update 和 execute 方法都可以用于数据写入，区别是 update 方法一般用于增删改操作 ，因为 update 方法会返回语句操作受影响的行数，便于在程序中判断此次操作成功与否；execute 方法没有返回值，主要用于数据库 ddl 操作。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1552712552327-e0577b57-757a-42d0-81a6-476f267b2d6d.png#align=left&display=inline&height=234&name=image.png&originHeight=257&originWidth=343&size=14880&status=done&width=312)

## 环境依赖
修改 pom 文件，引入 JDBC 、MySQL 相关依赖，同时 Demo 中使用到了 lombok ,还需要引入 lombok 依赖，有关 lombok 可以看这篇文章。 

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

## 配置数据源
使用Spring Boot 自动配置，在/src/main/resources/application.properties 中配置数据源信息

```
spring.datasource.driverClassName=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/springboot_db?serverTimezone=GMT
spring.datasource.username=root
spring.datasource.password=123456
```

## 脚本初始化
使用下面的脚本初始化测试数据库和数据表。

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

## JdbcTemplate 操作数据库
### 建立实体类
建立与 t_book 表对应的实体类，@Data 注解用于生成 Getter 和 Setter 方法，@Builder 注解用于生成构造函数。

```java
@Data
@Builder
public class Book {
    private long id;
    private String bookName;
    private String author;
    private double price;
}
```

### 创建Dao类
创建的 BookDao 类，引入JdbcTemplate 对象用于操作数据库的，@Repository 注解用于说明此类为数据操作类。

```java
@Repository
public class BookDao {
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    public int insertBook(Book book){
        int result = jdbcTemplate.update(
                "insert into t_book(book_name, author,price) values(?,?,?)",
                book.getBookName(),
                book.getAuthor(),
                book.getPrice()
        );
        return result;
    }


    public int deleteBook(long id){
        int result = jdbcTemplate.update("delete from t_book where id = ?",id);
        return result;
    }

    public int updateBook(Book book){
        int result = jdbcTemplate.update(
                "update t_book set book_name=?,author=?,price=? where id=?",
                book.getBookName(),
                book.getAuthor(),
                book.getPrice(),
                book.getId());
        return result;
    }

    public Book getBook(long id){
        Book book = jdbcTemplate.queryForObject("select id,book_name,author,price from t_book where id=?",
                new Object[]{id},
                new RowMapper<Book>(){
                    @Override
                    public Book mapRow(ResultSet rs, int rowNum) throws SQLException {
                        return Book.builder()
                                .id(rs.getLong(1))
                                .bookName(rs.getString(2))
                                .author(rs.getString(3))
                                .price(rs.getDouble(4))
                                .build();
                    }
        });
        return book;
    }


    public List<Book> getBooks(){
        List<Book> books = jdbcTemplate.query("select id,book_name,author,price from t_book",new RowMapper<Book>(){
            @Override
            public Book mapRow(ResultSet rs, int rowNum) throws SQLException {
                return Book.builder()
                        .id(rs.getLong(1))
                        .bookName(rs.getString(2))
                        .author(rs.getString(3))
                        .price(rs.getDouble(4))
                        .build();
            }
        });
        return books;
    }

    public List<Long> getBookids(){
        List<Long> bookids = jdbcTemplate.queryForList("select id from t_book",
                Long.class
        );
        return bookids;
    }
}
```

执行方法进行测试

## 参考
geektime - 玩转 Spring 全家桶<br />[Spring Boot 揭秘与实战系列](http://blog.720ui.com/columns/springboot_all/)

## 源代码
**示例源代码** [Spring-family ](https://github.com/SongWenJie/Spring-family.git)
