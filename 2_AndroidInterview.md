1.  **Android Context**

   Context是一个抽象类，有两个类继承它，ContextWrapper和ContextImp

   ContextWrapper是一个包装类, 内部通过mBase这个变量来调用方法做事情, mBase是一个ContextImp对象

   Context数量 = Activity数量 + Service数量 + 1 （1为Application）

   getApplicationContext() 和 getApplication() 的区别

   两者都返回同一个**Application**对象 ,这两个函数的**作用范围**不同

   getApplicationContext()是在**Context 定义**的，所以说只要继承Context的子类都可以调用getApplicationContext()方法。 

   而 getApplication()在Context并没有定义，只是**在Activity和Service中定义**了getApplication()方法

   ![Context](https://github.com/KaiShi1222/AndroidStudy/raw/main/Android_Context.png)

2. **include, merge, ViewStub** 

   <include/> : 用于布局的**重用**

   使用时需要注意的点: 

   * include 添加**id**,会**覆盖**被include的xml文件根节点ID
   * include标签如果使用**layout_xx**属性,会**覆盖**被include的xml文件根节点对应的layout_xx属性
   * 一个xml布局文件有**多个include标签**需要设置**ID**,才能找到相应子View的控件

   <merge/>: 用于**减少布局层级**

   使用时需要注意的点: 

   * merge标签必须使用在**根布局**
   * ViewStub标签中的layout布局不能使用merge标签
   * 因为merge标签并不是View,所以在通过LayoutInflate.inflate(layout, **parentContainer**, **true**)方法渲染的时候,第二个参数必须指定一个父容器,且第三个参数必须为true,也就是**必须为merge下的视图指定一个父亲节点**.

   <viewStub/>: 用于**按需加载**, 实际是一个宽度和高度都为0的View

   使用时需要注意的点: 

   * ViewStub标签不支持merge标签
   * ViewStub的inflate只能被调用一次,第二次调用会抛出异常,setVisibility可以被调用多次,但不建议这么做(ViewStub 调用过后,可能被GC掉,再调用setVisibility()会报异常)
   * 为ViewStub赋值的android:layout_XX属性会替换待加载布局文件的根节点对应的属性

3. **Android View事件分发**

   一个Activity包含了一个Window对象，这个对象是由PhoneWindow来实现的。PhoneWindow将DecorView作为整个应用窗口的根View，而这个DecorView又将屏幕划分为两个区域：一个是TitleView，另一个是ContentView，而我们平时所写的就是展示在ContentView中的。

   ![Window](https://github.com/KaiShi1222/AndroidStudy/raw/main/Android_Window.png)

   触摸事件对应的是**MotionEvent**类，事件的类型主要有如下三种：

   * ACTION_DOWN
   * ACTION_MOVE
   * ACTION_UP

​       View事件分发本质就是对**MotionEvent事件分发**的过程。即当一个MotionEvent发生后，系统将这个点击事件**传递**到一个具体的View上

​		事件分发过程由三个方法共同完成：

* dispatchTouchEvent：用于进行事件分发，返回值受当前onTouchEvent和它的下级View的dispatchTouchEvent影响

* onInterceptTouchEvent: 在dispatchTouchEvent方法内部调用, 表示是否拦截事件

* onTouchEvent:  在dispatchTouchEvent方法内部调用， 表示是否消费事件

当产生一个点击事件的时候，它的传递顺序是从Activity -> Window -> View


如果一个View处理了事件，onTouchEvent返回true, 这次事件分发结束

如果View的onTouchEvent返回false, 则它的父View的onTouchEvent会调用，如果父View也不处理，继续向上返回，最终传递给Activity处理

![Dispatch](https://github.com/KaiShi1222/AndroidStudy/raw/main/Android_View_Dispatch.png)

```java
// 伪代码
public boolean dispatchTouchEvent(MotionEvent ev) {
    boolean consume = false;
    if (onInterceptTouchEvent(ev)) {
        consume = onTouchEvent(ev);
    } else {
        coonsume = child.dispatchTouchEvent(ev);
    }
    return consume;
}
```

一些重要的结论:

1. 事件传递优先级：onTouchListener.onTouch > onTouchEvent > onClickListener.onClick。

2. View的enable属性不影响onTouchEvent的默认返回值。

3. 通过requestDisallowInterceptTouchEvent方法可以在子元素中干预父元素的事件分发过程，但是ACTION_DOWN事件除外。因为dispatchTouchEvent方法会重置DISALLOW_INTERCEPT标志位

------

滑动冲突: 

* 内外部滑动方向一致
* 内外部滑动方向不一致
* 上面两种情况嵌套

外部拦截法：指点击事件都**先经过父容器**的拦截处理，如果父容器需要此事件就拦截，否则就不拦截。具体方法：需要重写父容器的**onInterceptTouchEvent**方法，在内部做出相应的拦截。
内部拦截法：指父容器不拦截任何事件，而将所有的事件都传递给子容器，如果子容器需要此事件就直接消耗，否则就交由父容器进行处理。具体方法：需要配合**requestDisallowInterceptTouchEvent**方法。

针对第一种，可以通过判断dx或者dy的距离差判断

针对第2,3 种，需要通过具体的业务逻辑来区分了





















