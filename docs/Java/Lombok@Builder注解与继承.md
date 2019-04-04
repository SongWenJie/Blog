# Lombok @Builder注解与继承

Builder 模式的链式调用写起来很方便，自己实现构造模式要在 POJO 类中写较多代码，Lombok 的 @Builder注解可以方便的支持 Builder 模式。但是在开发实际项目时，使用 Lombok @Builder注解在继承场景下通过 Builder 模式创建对象，会出现 Lombok @Builder注解不会为继承的字段生成代码的问题。

简化一下场景：
```java
@NoArgsConstructor
@AllArgsConstructor
public class Parent {
    private String b;
}
```

```java
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Child extends Parent {
    private String b;
}
```

在通过构造模式创建 Child 对象时，不能链式调用 a() 方法。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1554360413556-53c2d1e1-89be-4ef0-b4f6-da3731e69426.png#align=left&display=inline&height=99&name=image.png&originHeight=109&originWidth=507&size=15672&status=done&width=461)

即使给父类Parent也添加@Builder注解，依然无法调用。
```java
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Parent {
    private String b;
}
```
![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1554360692546-23322e9e-eeb6-46cb-8726-4ffe34c8804f.png#align=left&display=inline&height=58&name=image.png&originHeight=64&originWidth=622&size=8522&status=done&width=565)

<a name="1e819ee4"></a>
## 自己实现 Builder 模式
使用 Lombok @Builder注解 可以很方便的使用构造模式， 我们也可以自己实现 Builder 模式，这将有助于我们理解 Builder 模式在继承场景下问题的本质。

自己实现 Builder 模式主要有四个步骤：
1. 在POJO 类中创建 builder 方法，创建并返回 Builder 对象
1. 需要创建一个内部静态 Builder 类，并且在 Builder 类中创建和POJO 类中相同的字段
1. 为每个字段提供 setter 方法，并且返回对象本身，实现链式调用
1. 在静态 Builder 类创建 build 方法，创建并返回 POJO 对象 

```java
public class Parent {
    private String b;

    public static Parent.ParentBuilder builder() {
        return new Parent.ParentBuilder();
    }

    public Parent() {
    }

    public Parent(String b) {
        this.b = b;
    }

    public static class ParentBuilder {
        private String b;

        ParentBuilder() {
        }

        public Parent.ParentBuilder b(String b) {
            this.b = b;
            return this;
        }

        public Parent build() {
            return new Parent(this.b);
        }
    }
}
```

```java
public class Child extends Parent {
    private String b;

    public static Child.ChildBuilder builder() {
        return new Child.ChildBuilder();
    }

    public Child() {
    }

    public Child(String b) {
        this.b = b;
    }

    public static class ChildBuilder {
        private String b;

        ChildBuilder() {
        }

        public Child.ChildBuilder b(String b) {
            this.b = b;
            return this;
        }

        public Child build() {
            return new Child(this.b);
        }
    }
}
```

其实上面的代码就是 Lombok @Builder注解在背后为我们做的事情，也解释了为什么 Builder 模式在继承场景下会出现问题。**类是继承的，但类中的 builder 类并无继承关系**。<br /><br />
<a name="2b1267de"></a>
## Builder 模式下的继承关系
那么这个问题就无法解决了吗？如果没有办法解决，Builder 模式的威力将大打折扣。幸运的是，我从网上寻找到了一种解决方式。<br />**原文：**

> <a name="54534343"></a>
# Lombok’s @Builder annotation and inheritance
> I’ve written about [Project Lombok’s](http://projectlombok.org/) _@Builder_ annotation before (see [here](https://www.donneo.de/2015/05/12/project-lomboks-builder-annotation/) and [here](https://www.donneo.de/2015/07/14/project-lomboks-builder-annotation-and-generics/)). We’ve started using it in our project some time ago in favour of the code generation library [PojoBuilder](https://github.com/mkarneim/pojobuilder). One thing has bugged me though during that time: Lombok’s _@Builder_ annotation won’t generate code for inherited fields. It turns out, there is a solution to this problem.
> We have been using _@Builder_ on the class itself, but you can also put it on a class’s constructor or on a static method. In that case, Lombok will create a setter method on the builder class for every parameter of the constructor/method. That means you can create a custom constructor with parameters for all the fields of the class including its superclass.
> 
| @AllArgsConstructor
**public**class** Parent {
  **private** String a;
}
 
**public**class**extends** Parent {
 
  **private** String b;
 
  @Builder
  **private** Child(String a, String b){
    **super**(a);
    **this**.b = b;
  }
} |
| --- |

> By setting the visibility of the constructor to _private_ you can make sure that it is only available to Lombok (thanks to Mathias for the tip!). As a result you can then use the generated builder like this:
> 
| Child.builder().a("testA").b("testB").build() |
| --- |

> The [official documentation explains this](https://projectlombok.org/features/Builder.html), but it doesn’t explicitly point out that you can facilitate it in this way.



**我尝试着做了翻译：**

我曾经写过有关 Lombok _@Builder _注解的文章。不久前，我们开始在项目中使用到了它。但在此期间，有一件事情困扰着我： Lombok _@Builder _注解不会为继承的字段生成代码。事实证明，这个问题有一个解决方案。

我们通常都是将 _@Builder _注解用于类本身，但是同样可以将其用于类的构造方法或者是静态方法上。如果是这样， Lombok 会在 builder 类中为构造方法或者静态方法的每一个参数创建 setter 方法。这意味着，你可以创建一个自定义的构造方法，其中包含该类（包括其超类）所有字段的参数。

```java
@AllArgsConstructor
public class Parent {
  private String a;
}
 
public class Child extends Parent {
  private String b;
 
  @Builder
  private Child(String a, String b){
    super(a);
    this.b = b;
  }
}
```

通过将构造方法的可见性设置成私有的，可以确保其只对 Lombok 可用。因此你可以使用生成的构建器，如下：
```java
Child.builder().a("testA").b("testB").build()
```

官方文档对其进行了解释，但是并没有明确指出可以通过这种方式提供便利。

<a name="d17a0f0b"></a>
## 参考
[Lombok’s @Builder annotation and inheritance](https://reinhard.codes/2015/09/16/lomboks-builder-annotation-and-inheritance/)
