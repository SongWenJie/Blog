# JavaScript 词法作用域不完全指北

在 **[JavaScript 词法作用域不完全指北](https://mp.weixin.qq.com/s?__biz=MzIxNjc2MzI2NQ==&mid=2247483911&idx=1&sn=ad7b5bedfdfd3527bd6d2489d0f632fd&chksm=97855ce9a0f2d5ff23d1e9151b39f262ab849eac92ae230b42859898b24a88856eb8e2806b07&token=1162220155&lang=zh_CN#rd) **中，我们介绍了作用域的概念以及 JavaScript 引擎、编译器和作用域的关系。作用域有两种主要的工作模型：**词法作用域**和**动态作用域**。其中最为普遍的也是大多数编程语言所采用的是词法作用域，我们主要对其进行研究学习。<br />在传统编译语言的流程中， 程序中的一段源代码在执行之前会经历三个步骤， 统称为“编译”。

- 分词/词法分析（Tokenizing/Lexing）

这个过程会将由字符组成的字符串分解成（对编程语言来说） 有意义的代码块， 这些代码块被称为词法单元。

- 解析/语法分析（Parsing）

这个过程是将词法单元流（数组） 转换成一个由元素逐级嵌套所组成的代表了程序语法结构的树。 这个树被称        为“抽象语法树”（Abstract Syntax Tree， AST）。

- 代码生成

将“抽象语法树” 转换为可执行代码的过程称被称为代码生成。 这个过程与语言、 目标平台等息息相关。<br />第一个步骤也叫作词法化，词法作用域就是定义在词法阶段的作用域。简单地说，词法作用域是由你写代码时将变量和块作用域写在哪里来决定的，词法分析器处理代码时会保持作用域不变。<br />我们通过以下代码来分析一下词法作用域：

```javascript
function foo(a){
  var b = a * 2;
  function bar(c){
    console.log(a,b,c);
  }
  bar(b * 3);
}
foo(2); //2 4 12
```

在实例代码中，会有三个逐级嵌套的作用域。

1. 包含着全局作用域，其中有一个标识符 foo

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1559920774376-ebe4afa4-0ef8-4494-80e4-cb8e8d63f002.png#align=left&display=inline&height=179&name=image.png&originHeight=224&originWidth=297&size=10721&status=done&width=237.6)

2. 包含着 foo 所创建的作用域，其中有三个标识符：a，bar，b

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1559920856838-52624912-c562-4c39-be1e-bc8a9b85acc8.png#align=left&display=inline&height=177&name=image.png&originHeight=221&originWidth=314&size=11183&status=done&width=251.2)

3. 包含着 bar 所创建的作用域，其中有一个标识符：c

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1559920974779-e67a44bd-6702-4313-a1c6-47842fa176ed.png#align=left&display=inline&height=174&name=image.png&originHeight=218&originWidth=291&size=10776&status=done&width=232.8)

**引擎使用作用域的结构和相互之间的位置关系来查找标识符**。我们在上篇文章中讲过，引擎在作用域中进行变量查找的过程，是从当前作用域逐级向外，直到遇到第一个匹配的标识符结束。<br />在实例代码中，引擎执行 `console.log(a,b,c);` 声明，并查找变量 a , b , c 的引用。首先从最内部的作用域，也就是 bar 函数的作用域开始查找，引擎无法在这里查找到变量 a ,便会到上一级所嵌套的 foo 函数作用域中进行查找。引擎在这里找到了变量 a 的引用，便会停止对变量 a 引用的查询。对 b 来说也是一样的。对 c 来说，引擎在 bar 函数作用域中就会找到它。<br />引擎会在作用域中找到第一个匹配的标识符时停止查找。也就是说，在多层的嵌套作用域中可以定义同名的标识符，内部的标识符会遮蔽外部的标识符，这叫作“遮蔽效应”。<br />**词法作用域意味着作用域是由书写代码时函数的位置来决定的。编译的词法分析阶段基本能够知道全部标识符在哪里以及是如何声明的，从而预测在引擎执行代码过程中如何对它们进行查找**。

<a name="3NkEZ"></a>
## 参考

- 《你不知道的JavaScript》
- 《深入理解JavaScript特性
