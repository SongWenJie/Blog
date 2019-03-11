## form表单元素

```javascript
<avue-form 
ref="form" 
v-model="formData"  
:option="formOption" 
@reset-change="emptytChange" 
@submit="submit">
```

```javascript
export default {
    name: 'form',
    data() {
      return {
        formData:[],
        formOption:{}
      }
    },
  
    methods: {
      //form
      emptytChange(){
        this.$message.success('清空方法回调');
      },
      submit () {
        this.$message.success('当前数据'+JSON.stringify(this.formData));
      }
    }
  }
```

![mark](http://songwenjie.vip/blog/20190307/DSQS5I7VTvL0.png?imageslim)

`reset-change ` 为清空表单的回调方法，清空表单内容，但是默认值不会被清空。

`v-model` 绑定属性 `formData` 存放表单数据

`submit` 为表单提交的回调方法，用以持久化表单数据等



### Radio 单选框

**在一组备选项中进行单选**

```javascript
formOption:{
  column: [{
    label: "性别",
    prop: "sex",
    span: 6,
    type: "radio",
    dicData: [{ //数据项
      label: '男',
      value: 0
    },{
      label: '女',
      value: 1
    }],
    valueDefault: 0, //默认值
    change:({value,column})=>{
      this.$message.success('value:' + value)
    }
  }]
}
```

`change` 回调方法，返回当前的值 `value` 和列的属性 `column` 

![mark](http://songwenjie.vip/blog/20190307/OD2QiHrakxGQ.png?imageslim)

**禁用选项**，在 `dicData` 相应选项中配置 	`disabled: true`

 ```javascript
formOption:{
  column: [{
    label: "性别",
    prop: "sex",
    span: 6,
    type: "radio",
    dicData: [{ //数据项
      label: '男',
      value: 0
    },{
      label: '女',
      value: 1
    },{
      label: '女',
      value: -1 ,
      disabled: true
    }],
    valueDefault: 0, //默认值
    change:({value,column})=>{
      this.$message.success('value:' + value)
    }
  }]
}
 ```

![mark](http://songwenjie.vip/blog/20190307/cEpFFV6e6JQx.png?imageslim)



### Checkbox 多选框

**一组备选项中进行多选**

```javascript
formOption:{
  column: [{
       label: "权限: ",
       prop: "grade",
       span: 6,
       type: "checkbox",
       dicData: [{
         label: '权限1',
         value: 1
       },{
         label: '权限2',
         value: 2
       },{
         label: '权限3',
         value: 3
       }],
       valueDefault: [1,2], //默认
       row: true //阻止依次排列，独占一行
     }]
   }]
}
```

![mark](http://songwenjie.vip/blog/20190307/TXzYcsvM7Rr7.png?imageslim)

#### 禁用选项

在 `dicData` 相应选项中配置 `disabled: true`

```javascript
formOption:{
  column: [{
       label: "权限: ",
       prop: "grade",
       span: 12,
       type: "checkbox",
       dicData: [{
         label: '权限1',
         value: 1
       },{
         label: '权限2',
         value: 2
       },{
         label: '权限3',
         value: 3
       },{
         label: '无权限',
         value: -1,
         disabled: true
       }],
       valueDefault: [1,2], //默认
       row: true //阻止依次排列，独占一行
     }]
   }]
}
```

![mark](http://songwenjie.vip/blog/20190307/eVascjdbQENO.png?imageslim)

### Input 输入框

**通过鼠标或键盘输入字符**

#### 普通输入框

```javascript
formOption:{
  column: [{
    label: '用户名',
    prop: 'name',
    type: 'input'
  }]
}
```

![mark](http://songwenjie.vip/blog/20190307/Y753aPDQhloo.png?imageslim)

#### 禁用状态

通过 `disabled` 属性指定是否禁用 input 组件

```javascript
formOption:{
  column: [{
    label: '用户名',
    prop: 'name',
    type: 'input',
    disabled: true
  }]
}
```



![mark](http://songwenjie.vip/blog/20190307/tyD8eDFDqxdr.png?imageslim)

#### 可清空

设置 `clearable` 属性为 `true ` 即可得到一个可清空的输入框（ 默认为true ）

```javascript
formOption:{
  column: [{
    label: '用户名',
    prop: 'name',
    type: 'input',
    clearable: true
  }]
}
```

![mark](http://songwenjie.vip/blog/20190307/pG8vRV8orsn0.png?imageslim)

#### 带 icon 的输入框

 `prefix-icon` 属性在 input 组件首部增加显示图标

 `suffix-icon` 属性在 input 组件尾部增加显示图标

```javascript
formOption:{
  column: [{
    label: '用户名',
    prop: 'name',
    type: 'input',
    prefixIcon: 'el-icon-search'
  }]
}
```

![mark](http://songwenjie.vip/blog/20190307/365Kn3sO7UrA.png?imageslim)

```
formOption:{
  column: [{
    label: '用户名',
    prop: 'name',
    type: 'input',
    suffixIcon: 'el-icon-search'
  }]
}
```

![mark](http://songwenjie.vip/blog/20190307/0LAJJXghz5U8.png?imageslim)

#### 复合型输入框

**可前置或后置元素，一般为标签或按钮**

` prepend` 属性在 input 组件前增加标签

 `append` 属性在 input 组件后增加标签

```javascript
formOption:{
  column: [{
    label: '网站',
    prop: 'website',
    type: 'input',
    prepend: 'http://'
  }]
}
```

![mark](http://songwenjie.vip/blog/20190307/S0mnq87I5Ihw.png?imageslim)

```javascript
formOption:{
  column: [{
    label: '网站',
    prop: 'website',
    type: 'input',
    prepend: 'http://',
    append: '.com'
  }]
}
```



![mark](http://songwenjie.vip/blog/20190307/n53NJRqXkVQ8.png?imageslim)

#### 文本域

用于输入多行文本信息，设置 `type` 为 `textarea` 

```javascript
formOption:{
  column: [{
    label: '内容',
    prop: 'content',
    type: 'textarea'
  }]
}
```

![mark](http://songwenjie.vip/blog/20190307/vunVuGWorxi9.png?imageslim)

#### 密码框

用于输入密码信息，设置 `type` 为 `password`

```
formOption:{
  column: [{
    label: '密码',
    prop: 'password',
    type: 'password'
  }]
}
```

![mark](http://songwenjie.vip/blog/20190307/HUacMWzaCL6J.png?imageslim)

### Select 选择器

当选项过多时，使用下拉菜单展示并选择内容。当 Radio 单选框选项比较多时，建议使用 select 选择器。

```javascript
formOption:{
  column: [{
    label: '下拉列表',
    prop: 'select',
    type: 'select',
    dicData: [{
      label: '下拉选项1'
      value: 1
    },{
      label: '下拉选项2'
      value: 2
    },{
      label: '下拉选项3'
      value: 3
    },{
      label: '下拉选项4'
      value: 4
    }]
  }]
}
```

 ![mark](http://songwenjie.vip/blog/20190308/jTnRHClUz7TU.png?imageslim)

#### 禁用状态

设置 `disabled` 属性为 `true`，则整个选择器不可用

```javascript
formOption:{
  column: [{
    label: '下拉列表',
    prop: 'select',
    type: 'select',
    disabled: true,
    dicData: [{
      label: '下拉选项1'
      value: 1
    },{
      label: '下拉选项2'
      value: 2
    },{
      label: '下拉选项3'
      value: 3
    },{
      label: '下拉选项4'
      value: 4
    }]
  }]
}
```

![mark](http://songwenjie.vip/blog/20190308/1mjIJohFhbM5.png?imageslim)

#### 禁用选项

设置 `dicData` 字典中的某一项的 `disabled` 属性为 `true`，则此选项不可用

```javascript
formOption:{
  column: [{
    label: '下拉列表',
    prop: 'select',
    type: 'select',
    dicData: [{
      label: '下拉选项1'
      value: 1
    },{
      label: '下拉选项2'
      value: 2,
      disabled: true
    },{
      label: '下拉选项3'
      value: 3
    },{
      label: '下拉选项4'
      value: 4
    }]
  }]
}
```



![mark](http://songwenjie.vip/blog/20190308/RFS2FAPxgfOI.png?imageslim)



#### 多选

设置 `multiple`  属性为 `true`，可启用多选

```javascript
formOption:{
  column: [{
    label: '下拉列表',
    prop: 'select',
    type: 'select',
    multiple: true
    dicData: [{
      label: '下拉选项1'
      value: 1
    },{
      label: '下拉选项2'
      value: 2
    },{
      label: '下拉选项3'
      value: 3
    },{
      label: '下拉选项4'
      value: 4
    }]
  }]
}
```

![mark](http://songwenjie.vip/blog/20190308/KpykRQew6fBV.png?imageslim)

#### 可搜索

设置  `filterable` 属性为 `true`，可以利用搜索功能快速查找选项

```javascript
formOption:{
  column: [{
    label: '下拉列表',
    prop: 'select',
    type: 'select',
    filterable: true
    dicData: [{
      label: '下拉选项1'
      value: 1
    },{
      label: '下拉选项2'
      value: 2
    },{
      label: '下拉选项3'
      value: 3
    },{
      label: '下拉选项4'
      value: 4
    }]
  }]
}
```

![mark](http://songwenjie.vip/blog/20190308/qhGS6h7dGLM7.png?imageslim)=>![mark](http://songwenjie.vip/blog/20190308/lElKQwjUHLu5.png?imageslim)

### Cascader 级联选择器

当一个数据集合有清晰的层级结构时，可通过级联选择器逐级查看并选择。

设置  `type`属性为 `cascader`，并且在 `dicData` 属性中增加 `children` 下一级选择器属性

```javascript
formOption:{
  column: [{
    label: '编程语言',
    prop: 'cascader',
    type: 'cascader',
    dicData:[{
       label: '前端',
       value: '前端',
       children: [{
         label: 'Html',
         value: 'Html'
       },{
         label: 'Css',
         value: 'Css'
       },{
         label: 'JavaScript',
         value: 'JavaScript'
       }]
     },{
       label: '后端',
       value: '后端',
       children: [{
         label: 'Java',
         value: 'Java'
       },{
         label: 'C#',
         value: 'C#'
       },{
         label: 'Go',
         value: 'Go'
       }]
     }]
  }]
}
```



![mark](http://songwenjie.vip/blog/20190308/HARuwrb03vnP.png?imageslim)

#### 默认选项

设置 `valueDefault` 属性，默认选中此项

```javascript
formOption:{
  column: [{
    label: '编程语言',
    prop: 'cascader',
    type: 'cascader',
    valueDefault: ['前端','JavaScript']
    dicData:[{
       label: '前端',
       value: '前端',
       children: [{
         label: 'Html',
         value: 'Html',
         disabled: true
       },{
         label: 'Css',
         value: 'Css'
       },{
         label: 'JavaScript',
         value: 'JavaScript'
       }]
     },{
       label: '后端',
       value: '后端',
       children: [{
         label: 'Java',
         value: 'Java'
       },{
         label: 'C#',
         value: 'C#'
       },{
         label: 'Go',
         value: 'Go'
       }]
     }]
  }]
}
```

![mark](http://songwenjie.vip/blog/20190308/dnaCw17htvpj.png?imageslim)

#### 禁用选项

设置 `dicData` 字典中的某一项的 `disabled` 属性为 `true`，则此选项不可用

```javascript
formOption:{
  column: [{
    label: '编程语言',
    prop: 'cascader',
    type: 'cascader',
    dicData:[{
       label: '前端',
       value: '前端',
       children: [{
         label: 'Html',
         value: 'Html',
         disabled: true
       },{
         label: 'Css',
         value: 'Css'
       },{
         label: 'JavaScript',
         value: 'JavaScript'
       }]
     },{
       label: '后端',
       value: '后端',
       children: [{
         label: 'Java',
         value: 'Java'
       },{
         label: 'C#',
         value: 'C#'
       },{
         label: 'Go',
         value: 'Go'
       }]
     }]
  }]
}
```

![mark](http://songwenjie.vip/blog/20190308/9Y629il9xTYL.png?imageslim)

#### 可搜索

设置  `filterable` 属性为 `true`，可以利用搜索功能快速查找选项

```javascript
formOption:{
  column: [{
    label: '编程语言',
    prop: 'cascader',
    type: 'cascader',
    filterable: true,
    dicData:[{
       label: '前端',
       value: '前端',
       children: [{
         label: 'Html',
         value: 'Html'
       },{
         label: 'Css',
         value: 'Css'
       },{
         label: 'JavaScript',
         value: 'JavaScript'
       }]
     },{
       label: '后端',
       value: '后端',
       children: [{
         label: 'Java',
         value: 'Java'
       },{
         label: 'C#',
         value: 'C#'
       },{
         label: 'Go',
         value: 'Go'
       }]
     }]
  }]
}
```

![mark](http://songwenjie.vip/blog/20190308/jNHsAVC2SCR8.png?imageslim)

### Switch 开关

表示两种相互对立的状态间的切换，多用于触发「开/关」

设置  `type`属性为 `switch ` 

```javascript
formOption:{
  column: [{
    label: '开关',
    prop: 'switch',
    type: 'switch',
    valueDefault: 0,
    dicData:[{
       label: '关',
       value: 0,
     },{
       label: '开',
       value: 1,
     }]
  }]
}
```



### DateTime 日期

#### 日期

```javascript
formOption:{
  column: [{
    label: '日期',
    prop: 'date',
    type: 'date'
  }]
}
```

![mark](http://songwenjie.vip/blog/20190308/8JVUGH4PN31D.png?imageslim)

#### 日期范围

```javascript
formOption:{
  column: [{
    label: '日期范围',
	prop: 'daterange',
	type: 'daterange',
	startPlaceholder: '开始日期',
	endPlaceholder: '结束日期'
  }]
}
```

![mark](http://songwenjie.vip/blog/20190308/T18eCayjHsn4.png?imageslim)



#### 时间

```javascript
formOption:{
  column: [{
    label: '时间',
    prop: 'time',
    type: 'time'
  }]
}
```

![mark](http://songwenjie.vip/blog/20190311/z2G5cOxkukA6.png?imageslim)

#### 时间范围

```
formOption:{
  column: [{
    label: '时间范围',
	prop: 'timerange',
	type: 'timerange',
	startPlaceholder: '开始时间',
	endPlaceholder: '结束时间',
  }]
}
```

![mark](http://songwenjie.vip/blog/20190311/7BfcD3ak0iCk.png?imageslim)

#### 时间日期

```javascript
formOption:{
  column: [{
    label: '时间日期',
	prop: 'datetime',
	type: 'datetime'
  }]
}
```

![mark](http://songwenjie.vip/blog/20190311/RaeVds4MI11r.png?imageslim)

#### 时间日期范围

```javascript
formOption:{
  column: [{
    label: '时间日期范围',
	prop: 'datetimerange',
	type: 'datetimerange',
	startPlaceholder: '开始时间日期',
	endPlaceholder: '结束时间日期'
  }]
}
```

![mark](http://songwenjie.vip/blog/20190311/sAsdzclMgT6J.png?imageslim)



#### 年

```javascript
formOption:{
  column: [{
    label: '年',
	prop: 'year',
	type: 'year'
  }]
}
```



![mark](http://songwenjie.vip/blog/20190311/muyvAlneBE44.png?imageslim)

#### 月

```javascript
formOption:{
  column: [{
    label: '月',
	prop: 'month',
	type: 'month'
  }]
}
```

![mark](http://songwenjie.vip/blog/20190311/aBkkkLTlfwB3.png?imageslim)

#### 周

```javascript
formOption:{
  column: [{
    label: '周',
	prop: 'week',
	type: 'week',
    format: 'yyyy年WW周'
  }]
}
```

![mark](http://songwenjie.vip/blog/20190311/zPtODeSHk1Sy.png?imageslim)

#### 多个日期

```javascript
formOption:{
  column: [{
    label: '多个日期',
	prop: 'dates',
	type: 'dates'
  }]
}
```

![mark](http://songwenjie.vip/blog/20190311/YqSVgKg5uY15.png?imageslim)

### 多级联动

第一级下拉列表设置联动选项 `cascaderItem: ['city', 'area']`

```javascript
formOption:{
  props: {
    label: 'name',
    value: 'code'
  },
  column: [
    {
      label: '省份',
      prop: 'province',
      type: 'select',
      cascaderItem: ['city', 'area'],
      dicUrl: `/api/area/getProvince`,
    },
    {
      label: '城市',
      prop: 'city',
      type: 'select',
      dicFlag: false,
      dicUrl: `/api/area/getCity/{{key}}`,
    },
    {
      label: '地区',
      prop: 'area',
      dicFlag: false,
      type: 'select',
      dicUrl: `/api/area/getArea/{{key}}`,
    }]
}
```

![mark](http://songwenjie.vip/blog/20190311/AxtmGwHMu1OS.png?imageslim)



### Tree 树型

设置  `type ` 属性为  `tree` 

可以设置 `parent `属性来控制父类是否可以勾选，默认为`true`

`defaultExpandAll` 属性是否展开全部树形结构，默认`true`

`multiple`设置是否为多选

```javascript
  const dic = [{
    name: '全栈',
    value: '全栈',
    children: [{
      name: '前端',
      value: '前端',
      children: [{
        name: 'JavaScript',
        value: 'JavaScript'
      },{
        name: 'CSS',
        value: 'CSS'
      },{
        name: 'Html',
        value: 'Html'
      }]
    },{
      name: '后端',
      value: '后端',
      children: [{
        name: 'C#',
        value: 'C#'
      },{
        name: 'Java',
        value: 'Java'
      },{
        name: 'Go',
        value: 'Go'
      }]
    }]
  }]
```

#### 多选

设置 `multiple` 多选属性为 `true`

```javascript
formOption:{
  column: [{
    label: '多选',
	prop: 'multiple',
    type: 'tree',
    multiple: true,
    dicData: dic,
    props:{
      label: 'name'
    }
  }]
}
```

![mark](http://songwenjie.vip/blog/20190311/nNWrzXJX1Ima.png?imageslim)

#### 单选

设置 `multiple` 多选属性为 `false`

```javascript
formOption:{
  column: [{
    label: '多选',
	prop: 'single',
    type: 'tree',
    multiple: false,
    dicData: dic,
    props:{
      label: 'name'
    }
  }]
}
```

![mark](http://songwenjie.vip/blog/20190311/005c5XTtHnHY.png?imageslim)



### Number 计数器

设置  `type ` 属性为  `number `

```javascript
formOption:{
  column: [{
    label: '计数器',
	prop: 'number',
	type: 'number'
  }]
}
```

![mark](http://songwenjie.vip/blog/20190311/9qqtLInPaEsX.png?imageslim)

#### 精度

设置  `precision`  属性接收一个  `Number` 控制数值精度，默认为0

```javascript
formOption:{
  column: [{
    label: '计数器',
	prop: 'number',
	type: 'number',
	precision:2
  }]
}
```

![mark](http://songwenjie.vip/blog/20190311/6t5w7ulwF7Cb.png?imageslim)

#### 最大值、最小值

使用 `minRows` 属性设置最小值； `maxRows` 属性设置最大值。

```javascript
formOption:{
  column: [{
    label: '计数器',
	prop: 'number',
	type: 'number',
	minRows: 0,
	maxRows: 3
  }]
}
```

![mark](http://songwenjie.vip/blog/20190311/Qmwx8GizJeCV.png?imageslim)![mark](http://songwenjie.vip/blog/20190311/G2gwpbLYsyrC.png?imageslim)

#### 按钮位置

设置 `controls-position` 属性可以控制按钮位置。

```javascript
formOption:{
  column: [{
    label: '计数器',
	prop: 'number',
	type: 'number',
	controlsPosition: 'left'
  }]
}
```

![mark](http://songwenjie.vip/blog/20190311/BKd3Evqs2pfa.png?imageslim)

```javascript
formOption:{
  column: [{
    label: '计数器',
	prop: 'number',
	type: 'number',
	controlsPosition: 'right'
  }]
}
```

![mark](http://songwenjie.vip/blog/20190311/pSbk4ERxff2e.png?imageslim)