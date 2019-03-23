# Spring 声明式事务

在 Spring 中，提供了很多抽象，可以帮助我们在不同的框架中以相同的方式来进行数据操作。其中最重要的一块就是事务抽象。Spring 提供了一致的事务模型，不管是使用 JDBC、Hibernate 还是 myBatis ,都可以使用一样的方式进行事务管理。

在 Spring Boot 中进行事务管理有两种方式，第一种是编程式事务，第二种是声明式事务。因为声明式事务基于注解，使用方便，现在开发中多采用这种方式。本文介绍如何在 Spring Boot 中使用声明式事务进行事务管理。

<a name="641cf522"></a>
## 环境依赖
引入 JDBC 依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

引入 MySQL 依赖
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

<a name="321b4184"></a>
## 数据源配置
在 **src/main/resources/application.properties** 中配置数据源信息。
```
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/foo?serverTimezone=GMT
spring.datasource.username=root
spring.datasource.password=123456
```
<a name="d41d8cd9"></a>
## 
<a name="1e2d1932"></a>
## 声明式事务
<a name="97250d7a"></a>
### 开启事务注解
在 Spring Boot 中使用 @EnableTransactionManagement 开启事务注解使用

```java
@EnableTransactionManagement
@SpringBootApplication
public class DeclarativeTransactionDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DeclarativeTransactionDemoApplication.class, args);
    }
}
```

<a name="9c533e32"></a>
### 自定义异常类
自定义一个异常类 RollbackException

```java
public class RollbackException extends Exception {
}
```

<a name="4a3276c3"></a>
### Service 相关
在类或者方法上使用 @Transactional 注解，表明使用声明式事务进行事务管理。这里使用的两个方法，insertRecord 不抛出异常，不会进行事务回滚；insertRecordThenRollback 抛出 RollbackException 异常，会进行事务回滚。

```java
public interface TestService {
    void insertRecord();
    void insertRecordThenRollback() throws RollbackException;
}
```

```java
@Service
public class TestServiceImpl implements TestService{

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Override
    @Transactional
    public void insertRecord() {
        jdbcTemplate.update("insert into test (name) values ('foo')");
    }

    @Override
    @Transactional(rollbackFor = RollbackException.class)
    public void insertRecordThenRollback() throws RollbackException {
        jdbcTemplate.update("insert into test (name) values ('foo')");
        throw new RollbackException();
    }
}
```
<a name="d41d8cd9-1"></a>
## 
<a name="93b824b5"></a>
### 单元测试
使用单元测试测试上述的两个方法，分别打印方法执行前后数据库 test表中的记录数。

```java
@Slf4j
@SpringBootTest
@RunWith(SpringRunner.class)
public class TestServiceImplTest {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Autowired
    private TestService testService;

    @Test
    public void insertRecord() {
        log.info("before insertRecord: {}",jdbcTemplate.queryForObject("select count(*) from test",Long.class));
        testService.insertRecord();
        log.info("after insertRecord: {}",jdbcTemplate.queryForObject("select count(*) from test",Long.class));
    }

    @Test
    public void insertRecordThenRollback() {
        log.info("before insertRecordThenRollback: {}",jdbcTemplate.queryForObject("select count(*) from test",Long.class));
        try{
            testService.insertRecordThenRollback();
        }catch (RollbackException e){
            log.info("after insertRecordThenRollback: {}",jdbcTemplate.queryForObject("select count(*) from test",Long.class));
        }
    }
}
```

测试结果：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1552891099395-5dce468b-b3e5-4ae0-8449-549ef30c101b.png#align=left&display=inline&height=82&name=image.png&originHeight=90&originWidth=1231&size=23340&status=done&width=1119)<br />
![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1552891125768-70fac48b-319d-46aa-a15d-02cb1dcad8f3.png#align=left&display=inline&height=44&name=image.png&originHeight=48&originWidth=1391&size=16874&status=done&width=1265)

<a name="d17a0f0b"></a>
## 参考
geektime - 玩转 Spring 全家桶<br />[Spring Boot 揭秘与实战系列](http://blog.720ui.com/columns/springboot_all/)

<a name="81cb1f5d"></a>
## 源代码
**示例源代码** [Spring-family ](https://github.com/SongWenJie/Spring-family.git)
