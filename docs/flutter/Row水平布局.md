# Row 水平布局

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

### crossAxisAlignment: 交叉轴的排列方式
主轴在此处都以 textDirection: TextDirection.ltr, mainAxisAlignment: MainAxisAlignment.start 为例。

verticalDirection 是垂直方向的排列方式。当值为down（向下排列）时起始位置在顶部。当值为up（向上排列）是起始位置在底部。

#### start
将子元素在**交叉轴上起点对齐**；根据verticalDirection属性确定交叉轴方向的起点。

- CrossAxisAlignment.start & VerticalDirection.down
```dart
crossAxisAlignment: CrossAxisAlignment.start,
verticalDirection: VerticalDirection.down,
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568895505583-dd50a894-1ca6-4dd3-9dd1-5f7901d73182.png#align=left&display=inline&height=423&name=image.png&originHeight=846&originWidth=510&size=35557&status=done&width=255)

- CrossAxisAlignment.start & VerticalDirection.up
```dart
crossAxisAlignment: CrossAxisAlignment.start,
verticalDirection: VerticalDirection.up,
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568895612816-c68192e3-8563-4546-bfd6-a26c7d1b87ec.png#align=left&display=inline&height=424&name=image.png&originHeight=848&originWidth=510&size=34872&status=done&width=255)

#### end
将子元素在**交叉轴上终点对齐**；根据verticalDirection属性确定交叉轴方向的起点。

- CrossAxisAlignment.end & VerticalDirection.down

```dart
crossAxisAlignment: CrossAxisAlignment.end,
verticalDirection: VerticalDirection.down,
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568895775526-3ca72bb6-b4b9-4843-b7ea-b573722b90eb.png#align=left&display=inline&height=424&name=image.png&originHeight=847&originWidth=510&size=34852&status=done&width=255)

- CrossAxisAlignment.end& VerticalDirection.up
```dart
crossAxisAlignment: CrossAxisAlignment.end,
verticalDirection: VerticalDirection.up,
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568895936602-fad9b536-d554-496e-82a7-ff0ced4b39a3.png#align=left&display=inline&height=423&name=image.png&originHeight=846&originWidth=510&size=35693&status=done&width=255)

#### center
将子元素在**交叉轴上居中对齐**。设置 verticalDirection 无改变。

- CrossAxisAlignment.center & VerticalDirection.down
```dart
crossAxisAlignment: CrossAxisAlignment.center,
verticalDirection: VerticalDirection.down,
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568896146996-182d9bec-1070-46d5-bb31-19b0082da39d.png#align=left&display=inline&height=424&name=image.png&originHeight=848&originWidth=510&size=34589&status=done&width=255)

- CrossAxisAlignment.center & VerticalDirection.up
```dart
crossAxisAlignment: CrossAxisAlignment.center,
verticalDirection: VerticalDirection.up,
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568896146996-182d9bec-1070-46d5-bb31-19b0082da39d.png#align=left&display=inline&height=424&name=image.png&originHeight=848&originWidth=510&size=34589&status=done&width=255)

#### stretch
将子元素在**交叉轴上拉伸**。设置 verticalDirection 无改变。

- CrossAxisAlignment.stretch& VerticalDirection.down
```dart
crossAxisAlignment: CrossAxisAlignment.stretch,
verticalDirection: VerticalDirection.down,
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568896333624-310cec18-d752-4f91-9653-00ab0c51d796.png#align=left&display=inline&height=424&name=image.png&originHeight=847&originWidth=510&size=35650&status=done&width=255)

- CrossAxisAlignment.stretch& VerticalDirection.up
```dart
crossAxisAlignment: CrossAxisAlignment.stretch,
verticalDirection: VerticalDirection.up,
```

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568896394578-38b892fb-d53e-4440-8822-08f8a2b65fbe.png#align=left&display=inline&height=424&name=image.png&originHeight=847&originWidth=510&size=35654&status=done&width=255)

#### baseline
基准线适用于文字。使用baseline，必须设置textBaseline属性。textBaseline 包括字母文字（alphabetic，如：英语）和表意文字（ideographic，如：汉语）。
```dart
Widget _buildContainer() {
    var container = Container(
      child: Row(
        textDirection: TextDirection.ltr,
        mainAxisAlignment: MainAxisAlignment.start,
        verticalDirection: VerticalDirection.down,
        crossAxisAlignment: CrossAxisAlignment.baseline,
        textBaseline: TextBaseline.alphabetic,
        children: <Widget>[
          Text(
            'Flutter',
            style: TextStyle(color: Colors.blue, fontSize: 30.0),
          ),
          Text(
            'Flutter',
            style: TextStyle(color: Colors.orange, fontSize: 20.0),
          )
        ],
      ),
    );
    return container;
  }
```

- 不设置基准线

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568943176345-67a5d3b8-39c1-442e-83aa-efe16be5c971.png#align=left&display=inline&height=528&name=image.png&originHeight=1055&originWidth=643&size=44819&status=done&width=321.5)

- 设置基准线（baseline）

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568943208646-f47c9243-4c2e-413b-ae52-f75a3965a082.png#align=left&display=inline&height=528&name=image.png&originHeight=1056&originWidth=643&size=45144&status=done&width=321.5)

### mainAxisSize: 主轴占用空间大小
mainAxisSize 只有两个值，一个是min，一个是max。默认是max。
当值为min时候。主轴缩紧，变为最小。此时即使mainAxisAlignment:MainAxisAlignment.spaceAround，主轴的长度仍为最小状态。

- mainAxisSize: MainAxisSize.max & mainAxisAlignment: MainAxisAlignment.spaceAround

        ,
![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568946442306-27444f1c-f9f2-4935-b82c-cae95f14c3c8.png#align=left&display=inline&height=529&name=image.png&originHeight=1058&originWidth=643&size=40763&status=done&width=321.5)

- mainAxisSize: MainAxisSize.min & mainAxisAlignment: MainAxisAlignment.spaceAround

![image.png](https://cdn.nlark.com/yuque/0/2019/png/291118/1568946596597-b5f398cb-32fe-4355-902e-faf9ee23d322.png#align=left&display=inline&height=529&name=image.png&originHeight=1057&originWidth=643&size=40346&status=done&width=321.5)
