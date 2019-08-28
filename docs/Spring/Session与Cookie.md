# Session Cookie

## 会话
Web 程序中常用的技术，用来跟踪用户的整个会话。由于http协议是无状态的协议，为了能够记住请求的状态，引入了会话跟踪技术 Cookie与Session。**Cookie通过在客户端记录信息确定用户身份**，**Session通过在服务器端记录信息确定用户身份**。

一次会话指的是：就好比打电话，A给B打电话，接通之后，会话开始，直到挂断电话，该次会话就结束了，而浏览器访问服务器，就跟打电话一样，浏览器A给服务器发送请求，访问web程序，该次会话就已经接通，其中不管浏览器发送多少请求(就相当于接通电话后说话一样)，都视为一次会话，直到浏览器关闭，本次会话结束。其中注意，一个浏览器就相当于一部电话，如果使用火狐浏览器，访问服务器，就是一次会话了，然后打开google浏览器，访问服务器，这是另一个会话，虽然是在同一台电脑，同一个用户在访问，但是，这是两次不同的会话。

一个浏览器访问一个服务器就能建立一个会话，如果其他电脑，都同时访问该服务器，就会创建很多会话，就拿一些购物网站来说，我们访问一个购物网站的服务器，会话就被创建了，然后就点击浏览商品，对感兴趣的商品就先加入购物车，等待一起付账，这看起来是很普通的操作，但是想一下，如果有很多别的电脑上的浏览器同时也在访问该购物网站的服务器，跟我们做类似的操作呢？服务器又是怎么记住用户，怎么知道用户A选择的任何商品都应该放在A的购物车内，不论是用户A什么时间选择的，都不能放入用户B或用户C的购物车内的呢？所以就有了Cookie 和Session 这两个技术，就像上面说的那样，Cookie 和 Session用来跟踪用户的整个会话。

## 理解 Session 与 Cookie
从网上看到一个会员卡的例子来解释 Session 与 Cookie ，非常的形象。

假如一个咖啡店有喝5杯咖啡免费赠一杯咖啡的优惠，然而一次性消费5杯咖啡的机会微乎其微，这时就需要某种方式来纪录某位顾客的消费数量。想象一下其实也无外乎下面的几种方案：

1、该店的店员很厉害，能记住每位顾客的消费数量，只要顾客一走进咖啡店，店员就知道该怎么对待了。这种做法就是协议本身支持状态。但是HTTP协议本身是无状态的。
2、发给顾客一张卡片，上面记录着消费的数量，一般还有个有效期限。每次消费时，如果顾客出示这张卡片，则此次消费就会与以前或以后的消费相联系起来。这种做法就是在客户端保持状态。也就是cookie。 顾客就相当于浏览器，Cookie如何工作，下面会详细讲解。
3、发给顾客一张会员卡，除了卡号之外什么信息也不纪录，每次消费时，如果顾客出示该卡片，则店员在店里的纪录本上找到这个卡号对应的纪录添加一些消费信息。这种做法就是在服务器端保持状态。

由于HTTP协议是无状态的，而出于种种考虑也不希望使之成为有状态的，因此，后面两种方案就成为现实的选择。具体来说cookie机制采用的是在客户端保持状态的方案，而session机制采用的是在服务器端保持状态的方案。同时我们也看到，由于采用服务器端保持状态的方案在客户端也需要保存一个标识，所以 Session 机制可能需要借助于Cookie 机制来达到保存标识的目的，但实际上它还有其他选择。

## Cookie详解
我们还是继续使用会员卡的例子，来分析 Cookie ，也就是上面所提到的第 2 种方式。

1、如何分发会员卡？
由服务器进行创建，也就相当于咖啡店来创建会员卡，在创建会员卡的同时，设置会员卡中的内容。同时在用户第一次到店消费时分发给用户，也就是在客户端第一次请求服务器的时候返回 Cookie，通过 HTTP 响应头中的 Set-Cookie 完成。

```java
Cookie cookie = new Cookie(key,value);　　//以键值对的方式存放内容，
response.addCookie(cookie);　　//发送回浏览器端
注意：一旦cookie创建好了，就不能在往其中增加别的键值对，但是可以修改其中的内容，
cookie.setValue();　　//将key对应的value值修改
```

2、客户如何使用会员卡，cookie在客户端是如何工作的，工作原理是什么？
具体流程见下图：
![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1565774584102-de6fed7e-998b-431a-9423-e8adffc91e61.png#align=left&display=inline&height=399&name=image.png&originHeight=548&originWidth=1199&size=46823&status=done&width=872)

Cookie 由服务器创建好返回给客户端之后，客户端会将 Cookie 缓存到内存中或者持久化到硬盘中。然后在之后的请求中带上这个 Cookie，以此来证明自己的身份。就相当于咖啡店创建好了会员卡，并且已经设置了其中的内容，交到了客户手中，下次客户过来时，就带着会员卡过来，就知道你是会员了，然后咖啡店就拿到你的会员卡对其进行操作。

3、会员卡的有效日期？也就是cookie的有效日期
这个可以自由设置，默认是关闭浏览器，cookie就没用了。
　　cookie.setMaxAge(expiry);　　//设置Cookie 被浏览器保存的时间。
　　expiry：单位秒，默认为-1，

  - expiry=-1：代表浏览器关闭后，也就是会话结束后，Cookie 就失效了，也就没有了。
  - expiry>0：代表浏览器关闭后，Cookie 不会失效，仍然存在。并且会将Cookie 保存到硬盘中，直到设置时间过期才会被浏览器自动删除，
  - expiry=0：删除Cookie 。不管是之前的expiry=-1还是expiry>0，当设置expiry=0时，Cookie 都会被浏览器给删除。

4、会员卡的使用范围？
比如星巴克在北京有一个分店，在上海也有一个分店，我们只是在北京的星巴克办理了会员卡，那么当我们到上海时，就不能使用该会员卡进行打折优惠了。而Cookie 也是如此，可以设置服务器端获取Cookie 的访问路径，而并非在服务器端的web项目中所有的servlet都能访问该Cookie 。

Cookie 默认路径：当前访问的servlet父路径。
例如：http://localhost:8080/test/a/b/c/SendCookieServlet
　　默认路径：/test/a/b/c　　
也就是说，在该默认路径下的所有Servlet都能够获取Cookie，/test/a/b/c/MyServlet　这个MyServlet就能获取到Cookie 。
　　修改cookie的访问路径：
　　setPath("/")；　　//在该服务器下，任何项目，任何位置都能获取到Cookie ，　　　　　　　　
　　setPath("/test/");　　//在test项目下任何位置都能获取到Cookie 。
### 
### 总结
工作流程：
1. 服务器创建Cookie ，保存少量数据（有限制），发送给浏览器。
2. 浏览器获得服务器发送的Cookie 数据，将自动的保存到浏览器端。
3. 下次访问时，浏览器将自动携带Cookie 数据发送给服务器。

Cookie 操作：
1.创建Cookie ：new Cookie(name,value)
2.发送Cookie 到浏览器：HttpServletResponse.addCookie(Cookie)
3.servlet接收Cookie ：HttpServletRequest.getCookies()  //浏览器发送的所有cookie

Cookie 特点：
1. 每一个Cookie 文件大小：4kb ， 如果超过4kb浏览器不识别
2. 一个web站点（web项目）：发送20个
3.一个浏览器保存总大小：300个
4.**Cookie ****不安全，可能泄露用户信息**。浏览器支持禁用cookie操作。
5. 默认情况生命周期：与浏览器会话一样，当浏览器关闭时销毁 Cookie。

## Session详解
会员卡的例子的第3种方法，发给顾客一张会员卡，除了卡号之外什么信息也不纪录，每次消费时，如果顾客出示该卡片，则店员在店里的纪录本上找到这个卡号对应的纪录添加一些消费信息。这种做法就是在服务器端保持状态。　这就是 Session 的用法，在服务器端来保持状态，保存一些用户信息。

1、Session 的工作原理
首先浏览器请求服务器访问服务器时，程序需要为客户端的请求创建一个 Session 的时候，服务器首先会检查这个客户端请求是否已经包含了一个Session标识、称为SessionId，如果已经包含了一个SessionId则说明以前已经为此客户端创建过Session，服务器就按照SessionId把这个Session检索出来使用；如果客户端请求不包含SessionId，则服务器为此客户端新创建一个Session并且生成一个与此Session相关联的SessionId，SessionId的值应该是一个既不会重复，又不容易被找到规律以仿造的字符串，这个SessionId将在本次响应中返回到客户端保存，保存这个SessionId的方式就可以是Cookie，这样在后续的交互过程中，浏览器可以自动的按照规则把这个标识发回给服务器，服务器根据这个SessionId就可以找得到对应的Session。
具体流程见下图：
![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1565774885601-4df8b96d-1904-4e35-ab90-de895f6e4bbd.png#align=left&display=inline&height=396&name=image.png&originHeight=544&originWidth=1258&size=51795&status=done&width=914.9090909090909)

2、如何获取记录本？也就是获取Session
为什么是通过request来获取session，可以这样理解，在获取session时，需要检测请求中是否有session标识，所以需要用request来获取。

```java
request.getSession();　　//如果没有将创建一个新的，等效getSession(true);
request.getSession(boolean);　　//true：没有将创建，false：没有将返回null
```

3、如何记录消费信息？也就是Session 的属性操作
在 Session 中记录信息，一般是用户信息。

```java
session.setAttrubute(key,value);//记录信息
session.getAttribute(key);//获取信息
```


4、Session 的生命周期
可以想象一下会员卡的例子，除非顾客主动对店家提出销卡，否则店家绝对不会轻易删除顾客的信息。对Session 来说也是一样的，除非程序通知服务器删除一个 Session ，否则服务器会一直保留，程序一般都是在用户退出登录的时候发个指令去删除Session 。也就是说只有用户是登录状态才会记录用户信息。


有一种误解是"只要关闭浏览器，Session 就消失"，之所以会有这种误解是因为大部分Session机制都使用会话Cookie来保存 SessionId，默认情况下关闭浏览器，SessionId就会消失，再次连接服务器时也就无法找到原来的Session。所以关闭浏览器消失的不是 Session，而是 SessionId（Cookie）。浏览器从来不会主动在关闭之前通知服务器它将要关闭，因此服务器根本不会有机会知道浏览器已经关闭，所以根本不会有删除 Session 的机会。正是基于这种机制，迫使服务器为Session设置了一个失效时间，一般是30分钟，当距离客户端上一次使用Session的时间超过这个失效时间时，服务器就可以认为客户端已经停止了活动，才会把Session删除以节省存储空间。    


我们也可以自己来控制session的有效时间：
```java
session.invalidate()将session对象销毁
session.setMaxInactiveInterval(int interval) 设置有效时间，单位秒
```
　　　　　　　

5、Session 的客户端标识    
由于采用服务器端保持状态的方案在客户端也需要保存一个标识 SessionId，所以 Session 机制可能需要借助于Cookie 机制来达到保存标识的目的，但实际上它还有其他选择，例如Local Storage、Session Storage或其它数据存储形式中。Cookie 的优势是浏览器默认会在 HTTP请求头中携带 Cookie 信息（如果有Cookie），而其他方式，例如Local Storage、Session Storage 则需要在发起 HTTP请求时手动添加Cookie 信息到HTTP请求头中。

## 参考
[理解HTTP session原理及应用](https://blog.csdn.net/zhouqixiang/article/details/1921606)
