# Spring 声明式事务的本质

话接上文，我们在前文中介绍了 Spring 声明式事务的使用及属性详解，在 Spring 中使用声明式事务进行事务管理s虽说简单又方便，但是却有一个小小的"坑"。假设有这么一个场景，有个接口，它包含两个方法 a 和 b，然后有一个类实现了该接口。在该实现类里 a方法上使用事务注解、b方法上不使用事务注解，然后 b方法里调用a方法，此时调用b方法，会有事务吗？答案是不会。<br /><br />
<a name="7dccc266"></a>
## Spring 事务失效
我们继续使用上文中的实例代码
<a name="4a3276c3"></a>
### Service 相关
```java
public interface TestService {
    void a() throws RollbackException;
    void b() throws RollbackException;
}
```

```java
@Service
public class TestServiceImpl implements TestService{ 
    @Autowired
    private JdbcTemplate jdbcTemplate;
  
    @Override
    @Transactional(rollbackFor = RollbackException.class)
    public void a() throws RollbackException {
        jdbcTemplate.update("insert into test (name) values ('foo')");
        throw new RollbackException();
    }

    @Override
    public void b() throws RollbackException {
        a();
    }
}
```

<a name="93b824b5"></a>
### 单元测试
在调用方法 b 之前和之后分别打印表中的记录数。

```java
@Slf4j
@SpringBootTest
@RunWith(SpringRunner.class)
public class TestServiceImplTest {
	@Test
  public void b(){
      log.info("before invoke method b: {}",jdbcTemplate.queryForObject("select count(*) from test",Long.class));
      try{
          testService.b();
      }catch (RollbackException e){
          log.info("after invoke method b: {}",jdbcTemplate.queryForObject("select count(*) from test",Long.class));
      }
  }
}
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1552979214566-8f99d509-1f36-4a6a-8627-67acec7cc771.png#align=left&display=inline&height=46&name=image.png&originHeight=51&originWidth=1429&size=17164&status=done&width=1299)

从结果中可以验证我们的结论，事务并没有生效。

<a name="711c8694"></a>
## 声明式事务原理解析
**Spring 的声明式事务本质上是通过 AOP 增强了类的功能，而 Spring 的 AOP 本质上就是为类做了一个代理，我们看似调用的是自己写的类，实际上调用的是增强后的代理类。所以事务是由代理加进去的，所以关键就是代理如何生成。在 Spring 中有两种生成代理的方式，JDK动态代理和CGLIB。JDK 动态代理只能用于实现了接口的类，CGLIB则是否实现接口都可以**。<br /><br /><br />对于没有实现接口的类，CGLIB 只能通过继承重写的方式生成代理类。所以对于方法的访问范围是有要求的。private 方法不能被继承，final 方法不能被重写，static 方法和继承不相干，所以使用它们修饰的方法事务失效。public 方法，protected 方法可以被重写以添加事务代码，对于 package 方法来说，如果生成的子类位于同一个包里，就可以被重写以添加事务代码。所以它们本质上都可以支持事务，但是 Spring 选择让 protected 方法和package 方法不支持事务，所以只有public方法支持事务。<br /><br /><br />所以，**只要是以代理方式实现的声明式事务，无论是JDK动态代理，还是CGLIB直接写字节码生成代理，都只有public 方法上的事务注解才起作用。而且必须在代理类外部调用才行，如果直接在目标类里面调用，事务照样不起作用**。<br /><br />
<a name="10e8aeb9"></a>
## Spring 事务失效的解决方法
了解了 Spring 事务的本质后，Spring 事务失效的问题就很好解决了。通过访问增强后的代理类的方法，而不是直接访问自身的方法。具体可以采用下面的方式：
<a name="6e56ee98"></a>
### 注入自己的实例
注入自己的实例，这种方式比较容易理解。

```java
@Service
public class TestServiceImpl implements TestService{
    @Autowired
    private TestService testService; 
  
    @Override
    public void b() throws RollbackException {
        testService.a();
    }
}
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1552985176542-abf9dcf0-7ad0-4f01-96f9-2509945ae1d2.png#align=left&display=inline&height=45&name=image.png&originHeight=49&originWidth=1412&size=17543&status=done&width=1284)

<a name="6607b84e"></a>
### 获取当前代理
获取当前代理，这样写避免了自己调用自己的实例。使用这种方式必须引入 spring-boot-starter-aop 依赖，并且在启动类设置 exposeProxy = true。

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

```java
@EnableAspectJAutoProxy(exposeProxy = true)
@EnableTransactionManagement
@SpringBootApplication
public class DeclarativeTransactionDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DeclarativeTransactionDemoApplication.class, args);
    }
}
```

```java
@Service
public class TestServiceImpl implements TestService{
    @Autowired
    private TestService testService; 
  
    @Override
    public void b() throws RollbackException {
        ((TestService) (AopContext.currentProxy())).a();
    }
}
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1552985549987-6529de9a-f469-4e87-bcec-f4aaea3f719b.png#align=left&display=inline&height=46&name=image.png&originHeight=51&originWidth=1418&size=17439&status=done&width=1289)
<a name="d41d8cd9"></a>
### 
<a name="62076986"></a>
### 再加一层事务
在方法 b 上添加 与方法 a 相同的 @Transactional 注解开启事务

```java
@Service
public class TestServiceImpl implements TestService{
    @Autowired
    private TestService testService; 
  
    @Override
    @Transactional(rollbackFor = RollbackException.class)
    public void b() throws RollbackException {
        a();
    }
}
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1552985889930-42ba7f15-f2d0-420a-9622-2405a14ff12b.png#align=left&display=inline&height=46&name=image.png&originHeight=51&originWidth=1409&size=17221&status=done&width=1281)

<a name="d17a0f0b"></a>
## 参考
geektime - 玩转 Spring 全家桶<br />[Spring Boot 揭秘与实战系列](http://blog.720ui.com/columns/springboot_all/)<br />[我是如何在面试别人Spring事务时“套路”对方的](https://mp.weixin.qq.com/s?__biz=MzU1NzY1Nzc1OQ==&mid=2247484017&idx=1&sn=a2055640b142fc4cfa5a9901d22ec57f&chksm=fc333981cb44b09782c196da25dc613b38a9ede52de073bfaaaef6214a9deb294b6527091892&scene=21#wechat_redirect)

<a name="81cb1f5d"></a>
## 源代码
**示例源代码** [Spring-family ](https://github.com/SongWenJie/Spring-family.git)
