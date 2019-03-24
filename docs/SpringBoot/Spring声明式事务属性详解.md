# Spring 声明式事务属性详解

在上文中，我们介绍了如何在 Spring Boot 中使用声明式事务进行事务管理。现在我们来看一下与声明式事务注解相关属性的配置。

<a name="d35b05cf"></a>
## @Transactional
@Transactional 这个注解是一些（和事务相关的）元数据，在运行时被事务基础设施读取消费，并使用这些元数据来配置bean的事务行为。大致来说具有两方面功能，一是表明该方法要参与事务，二是配置相关属性来定制事务的参与方式和运行行为。<br /><br />
<a name="transactionalManager"></a>
### transactionalManager
**事务的管理方式，**使用默认的 dataSourceManager 即可

<a name="propagation"></a>
### propagation
**事务传播特性，有 7 种方式。**默认值是PROPAGATION_REQUIRED,当前有事务就用当前的，没有就用新的。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1552902265650-b7edb3a9-a730-4f16-a1fc-f39c12e9d3af.png#align=left&display=inline&height=502&name=image.png&originHeight=552&originWidth=827&size=153866&status=done&width=752)<br />
<a name="d41d8cd9"></a>
### 
<a name="Isolation"></a>
### Isolation
**事务隔离性，对应数据库的隔离级别。**默认值是 DEFAULT(-1) ，意味着使用数据库本身设置的全局隔离级别。如果我们不适用默认值，而是在程序中设置下面的隔离性，便会在此次事务操作中使用会话级别的相应的隔离级别。<br />![](https://cdn.nlark.com/yuque/0/2019/png/291118/1552876583067-3012ad47-3023-4dae-8dda-3105162ffa10.png#align=left&display=inline&height=336&originHeight=620&originWidth=1375&status=done&width=746)
<a name="d41d8cd9-1"></a>
### 
<a name="timeout"></a>
### timeout
**超时时间，单位为秒，默认值是 -1 ，表示不限制超时时间。**设置 timeout 后，当事务执行时间超过设置的超时时间，便会进行事务回滚，关闭事务。

我们继续使用上一篇文章中的示例，测试一下 timeout 属性设置带来的影响。<br />我们先设置超时时间 timeout = 1，然后让事务操作休眠 3 秒。
```java
@Transactional(timeout = 1 )
public void timeout() throws Exception {
    jdbcTemplate.execute("select sleep(3)");
    jdbcTemplate.update("insert into test (name) values ('foo')");
}
```

单元测试
```java
@Test
    public void timeout()  {
        log.info("before timeout: {}",jdbcTemplate.queryForObject("select count(*) from test",Long.class));
        try {
            testService.timeout();
        }catch (Exception e){
            log.info("after timeout: {}",jdbcTemplate.queryForObject("select count(*) from test",Long.class));
        }
    }
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1552897136574-06bdf697-4ff8-4a74-8651-1acf0b66349b.png#align=left&display=inline&height=45&name=image.png&originHeight=49&originWidth=1351&size=16289&status=done&width=1228)

从结果中可以看到，触发了超时异常，事务进行了回滚操作。

再来看一下，设置超时时间，但是不触发限制的情况。事务操作依然休眠 3 秒，修改超时时间 timeout = 5。

```java
@Transactional(timeout = 5)
public void timeout() throws Exception {
    jdbcTemplate.execute("select sleep(3)");
    jdbcTemplate.update("insert into test (name) values ('foo')");
}
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1552897593382-bb1b78a7-2a05-4972-8e70-850b64ac5819.png#align=left&display=inline&height=29&name=image.png&originHeight=32&originWidth=1330&size=8828&status=done&width=1209)

没有触发超时异常，事务正产提交。

<a name="readOnly"></a>
### readOnly
**只读属性，默认值是 false。**设置 readOnly = true 表示开启只读事务，事务中只能执行读操作，不能执行写操作。

设置只读属性 readOnly = true，并且在事务中执行写操作语句。
```java
@Transactional(readOnly = true)
public void readOnly() throws Exception {
    jdbcTemplate.update("insert into test (name) values ('foo')");
}
```

单元测试
```java
@Test
public void readOnly(){
    try {
        testService.readOnly();
    }catch (Exception e){
        log.info("Exception: {}",e.toString());
    }
}
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1552899484789-110ef97e-e9e6-4c3c-8056-51a419a6c52a.png#align=left&display=inline&height=90&name=image.png&originHeight=99&originWidth=1408&size=32473&status=done&width=1280)

从打印的异常信息中可以看到，触发了只读异常，不允许进行数据写操作。

<a name="f04059f0"></a>
### 如何判断回滚
rollbackFor 属性可以指定需要进行事务回滚的异常类型；noRollbackFor 属性可以指定即使发生此类异常，也不要进行事务回滚的异常类型。默认的回滚规则是只把runtime, unchecked exceptions标记为回滚，即RuntimeException及其子类，Error默认也导致回滚。Checked exceptions默认不导致回滚。

我们在事务执行中抛出 Exception 异常类型，但是指定rollbackFor属性为我们自定义的 RollbackException 异常类型。
```java
@Override
@Transactional(rollbackFor = {RollbackException.class})
public void rollbackForException() throws Exception {
    jdbcTemplate.update("insert into test (name) values ('foo')");
    throw new Exception();
}
```
单元测试
```java
@Test
public void rollbackForException(){
    log.info("before rollbackForException: {}",jdbcTemplate.queryForObject("select count(*) from test",Long.class));
    try{
        testService.rollbackForException();
    }catch (Exception e){
        log.info("after rollbackForException: {}",jdbcTemplate.queryForObject("select count(*) from test",Long.class));
    }
}
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1552900835684-597b746e-e673-4fdc-974a-87d6c241dc6a.png#align=left&display=inline&height=85&name=image.png&originHeight=94&originWidth=1434&size=21705&status=done&width=1304)

从结果中可以看到，虽然事务执行过程中抛出了异常，但是并没有进行事务回滚操作。这是因为抛出的异常类型和rollbackFor 属性指定的要进行事务回滚的异常类型不匹配。解决的方式有两种：

第一种是修改 rollbackFor 属性的异常类型与抛出的异常类型相同
```java
@Override
@Transactional(rollbackFor = {Exception.class})
public void rollbackForException() throws Exception {
    jdbcTemplate.update("insert into test (name) values ('foo')");
    throw new Exception();
}
```

![](https://cdn.nlark.com/yuque/0/2019/png/291118/1552901313335-71d0fc3a-c6ef-4cec-a358-89392809998a.png#align=left&display=inline&height=40&originHeight=77&originWidth=1438&status=done&width=746)

第二种是可以指定多个 rollbackFor 异常类型，添加一个 Exception 类型的实例
```java
@Transactional(rollbackFor = {RollbackException.class,Exception.class})
public void rollbackForException() throws Exception {
    jdbcTemplate.update("insert into test (name) values ('foo')");
    throw new Exception();
}
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1552901576135-20c23e7a-cc18-4be0-8521-9dee786c6222.png#align=left&display=inline&height=85&name=image.png&originHeight=93&originWidth=1426&size=21247&status=done&width=1296)<br />

<a name="d17a0f0b"></a>
## 参考
geektime - 玩转 Spring 全家桶<br />[Spring Boot 揭秘与实战系列](http://blog.720ui.com/columns/springboot_all/)

<a name="81cb1f5d"></a>
## 源代码
**示例源代码** [Spring-family ](https://github.com/SongWenJie/Spring-family.git)
