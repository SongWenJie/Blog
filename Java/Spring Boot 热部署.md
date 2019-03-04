我们知道 Java 是一门编译型语言，在本地开发时，如果我们修改了代码，需要重启一下服务器，并且重新运行代码。热部署可以解决掉这个问题，有了热部署之后，只需要重新 Build 一下就可以看到修改代码后的效果，不需要重启服务器。

在 Spring Boot 中，我们可以通过 spring-boot-devtools 实现热部署。

maven pom.xml 文件中添加如下依赖：

```java
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-devtools</artifactId>
     <optional>true</optional>
</dependency>
```

并且修改 <build><plugins> 下的节点

```java
<build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <fork>true</fork>
                </configuration>
            </plugin>
        </plugins>
</build>
```

如果不想手动 Build 项目，可以在 IDEA 中配置自动 Build

File->Build->Compiler,选中 Build peoject automatically

![mark](http://songwenjie.vip/blog/20190228/U51CE7fnuGUO.png?imageslim)

然后组合键 Ctrl + Shift  + Alt + / ,选中 Registry ,使 compiler.automake.allow.when.app.running  选中

![mark](http://songwenjie.vip/blog/20190228/jfyO1zIixkhs.png?imageslim)

重启 IDEA ,就可以使用热部署功能了。需要注意的是，这种方式只适用于 Spring Boot 项目。