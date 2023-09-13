## Layout与视图View
Layout和View的通用属性: padding, margin
视图view的共有属性
 ```
- layout_width:有match_parent（和父级属性一样）和wrap_content（包裹内容，有多少写多少）
- layout_height:同上
- id
- gravity:表示内容在View的重心位置，center居中
- layout_gravity:标识View在Layout中的重心位置
- ﻿﻿background 
```
xml布局文件的一个个view节点，最终也是会被解析成java代码类


## 常见容器视图
### LinearLayout
特有的属性orientation，子view的排列方向
子view特有的属性layout_weight，各个子view所占的比例

### RelativeLayout
子view特有的属性
```
相对位置:layout_below/layout_above/layout_toLeftOf
在父容器中的位置: layout_centerInParent ...
子view与父view对齐: layout_alignBottom...
子view与父容器对齐: layout_alignParentBottom
```

### FrameLayout
容器内的view一层盖一层地排列
子view一般会使用layout_gravity 实现排列在父容器的上下左右位置

### ConstrainLayout
约束布局，为子view添加约束来确定位置
优点：减少嵌套，来减少过度绘制，从而优化布局；可以直接拖拽的方式布局
缺点：修改的时候容易错乱，代码可读性变差

## Logcat
级别越来越严格 
Logcat.v verbose详细的
Logcat.i info
Logcat.debug
Logcat.w warn
Logcat.e error
Logcat.a assert

