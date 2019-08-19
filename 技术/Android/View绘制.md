[TOC]

## View绘制

参考[Android应用层View绘制流程与源码分析](https://blog.csdn.net/yanbober/article/details/46128379)

### View.onMeasure

- 系统中的测量流程请参阅

  - {@link android.view.ViewGroup#getChildMeasureSpec}
  - D:\Android-Sdk\sdk\platforms\android-28\android\view\ViewGroup.java(getChildMeasureSpec#6771)
  
- 最顶层DecorView测量时的MeasureSpec是由ViewRootImpl中getRootMeasureSpec方法确定：
  LayoutParams宽高参数均为MATCH_PARENT，specMode是EXACTLY，specSize为物理屏幕大小
  
- getMeasuredWidth()和getWidth()分别对应于视图绘制的measure和layout阶段

  getMeasureWidth 获取的是View原始的大小，也就是这个View在XML文件中配置或者是代码中设置的大小，必须在onMeasure回调之后调用才返回有效值

  getWidth()获取的是其在父容器中所占的最终宽度，是这个View最终显示的大小，这个大小有可能等于原始的大小，也有可能不相等；必须在layout阶段完成后才能返回有效值

  所以getMeasureWidth 并且不一定等getWidth

- ViewGroup类提供了measureChild，measureChild和measureChildWithMargins方法，简化了父子View的尺寸计算

- 只要是ViewGroup的子类就必须要求LayoutParams继承子MarginLayoutParams，否则无法使用layout_margin参数



### View.Layout

layout也是从顶层父View向子View的递归调用view.layout方法的过程，即父View根据上一步measure子View所得到的布局大小和布局参数，将子View放在合适的位置上。

具体layout核心主要有以下几点：

- View.layout方法可被重载，ViewGroup.layout为final的不可重载，ViewGroup.onLayout为abstract的，子类必须重载实现自己的位置逻辑。
- layout操作完成之后得到的是对每个View进行位置分配后的mLeft、mTop、mRight、mBottom，这些值都是相对于父View来说的
- 凡是layout_XXX的布局属性基本都针对的是包含子View的ViewGroup的，当对一个没有父容器的View设置相关layout_XXX属性是没有任何意义的



### View.draw



## DrawerLayout官方控件解析

### onMeasure方法

- 在该方法中，除了计算宽高，还会对侧边栏控件的Elevation进行设置，对状态栏进行设置：根据getFitsSystemWindows处理WindowInsets，对内容容器设置topMargin进行偏移，

  侧边栏控件则通过ScrimInsetsFrameLayout对WindowInsetsCompat进行处理

### onDraw方法

- 先判断是否有设置状态栏背景
- 是否大于21，是则通过WindowInsets.getSystemWindowInsetTop获取状态栏高度
- 然后绘制状态栏



### drawChild方法

- 该方法由ViewGroup触发调用
- 在该方法中实现侧边菜单滑动时阴影区域的绘制
- 通过ViewDragHelper拖动侧边菜单栏，引起菜单栏getRight的不断变化，getRight赋值给阴影区域的left进行绘制
- 阴影区域的颜色值，在computeScroll中进行不断变化