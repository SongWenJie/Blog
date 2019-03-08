### # 字典

日常开发中经常会遇到这样的场景，为了减少数据库存储信息的大小，会对业务中的一些概念进行编码后再存储的到数据库。比如电商系统中的评价星数1星到5星，在数据库中存储的是数字1-5；性别男女在数据库中用0和1进行存储。当进行页面展示时，需要进行一次转换，将编码转换为业务概念进行展示。

对于这种场景，avue 分别支持**本地字典**和**后台接口字典**进行转换。简单的转换可以使用本地字典进行，比如男女，是否。对于复杂的或者可能会发生变化的转化，应该使用后台接口字典的方式。

#### 本地字典

本地字典只要设置 `column` 数组对象中的属性`dicData`为一个`Array`数组即可，但是要注意返回的**字典中value类型和数据的类型必须要对应**。

```javascript
  column:[
      {
      label: '评价',
      prop: 'evaluation',
      type: 'select',
      dicData: [
        {
          label: '五星', value: 5
        },
        {
          label: '四星', value: 4
        },
        {
          label: '三星', value: 3
        },
        {
          label: '二星', value: 2
        },
        {
          label: '一星', value: 1
        }]
     }
  ]
```

![mark](http://songwenjie.vip/blog/20190306/iBCWFg5I1x4B.png?imageslim)

此处应该注意属性 `type` ,该属性规定了在 form 表单中此字段对应的元素类型，常用的有以下几种：

1. **下拉列表 `type: select`**

   ![mark](http://songwenjie.vip/blog/20190306/yOcEnfsSn0gu.png?imageslim)

2. **级联下拉列表 `type:cascader`**

   ```javascript
     column: [
       {
         label: '分类',
         prop: 'category',
         type: 'cascader',
         dicData: [{
           label: 'A',
           value:  '00',
           children: [
             {
               label: 'A1',
               value: '01'
             },
             {
               label: 'A2',
               value: '02'
             }
           ]
         },{
           label: 'B',
           value:  '10',
           children: [
             {
               label: 'B1',
               value: '11'
             },
             {
               label: 'B2',
               value: '12'
             }
           ]
         }]
       }
     ]
   ```

   ```javascript
     export default {
       name: 'book',
       data() {
         return {
           data: [{
               "bookId":1,
               "bookName":"江湖不远",
               "press":"学林出版社",
               "author":"鲍鹏山",
               "evaluation":5,
               "category":['00','01']
             },{
               "bookId":2,
               "bookName":"见识",
               "press":"中信出版社",
               "author":"吴军",
               "evaluation":5,
               "category":['00','02']
             },{
               "bookId":3,
               "bookName":"态度",
               "press":"中信出版社",
               "author":"吴军",
               "evaluation":4,
               "category":['10','11']
             }]
         }
       }
   }
   ```

   ![mark](http://songwenjie.vip/blog/20190306/XadBTsIKUfDK.png?imageslim)

3. 多选框 `type: checkbox`

   ```javascript
     column: [
      {
         label: '多选项',
         prop: 'checkbox',
         type: 'checkbox',
         dicData: [
           {
             label: '权限1',
             value: 1
           },
           {
             label: '权限2',
             value: 2
           },
           {
             label: '权限3',
             value: 3
           },
           {
             label: '禁止项',
             disabled:true,
             value: -1
           },
         ]
       }
     ]
   ```

   ```javascript
     export default {
       name: 'book',
       data() {
         return {
           data: [{
               "bookId":1,
               "bookName":"江湖不远",
               "press":"学林出版社",
               "author":"鲍鹏山",
               "evaluation":5,
               "checkbox":[1,2]
             },{
               "bookId":2,
               "bookName":"见识",
               "press":"中信出版社",
               "author":"吴军",
               "evaluation":5,
               "checkbox":[2,3]
             },{
               "bookId":3,
               "bookName":"态度",
               "press":"中信出版社",
               "author":"吴军",
               "evaluation":4,
               "checkbox":[1,3]
             }]
         }
       }
   }
   ```

   ![mark](http://songwenjie.vip/blog/20190306/NY8CJjkujpm4.png?imageslim)

   4. 树型 `type: tree`

      ```javascript
       column: [
         {      
            label: '树型',
            prop: 'tree',
            type: 'tree',
            dicData: [
              {
                label: '山东省',
                value: '370000',
                children:[{
                  label: '济南市',
                  value: '370100'
                },{
                  label: '日照市',
                  value: '371100'
                }]
              },{
                label: '江苏省',
                value: '320000',
                children:[{
                  label: '南京市',
                  value: '320100'
                },{
                  label: '徐州市',
                  value: '370300'
                }]
              }
            ]    
          }
        ]
      ```

      ```javascript
        export default {
          name: 'book',
          data() {
            return {
              data: [{
                  "bookId":1,
                  "bookName":"江湖不远",
                  "press":"学林出版社",
                  "author":"鲍鹏山",
                  "evaluation":5,
                  "tree":'320000'
                },{
                  "bookId":2,
                  "bookName":"见识",
                  "press":"中信出版社",
                  "author":"吴军",
                  "evaluation":5,
                  "tree":'370100'
                },{
                  "bookId":3,
                  "bookName":"态度",
                  "press":"中信出版社",
                  "author":"吴军",
                  "evaluation":4,
                  "tree":'371100'
                }]
            }
          }
      }
      ```


   ![mark](http://songwenjie.vip/blog/20190306/mWWAKBcz9pmB.png?imageslim)

#### 后台接口字典

avue 支持给每一列属性单独配置网络字典,配置 `column` 数组对象中的属性`dicUrl`

```
  column:[
      {
      label: '评价',
      prop: 'evaluation',
      type: 'select',
      dicUrl： 'http://127.0.0.1:9998/book/book/getData'
     }
  ]
```

**默认的 key-value 为 label-value, props可以配置 key-value 的值**

```javascript
  column:[
      {
      label: '评价',
      prop: 'evaluation',
      type: 'select',
      dicUrl： 'http://127.0.0.1:9998/book/book/getData',
      props: {
        label: 'text',
        value: 'val'
      }
     }
  ]
```

接口返回的数据：

```json
[{"text":"五星","val":5},{"text":"四星","val":4},{"text":"三星","val":3},{"text":"二星","val":2},{"text":"一星","val":1}]
```