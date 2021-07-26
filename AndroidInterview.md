1. **通过Acitivty的xml标签来改变任务栈的默认行为**

   - 使用`android:launchMode="standard|singleInstance|singleTask|singleTop"`来控制Acivity任务栈。

     **任务栈**是一种后进先出的结构。位于栈顶的Activity处于焦点状态,当按下back按钮的时候,栈内的Activity会一个一个的出栈,并且调用其`onDestory()`方法。如果栈内没有Activity,那么系统就会回收这个栈,每个APP默认只有一个栈,以APP的包名来命名.

     - standard : 标准模式,每次启动Activity都会创建一个新的Activity实例,并且将其压入任务栈栈顶,而不管这个Activity是否已经存在。Activity的启动三回调(*onCreate()->onStart()->onResume()*)都会执行。

     - singleTop : 栈顶复用模式.这种模式下,如果新Activity已经位于任务栈的栈顶,那么此Activity不会被重新创建,所以它的启动三回调就不会执行,同时Activity的`onNewIntent()`方法会被回调.如果Activity已经存在但是不在栈顶,那么作用与*standard模式*一样.
     - singleTask: 栈内复用模式.创建这样的Activity的时候,系统会先确认它所需任务栈已经创建,否则先创建任务栈.然后放入Activity,如果栈中已经有一个Activity实例,那么这个Activity就会被调到栈顶,`onNewIntent()`,并且singleTask会清理在当前Activity上面的所有Activity.**(clear top)**
     - singleInstance : 加强版的singleTask模式,这种模式的Activity只能单独位于一个任务栈内,由于栈内复用的特性,后续请求均不会创建新的Activity,除非这个独特的任务栈被系统销毁了

   Activity的堆栈管理以ActivityRecord为单位,所有的ActivityRecord都放在一个List里面.可以认为一个ActivityRecord就是一个Activity栈

2. **Android中的动画**

   分为三种: View动画, 帧动画(本质也是View动画)，属性动画

   * View动画

     四种变化: 平移，缩放，旋转，透明度动画

     分别对应Animation四个子类, 这四种动画可以通过**xml**或**代码**来定义

     ```kotlin
     val animation: Animation = AnimationUtils.loadAnimation(this, R.anim.animation_test)
     val textView = findViewById<TextView>(R.id.tv)
     textView.startAnimation(animation)
     
     
     val alphaAnimation = AlphaAnimation(0f, 1f)
     alphaAnimation.duration = 300
     alphaAnimation.fillAfter = true
     textView.startAnimation(alphaAnimation)
     ```

     自定义View动画 : 继承于Animation 重写initialize和applyTransformation方法

   * 帧动画

     顺序播放一组定义好的图片，由AnimationDrawable来使用

     ```kotlin
     val button = findViewById<Button>(R.id.bt)
     button.setBackgroundResource(R.drawable.frame_animation)
     val drawable = button.background as AnimationDrawable
     button.setOnClickListener {
         drawable.start()
     }
     ```

   * 属性动画

     在一个时间间隔，完成**对象**从一个**属性值**到另一个属性值的**改变**

     ```kotlin
     val colorAnim = ObjectAnimator.ofInt(button, "textColor", 0xFFFF8080.toInt(), 0xFF8080FF.toInt())
     colorAnim.duration = 3000
     colorAnim.setEvaluator(ArgbEvaluator())
     colorAnim.repeatCount = ValueAnimator.INFINITE
     colorAnim.repeatMode = ValueAnimator.REVERSE
     colorAnim.start()
     
     // 动画集合
     val colorAnim = ObjectAnimator.ofInt(button, "textColor", 0xFFFF8080.toInt(), 0xFF8080FF.toInt())
     colorAnim.duration = 3000
     colorAnim.setEvaluator(ArgbEvaluator())
     colorAnim.repeatCount = ValueAnimator.INFINITE
     colorAnim.repeatMode = ValueAnimator.REVERSE
     
     val animatorSets = AnimatorSet()
     animatorSets.apply {
         playTogether(colorAnim)
         duration = 5 * 1000
         start()
     }
     ```

     通过xml添加属性动画

     ```xml
     <?xml version="1.0" encoding="utf-8"?>
     <set xmlns:android="http://schemas.android.com/apk/res/android"
         android:ordering="together">
         <objectAnimator
             android:propertyName="x"
             android:duration="300"
             android:valueTo="200"
             android:valueType="intType"/>
     
         <objectAnimator
             android:propertyName="y"
             android:duration="300"
             android:valueTo="300"
             android:valueType="intType"/>
     </set>
     ```

     ```kotlin
     val set = AnimatorInflater.loadAnimator(this, R.animator.property_animator)
     set.setTarget(button)
     set.start()
     ```

     对任意属性做动画，有三种方法

     * 如果有权限，给对象添加set, get方法

     * 用一个类包装对象，间接提供set, get方法

       ```kotlin
       private class ViewWrapper(target: View) {
       
           private var mTarget = target
       
           fun getWidth(): Int {
               return mTarget.layoutParams.width
           }
       
           fun setWidth(width: Int) {
               mTarget.layoutParams.width = width
               mTarget.requestLayout()
           }
       
       }
       ```

     * 用ValueAnimator，监听动画，实现属性的改变

       ```kotlin
         fun performAnimate(target: View, start: Int, end: Int) {
               val valueAnimator = ValueAnimator.ofInt(1, 100)
               valueAnimator.addUpdateListener {
                   ValueAnimator.AnimatorUpdateListener {
                       val intEvaluator = IntEvaluator()
                       val currentValue: Int = it.animatedValue as Int
                       Log.d("luo", "currentValue: $currentValue")
                       val fraction = it.animatedFraction
                       target.layoutParams.width = intEvaluator.evaluate(fraction, start, end)
                       target.requestLayout()
                   }
               }
               valueAnimator.duration = 500
               valueAnimator.start()
           }
       ```
       
       

   3. **ANR**

      1. 什么是ANR

         ANR是指应用程序未响应，Android系统对于一系列事件需要在**一定时间内完成**，如果超出预定时间**未能响应**或者**响应时间过长**，都会造成ANR. 具体来说是system_server进程启动倒计时，如果应用进程在规定的时间没有执行完需要的任务，触发ANR

      2. 哪些场景会造成ANR呢

      * Service Timeout: 比如**前台**服务在20s内未执行完成
      * BroadcastQueue Timeout：比如**前台**广播在10s内未执行完成

      3. ANR是怎么判断超时时间的

         system_server在控制每个组件的生命周期回调的时候，通过handler发送延时消息，如果超时，就触发ANR，如果没有，就将这个消息从消息队列中取出

         在判断Activity的InputDispatching超时的情况下，是InputDispatcher发现**上一个事件没有处理完，新的事件又进来了**，才会去走ANR。

      4. ANR发生后操作

         - 将am_anr信息输出到EventLog，也就是说ANR触发的时间点最接近的就是EventLog中输出的am_anr信息
         - 收集以下重要进程的各个线程调用栈trace信息，保存在**data/anr/traces.txt**文件
           - 当前发生ANR的进程，system_server进程以及所有persistent进程
           - audioserver, cameraserver, mediaserver, surfaceflinger等重要的**native**进程
           - CPU使用率排名前5的进程
         - 将发生ANR的reason以及CPU使用情况信息输出到main log
         - 将traces文件和CPU使用情况信息保存到dropbox，即data/system/dropbox目录
         - 对用户可感知的进程则弹出ANR**对话框**告知用户，对用户不可感知的进程发生ANR则直接杀掉

         Ref

         1. [彻底理解安卓应用无响应机制](http://gityuan.com/2019/04/06/android-anr/)

   

   4. **Android 消息机制**

      1. 所谓消息机制，就是一个线程开启循环模式，不断的从一个消息队列中取出消息并处理。

         当这个消息队列没有消息的时候，该线程会挂起等待。其他线程通过发送消息到消息队列

         让这个线程取出处理。

         在Android中，消息机制主要由Handle , MessageQueue, Looper, Message实现

         * Handle的作用在于发送和处理消息，也就是我们常用的sendMessage和handleMessage方法

         * MessageQueue主要用于插入和读取消息。对应为enqueueMessage和next方法

         * Looper用于开启循环，不断的遍历消息队列中是否有消息，有的话就处理

         * Message为消息本身

           ![](/home/shikai/Working/myself/study/handler_java.jpg)

      当通过send或者post发送消息后，最后都会调用MessageQueue的enqueueMessage方法，该方法用于插入消息。该方法内部是由单链表实现的，这样做的好处在于方便插入和删除。

      当调用Looper.loop方法开启循环, 内部会调用MessageQueue的next方法, 取出消息，并通过msg.target.dispatchMessage处理

      其中target为发送这个message的handle对象. 这个函数内部实现逻辑为如果msg注册了callback,则消息处理handleCall

      如果实现了callBack接口，消息处理由这个callBack接口的handleMessage处理，否则由Handle的handleMessage处理

      

      主线程是已经在ActivityThread中通过Looper.prepareMainLooper创建Looper并调用Looper.loop开启循环，如何在其他线程中使用需要手动开启.

      https://segmentfault.com/a/1190000022551072

      Q: 为什么 Android 非要规定只有主线程才能更新 UI

      A: 因为Android UI 控件线程不安全, 多线程访问可能会造成UI状态的不可预期

      ​    如果都对UI控件加锁, 会让UI控件访问逻辑复杂。并且加锁会让访问的效率变低 

      Q:  Handler 内部是如何完成线程切换的呢(https://juejin.cn/post/6844903886675935239)

      A: 通过ThreadLocal, ThreadLocal是线程内部的数据存储类，以线程为单位不同线程可以获得不同的数据副本

      ​    创建Handle时，会持有这个线程的Looper对象和MessageQueue对象，该Looper对象和handler处于同一个线程. 并存储在ThreadLocal中

      ​    当B线程发送一个消息时, 消息最终会通过msg.target.dispatchMessage(msg)处理

      ![](/home/shikai/Working/myself/study/handle_class.jpg)

      **线程的转换由 Looper 完成，handleMessage() 所在线程由 Looper.loop() 调用者所在线程决定**，不是创建handler的线程。

      Looper 有退出的功能，但是主线程的 Looper 不允许退出；

      异步线程的 Looper 需要自己调用 Looper.myLooper().quit()退出

      http://gityuan.com/2015/12/26/handler-message-framework/

      Q: 为什么主线程可以直接使用Handle

      A: 因为ActivityThread的main方法已经创建了Looper并开启了循环

      Q:  一个线程能否创建多个Handler，Handler跟Looper之间的对应关系

       A: 个Thread只能有一个Looper，一个MessageQueen，可以有多个Handler

      ​     以一个线程为基准，他们的数量级关系是： Thread(1) : Looper(1) : MessageQueue(1) : Handler(N)

      Q: Android中为什么主线程不会因为Looper.loop()里的死循环卡死

      A:    死循环并非运行在新线程，**死循环就运行在主线程**。死循环在没有消息时，就会**阻塞**。我们之所有感受不到阻塞，是因为我们所有的常规代码，都是**被动执行**的：主线程接收消息，主线程执行代码，执行完代码后取下一条消息，没有消息主线程阻塞。**我们的代码都执行了，当然体会不到阻塞**。

      死循环又如何去处理其他事务呢？通过创建新线程的方式。

      真正会卡死主线程的操作是在回调方法onCreate/onStart/onResume等操作时间过长，会导致掉帧，甚至发生ANR，looper.loop本身不会导致应用卡死。

      Ref: Handle

      1. https://www.huaweicloud.com/articles/029b0f537168b7328a4bf621c127c1a3.html
      2. http://gityuan.com/2015/12/26/handler-message-framework/
      3. https://juejin.cn/post/6844903886675935239
      4. https://segmentfault.com/a/1190000022551072

   5. **Android 序列化**

       以字节流的形式, 将对象的状态信息转换为可以存储或传输的形式的过程,

      这也回答了为什么要序列化: 

      * 当应用意外退出时候，保存对象，用于恢复

      * 用于传输(网络, IPC)

       

      序列化方式: Java自带的Serializable, Android的Parcelable

      1. Serializable

         通过Serializable序列化后，恢复的对象内容相同，但不是同一个对象

         ObjectOutputStream 和 ObjectInputStream实现

        通过指定UID，可以很大程度避免反序列化的失败

        静态成员属于类，不属于对象，因此不会参加序列化的过程

        使用transient的对象不参与序列化

      2. Parcelable

      ​      Android系统提供的接口，实现这个接口，对象可以通过Intent传递

      ​       createFromParcel()和writeToParcel()方法中对应变量读写的顺序必须是一致的，否则序列化会失败

      ​       Parcelable是在native层通过读写内存实现的

       两者区别: 

       Serializable

       由于存在大量I/O操作，效率比较低

        使用上，Parcelable相对麻烦一点，但是效率比较高

        Parcelable主要用于内存的序列化， Serializable用于网络或者本地存储，网络传输也可以通过Gson等方式

6. **MVC, MVP, MVVM**

* MVC: 分为Model, View, Controller

  Model负责数据逻辑的处理

  View 负责数据的显示

  Controller负责业务逻辑的处理

  View 产生事件，通知到 Controller，Controller 中进行一系列逻辑处理，之后通知给 Model 去更新数据，Model 更新数据后，再将数据结构通知给 View 去更新界面。Model直接操作View

![image](https://github.com/KaiShi1222/AndroidStudy/raw/main/Android_MVC.png)

​    在Android中，Activity和Fragment既承担了View的工作，也承担了Controller的职责

1. 这导致了当业务逻辑非常复杂的时候Activity的代码过于臃肿，难于维护

2. 难以写test case

3. Model和View直接交互，耦合性比较强

* MVP分为Model, View, Presenter

MVP和MVC类似，Presenter取代了Controller的位置，presenter作为View和model沟通的桥梁,实现了View和model的解耦

View只负责UI的显示

Model负责对数据的操作 

![image](https://github.com/KaiShi1222/AndroidStudy/raw/main/Android_MVP.png)

MVP的优点:

1. 将View和Model层解耦, View是可能经常改变的部分，当View改变的时候，Model不需要修改

2. 方便进行unit test

3. 模块间的层次清晰，职责分明

4. 增强了组件的重用性，比如不同界面需要相同的数据，Model可重用

MVP的不足/注意的地方

1. 会添加过多的接口，接口的颗粒度不好掌握，文件数量会大大增加

如何优化: 采用泛型定义契约类，将model、view、presenter定义在一个契约类中

​       一个契约类对应一个业务模块

2. View和presenter互相持有，形成耦合

3. 当业务非常复杂的时候，Presenter依然会比较重，难以维护

* MVVM: 分为Model, View, ViewModel

同样的Model负责处理数据, View负责UI, ViewModel负责逻辑

这里的ViewModel指的是ViewMode层，是一个抽象的概念，和Jetpack的具体组件

ViewModel无关，只是Android Jetpack中的ViewModel可以充当MVVM架构的VM层

 

MVVM的本质是数据驱动，ViewModel不持有view，进一步解耦合

 

Android官方提出的Jetpack MVVM是对MVVM的一种实践

View层: Activity和Fragment

ViewModel层: Jetpack的ViewModel, LiveData

用于持有和UI元素相关的数据，以保证这些数据在屏幕旋转时不会丢失，并且还要提供接口给View层调用以及和仓库层进行通信。

Model层：包含repository层，也就是仓库层，用于处理数据从本地获取还是从网络获取

本地使用Room或者SP等，网络比较常用的就是retrofit发送网络请求，访问服务器

![Android_MVVM](https://github.com/KaiShi1222/AndroidStudy/raw/main/Android_MVVM.png)

 

LiveData具有生命周期感知能力，通过观察者模式，当数据更新后，自动将数据推送给活跃的UI组件

ViewModel比需要更新的View的生命周期长，因此不要持有View对象，包括context对象

MVVM的好处:  MVP 的基础上，MVVM 把 View 和 ViewModel 也进行了解耦

Ref:

1. https://developer.android.com/jetpack/guide

2. https://zhuanlan.zhihu.com/p/346711847

3. https://zhuanlan.zhihu.com/p/83635530

 

总结：这些架构都是为了具体的项目而服务的，为了是解耦和维护。针对简单的项目MVC或者MVP也可以才用

选择适合项目的架构即可，没有哪一种架构一定好，需要避免过度的设计。

7. **Android系统启动流程**

   ![](/home/shikai/Working/myself/study/android-booting.jpg)

   当系统按下上电后，加载引导程序Bootloader到内存，Bootloader的主要作用是检查RAM，初始化硬件参数等功能。

   接着启动kernel, 创建swapper进程，这是系统的第一个进程pid = 0. 这个进程主要用于进程管理、内存管理，加载一些驱动. 

   再创建Linux系统的内核进程

   当linux内核启动后，系统会启动init进程，这是第一个用户进程pid = 1

   * init进程

     进程启动后主要做如下几件事

     * 查找并解析init.rc文件, 创建和挂载一些文件的目录
     * 初始化子进程退出的处理函数

     接着调用property_init函数对属性初始化, 属性初始化以后，接着启动属性服务， 即创建Socket, 并设置监听，当有数据到来时init进程调用相应的处理函数进行处理

     * 解析init.rc创建ServiceManger进程

     * 解析init.rc创建 **Zygote** 进程

     Zygote进程是Android系统的第一个Java进程(即虚拟机进程)

     init进程还启动`servicemanager`(binder服务管家)、`bootanim`(开机动画)等重要服务

   * Zygote进程

     Zygote进程启动Java虚拟机, 并注册JNI方法, 通过JNI调用从Native层到Java层，调用ZygoteInit的main方法

     这个main方法主要做几件事

     * 创建一个zygote的服务端的Socket
     * 预加载类和资源 drawable和color资源
     * 等待AMS请求创建新的应用程序进程
     * 创建system_server进程 (通过fork)

   * system_server进程

     system_server进程用于创建系统服务，比如AMS, PMS, WMS等

     system_server会启动一个Binder线程池，用于和其他进程通信

     在system_server的main方法中，主要做几件事

     * 加载so库
     * 创建SystemServerManager,主要对各种服务进行创建，管理
     * 启动各种服务

   * 最后AMS会启动Launcher应用，将安装的应用图标显示出来

     ![](/home/shikai/Working/myself/study/android-boot.jpg)