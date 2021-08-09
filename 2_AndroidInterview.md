1.  **Android Context**

   Context是一个抽象类，有两个类继承它，ContextWrapper和ContextImp

   ContextWrapper是一个包装类, 内部通过mBase这个变量来调用方法做事情, mBase是一个ContextImp对象

   Context数量 = Activity数量 + Service数量 + 1 （1为Application）

   getApplicationContext() 和 getApplication() 的区别

   两者都返回同一个**Application**对象 ,这两个函数的**作用范围**不同

   getApplicationContext()是在**Context 定义**的，所以说只要继承Context的子类都可以调用getApplicationContext()方法。 

   而 getApplication()在Context并没有定义，只是**在Activity和Service中定义**了getApplication()方法

   ![Context](https://github.com/KaiShi1222/AndroidStudy/raw/main/Android_Context.webp)

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

子元素是否能接受事件:需要满足两个条件

1. 子元素在播放动画
2. 点击事件的坐标落在子元素区域内

View对点击事件的处理

1. 首先判断是否设置onTouchListener, onTouch返回true时，onTouchEvent就不会调用

2. onTouchEvent 内部会对具体的action进行相应， 只要View的CLICKABLE或LONG_CLICKABLE有一个为true就会消耗这个事件

------

滑动冲突: 

* 内外部滑动方向不一致
* 内外部滑动方向一致
* 上面两种情况嵌套

外部拦截法：指点击事件都**先经过父容器**的拦截处理，如果父容器需要此事件就拦截，否则就不拦截。具体方法：需要重写父容器的**onInterceptTouchEvent**方法，在内部做出相应的拦截。
内部拦截法：指父容器不拦截任何事件，而将所有的事件都传递给子容器，如果子容器需要此事件就直接消耗，否则就交由父容器进行处理。具体方法：需要配合**requestDisallowInterceptTouchEvent**方法。

针对第一种，可以通过判断dx或者dy的距离差判断

针对第2,3 种，需要通过具体的业务逻辑来区分了



4. AAR 和 Jar区别**

* AAR可以包含Android resource和manifest文件(必须)
* AAR可以包含C/C++ library

当构建多个应用，这些应用使用相同的组建时

当构建一个应用，这个应用构建不同的变体版本（免费，付费，国内，国际）

Ref: https://developer.android.com/studio/projects/android-library?hl=zh-cn#PrivateResources

版本依赖冲突的解决方案: https://blog.csdn.net/yuzhiqiang_1993/article/details/78214812

1. ./gradlew -q app:dependencies

   ```groovy
   api("dependencies_name") {
           exclude group: 'com.android.support', module: 'support-v13'
           exclude group: 'com.android.support', module: 'support-vector-drawable'
       }
   ```

2. Grovvy脚本修改版本号解决冲突

   ```groovy
   configurations.all {
       resolutionStrategy.eachDependency { DependencyResolveDetails details ->
           def requested = details.requested
           if (requested.group == 'com.android.support') {
               if (!requested.name.startsWith("multidex")) {
                   details.useVersion '28.0.0'
               }
           }
       }
   }
   ```

3. 迁移到androidx

5. **Android 线程池**

   * Thread
   * AsyncTask: 封装了 Executor和Handler
   * HandlerThread: 继承Thread，可以使用Handler, 在run方法中创建Looper并开启循环, 需要通过quit和quitSafely退出
   * IntentService: 本质是一个Service, 内部封装了HandlerThread和Handler

   线程池分类: Android中线程池的概念来自Java的Executor, Executor是个接口, 具体的实现是ThreadPoolExecutor, 

   它可以配置线程核心数，线程池最大的线程数, 非核心线程keepAliveTime等等

   Android中线程池分为

   * FixedThreadPool: 只有核心线程, 并且数量固定的,也不会被回收

   * SingleThreadPool

     **只有一个核心线程**,确保所有的任务都在同一线程中按序完成.因此不需要处理线程同步的问题.

   * CachedThreadPool

     **只有非核心线程**,最大线程数非常大,所有线程都活动时会为新任务创建新线程,否则会利用空闲线程(60s空闲时间,过了就会被回收,所以线程池中有0个线程的可能)处理任务.

   ​	优点:任何任务都会被立即执行(任务队列SynchronousQuue相当于一个空集合);比较适合执行大量的耗时较少的任务.

   * ScheduledThreadPool

     **核心线程数固定**,非核心线程（闲着没活干会被**立即回收数**）没有限制.

     优点:执行**定时**任务以及有**固定周期**的重复任务

6. **Fragment与Activity之间的交互**

   * Activity传递数据给Fragment: 使用Bundle

     ```java
      Bundle bundle = new Bundle();    
      bundle.putString("message", "I love Google");
      fragment.setArguments(bundle);
     ```

   * Fragment传递数据给Activity: 回调接口

   * EventBus实现fragment向Activity传值

   * LiveData

   ​    

      Ref

   1. https://cloud.tencent.com/developer/article/1394289
   2. 

7. **内存泄漏**

   * 单例造成的内存泄漏

   ```java
   public class AppManager {
     private static AppManager instance;
     private Context context;
       
     private AppManager(Context context) {
         this.context = context;
     }
     
     public static AppManager getInstance(Context context) {
       if (instance == null) {
           instance = new AppManager(context);
       }
       return instance;
     }
   }
   ```

   * 集合造成的内存泄漏

     HashMap、Vector等的使用容易出现内存泄露，比如这个集合类是全局性的变量，只有添加元素的方法，没有移除的方法，或者将集合对象设置为null

     对GC来说这个对象是不可以回收的

     ```java
     Vector v = new Vector(10);
     for (int i = 1; i < 100; i++) {
         Object o = new Object();
         v.add(o);
         o = null;   
     }
     
     ```

   * 使用广播没有unregister, 文件流没有关闭等

   * Handler 造成的内存泄漏: 本质原因是因为线程，不是内部类持有外部类引用

     可以使用静态内部类和弱引用的方式



8. implement 和api区别

   https://stackoverflow.com/questions/44413952/gradle-implementation-vs-api-configuration

   api : 传递性

   implement: 没有传递性

















