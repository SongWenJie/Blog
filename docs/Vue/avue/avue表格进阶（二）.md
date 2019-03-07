** 本文主要包含 avue表格 以下内容: **
- 行操作
- 合计
- 合并行列

###  # 行操作

#### 行单击

```javascript
<avue-crud 
:data="data" 
:option="option" 
@row-click="handleRowClick">
</avue-crud>
```

单击回调方法 `row-click` 同时返回当前行的 `row `数据,`event` 当前的操作对象,`column` 当前列的属性。

```javascript
methods: {
      handleRowClick(row,event,column){
        this.$message.success(JSON.stringify(row))
      }
}
```

![mark](http://songwenjie.vip/blog/20190306/RHXoU3eMq7rL.png?imageslim)

#### 行双击

```javascript
<avue-crud 
:data="data" 
:option="option" 
@row-dblclick="handleRowDBLClick">
</avue-crud>
```

与行单击一样，行双击回调方法 `row-dblclick` 同时返回当前行的 `row `数据,`event` 当前的操作对象,`column` 当前列的属性。

![mark](http://songwenjie.vip/blog/20190306/tNNNT4dAG4dx.png?imageslim)

```javascript
methods: {
      handleRowDBLClick(row,event,column){
        this.$message.success(JSON.stringify(row))
      }
}
```

#### 行单选

```javascript
<avue-crud 
ref="crud" 
:data="data" 
:option="option0" 
@current-row-change="handleCurrentRowChange">
</avue-crud>
```

回调方法 `current-row-change` ,返回当前行的`row`数据

```
methods: {
      handleCurrentRowChange(row){
        this.$message.success(JSON.stringify(row))
      }
}
```

可以通过代码 `this.$refs.crud.setCurrentRow(row)` 选中某一行，`row` 为行数据。



### # 合计

设置 `showSummary` 为 `true` 就会在表格尾部展示合计行

![mark](http://songwenjie.vip/blog/20190307/QOsAuYH2AoaC.png?imageslim)

可以通过`sumText` 属性自定义合计行文案

```javascript
showSummary: true,
sumText: '自定义合计'
```

![mark](http://songwenjie.vip/blog/20190307/NtOhAV8fLMwv.png?imageslim)

配置 `sumColumnList` 数组你要显示的统计的字段，有 `name` 和 `type` 两个属性，`name` 表示要显示统计的字段，`type` 表示统计的类型， 可以为 sum 相加|avg 平均值|count 计数。

以书籍价格列为例：

#### 相加 sum

```javascript
showSummary: true,
sumText: '自定义合计',
sumColumnList: [
  {
    name: 'price',
    type: 'sum'
  }
]
```

![mark](http://songwenjie.vip/blog/20190307/va69GBBLepmp.png?imageslim)



#### 平均值 avg

```javascript
showSummary: true,
sumText: '自定义合计',
sumColumnList: [
  {
    name: 'price',
    type: 'avg'
  }
]
```

![mark](http://songwenjie.vip/blog/20190307/3WtCPshPRuOi.png?imageslim)

#### 计数 count

```javascript
showSummary: true,
sumText: '自定义合计',
sumColumnList: [
  {
    name: 'price',
    type: 'count'
  }
]
```

![mark](http://songwenjie.vip/blog/20190307/bubLkEpFR7Gi.png?imageslim)



### 合并行列

通过给传入 `span-method` 属性绑定的方法可以实现合并行或列，方法的参数是一个对象，里面包含当前行`row `、当前列 `column` 、当前行号 `rowIndex` 、当前列号 `columnIndex` 四个属性。

该函数可以返回一个包含两个元素的数组，第一个元素代表 `rowspan` ，第二个元素代表 `colspan` ：

`[rowspan,colspan]`  代表几行几列，例如 [1,2] 表示1行2列，进行了列合并；[2,1] 表示2行1列，进行了行合并； [0,0] 表示0行0列，不显示此行此列。

 也可以返回一个键名为`rowspan`和`colspan`的对象：

```javascript
{
    rowspan:1,
    colspan:2
}
```



#### 合并列

```javascript
<avue-crud
  :data="data1"
  :option="option2"
  :span-method="mergeRow"
></avue-crud>
```

```javascript
methods: {
  mergeRow({ row, column, rowIndex, columnIndex }){
    if(rowIndex % 2 === 0){ //如果为偶数行
      if(columnIndex === 0){
        return [1,2] //如果为第1列，则跨2列
      }else if(columnIndex === 1){
        return [0,0] //如果为第2列，则不显示
      }
    }
  }
}
```

![mark](http://songwenjie.vip/blog/20190307/au23BTPO3j2g.png?imageslim)



#### 合并行

```javascript
<avue-crud
  :data="data1"
  :option="option2"
  :span-method="mergeCol"
></avue-crud>
```

````javascript
methods: {
  mergeCol({row,column,rowIndex,columnIndex}){
    if(columnIndex === 2){ //如果为第3列
      if(rowIndex % 2 === 0){ //如果为偶数行，则跨2行
        return {
          rowspan:2,
          colspan:1
        }
      }else{
        return {
          rowspan:0,
          colspan:0
        }
      }
    }
  }
}
````

![mark](http://songwenjie.vip/blog/20190307/5aBU9xcCEM4H.png?imageslim)
