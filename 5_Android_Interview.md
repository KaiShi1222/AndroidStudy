1. **Android 性能优化工具使用**

   AS 3.0以上，通过Android Profiler分析性能， 它可以分析CPU, 内存，网络等的性能

   点击Sessions，可以选择要分析的进程

   对于CPU的分析， 可以选择System Trace或Method and function Trace

   Method and function可以了解一段时间内执行了哪些函数，以及这些函数消耗的CPU资源

   * CPU性能分析器界面包括多个时间轴

   时间时间轴：显示Activity生命周期交替, 用户与设备的交互等

   CPU时间轴：显示当前CPU的占用率，进程中运行的线程的数量等

   ​		线程时间轴: 显示线程状态

           1. 绿色: 活跃状态
              2. 黄色: 正在活跃状态， 但等待I/O执行
              3. 灰色: 表示线程正在休眠, 不消耗CPU

   通过AS可以导出CPU使用情况文件.trace, 方便后续查看

   * 内存系能分析

     通过Memory Profiler进行分析

     它会计算出这个应用消耗的内存，

     1. 包括Java, Kotlin代码创建的对象
     2. native 的代码C/C++ 分配的对象
     3. Graphics
     4. Stack: 应用的堆栈

     内存分配的具体情况

     1. 分配了哪些对象，占用了多少空间
     2. 对象的创建和销毁时机
     3. 对象分配的轨迹，包括在哪个线程中

     通过Dump Java Heap，分析Activity/Fragment引起的内存泄漏

     可以通过屏幕的旋转，应用的前后台切换，触发内存泄漏

2. **Android性能优化**

   * 布局优化

     * 减少布局的层级

       可以通过merge标签

     * 相同层级，线性布局和相对布局都能实现的情况下,采用线性布局

       因为如果没有android:width

3. Java double to String  不使用library

```java
Double d = 123.45d;
String s2 = "" + d;
System.out.println(s2.getClass().getName());
```

4. 委托模式

   委托模式中，有**两个对象**参与处理**同一个请求**，接受请求的对象将请求委托给另一个对象来处理

   ```java
   class RealPrinter { // the "delegate"
        void print() { 
          System.out.print("something"); 
        }
    }
    
    class Printer { // the "delegator"
        RealPrinter p = new RealPrinter(); // create the delegate 
        void print() { 
          p.print(); // delegation
        } 
    }
    
    public class Main {
        // to the outside world it looks like Printer actually prints.
        public static void main(String[] args) {
            Printer printer = new Printer();
            printer.print();
        }
    }
   ```

   通过接口，实现委托，更加灵活

   ```java
   interface I {
        void f();
        void g();
    }
    
    class A implements I {
        public void f() { System.out.println("A: doing f()"); }
        public void g() { System.out.println("A: doing g()"); }
    }
    
    class B implements I {
        public void f() { System.out.println("B: doing f()"); }
        public void g() { System.out.println("B: doing g()"); }
    }
    
    class C implements I {
        // delegation
        I i = new A();
    
        public void f() { i.f(); }
        public void g() { i.g(); }
    
        // normal attributes
        public void toA() { i = new A(); }
        public void toB() { i = new B(); }
    }
    
    
    public class Main {
        public static void main(String[] args) {
            C c = new C();
            c.f();     // output: A: doing f()
            c.g();     // output: A: doing g()
            c.toB();
            c.f();     // output: B: doing f()
            c.g();     // output: B: doing g()
        }
    }
   ```

   Kotlin对委托实现了语言层面的支持

   * 类委托

   ```kotlin
   interface IPlay {
   
       fun play()
   
       fun upgrade()
   }
   
   class RealPlayer(private val name: String) : IPlay {
   
       override fun play() {
           println("$name start playing")
       }
   
       override fun upgrade() {
           println("$name upgrade")
       }
   
   }
   
   class RealPlayer2(private val name: String) : IPlay {
   
       override fun play() {
           println("$name start playing by RealPlayer2")
       }
   
       override fun upgrade() {
           println("$name upgrade by RealPlayer2")
       }
   
   }
   
   class DelegatePlayer(private val iPlay: IPlay): IPlay by iPlay
   
   fun main() {
       val realPlayer = RealPlayer("Kevin")
       var delegatePlayer = DelegatePlayer(realPlayer)
       delegatePlayer.play()
       delegatePlayer.upgrade()
   
       val realPlayer2 = RealPlayer2("luo")
       delegatePlayer = DelegatePlayer(realPlayer2)
       delegatePlayer.play()
       delegatePlayer.upgrade()
   }
   ```

   * 属性委托

      val/var <属性名>: <类型> by <表达式>

     属性的委托不必实现任何的接口，但是需要提供一个 `getValue()` 函数（与 `setValue()`——对于 *var* 属性)

     ```kotlin
     class T {
         var prop: String by Delegate()
     }
     
     class Delegate {
         
         // thisRef的类型必须是拥有这个属性的类的类型或者超类类型
         // property —— 必须是类型 KProperty<*> 或其超类型
         operator fun getValue(thisRef: Any, property: KProperty<*>): String {
             return "$thisRef, thank you for delegating '${property.name}' to me!"
         }
     
         //value的类型必须是属性的类型或者它的子类
         operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
             println("$value has been assigned to '${property.name}' in $thisRef.")
         }
     }
     
     fun main() {
          println(T().prop)
          T().prop = "new Prop"
     }
     
     // 输出结果
     T@421faab1, thank you for delegating 'prop' to me!
     new Prop has been assigned to 'prop' in T@421faab1.
     ```

      Kotlin **1.4 ** 可以把一个属性的setter和getter委托给另一个属性, 另一个属性可以是

     * 这个类的成员属性或者扩展属性
     * 另一个类的成员属性或者扩展属性
     * 顶层属性

     ```kotlin
     var topInt = 10
     
     class ClassWithDelegate(val anotherClassInt: Int)
     
     class MyClass(var memberInt: Int, val anotherClassInstance: ClassWithDelegate) {
         var propOne: Int by this::memberInt
         var propTwo: Int by ::topInt
         val propThree:Int by anotherClassInstance::anotherClassInt
     }
     
     var MyClass.extension: Int by ::topInt
     ```

     **应用场景**

     以一种向后兼容的方式重命名一个属性的时候：引入一个新的属性

     ```kotlin
     class MyTest {
     
         var newName: Int = 0
     
         @Deprecated(message = "Dep", ReplaceWith("newName"))
         var oldName: Int by this::newName
     }
     
     fun main() {
         val myTest = MyTest()
         myTest.oldName = 32
         println("newName: ${myTest.newName}")
     }
     
     // 输出结果
     newName: 32
     ```

     

   * Kotlin库函数对委托的支持

     * lazy函数

        接受一个 lambda 并返回一个 `Lazy <T>` 实例的函数

       第一次访问/调用，会执行lambda并返回结果, 后续只会返回结果

       ```kotlin
       val testString: String by lazy {
           println("lazy run")
           "hhh"
       }
       
       fun main() {
           println(testString)
           println(testString)
       }
       
       // 输出结果
       lazy run
       hhh
       hhh
       ```

       默认情况下，对于 lazy 属性的求值是**同步锁的（synchronized）**

       如果初始化每次都发生在和使用属性相同的线程, 可以使用LazyThreadSafetyMode.NONE

     * 可观察属性observable

       ```kotlin
       class User {
           var name: String by Delegates.observable("<no name>") {
               prop, old, new ->
               println("$old -> $new")
           }
       }
       
       fun main() {
           val user = User()
           user.name = "first"
           user.name = "second"
       }
       ```

     * 带条件的observable: **vetoable**

       ```kotlin
       var vetoableProp: Int by Delegates.vetoable(5) {
               _: KProperty<*>, oldValue: Int, newValue: Int ->
           println("vetoableProp :  $oldValue -> $newValue")
           newValue > oldValue
       
       }
       
       fun main {
           vetoableProp = 10
           vetoableProp = 3
           vetoableProp = 20
       }
       
       // 输出结果
       vetoableProp :  5 -> 10
       vetoableProp :  10 -> 3
       vetoableProp :  10 -> 20
       ```

     # TODO(Degel in Android)

   5. Kotlin Lambda

      Lambda是可以传递给其他函数的一小段代码

   * 作为函数的参数传递

   * 语法 : { x: Int, y: Int -> x + y } 始终在{ } 内

   * lambda如果是函数最后一个参数，可以放在括号外, 如果是函数的唯一实参，可以省略括号

     ```kotlin
     fun foo({ })
     fun foo() { }
     fun foo { }
     ```

   * 高阶函数: 种参数有函数类型或者返回值是函数类型的函数，都叫做高阶函数

   * ::method 

     一个**和函数具有相同功能**的对象, 一个函数类型的对象

   * Lambda 的返回值别写 return，如果写了，它会把这个作为它外层的函数的返回值来直接结束外层函数
   * **匿名函数不是函数，是一个对象**
   *  Kotlin 的 Lambda 是实实在在的对象

5. Jetpack

   官方定义:  Jetpack 是一个由**多个库组成的套件**

   能做什么: 通过google提供的各种组件，比如LiveData, ViewModel, Room等等, 帮助开发者更专注于**具体的业务开发**

   开发中都会遇到的通用问题, 比如生命周期管理, SQL语句的编写，内存泄漏, 通过官方提供给开发者的组件，更好的去解决上述的问题

   结合Android官方的MVVM best practice, 提升开发效率和应用质量

   

   * LiveData

     * 是什么:  一种**可观察的数据存储器**类, 它遵循其他组件，比如Activity/Fragment的生命周期, 在这些组建处于活跃状态的时候, 将数据推送给他们

     在他们销毁的时候，也就是onDestroy方法执行的时候， 自动解除订阅

     * 解决什么问题

       1. 不再需要手动处理生命周期
       2. 应用数据发生变化，自动推送到UI, 不要要关注Activity/Fragment的生命周期的状态
       3. 防止了内存泄漏

     * 怎么用

       通过观察者模式，通常在**ViewModel**中创建LiveData对象，这样做的原因在于Activity/Fragment属于View，只负责UI显示的部分. LiveData和特定的UI分离

       1. 创建LiveData

          ```kotlin
          class SampleViewModel (val name:String): ViewModel() {
          
              private val _nameLiveData = MutableLiveData<String>()
          
              val nameLiveData: LiveData<String>
                  get() = _nameLiveData
          
              fun setFirstName() {
                  _nameLiveData.postValue( name)
              }
          }
          ```

       2. 观察LiveData对象

          在UI界面创建ViewModel,通过observe方法观察LiveData对象

          ```kotlin
          class MainActivity : AppCompatActivity() {
          
              lateinit var viewModel: SampleViewModel
          
              override fun onCreate(savedInstanceState: Bundle?) {
                  super.onCreate(savedInstanceState)
                  setContentView(R.layout.activity_main)
                  // 创建VM
                  viewModel = ViewModelProvider(this).get(SampleViewModel::class.java)
                  observeViewModel()
                  initListeners()
              }
          
              private fun initListeners() {
                  btn_badge?.setOnClickListener {
                      viewModel.setFirstName()
                  }
              }
          
              private fun observeViewModel() {
          
                  viewModel.nameLiveData.observe(this, Observer {
                      showToast(it)
                  })
          
              }
          
              private fun showToast(value: String) {
                  Toast.makeText(this, value, Toast.LENGTH_LONG).show()
              }
          
          }
          ```

       3. 更新LiveData对象

          通过setValue和postValue更新LiveData对象

          setValue通过在主线程更新观察者, postValue在工作线程来更新

          https://betterprogramming.pub/everything-to-should-understand-about-livedata-507dd83adea7

          

   * ViewModel

     * 是什么: 是界面(Activity/Fragment)的辅助类, 它以注重**生命周期**的方式, 管理**界面数据** 

     * 解决什么问题: 页面状态管理的问题

       1. 因为屏幕旋转，语言切换等配置修改后，UI会重新创建, 比如一个网络请求，会再次请求数据

       ViewModel在界面发生改变后，会自动恢复，横竖屏对应同一个ViewModel，不需要重新获取数据

       2. 作用域内**单一实例**的特性，使得**多个fragment**之间可以方便**通信，**并且维护**同一个数据状态**
     3. 可以作为MVVM的VM层，不依赖View，实现代码的解耦
       
     http://www.droidmonk.com/jetpack/the-problem-that-viewmodel-solves/
       
     https://juejin.cn/post/6942464122273398820
       
       https://juejin.cn/post/6955491901265051661#heading-9
       
       https://juejin.cn/post/6951244272553181197
       
       https://blog.csdn.net/chuhe1989/article/details/109805408
       
       https://juejin.cn/post/6951244272553181197

     * ViewModel的创建

       首先通过ViewModelProvider的get方法获取ViewModel的实例，ViewModelProvider.Factory创建ViewModel, 并通过**ViewModelStore**这个类的成员HashMap做缓存

       ViewModel 对象存在了 **ComponentActivity** 的 mViewModelStore 对象中

       ```kotlin
       public ViewModelProvider(@NonNull ViewModelStoreOwner owner) {
               this(owner.getViewModelStore(), owner instanceof HasDefaultViewModelProviderFactory
                       ? ((HasDefaultViewModelProviderFactory) owner).getDefaultViewModelProviderFactory()
                       : NewInstanceFactory.getInstance());
       }
       
       public ViewModelProvider(@NonNull ViewModelStoreOwner owner, @NonNull Factory factory) {
           this(owner.getViewModelStore(), factory);
       }
       
       ```

     * ViewModel的销毁

       在 activity 销毁时，判断如果是**非配置改变**导致的destory时候， 调用onCleared

       ```kotlin
       // ViewModelStore.java 
       public final void clear() {
            for (ViewModel vm : mMap.values()) {
                   vm.clear();
            }
            mMap.clear();
       }
       
       // ComponentActivity 的构造方法
       
       public ComponentActivity() {
       
           ... 
       
           getLifecycle().addObserver(new LifecycleEventObserver() {
       
               @Override
               public void onStateChanged(@NonNull LifecycleOwner source,
                       @NonNull Lifecycle.Event event) {
       
                   if (event == Lifecycle.Event.ON_DESTROY) {
                       // Clear out the available context
                       mContextAwareHelper.clearAvailableContext();
                       // And clear the ViewModelStore
                       if (!isChangingConfigurations()) {
                           getViewModelStore().clear();
       
                       }
                   }
               }
       
           });
       
           ...
       
       }
       ```

     * 为什么可以恢复数据

       Activity 在因配置更改而销毁重建过程中会先调用 **onRetainNonConfigurationInstance** 保存 **viewModelStore** 实例。

       在重建后可以通过 **getLastNonConfigurationInstance** 方法获取之前的 viewModelStore 实例。而ViewModel存储在viewModelStore 的map中

       ————————————————

       其实是用到了Activity的一个子类ComponentActivity，然后重写了onRetainNonConfigurationInstance()方法保存ViewModelStore，并在需要的时候，也就是重建的Activity中去通过getLastNonConfigurationInstance()方法获取到ViewModelStore实例。这样也就保证了ViewModelStore中的ViewModel不会随Activity的重建而改变。
       ————————————————
       版权声明：本文为CSDN博主「chuhe1989」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
       原文链接：https://blog.csdn.net/chuhe1989/article/details/109805408

       ```kotlin
       @Override
       @Nullable
       @SuppressWarnings("deprecation")
       
       public final Object onRetainNonConfigurationInstance() {
       
           // Maintain backward compatibility.
           Object custom = onRetainCustomNonConfigurationInstance();
       
           ViewModelStore viewModelStore = mViewModelStore;
       
           // 若 viewModelStore 为空，则尝试从 getLastNonConfigurationInstance() 中获取
           if (viewModelStore == null) {
               // No one called getViewModelStore(), so see if there was an existing
               // ViewModelStore from our last NonConfigurationInstance
               NonConfigurationInstances nc =
                      (NonConfigurationInstances) getLastNonConfigurationInstance();
               if (nc != null) {
                   viewModelStore = nc.viewModelStore;
               }
       
           }
       
       // 依然为空，说明没有需要缓存的，则返回 null
           if (viewModelStore == null && custom == null) {
               return null;
           }
       
       // 创建 NonConfigurationInstances 对象，并赋值 viewModelStore
           NonConfigurationInstances nci = new NonConfigurationInstances();
           nci.custom = custom;
           nci.viewModelStore = viewModelStore;
       
           return nci;
       
       }
       ```

     * Fragment的通信

       通过ViewModelProvide**r传入相同的Activity实现**

     * ViewModel和AndroidViewModel的区别

        AndroidViewModel是ViewModel的字类，并且持有application context

     * viewModel和onSaveInstanceState()

       

   

   











































 