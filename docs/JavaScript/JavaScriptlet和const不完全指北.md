# JavaScript let和const不完全指北

<a name="NIXJW"></a>
## let
`let`  声明时 ES6 中最广为人知的特性之一，它和 `var`  声明功能类似，都用于变量声明，但是有着不同的作用域规则。<br />`var` 声明的变量是基于词法作用域的，仍然可以在变量声明所在块的外部访问到该变量，并且不会抛出错误。

```javascript
{
  {
    {
    	var deep = 'this is available from outer scope'
    }
  }
}
console.log(deep); //this is available from outer scope
```

但是如果在下面的几种情况下，抛出错误才是更好的方式：

- 访问内部变量破坏代码的封装性
- 同级的兄弟块使用相同的变量名
- 内部使用某个父级块中已经存在名称相同的变量

`let`  声明是 `var`  声明的一个替代方案。**`let` 关键字遵循块作用域，而不是默认的词法作用域**。使用 `var` 时，只能通过嵌套函数来创建更深的作用域；但是**使用 `let` 时，只需要通过 `{ }` 就可以创建新的作用域，无需创建新的函数**。所以我在前文中说，在 ES6 支持 `let`  和 `const`  之后，通过 IIFE 创建新的作用域已经成为了历史，IIFE 立即执行函数表达式已经完成了它的历史使命。

```javascript
let top ={}
{
    let inner = {}
    {
        let innermost = {}
    }
    // 在此处访问 innermost 会抛出异常
}
// 在此处访问 inner 会抛出异常
// 在此处访问 innermost 会抛出异常
```

`let` 一个非常有用的用法就是在 for 循环中使用 `let` ，则变量的作用域会封闭在循环体内。这一点和 `var` 声明变量恰恰相反。

```javascript
for (let i=0; i<10; i++) {
  console.log( i );
}
console.log( i ); //ReferenceError: i is not defined
```

<a name="xTLww"></a>
## const
和 ` let` 类似， `const` 声明也遵循块级作用域规则。

```javascript
const pi = 3.1415;
{
  const pi = 6;
  console.log(pi); //6
}
console.log(pi); //3.1415
```
 <br />虽然 `const` 和 `let` 都遵循块级作用域规则，但是也存在很多不同之处。

1. 使用 `const`  声明的变量必须在声明时就进行初始化

```javascript
const pi = 3.1415
const e // Missing initializer in const declaration
```


2. 使用 const 声明的变量无法重复赋值

```javascript
const people = ['song','coder']
 people = [] // Assignment to constant variable.
```

**需要重点注意的是，使用 **`**const**` ** 声明只是意味着所声明的变量会一直持有对同一个对象或基本值的引用，保持不变的只是这个引用，但是引用指向的值并不是不可改变的**。如下代码：

```javascript
const people = ['song','coder']
people.push('focus')
console.log(people) //["song", "coder", "focus"]
```

<br />`**const**` ** 声明只是会禁止变量绑定到一个新的引用**，我们可以使用 `const` 声明一个变量，然后将这个变量赋给一个 var 声明的变量。此时， `var`  声明的变量照常可以赋值其它引用。

```javascript
const people = ['song','coder']
var humans = people
humans = 'coderfocus'
console.log(humans) //coderfocus
```

<br />如果确实想要确保 `const` 变量的值不变，可以使用Object.freeze函数禁止扩展传入的对象，如下所示。

```javascript
const people = Object.freeze(
    ['song','coder']
)
people.push('focus')
//Cannot add property 2, object is not extensible at Array.push (<anonymous>)
```

<a name="mXOSV"></a>
## 暂时性死区
在 JavaScript 提升不完全指北（链接） 一文中，我们介绍了 JavaScript 提升的概念，**提升是指不管变量和函数声明在代码的哪个位置，都会提升到所在作用域的顶部**。如下代码。

```javascript
console.log(a); //undefined
var a = 2;
```

因为存在变量的提升，实际的代码执行流程如下。

```javascript
var a;
console.log(a); //undefined
a = 2;
```

如果我们使用 `let`  或者 `const`  替代 ` var` 声明变量，则会抛出异常。

```javascript
console.log(a); //ReferenceError: a is not defined
let a = 2;
```

```javascript
console.log(a); //ReferenceError: a is not defined
const a = 2;
```

这是因为 `let` 和 `const` 存在暂时性死区，所谓的暂时性死区就是指，**从作用域开始到 let（const） 声明的执行前，访问 let(const) 声明的变量都会报错**。<br />**`let`  和 `const`  声明的变量，也会存在提升，只是因为存在暂时性死区，所以看起来就像没有提升一样**。实际上，暂时性死区是为了 `const`  实现的，为了保持统一，也应用于 `let`  了。因为使用 `const`  声明的变量必须在声明时就进行初始化，如果没有暂时性死区，则可以在 `const`  声明执行前给提升的 `const`  变量赋值。 **暂时性死区就是为了确保只在声明时对 const 进行赋值而实现的**。
