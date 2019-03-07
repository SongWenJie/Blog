** 本文主要包含 avue表格 以下内容：**

- 面板
- 分页
- 排序
- 多选
- 筛选
- 搜索

### # 面板

**配置项：**

```javascript
expand:true,
//行数据的主键
idKey:'bookId',
```

**vue文件：**

`solt` 的名称为 `expand` 时即可，`props `为返回需要的数据，里面包括了当前行的相关数据

```javascript
<avue-crud ref="crud"
           :data="tableData"
           :option="tableOption"
           @on-load="getList">

  <template slot-scope="props" slot="expand">
    <el-tag>当前行的{{props}}</el-tag>
  </template>
  
</avue-crud>
```

![mark](http://songwenjie.vip/blog/20190305/8LQwN5B8yh3s.png?imageslim)



**`defaultExpandAll` 属性默认展开全部**

```javascript
expand:true,
//行数据的主键
idKey:'bookId',
//默认展开全部
defaultExpandAll: true
```

![mark](http://songwenjie.vip/blog/20190305/UQieSYw0jFit.png?imageslim)



### # 分页

首次加载调用 `on-load` 方法加载数据，调用网络接口获取分页信息，关键是要更新总页数 `page.total` 和分页数据 `data`	

**vue 文件：**

```javascript
<avue-crud
  :data="data"
  :option="option"
  :page="page"
  @on-load="onLoad">
</avue-crud>
```

**配置项：**

```javascript
export default {
  name: 'book',
  data() {
    return {
      data: [],//数据  
      page: {
        total: 0, // 总页数
        currentPage: 1, // 当前页数
        pageSize: 20 // 每页显示多少条
      },
      tableLoading: false,
      option: {...}//table配置项
    }
  },
  methods: {
    onLoad(page,params) {
      //等待
      this.tableLoading = true
      //异步发起网络请求
      fetchList(Object.assign({
        current: page.currentPage,
        size: page.pageSize
      })).then(response => {
        //更新数据  
        this.data = response.data.data.records
        //更新总页数
        this.page.total = response.data.data.total
        //取消等待
        this.tableLoading = false
      })
    }
  }
}
```

![mark](http://songwenjie.vip/blog/20190305/hGfTJrXCT7VF.png?imageslim)

### # 排序

**vue 文件：**

```javascript
<avue-crud 
:data="data" 
:option="option" 
@sort-change="sortChange">
</avue-crud>
```

**配置项：**

设置 `column` 数组对象中的属性 `sortable: true`

```javascript
column: [
  {
    label: '书籍名称',
    prop: 'bookName',
    sortable: true //排序
  }
]
```

![mark](http://songwenjie.vip/blog/20190305/eDNzIXHhOOFN.png?imageslim)

**回调方法 `sort-change` 返回排序后的数据，可以利用其搞点事情**

```javascript
methods: {
	sortChange(val){
		this.$message.success('排序'+ JSON.stringify(val));
	}
}
```



### # 多选

**vue文件：**

```javascript
<avue-crud ref="crud" 
:data="data" 
:option="option" 
@selection-change="selectionChange">
</avue-crud>
```

**配置项：**

设置属性selection为true

```javascript
selection: true
```

![mark](http://songwenjie.vip/blog/20190306/pHf3VFC4yEKe.png?imageslim)

**回调方法 `selection-change` 返回选中的数据**

```javascript
methods: {
	selectionChange(list){
		this.$message.success('选中'+ JSON.stringify(list));
	}
}
```

![mark](http://songwenjie.vip/blog/20190306/yqdmde7B0Tl6.png?imageslim)



### # 筛选

**vue 文件：**

```javascript
<avue-crud 
:data="data" 
:option="option">
</avue-crud>
```

**配置项：**

设置 `column` 数组对象中的 `filters` 为筛选的字典，`filterMethod `为自定义的筛选逻辑

```javascript
column: [
    {
      label: '评价',
      prop: 'evaluation',
      filters: [{ text: '5', value: 5 },{ text: '4', value: 4 }],
      filterMethod:  function (value,row,column) {
        return row.evaluation === value;
      }
    }
]
```

![mark](http://songwenjie.vip/blog/20190306/AKXB75N6ClHc.png?imageslim)

设置 `filterMultiple` 筛选的数据为多选还是单选，默认为 true ，支持多选。可以通过设置 `filterMultiple` 为 false 单选。

```javascript
column: [
    {
      label: '评价',
      prop: 'evaluation',
      filters: [{ text: '5', value: 5 },{ text: '4', value: 4 }],
      filterMultiple: false, 
      filterMethod:  function (value,row,column) {
        return row.evaluation === value;
      }
    }
]
```

![mark](http://songwenjie.vip/blog/20190306/dRuGJIqHx8F2.png?imageslim)



### # 搜索

**vue 文件：**

```javascript
<avue-crud 
:data="data" 
:option="option"  
@search-change="searchChange">
</avue-crud>
```

**配置项：**

配置 `column` 数组对象中的 `search` 属性为 `true`

```javascript
 column: [
    {
      label: '书籍名称',
      prop: 'bookName',
      search: true
    }
 ]
```

**点击搜索功能回调 `search-change` 方法，返回搜索的参数**

```javascript
methods: {
	searchChange(params){
		this.$message.success('参数：'+ JSON.stringify(params));
	}
}
```

搜索常和分页功能一起使用，一般的做法是在搜索回调方法中调用分页回调方法

```
methods: {
	searchChange(params){
		this.onLoad(this.page, params);
	}
}
```

![mark](http://songwenjie.vip/blog/20190306/XhPKyzCOxWqa.png?imageslim)

通过 `searchDefault` 设置表格搜索的默认值

```javascript
 column: [
    {
      label: '书籍名称',
      prop: 'bookName',
      search: true,
      searchDefault: '见识'
    }
 ]
```

![mark](http://songwenjie.vip/blog/20190306/AJxoXvfjyjk6.png?imageslim)