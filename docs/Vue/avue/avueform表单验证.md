对于 web 开发来说，表单验证是永远跨不过去的坎。一般来说，客户端和服务端都需要进行表单验证。客户端验证是为了防止每次验证都发起网络请求，影响用户使用体验；但是也需要在表单提交时进行服务端验证，因为客户端验证是不可靠的，用户可以在浏览器跳过客户端验证直接提交。

对于客户端表单验证，通常要通过编写大量的重复代码实现。本文研究 `avue` 框架如何基于配置进行表单验证。`avue` 是基于 `element-ui` 做了一层封装，而 `element-ui`  的表单验证基于 `async-validator`  [async-validator 项目地址](https://github.com/yiminghe/async-validator) 。

`async-validator` 支持以下验证类型(type)：

- `string`：必须是类型`string`。`This is the default type.`

- `number`：必须是类型`number`。

- `boolean`：必须是类型`boolean`。

- `method`：必须是类型`function`。

- `regexp`：必须是`RegExp`创建新项时不生成异常的实例或字符串`RegExp`。

- `integer`：必须是类型`number`和整数。

- `float`：必须是类型`number`和浮点数。

- `array`：必须是由...确定的数组`Array.isArray`。

- `object`：必须是类型`object`而不是`Array.isArray`。

- `enum`：价值必须存在于`enum`。

- `date`：值必须有效，由确定 `Date`

- `url`：必须是类型`url`。

- `hex`：必须是类型`hex`。

- `email`：必须是类型`email`。


结合 `avue`、`element-ui`、`async-validator` 项目文档， 总结了以下常用的表单验证规则：

### 必填必选

使用 `required`  属性定义需要。

#### 必填

```javascript
rules:[{
  required: true,
  message: "请输入活动名称",
  trigger: "blur"
}]
```

![mark](http://songwenjie.vip/blog/20190313/5lnyTYlGQgkM.png?imageslim)

#### 必选

````javascript
rules:[{
  type: 'array',
  required: true,
  message: "请选择活动性质",
  trigger: 'change'
}]
````

![mark](http://songwenjie.vip/blog/20190313/tEA7FAJxOzO0.png?imageslim)

![mark](http://songwenjie.vip/blog/20190313/52eUxRA6lLlm.png?imageslim)

### 范围

使用 `min` 和 `max` 属性定义范围。

对于`string`和`array`类型进行比较`length`，对于`number`类型，数量不得小于`min`或大于`max`。

```javascript
rules: [{
    type: 'string',
    message: '字符串长度必选在5-10位',
    min: 5,
    max: 10,
    trigger: 'blur'
}]
```

![mark](http://songwenjie.vip/blog/20190313/IXLR4I8tJVo8.png?imageslim)![mark](http://songwenjie.vip/blog/20190313/TU4EIJ83RTSN.png?imageslim)

### 长度

使用 `len` 属性验证字段的确切长度。

对于 `string` 和 `array` 类型比较`length`，对于 `number` 类型，此属性指示完全匹配`number`，即，它可能仅严格等于`len`。

如果 `len ` 属性与 `min `和`max `范围属性组合，`len ` 则优先。

```javascript
rules: [{
  type: 'string',
  message: '字符串长度必须为5位',
  len: 5,
  trigger: 'blur'
}]
```

![mark](http://songwenjie.vip/blog/20190313/yv0GO0POpSRt.png?imageslim)![mark](http://songwenjie.vip/blog/20190313/e4Oe0yXpGsea.png?imageslim)

### 邮箱

邮箱 `email` 是 `async-validator` 内部支持的验证类型。

```java
rules: [{
  type: 'email',
  message: '请输入正确的邮箱地址',
  trigger: 'blur'
}]
```

![mark](http://songwenjie.vip/blog/20190313/Rt367lIRFaB6.png?imageslim)![mark](http://songwenjie.vip/blog/20190313/eQTgSyy1YyTS.png?imageslim)

### 正则表达式

`pattern ` 规则属性指示一个正则表达式的值必须匹配, 才能通过验证。

```javascript
rules: [{
  required: true,
  pattern: /^1[34578]\d{9}$/,
  message: '请输入正确的手机号',
  trigger: 'blur'
}]
```

![mark](http://songwenjie.vip/blog/20190313/9EOS0Cdp8Hka.png?imageslim)![mark](http://songwenjie.vip/blog/20190313/ftLNEXsU1uIy.png?imageslim)

### 多规则验证

在实际业务中，一个字段经常会有多个验证规则。比如，必填且长度为5.

```javascript
rules: [{
  required: true,
  message: '请输入内容',
  trigger: 'blur'
},{
  len: 5,
  message: '内容长度必须为5位',
  trigger: 'blur'
}]
```

![mark](http://songwenjie.vip/blog/20190313/uxkSK3oQGNTN.png?imageslim)![mark](http://songwenjie.vip/blog/20190313/KOb7x7ASpLB3.png?imageslim)

### 自定义验证

实际开发中，会有一些需要自定义（定制）的验证。比如密码的再次确认。

使用 `validator` 属性指向自定义的验证方法，便可以使用自定义验证。

```javascript
export default {
    data() {
      var validatePass = (rule, value, callback) =>{
        if (value === '') {
          callback(new Error('请输入密码'));
        } else {
          callback();
        }
      };
      var validatePassAgain = (rule, value, callback) => {
        if (value === '') {
          callback(new Error('请再次输入密码'));
        } else if (value !== this.formData.password) {
          callback(new Error('两次输入密码不一致!'));
        } else {
          callback();
        }
      };
      return {
        formData:{
        },
        formOption:{
          column:[{
            label:'密码',
            prop:'password',
            hide:true,
            rules: [{ 
                validator: validatePass, 
                trigger: 'blur' 
            }]
          }, {
            label:'确认密码',
            prop:'confirmPassword',
            hide:true,
            rules: [{ 
                validator: validatePassAgain, 
                trigger: 'blur' 
            }]
         }
      }
    },
    methods: {
    }
}
```

![mark](http://songwenjie.vip/blog/20190313/FETPC1v156LQ.png?imageslim)![mark](http://songwenjie.vip/blog/20190313/qtIidk4pSz9a.png?imageslim)![mark](http://songwenjie.vip/blog/20190313/5gc0kOqToerx.png?imageslim)