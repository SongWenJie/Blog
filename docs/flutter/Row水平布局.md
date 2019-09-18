# Row水平布局

## Row水平布局
子元素 children 的排列方式由这两个属性决定 textDirection 和 verticalDirection 。

verticalDirection时水平方向的排列方式。当值为down（向下排列）时起始位置在顶部。当值为up（向上排列）是起始位置在底部。

### textDirection: children在水平方向的摆放顺序 
textDirection决定水平方向的排列方式：`TextDirection.ltr`从左往右排列（把左当作起始位置），`TextDirection.rtl`从右往左排列（把右当作起始位置）。
#### ltr
textDirection: TextDirection.ltr

```dart
Widget _buildContainer() {
    var container = Container(
      child: Row(
        textDirection: TextDirection.ltr,
        children: <Widget>[
          Container(
            color: Colors.red,
            height: 100,
            width: 100,
          ),
          Container(
            color: Colors.blue,
            height: 50,
            width: 100,
          ),
          Container(
            color: Colors.black,
            height: 75,
            width: 75,
          )
        ],
      ),
    );
    return container;
  }
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568886821155-752b1da3-c037-48cf-a812-9f453cf5dd52.png#align=left&display=inline&height=427&name=image.png&originHeight=854&originWidth=514&size=43202&status=done&width=257)

#### rtl
textDirection: TextDirection.rtl

```dart
  Widget _buildContainer() {
    var container = Container(
      child: Row(
        textDirection: TextDirection.rtl,
        children: <Widget>[
          Container(
            color: Colors.red,
            height: 100,
            width: 100,
          ),
          Container(
            color: Colors.blue,
            height: 50,
            width: 100,
          ),
          Container(
            color: Colors.black,
            height: 75,
            width: 75,
          )
        ],
      ),
    );
    return container;
  }
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568887017320-a84cb93b-1fe8-49df-a9ca-ada7431e2adb.png#align=left&display=inline&height=425&name=image.png&originHeight=850&originWidth=513&size=39919&status=done&width=256.5)
### mainAxisAlignment: 主轴排列方式
对于 Row 主轴（mainAxis）是x轴，交叉轴（crossAxis）是y轴。

![](https://cdn.nlark.com/yuque/0/2019/webp/291118/1568887274469-f72c5160-21e6-4cc4-a2eb-cb4983fdd2c8.webp#align=left&display=inline&height=422&originHeight=422&originWidth=804&size=0&status=done&width=804)
#### start (默认值)
将children放置在**主轴的起点**，根据textDirection属性排列水平方向的摆放顺序。

- MainAxisAlignment.start & TextDirection.ltr
```dart
textDirection: TextDirection.ltr,
mainAxisAlignment: MainAxisAlignment.start,
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568887735936-f6340112-76c0-4c10-ae91-1db42cb51ddb.png#align=left&display=inline&height=423&name=image.png&originHeight=845&originWidth=510&size=35255&status=done&width=255)

- MainAxisAlignment.start & TextDirection.rtl
```dart
textDirection: TextDirection.rtl,
mainAxisAlignment: MainAxisAlignment.start,
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568887921852-30b181e9-3614-48f7-855e-faff466073a0.png#align=left&display=inline&height=424&name=image.png&originHeight=848&originWidth=510&size=35057&status=done&width=255)

#### end
将children放置在**主轴的终点**，根据textDirection属性排列水平方向的摆放顺序。

- MainAxisAlignment.end & TextDirection.ltr
```dart
textDirection: TextDirection.ltr,
mainAxisAlignment: MainAxisAlignment.end,
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568888295129-a3080343-54c6-4b97-916a-c0337cc84330.png#align=left&display=inline&height=424&name=image.png&originHeight=848&originWidth=510&size=35793&status=done&width=255)

- MainAxisAlignment.end & TextDirection.rtl
```dart
textDirection: TextDirection.rtl,
mainAxisAlignment: MainAxisAlignment.end,
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568888325627-7a2fe14e-8349-4fd4-a55a-4908ca22fc86.png#align=left&display=inline&height=424&name=image.png&originHeight=847&originWidth=510&size=35509&status=done&width=255)

#### center
将children放置在**主轴的中间**，根据textDirection属性排列水平方向的摆放顺序。

- MainAxisAlignment.center & TextDirection.ltr
```dart
textDirection: TextDirection.ltr,
mainAxisAlignment: MainAxisAlignment.center,
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568892519713-57dcf4ed-da7f-4441-a539-cb75edb270f4.png#align=left&display=inline&height=424&name=image.png&originHeight=848&originWidth=510&size=35762&status=done&width=255)

- MainAxisAlignment.center & TextDirection.rtl
```dart
textDirection: TextDirection.rtl,
mainAxisAlignment: MainAxisAlignment.center,
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568892586845-84d6d5fd-a355-40fc-a02e-698740cd940f.png#align=left&display=inline&height=424&name=image.png&originHeight=847&originWidth=510&size=34553&status=done&width=255)

#### spaceBetween
将主轴方向上的**空白区域均分**，使得children之间的空白区域相等，**首尾child都靠近首尾，没有间隙**；根据textDirection属性排列水平方向的摆放顺序。

- MainAxisAlignment.spanBetween & TextDirection.ltr
```dart
textDirection: TextDirection.ltr,
mainAxisAlignment: MainAxisAlignment.spanBetween,
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568892901746-8ed5981b-8043-419d-9718-ed05dee27eed.png#align=left&display=inline&height=424&name=image.png&originHeight=848&originWidth=510&size=35497&status=done&width=255)

- MainAxisAlignment.spanBetween & TextDirection.rtl
```dart
textDirection: TextDirection.rtl,
mainAxisAlignment: MainAxisAlignment.spanBetween,
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568892962923-07663d7a-5741-44a0-b15c-5e8bf835f1e5.png#align=left&display=inline&height=424&name=image.png&originHeight=847&originWidth=510&size=35329&status=done&width=255)

#### spaceAround
将主轴方向上的**空白区域均分**，使得children之间的空白区域相等，但是**首尾空白区域为一半**；根据textDirection属性排列水平方向的摆放顺序。

- MainAxisAlignment.spaceAround& TextDirection.ltr
```dart
textDirection: TextDirection.ltr,
mainAxisAlignment: MainAxisAlignment.spaceAround,
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568893389013-5d835fce-4acc-42c0-ae20-7ba307291195.png#align=left&display=inline&height=424&name=image.png&originHeight=847&originWidth=510&size=35174&status=done&width=255)

- MainAxisAlignment.spaceAround& TextDirection.rtl
```dart
textDirection: TextDirection.rtl,
mainAxisAlignment: MainAxisAlignment.spaceAround,
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568893439279-aacfda63-e494-4c7f-8652-ef20790658d9.png#align=left&display=inline&height=424&name=image.png&originHeight=847&originWidth=510&size=35676&status=done&width=255)


#### spaceEvenly
将主轴方向上的**空白区域均分**，使得**children之间的空白区域相等，包括首尾空白区域**；根据textDirection属性排列水平方向的摆放顺序。

- MainAxisAlignment.spaceEvenly& TextDirection.ltr
```dart
textDirection: TextDirection.ltr,
mainAxisAlignment: MainAxisAlignment.spaceEvenly,
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568893904618-000c810d-c97f-4f27-9705-634521856ee3.png#align=left&display=inline&height=424&name=image.png&originHeight=848&originWidth=510&size=35051&status=done&width=255)

- MainAxisAlignment.spaceEvenly& TextDirection.rtl
```dart
textDirection: TextDirection.rtl,
mainAxisAlignment: MainAxisAlignment.spaceEvenly,
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568894435064-ce3f7163-e3a8-4236-aad1-286cf877659e.png#align=left&display=inline&height=424&name=image.png&originHeight=847&originWidth=510&size=35756&status=done&width=255)


