# JavaScript 提升不完全指北

我们直觉上会认为JavaScript 代码在执行时是由上到下一行一行执行的。 但实际上这并不完全正确， 有一种特殊情况会导致这个假设是错误的，这种情况叫做提升。**提升是指不管变量和函数声明在代码的哪个位置，都会提升到所在作用域的顶部**。
<a name="6okV1"></a>
## 提升
思考下面的代码的执行结果：

```javascript
a = 2;
var a;
console.log( a );
```
 <br />很多人会认为是  `undefined` , 因为 var a 声明在 a = 2 之后， 他们自然而然地认为变量被重新赋值了， 因此会被赋予默认值 undefined。 但是， 真正的输出结果是 2。<br />再思考如下代码：

```javascript
console.log( a );
var a = 2;
```

鉴于上一个代码片段所表现出来的某种非自上而下的行为特点， 你可能会认为这个代码片段也会有同样的行为而输出 2。 还有人可能会认为， 由于变量 a 在使用前没有先进行声明，因此会抛出 ReferenceError 异常。但是很遗憾，两种猜测都是不对的，真正的输出结果是 undefined 。 <br />为了搞清楚这个问题，我们需要回忆一下 [JavaScript 作用域不完全指北](https://mp.weixin.qq.com/s?__biz=MzIxNjc2MzI2NQ==&mid=2247483911&idx=1&sn=ad7b5bedfdfd3527bd6d2489d0f632fd&chksm=97855ce9a0f2d5ff23d1e9151b39f262ab849eac92ae230b42859898b24a88856eb8e2806b07&token=1162220155&lang=zh_CN#rd) 中提到的，JavaScript 是一门编译语言。当我们看到 var a = 2; 时， 可能会认为这是一个声明。 但 **JavaScript 实际上会将其看成两个声明： var a; 和 a = 2;。 第一个定义声明是在编译阶段进行的。 第二个赋值声明会被留在原地等待执行阶段**。<br />我们的第一段代码会被按照如下流程处理：

```javascript
var a;

a = 2;
console.log( a ); //2
```

其中第一部分是编译， 而第二部分是执行。<br />同理，第二段代码会被按照如下流程处理：

```javascript
var a;

console.log( a ); //undefined
a = 2;
```

这个过程就好像变量和函数声明从它们在代码中出现的位置被“移动”到了所在作用域的最上面。 这个过程就叫作提升。<br />有几个需要特别注意的地方：

1. **只是变量或者函数的声明被“移动”了，而赋值和其他的运行逻辑被留在了原地**
1. **每个作用域都会进行提升操作**

```javascript
foo();
function foo() {
	console.log( a ); // undefined
	var a = 2;
}
```

<br />被提升的不只是函数 foo()，还有函数 foo() 作用域中的变量 a 。处理后的代码流程如下：

```javascript
function foo() {
  var a;
	console.log( a ); // undefined
	a = 2;
}
foo();
```

3. **函数声明会被提升，但是函数表达式不会**

```javascript
foo(); // 不是 ReferenceError, 而是 TypeError!
var foo = function bar() {
	// ...
};
```

此处需要注意的是，运行 foo() 函数抛出的错误是 TypeError，而不是  ReferenceError。我们在作用域一文中讲到过这两种错误的区别，ReferenceError 是作用域判别失败，也就是嵌套的所有作用域中都不存在此标志符；而 TypeError 是作用域判别成功了，但是试图对这个变量的值做非法的操作，比如对一个非函数类型的值进行函数调用， 或着引用 null 或 undefined 类型的值中的属性。<br />示例代码中抛出 TypeError 错误就是因为对 undefined 做函数调用，根据这个能推断出**实际上函数表达式也被提升了，只是在执行前没有被赋值**。在这一点上，let 和 const 都是如此（这里不做探究，将会在后文中单独讲解），执行流程如下：

```javascript
var foo;
foo(); // 不是 ReferenceError, 而是 TypeError!
foo = function bar() {
	// ...
};
```

4. **函数优先，函数和变量都会被提升，但是函数会首先被提升，然后才是变量**

思考如下代码：
```javascript
foo(); // 1
var foo;
function foo() {
	console.log( 1 );
} 
foo = function() {
	console.log( 2 );
};
```
输出 1 而不是 2 ，引擎执行代码的实际流程如下:
```javascript
function foo() {
	console.log( 1 );
} 
foo(); // 1
foo = function() {
	console.log( 2 );
};
```
 var foo 尽管出现在 function foo()... 的声明之前， 但它是重复的声明，因此被忽略了， 因为**函数声明会被提升到普通变量之前**。<br />尽管**重复的 var 声明(没有赋值)会被忽略掉， 但出现在后面的函数声明还是可以覆盖前面的**。

```javascript
foo(); // 3
function foo() {
console.log( 1 );
}
var foo = function() {
console.log( 2 );
};
function foo() {
console.log( 3 );
}
```
这样会带来一个严重的问题，一个普通块内部的函数声明通常会被提升到所在作用域的顶部，但是重复定义的函数，后定义的函数会覆盖先定义的函数，这会造成严重的代码问题。

```javascript
foo(); // "b"
var a = true;
if (a) {
	function foo() { 
        console.log("a");
    }
}
else {
	function foo() {
    	console.log("b"); 
    }
}
```

<br />虽然 if 判断永远成立（因为 a = true），但是因为函数的提升和重复定义，会导致实际执行的永远是后定义的函数。本示例中的实际执行代码如下：

```javascript
//function foo() { 
//    console.log("a");   //被覆盖
//}
function foo() {
    console.log("b"); 
}
foo(); // "b"
```

