**avue是一个配置化前端框架，一切基于配置**

### 表格基本配置

使用 `<avue-crud :data="data" :option="option"></avue-crud>` 构建表格

`data` 为数据的对象数组，`option` 为表格要配置的数据列

因为  `option`  为表格要配置的数据列,当表格列数比较多的时候，配置项也会急剧膨胀，配置项和业务代码混杂在一起，也不利于项目代码的可读性和可维护性。为了解决这个问题，可以将配置代码和业务代码分离。

业务代码：

![mark](http://songwenjie.vip/blog/20190305/iaP30z5vJvSb.png?imageslim)![mark](http://songwenjie.vip/blog/20190305/41q9M4ScVIFf.png?imageslim)

配置代码：

![mark](http://songwenjie.vip/blog/20190305/p4UssYwJkk07.png?imageslim)

### 丰富表格选项

我们还可以通过配置丰富表格选项：

**显示边框：**

`border: true`

![mark](http://songwenjie.vip/blog/20190305/2uguhsFXDFHw.png?imageslim)

**显示斑马纹：**

`stripe: true`

![mark](http://songwenjie.vip/blog/20190305/A4BBfULpycno.png?imageslim)

**显示索引：**

`index: true,
indexLabel: '序号'`

![mark](http://songwenjie.vip/blog/20190305/2VFGUinvWeHw.png?imageslim)

**显示多选框：**

`selection: true`

![mark](http://songwenjie.vip/blog/20190305/THTzWhweNJwp.png?imageslim)

**显示表头：**

`showHeader: true`

![mark](http://songwenjie.vip/blog/20190305/GxAtvpDJYfGS.png?imageslim)

**复杂表头：**

```javascript
column: [
    {
      label: '名称及作者',
      prop: 'nameAndAuthor',
      children:[
        {
          label: '书籍名称',
          prop: 'bookName'
        },
        {
          label: '作者',
          prop: 'author'
        }
      ]
    }
]
```

![mark](http://songwenjie.vip/blog/20190305/Uv06akIKXtuB.png?imageslim)