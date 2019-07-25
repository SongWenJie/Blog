# Spring 控制器参数详解

<a name="4dbol"></a>
#### 通过 URL 获取参数
![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1563779533489-9776589f-b449-41ad-902c-0bcd18c19248.png#align=left&display=inline&height=135&name=image.png&originHeight=211&originWidth=951&size=53568&status=done&width=608.64)<br />URL形式：localhost:8080/getBook2/bookName=martin&bookCode=123456<br />适用于get方式提交，不适用于post方式提交。<br />使用此种方式，如果我们不给参数赋值，那么spring会将空着的参数默认按照String型空字符串处理。所以，如果是String型的参数，为空时不报错的；若是非String型参数，则必须非空。（使用包装类可以解决）

<a name="iqCUD"></a>
#### 通过使用实体类获取参数
适用于get请求方式和post请求方式<br />对于post请求，建议使用此种方式。get请求方式参数比较多时，也建议使用此方式。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1563780595873-78b68a48-612d-419c-a37e-d85ca14917b2.png#align=left&display=inline&height=279&name=image.png&originHeight=436&originWidth=952&size=108998&status=done&width=609.28)

<a name="U7YX8"></a>
#### 通过@PathVariable获取路径中的参数
适用于get请求方式<br />请求路径形式：localhost:8080/getBook4/martin
```java
    @GetMapping("/getBook4/{bookName}")
    public Map<String,Object> getBook4(@PathVariable String bookName){
        Map<String, Object> paramMap = new HashMap<String, Object>();
        paramMap.put("bookName", bookName);
        return paramMap;
    }
```

<a name="7TZXH"></a>
#### 通过@RequestParam 获取参数
该注解用来处理Content-Type: 为 `application/x-www-form-urlencoded`编码的内容.

常用来处理简单类型的绑定，通过Request.getParameter() 获取的String可直接转换为简单类型的情况（ String--> 简单类型的转换操作由ConversionService配置的转换器来完成）；因为使用request.getParameter()方式获取参数，所以可以处理get 方式中queryString的值，也可以处理post方式中 body data的值。<br />适用于get请求方式和post请求方式

@RequestParam 支持下面四种参数，可以对参数进行更为详细的控制<br />defaultValue 如果本次请求没有携带这个参数，或者参数为空，那么就会启用默认值<br />name 绑定本次参数的名称，要跟URL上面的一样<br />required 这个参数是不是必须的<br />value 跟name一样的作用，是name属性的一个别名
```java
   @GetMapping("/getBook5")
    public Map<String,Object> getBook5(@RequestParam String bookName){
        Map<String, Object> paramMap = new HashMap<String, Object>();
        paramMap.put("bookName", bookName);
        return paramMap;
    }

    @PostMapping("/postBook2")
    public Map<String,Object> postBook2(@RequestParam String bookName){
        Map<String, Object> paramMap = new HashMap<String, Object>();
        paramMap.put("bookName", bookName);
        return paramMap;
    }
```
<a name="STnA1"></a>
#### @RequestParam with List or array
我们可以在调用中为参数指定多个值<br />http://localhost:8080/getBook8?bookNames=1,2<br />或<br />http://localhost:8080/getBook8?bookNames=1&bookNames=2
```java
@GetMapping("/getBook8")
public Map<String,Object> getBook8(@RequestParam List<String> bookNames){
    Map<String, Object> paramMap = new HashMap<String, Object>();
    paramMap.put("bookNames", bookNames);
    return paramMap;
}
```
对于对象类型的数组或集合，建议使用 @RequestBody 参数采用 Json 数据格式 
<a name="bl8FO"></a>
#### 通过@RequestBody 获取参数
该注解常用来处理Content-Type: 不是application/x-www-form-urlencoded编码的内容，例如application/json, application/xml等。<br />支持get请求方式和post请求方式，但是不建议get请求使用此方式。此方式是在请求体中携带参数，用于get请求容易发生匹配错误，也不符合HTTP get标准。

```java
    @GetMapping("/getBook6")
    public Map<String,Object> getBook6(@RequestBody String bookName){
        Map<String, Object> paramMap = new HashMap<String, Object>();
        paramMap.put("bookName", bookName);
        return paramMap;
    }

    @PostMapping("/postBook3")
    public Map<String,Object> postBook3(@RequestBody Book book){
        Map<String, Object> paramMap = new HashMap<String, Object>();
        paramMap.put("bookName", book.getBookName());
        paramMap.put("bookCode", book.getBookCode());
        return paramMap;
    }
```

<a name="34pyi"></a>
#### 通过@ModelAttribute获取参数
@ModelAttribute<br />该注解有两个用法，一个是用于方法上，一个是用于参数上；<br />用于方法上时：  通常用来在处理@RequestMapping之前，为请求绑定需要从后台查询的model；<br />用于参数上时： 用来通过名称对应，把相应名称的值绑定到注解的参数bean上；要绑定的值来源于：<br />A） @SessionAttributes 启用的attribute 对象上；<br />B） @ModelAttribute 用于方法上时指定的model对象；<br />C） 上述两种情况都没有时，new一个需要绑定的bean对象，然后把request中按名称对应的方式把值绑定到bean中。

```java
    @GetMapping("/getBook7/{bookName}/{bookCode}")
    public Map<String,Object> getBook7(@ModelAttribute Book book){
        Map<String, Object> paramMap = new HashMap<String, Object>();
        paramMap.put("bookName", book.getBookName());
        paramMap.put("bookCode", book.getBookCode());
        return paramMap;
    }

    @PostMapping("/postBook4")
    public Map<String,Object> postBook4(@ModelAttribute Book book){
        Map<String, Object> paramMap = new HashMap<String, Object>();
        paramMap.put("bookName", book.getBookName());
        paramMap.put("bookCode", book.getBookCode());
        return paramMap;
    }
```
<a name="6lpr1"></a>
#### 问题： 在不给定注解的情况下，参数是怎样绑定的？
通过分析AnnotationMethodHandlerAdapter和RequestMappingHandlerAdapter的源代码发现，方法的参数在不给定参数的情况下：<br />若要绑定的对象时简单类型：  调用@RequestParam来处理的。  <br />若要绑定的对象时复杂类型：  调用@ModelAttribute来处理的。<br />这里的简单类型指java的原始类型(boolean, int 等)、原始类型对象（Boolean, Int等）、String、Date等ConversionService里可以直接String转换成目标对象的类型；


<a name="RSFqJ"></a>
#### 数据格式（类型）很重要
form是通过name进行参数匹配的<br />json是通过反序列化进行参数匹配的，所以对get请求来说，使用json格式，无法匹配多个参数，但是可以使用实体类进行封装
