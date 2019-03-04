Liskov替换原则（Liskov Substitution Principle）是一组用于创建**继承层次结构**的指导原则。按照Liskov替换原则创建的继承层次结构中，客户端代码能够放心的使用它的任意类或子类而不担心所期望的行为。

## Liskov替换原则定义

> 如果S是T的子类型，那么所有的T类型的对象都可以在不破坏程序的情况下被S类型的对象替换。

- 基类型：客户端引用的类型（T）。子类型可以重写（或部分定制）客户端所调用的基类的任意方法。
- 子类型：继承自基类型（T）的一组类（S）中的任意一个。客户端不应该，也不需要知道它们实际调用哪个具体的子类型。无论使用的是哪个子类型实例，客户端代码所表现的行为都是一样的。

## Liskov替换原则的规则

要应用Liskov替换原则就必须遵守两类规则：

1.**契约规则（与类的期望有关）**

- 子类型不能加强前置条件
- 子类型不能削弱后置条件
- 子类型必须保持超类型中的数据不变式

2.**变体规则（与代码中能被替换的类型有关）**

- 子类型的方法参数必须是支持逆变的
- 子类型的返回类型必须是支持协变的
- 子类型不能引发不属于已有异常层次结构中的新异常



## 契约

我们经常会说，要面向接口编程或面向契约编程。然后，除了表面上的方法签名，**接口所表达的只是一个不够严谨的契约概念**。

作为方法编写者，要确保方法名称能反应出它的真实目的，同时参数名称要尽可能使描述性的。

```c#
public decimal CalculateShippingCost(int count,decimal price)
{
    return count * price;
}
```

然而，方法签名并没有包含方法的契约信息。比如price参数是decimal类型的，这就表明任何decimal类型的值都是有限的。但是price参数的意义是价格，显然价格不能是负数。为了做到这一点，要在方法内部实现一个前置条件。

### 前置条件

**前置条件（precondition）是一个能保障方法稳定无错运行的先决条件**。所有方法在被调用钱都要求某些前置条件为真。

引发异常是一种强制履行契约的高效方式：

```c#
public class ShippingStrategy
{
	public decimal CalculateShippingCost(int count,decimal price)
	{
	    if(price <= Decimal.Zero)
	    {
	        throw new Exception();
	    }
	    return count * price;
	}
}
```

更好的方式是提供详尽的前置条件校验失败原因，便于客户端快速排查问题。此处抛出参数超出了有效范围，并且明确指出了是哪一个参数。

```c#
public class ShippingStrategy
{
	public decimal CalculateShippingCost(int count, decimal price)
	{
	    if (price <= Decimal.Zero)
	    {
	        throw new ArgumentOutOfRangeException("price", "price must be positive and non-zero");
	    }
	    return count * price;
	}
}
```

有了这些前置条件，客户端代码就必须在调用方法钱确保它们传递的参数值要处于有效范围内。当然，所有在前置条件中检查的状态必须是公开可访问的。私有状态不应该是前置条件检查的目标，只有方法参数和类的公共属性才应该有前置条件。

### 后置条件

**后置条件会在方法退出时检测一个对象是否处于一个无效的状态**。只要方法内改动了状态，就用可能因为方法逻辑错误导致状态无效。

方法的尾部临界子句是一个后置条件，它能确保返回值处于有效范围内。该方法的签名无法保证返回值必须大于零，要达到这个目的，必须通过客户端履行方法的契约来保证。

```c#
public class ShippingStrategy
{
	public decimal CalculateShippingCost(int count, decimal price)
	{
	    if (price <= Decimal.Zero)
	    {
	        throw new ArgumentOutOfRangeException("price", "price must be positive and non-zero");
	    }
	
	    decimal cost = count * price;
	
	    if (cost <= Decimal.Zero)
	    {
	        throw new ArgumentOutOfRangeException("cost", "cost must be positive and non-zero");
	    }
	    return cost;
	}
}
```

### 数据不变式

**数据不变式（data invariant）是一个在对象生命周期内始终保持为真的一个谓词；该谓词条件在对象构造后一直超出其作用范围前的这段时间都为真**。

数据不变式都是与期望的对象内部状态有关，例如税率为正值且不为零。在构造函数中设置税率，只需要在构造函数中增加一个防卫子句就可以防止将其设置为无效值。

```c#
public class ShippingStrategy
{
    protected decimal flatRate;
    public ShippingStrategy(decimal flatRate)
    {
        if(flatRate <= Decimal.Zero)
        {
            throw new ArgumentOutOfRangeException("flatRate", "flatRate must be positive and non-zero");
        }
        this.flatRate = flatRate;
    }
}
```

因为flatRate是一个受保护的成员变量，所以客户端只能通过构造函数来设置它。如果传入构造函数的值是有效的，就保证了ShippingStrategy对象在整个生命周期内的flatRate值都是有效的，因为客户没有地方可以修改它。但是，如果把flatRate定义为公共并且可设置的属性，为了保证数据不变式，就必须将防卫子句布置到属性设置器内。

```c#
public class ShippingStrategy
{
    private decimal flatRate;
    public decimal FlatRate
    {
        get
        {
            return flatRate;
        }
        set
        {
            if (value <= Decimal.Zero)
            {
                throw new ArgumentOutOfRangeException("flatRate", "flatRate must be positive and non-zero");
            }
            flatRate = value;
        }
    }
    public ShippingStrategy(decimal flatRate)
    {
        this.FlatRate = flatRate;
    }
}
```

### Liskov契约规则

在适当的时候，子类被允许重写父类的方法实现，此时才有机会修改其中的契约。**Liskov替换原则明确规定一些变更是被禁止的，因为它们会导致原来使用超类实例的客户端代码在切换至子类时必须要做更改**。

1.子类型不能加强前置条件

当子类重写包含前置条件的超类方法时，绝不应该加强现有的前置条件，这样做**会影响到那些已经假设超类为所有方法定义了最严格的前置条件契约的客户端代码**。

![mark](http://songwenjie.vip/blog/180904/H8ceL0igJC.png?imageslim)

```c#
public class WorldWideShippingStrategy : ShippingStrategy
{
    public override decimal CalculateShippingCost(int count, decimal price)
    {
        if (price <= Decimal.Zero)
        {
            throw new ArgumentOutOfRangeException("price", "price must be positive  and non-zero");
        }
        if (count <= 0)
        {
            throw new ArgumentOutOfRangeException("count", "count must be positive  and non-zero");
        }
        return count * price;
    }
}
```

2.子类型不能削弱后置条件

与前置条件相反，不能削弱后置条件。因为已有的客户端代码在原有的超类切换至新的子类时很可能会出错。

原有的方法后置条件是方法的返回值必须大于零，映射到现实场景就是购物金额不能为负数。

![mark](http://songwenjie.vip/blog/180904/H8ceL0igJC.png?imageslim)

```c#
public class WorldWideShippingStrategy : ShippingStrategy
{
    public override decimal CalculateShippingCost(int count, decimal price)
    {
        if (price <= Decimal.Zero)
        {
            throw new ArgumentOutOfRangeException("price", "price must be positive  and non-zero");
        }
      
        decimal cost = count * price;

        return cost;
    }
}
```

3.子类型必须保持超类型中的数据不变式

在创建新的子类时，它必须继续遵守基类中的所有数据不变式。这里是很容易出问题的，因为子类有很多机会来改变基类中的私有数据。

![mark](http://songwenjie.vip/blog/180904/H8ceL0igJC.png?imageslim)

```c#
public class ShippingStrategy
{
    public ShippingStrategy(decimal flatRate)
    {
        if (flatRate < Decimal.Zero)
        {
            throw new ArgumentOutOfRangeException("flatRate", "flatRate must be positive and non-zero");
        }
        this.flatRate = flatRate;
    }

    protected decimal flatRate;
}

public class WorldWideShippingStrategy : ShippingStrategy
{
    public WorldWideShippingStrategy(decimal flatRate) : base(flatRate)
    {
    }

    public  decimal FlatRate
    {
        get
        {
            return base.flatRate;
        }
        set
        {
            base.flatRate = value;
        }
    }
}
```

一种普遍的模式是，私有的字段有对应的受保护的或者公共的属性，属性的设置器中包含的防卫子句用来保护属性相关的数据不变式。**更好的方式是，在基类中控制字段的可见性并只允许引入防卫子句的属性设置器访问该字段，将来所有的子类都不再需要防卫子句检查**。

![mark](http://songwenjie.vip/Snipaste_2018-09-04_10-55-50.png?imageMogr2/thumbnail/x200/interlace/1/blur/1x0/quality/75)

```c#
public class ShippingStrategy
{
    public ShippingStrategy(decimal flatRate)
    {
        this.FlatRate = flatRate;
    }

    private decimal flatRate;
    protected decimal FlatRate
    {
        get
        {
            return flatRate;
        }
        set
        {
            if (value < Decimal.Zero)
            {
                throw new ArgumentOutOfRangeException("flatRate", "flatRate must be positive and non-zero");
            }
            flatRate = value;
        }
    }
}

public class WorldWideShippingStrategy : ShippingStrategy
{
    public WorldWideShippingStrategy(decimal flatRate) :base(flatRate)
    {
    }

    public new decimal FlatRate
    {
        get
        {
            return base.FlatRate;
        }
        set
        {
            base.FlatRate = value;
        }
    }
}
```



##协变和逆变

Liskov替换原则的剩余原则都与协变和逆变相关。首先要明确变体（variance）这个概念，变体这个术语主要应用于复杂层次类型结构中**以定义子类型的期望类型**，有点类似于多态。在C#语言中，变体的实现有协变和逆变两种。

### 协变

下图展示了一个非常小的类层次结构，包含了基（超）类Supertype和子类Subtype。

![mark](http://songwenjie.vip/blog/180904/Fg6ggAcKbc.png?imageslim)

**多态是一种子类型被看做基类型实例的能力**。任何能够接受Supertype类型实例的方法也可以接受Subtype类型实例，客户端不需要做类型转换，也不需要知道任何子类相关的信息。

如果我们引入一个通过**泛型参数**使用Supertype和Subtype的类型时，就进入了变体（variance）的主题。因为有了协变，一样可以用到多态这个强大的特性。当有方法需要ICovariant<Supertype>的实例时，完全可以使用ICovariant<Subtype>的实例替代之。

![mark](http://songwenjie.vip/blog/180904/iam3cm5Egj.png?imageslim)

举一个从仓储库中获取对象的例子帮助理解：

```c#
public class Entity
{
    public Guid ID { get; set; }

    public string Name { get; set; }
}

public class User:Entity
{
    public string Email { get; set; }

    public DateTime DateOfBirth { get; set; }
}
```

因为User类和Entity类之间是继承关系，所以我们也想在仓储实现上存在继承层次结构，通过重写基类方法返回不同具体类型对象。
```c#
public class EntityRepository
{
    public virtual Entity GetByID(Guid ID)
    {
        return new Entity();
    }
}

public class UserRepository : EntityRepository
{
    public override User GetByID(Guid ID)
    {
        return new User();
    }
}
```

![mark](http://songwenjie.vip/blog/180904/H8ceL0igJC.png?imageslim)

结果就会发现编译不通过。**因为不使用泛型类型，C#方法的返回类型就不是协变的。**换句话说，这种情况下（普通类）的继承是不具备协变能力的。

![mark](http://songwenjie.vip/blog/180904/G2cam02Jhk.png?imageslim)

![mark](http://songwenjie.vip/Snipaste_2018-09-04_10-55-50.png?imageMogr2/thumbnail/x200/interlace/1/blur/1x0/quality/75)

有两种方案可以解决此问题：

1.可以将UserRepository类的GetByID方法的返回类型修改回Entity类型，然后在该方法返回的地方应用多态将Entity类型的实例装换为User类型的实例。这种方式虽然客户解决问题，但是对于客户端并不友好，因为客户端必须自己做实例类型转换。

```c#
public class UserRepository : EntityRepository
{
    public override Entity GetByID(Guid ID)
    {
        return new User();
    }
}
```

2.可以把EntityRepository重新定义为一个需要泛型的类型，把Entity类型作为泛型参数传入。这个泛型参数是可以协变的，UserRepository子类可以为User类指定超类型。

```c#
public interface IEntityRepository<out T> where T:Entity
{
    T GetByID(Guid ID);
}

public class EntityRepository : IEntityRepository<Entity>
{
    public Entity GetByID(Guid ID)
    {
        return new Entity();
    }
}


public class UserRepository : IEntityRepository<User>
{
    public User GetByID(Guid ID)
    {
        return new User();
    }
}
```

新的UserRepository类的客户端无需再做向下的类型转换，因为直接得到就是User类型对象，而不是Entity类型对象。EntityRepository和UserRepository两个类的父子继承关系也得以保留。

### 逆变

协变是与方法返回类型的处理有关，而逆变是与方法参数类型的处理有关。

![mark](http://songwenjie.vip/blog/180904/FFCHdAd5F3.png?imageslim)

如图所示，泛型参数由关键字in标记，表示它是可逆变的。这表明层析结构已经被颠倒了：IContravariant<Subtype>成为了超类，IContravariant<Supertype>则变成了子类。

```c#
 public interface IEqualityComparer<in T> where T:Entity
 {
     bool Equals(T left, T right);
 }

 public class EntityEqualityComparer : IEqualityComparer<Entity>
 {
     public bool Equals(Entity left, Entity right)
     {
         return left.ID == right.ID;
     }
 }
```

```c#
IEqualityComparer<User> userComparer = new EntityEqualityComparer();
User user1 = new User();
User user2 = new User();
userComparer.Equals(user1, user2);
```

![mark](http://songwenjie.vip/blog/180904/H8ceL0igJC.png?imageslim)

如果没有逆变（接口定义中泛型参数前的in 关键字），编译时会直接报错。

![mark](http://songwenjie.vip/blog/180904/4JHaiHmBf5.png?imageslim)

错误信息告诉我们，无法将EntityEqualityComparer转换为IEqualityComparer<User>类型。直觉就是这样，因为Entity是基类，User是子类型。而如果IEqualityComparer支持逆变，现有的继承层次结构会被颠倒。此时可以**向需要具体类型参数的地方传入更通用的类型**。

### 不变性

除了逆变和协变的行为外，类型本身具有不变性。这里的不变性是指“**不会生成变体**”。既不可协变也不可逆变，必定是个非变体。具体到实现层面，定义中没有对in和out关键字的引用，这二者分别用来指定逆变和协变。**C#语言的方法参数类型和返回类型都是不可变的，只有在设计泛型时才能将类型定义为可协变的或可逆变的**。

### Liskov类型系统规则

- 子类型的方法参数必须是支持逆变的

- 子类型的返回类型必须是支持协变的

- 子类型不能引发不属于已有异常层次结构中的新异常

  异常机制的主旨就是**将错误的汇报和处理环节分隔开**。捕获异常后不做任何处理或只捕获最通用的Exception基类都是不可取的，二者结合就更糟糕了。从SystemException派生出来的异常基本都是根本无法处理和恢复的情况。好的做法总是从ApplicationException类派生自己的异常。



## 最后

Liskov替换原则是SOLID原则中最复杂的一个。需要理解契约和变体的概念才可以应用Liskov替换原则编写具有更高自适应能力的代码。**理想情况下，不论运行时使用的是哪个具体的子类型，客户端都可以只引用一个基类或接口而无需担心行为变化。**任何对Liskov替换原则定义规则的违背都应该被看作技术债务，应该尽早的偿还掉这些技术债务，否则后患无穷。



## 参考

《C#敏捷开发实践》