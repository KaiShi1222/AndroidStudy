# Kotlin

## 1.*Why Kotlin*

Kotlin出现的背景: 

* JetBrains需要一门“现代化”的语言去替代**Java**，去提升内部的开发效率

  Java写起来比较“啰嗦”，但是，JetBrains内部70%的IDE的都是Java编写的，全部重写耗费大量精力，因此新语言需要很好的**兼容**Java，可以基于现存项目直接开发，使用已有的Library等. 要有良好的性能，比如编译速度.

* JetBrains期望通过Kotlin提高IDE的销量. 

  它不通过Kotlin直接赚钱，通过开源这门语言，提高开发者对新语言和开发工具的关注度，改善开源社区，从而更好的占领市场，进行商业活动

* 新语言的话题性

  开发者相信JetBranis这个公司，也相信他们可以做好这个新的项目.这份信任让JetBranis相信公司可以从商业的获利， 而且更多的开发者们也会通过JetBranins的开发工具，让开发这件事变得更有趣

## 2. 适用场景

### 2.1 技术场景

* 服务器开发
* Android 开发
* 其他任何运行Java代码的地方

### 2.2 业务场景

## 3. Kotlin基础和Java的比较

Kotlin是一门静态类型类型语言，支持面向对象和面向函数编程

### 3. 1 变量

Kotlin通过val和var两个关键字声明变量

val 表示**不可变引用**，只能初始化一次

```kotlin
🏝
val a: Int = 1
// Int类型声明可以根据类型推导得出
val a = 1
```

相当于Java中的final变量

```java
☕
final int a = 1;
```

var表示**可变引用**，变量的值可以改变

```kotlin
🏝
var a: Int = 1
a = 2
```

等价于Java中的

```java
☕
int a = 1;
a = 2;
```

### 3. 2 函数

#### 3.2.1 函数声明

```kotlin
🏝
fun f(a: Int, b: Int): Int {
    return if (a > b) a else b
}
```

对于单个表达式的函数体，简写如下. 但对于有返回值的代码块，必须显示的声明返回值类型和return语句

```kotlin
🏝
fun f(a: Int, b: Int): Int = if (a > b) a else b
```

#### 3.2.2 函数参数

Kotlin中函数参数的定义方式为: *name*: *type*， 而且Kotlin函数参数支持命名参数

```kotlin
🏝
fun max(a: Int = 1, b: Int = 2) = if (a > b) a else b
// invoke
max() // 2
max(4, 1) // 4
max(a = 2, b = 3) // 3
max(0, b = 1) // 1
```

命名参数的好处在于，增强了代码的可读性

假设有一个函数用于校验用户信息，函数参数是3个Boolean值，分别代表isMarried, isMale, isNative

Java对应的代码可能为, 如果不通过IDE的提示和注释，很难第一眼看出每个参数的含义

```java
☕
public void verifyUserInfo(true, false, false) { /*...*/ }
```

如果用Kotlin写这个函数

```kotlin
🏝
open verifyUserInfo(isMarried = true, isMale = false, isNative = false) { /*...*/ }
```

当一个函数参数混合传递位置参数和命名参数的时候，所有的位置参数都放在第一个命名参数的前面

```kotlin
🏝
fun sum(a: Int, b: Int = 2, c: Int = 3) = a + b + c
// invoke
sum(1, c = 4) // 7
sum(c = 1, 4) // error
```

参数的默认值用于解决Java中**重载函数过多**的问题

做Android开发的小伙伴可能都写过类似如下的代码

```java
☕
public class CustomView extends View {

    public CustomView(Context context) {
        super(context);
    }

    public CustomView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
    }

    public CustomView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }
}
```

Kotlin可以实现的更加简洁

```kotlin
🏝
class CustomView @JvmOverloads constructor(context: Context, attrs: AttributeSet? = null, defStyleAttr: Int = 0) : View(context, attrs, defStyleAttr) {
	/* ... */
}
```

因为Java中没有默认参数的概念，如果频繁的从Java中调用Kotlin代码，每次都需要显示的指定所有的函数参数，比较麻烦

通过 @JvmOverloads注解，编译器会生成多个重载函数,方便Java调用

```java
☕
public CustomView(Context context) {
    super(context);
}

public CustomView(Context context, @Nullable AttributeSet attrs) {
    super(context, attrs);
}

public CustomView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
    super(context, attrs, defStyleAttr);
}
```

#### 3.3.3 函数的返回值

Java中函数的返回值类型有void类型, 在Kotlin中, Unit类型相当于Kotlin中的"void"

```java
☕
public void f() { ... }
```

```kotlin
🏝
open f(): Unit { ... }// 其中Unit可以省略
```

Unit和void的区别: Unit是一个类型，具体说是**只有一个**Unit实例的类型，可以当作函数的参数传递，void不能当作函数的参数

针对不需要返回值，Java的做法如下，只要函数的返回值不是void, 就必须显示的声明return语句

* 分开定义需要返回值的接口和不需要返回值的接口，如Callable和Runnable

  ```java
  ☕
  Callable<String> callable = new Callable<String>() {
      @Override
      public String call() throws Exception {
          // do something
          return "string";
      }
  };
  
  Runnable runnable = new Runnable() {
      @Override
      public void run() {
          // do something
      }
  };
  ```

  对于return关键字的理解: 如果没有@label, 返回到最近的fun， 如果有@label,返回到label

  [Kotlin官方文档关于return的描述](https://kotlinlang.org/docs/reference/returns.html)

* 通过java.lang.Void

  ```java
  ☕
  public Void f() {
      // do something
      return null; // 必须显示声明
  }
  ```

### 3.3 类和属性

#### 3.3.1 类

在Java中，经常会写大量的数据类, 比如我们创建一个Person类如下。这包含了大量的样板代码

```java
☕
public class Person {

    private String name;

    public Person(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

Kotlin中提供了简明的语法来声明他们

```kotlin
🏝
class Person(var name: String) // kotlin中类默认是public的
```

这里通过var声明了name属性是可变的. 在Kotlin中调用Java的Person类

```kotlin
🏝
val person = Person("Name")
person.name = "New name"              // 调用Java的setName
println("Name: ${person.name}")      // 调用Java的getName, 输出New Name, 这里用到了类似shell等编程语言中的字符串模板
```

在Java中调用Kotlin

```java
☕
public static void main(String[] args) {
    Person p = new Person("Name");
    p.setName("New Name");
    System.out.println("Name: " + p.getName());
}
```

* 构造方法

  Kotlin中的构造方法分为***主构造方法***，和***从构造方法***

  1. 主构造方法

  ```kotlin
  // 声明方式
  // 1
  class Person constructor(_name: String) {
      val name: String
      
      init {
          name = _name // 构造方法参数_name用于和属性区分，也可以用类似java的常用做法: this.name = name
      }
  }
  
  // 2
  class Person constructor(_name: String) {
      val name = _name // 参数初始化属性
  }
  
  // 3
  // 如果属性用构造方法参数初始化，可以用如下方式简写
  class Person constructor(val name: String)
  
  // 4
  // 进一步简化，如果constuctor没有注解或者可见性修饰符(如private), constructor可以省略
  class Person(val name: String)
  ```

  以上声明方式等价，第4种最简洁. 在真实的开发中，没有参数构造方法和直接通过参数初始化属性的方式能比较好的编码。

  和上面声明代码的等价的Java代码

  ```java
  ☕
  final class Person {
      final String name;
      
      public Person(String name) {
          this.name = name;
      }
      
      public String getName() {
          return name;
      }
  }
  ```

  2. 从构造方法

     在Java中有大量的重载构造方法，这在Kotlin中不那么常见。Kotlin中通过***提供参数默认值***和***命名参数***来替代Java的做法

     好处在于不需要写大量的重载构造方法，代码的可读性也得到了增强. 

     从构造方法的使用场景: 扩展(继承)一个类来添加多个构造方法. 比如Android开发中我们写自定义View继承了View这个类

     ```kotlin
     🏝
     class MyView : View {
         
         constructor(context: Context): super(context)
     
         constructor(context: Context, attributes: AttributeSet?) : super(context, attributes)
     
         constructor(context: Context, attributes: AttributeSet?, defStyleAttr: Int): super(context, attributes, defStyleAttr)
     
     }
     ```
     
     个人理解可以通过提供参数默认值的方式去替代多个构造函数
     
     ```kotlin
     🏝
     class MyView(context: Context, attributes: AttributeSet? = null, defStyleAttr = 0: Int = 0) : View(context: Context, attributes: AttributeSet?, defStyleAttr: Int) // View内部类默调用带有一个参数的构造方法时，attributes为null, defStyleAttr = 0
     ```

* 嵌套类

  首先来看一下Java的嵌套类。在Java中，可以把一个类定义到另外一个类的内部，叫做嵌套类(Nested class)

  when to use:  

  1. 只有外部类使用这个类
  2. 这个类想访问外部类的private属性或者方法

  这样做的好处: 把类放在一起，更好的封装了代码, 代码的结构更紧凑。类的关系更紧密，增强了代码的可读性。修改内部类代码的时候，值会影响他的外部类，增强了代码的可维护性

  Java中的嵌套类可以分为***静态嵌套类***和***非静态嵌套类***。非静态嵌套类又叫***inner class***

  从类的**行为**角度来说，静态的内部类在和外部类交互的时候，和其他顶层类一样。它不能直接访问外部类的属性和方法。只能通过外部类的引用访问

  非静态嵌套类由于持有一个外部类的对象, 所有可以直接访问外部类的属性和方法，就相当于访问自己的属性和方法一样。

  ```java
  ☕
  public class Outer {
  
      private final String outer = "outer";
  
      public Outer() {
          System.out.println("Create outer object");
      }
  
      public void testOuter() {
          System.out.println("outer test run");
      }
  
      static class StaticInner {
  
          public StaticInner() {
              System.out.println("Create static inner object");
          }
  
          public void testStaticInner() {
  //            testOuter(); // can not access
  //            System.out.println("Outer filed: " + outer); // can not access
              System.out.println("static inner test run");
          }
      }
  
      class Inner {
  
          public Inner() {
              System.out.println("Create inner object");
          }
  
          public void testInner() {
              Outer.this.testOuter(); // can access directly
              System.out.println("Outer filed: " + Outer.this.outer);
              System.out.println("inner test run");
          }
      }
  }
  
  ```

  在Kotlin中创建的内部类默认是***静态嵌套类*** ， 相当于Java的static class

  ```kotlin
  🏝
  class KOuter {
      
      val name: String = "outer"
      
      fun testOuter() {
          println("outer test run")
      }
      
      inner class Inner {
          
          fun getOuterReference() = this@KOuter
          
          fun testInner() {
              testOuter()
              print("outer name: $name")
          }
      }
  }
  ```

* 密封(sealed)类

  在Java中调用Kotlin

  ```kotlin
  🏝
  sealed class Expr {
      
      class Number(val value: Int) : Expr()
      
      class Sum(val left: Expr, val right: Expr) : Expr()
  }
  
  fun eval(e: Expr): Int = when(e) {
      is Expr.Number -> e.value
      is Expr.Sum -> eval(e.left) + eval(e.right)
  }
  ```

#### 3.3.2 属性

当不需要一个字段去存储属性的值的时候，可以通过Kotlin的自定义访问器实现, 完整的语法规则为

```kotlin
🏝
var <propertyName>[: <PropertyType>] [= <property_initializer>]
    [<getter>]
    [<setter>]
```

比如我们有一个矩形类，它可以自己根据长和宽判断是否是正方形, 每次都是通过计算获得isSquare的值

```kotlin
🏝
class Rectangle(val height: Int, val width: Int) {
    var isSquare: Boolen
        get() = height == width
}
```

Kotlin中可以通过特殊的标识符**field**来得到支持字段的值

假设有一个员工类，这个类有一个可变属性salary, 初始化值为1000, 通过自定义setter访问器,实现加薪, 其中field标识符就是属性salary的值, 初始值为1000

```kotlin
🏝
class Employee {
    var salary: Int = 1000
        set(value) {
            println("Before increase salary: $salary")
            field = value + 1000
            println("After increase salary: $salary")
        }
}

fun main() {
    val e = Employee()
    e.salary = 500  // 输出1000和1500
}
```

### 3.4 控制流

* if

  在Kotlin中if是表达式，这意味着if有返回值。这点和Java不同，Java中的if是语句

  ```kotlin
  🏝
  if (a > b) a else b
  // 代码块最后一个表达式的值作为if的返回值
  var max = if (a > b) {
      print("a is max")
      a
  } else {
      print("b is max")
      b
  }
  ```

* when

  在Kotlin中,when的作用类似于Java的switch语句，但是有些不同

  * Java中如果有switch分支遗漏了break语句，可能会导致bug

  ​       Kotlin中when匹配成功，只有对应的分支会执行

  * Java中的switch要求变量必须使用byte`, `short这样的基本类型，或者Enum类型, String类型(Java SE7后), 和一些包装类，比如Integer等

    Kotlin中的when可以接收任何类型的对象

  * Kotlin中的when可以不带参数, 如果不带参数，分支条件只能是boolean值

    ```kotlin
    🏝
    fun isOdd(x: Int, y: Int) {
        when {
            x % 2 == 0 && y % 2 == 0 -> println("x and y all evens")
            else -> println("x or y is not a even")
        }
    }
    ```

  * 从Kotlin **1.3** 开始, when的参数可以是一个变量

    ```kotlin
    🏝
    fun random() {
        when(val x = Random.nextInt(1, 5)) {
            1 -> print("x is 1")
            2 -> print("x is 2")
            else -> print("x is $x")
        }
    }
    ```

* while

  Kotlin和Java的while循环一样，分为while和do-while循环

  ```kotlin
  🏝
  while(condition) {/*...*/ }
  
  // 第一次循环会无条件执行, 后续condition为true在执行
  do {
      /*...*/
  } while(condition)
  ```

* for

  Kotlin中通过**区间**去替代Java中传统的for循环

  ```kotlin
  🏝
  // 注意右边的值总是区间的一部分，即区间是闭合的[1, 10]
  for(i in 1..10) {
      print("$i ") // 1 2 3 4 5 6 7 8 9 10
  }
  ```
  
  相当于Java中的
  
  ```java
  🏝
  for(int i=1; i<=10; i++) {
      System.out.println(i)
  }
  ```

kotlin中的for循环比Java的for循环强大, 或者说语法糖比较多

  ```kotlin
    🏝
    // downTo和step实际上是一个扩展函数
    for (i in 10 downTo 2 step 2) {
      println(i) // 10 8 6 4 2
    }

    val array = arrayOf("a", "b", "c")
    for ((index, value) in array.withIndex()) {
      println("the element at $index is $value")
    }

    for(i in 0 until 5) {
       println(i) // 0 1 2 3 4
    }
  ```

 

### 3.5 接口

Kotlin的接口和Java **8**的接口类似, 可以添加具体的方法实现

```kotlin
interface A {
    fun foo()
    fun bar() {
        println("default bar called")
    }
}
```

等价的Java 8 代码

```java
interface A {
    void foo();
    default void bar() {
        System.out.println("default bar called");
    } 
}
```

Kotlin和Java一样，接口可以继承。可以 和类一样添加自己的属性和方法，或者重载实现

```kotlin
interface Name {

    val name: String

}

interface PersonName : Name {

    val firstName: String

    val lastName: String

    override val name: String
        get() = "$firstName and $lastName"
}
```

对于实现了这个接口的类来说，只需要添加需要的实现即可

```kotlin
class Person(override val firstName: String, override val lastName: String) : PersonName
```

如果两个接口中定义了相同的方法(方法名一样，参数一样), 一个类实现这两个接口，必须自己实现这个方法

```kotlin
interface A {
    fun foo() { print("A") }
    fun bar()
}

interface B {
    fun foo() { print("B") }
    fun bar() { print("bar") }
}

class C : A, B {
    
    override fun foo() {
        super<A4>.foo()
        super<B4>.foo()
    }

    override fun bar() {
        super.bar()
    }
}
```

### 3.6 可见性修饰符

Java中如果不显示的标明final，你可以继承任何类并重写任何非private方法。这带来的问题是，如果父类修改，可能会对子类产生不确定的影响。

继承的主要作用在于: **多态**

**<\<Effective Java>>**中的建议: 要么为继承做好设计并写好文档，要么**禁止**这样做

Kotlin的设计思想直接采用了这样的做法，默认情况下所有的类和方法，都是final的

所以如下想让你的类或者方法被继承，需要显示的声明**open**, 如果一个方法重写了一个open方法，那么它本身也是**open**的

比如我们定义一个Car类和一个BMW类 :simle:

```kotlin

open class Car {

    fun launch() {
        println("Car launched")   
    }
    // 可以被重写
    open fun speedUp() {}
}

// BMW可以被继承
open class BMW : Car() {
    // 重写父类方法, 切这个函数不能被继承
    final override fun speedUp() {
        super.speedUp()
        println("BMW speedUp")
    }
}
```

Classes, objects, interfaces, constructors, functions, properties and their setters can have *visibility modifiers*. (Getters always have the same visibility as the property.)

Kotlin的可见性修饰符和Java类似，都有public, protected, private. 但是默认的可见性不同

Java为包能可见性, Kotlin为public

另外Kotlin中还有一中internal的可见性。类似Java的包可见性。它表明了模块内的封装。

关于模块: 官方给出的解释是一个可以是一个IntelliJ IDEA module ,或者maven project等等

在Android开发中，一个app可以由多个module组成，通过gradle最终编译成一个apk，个人理解可以在这些module中使用internal

和Java还有一个不同是，Kotlin的可见性还可以在顶层(top-lever)声明中使用，这也是一种抽象的方式(对外隐藏具体的实现)

|  修饰符   |    类成员     |  顶层声明   |
| :-------: | :-----------: | :---------: |
|  public   |  everywhere   | everywhere  |
| internal  |  in a module  | in a module |
| protected | in a subclass |    NONE     |
|  private  |  in a calss   |  in a file  |

p.s open是表示这个类的**可继承性**



## 5.*TODO*

function Nothing

function Any

for



### 6.Kotlin VS Java

1. 静态变量

```java
public static final String name = "Name";
```

在Kotlin中可以通过

```kotlin
const val name = "Name"

class A {
    companion object {
        const val name = "Go!"
    } 
}
```

2. 静态方法

   ```java
   public static void foo() {
       System.out.println("Hello");
   }
   
   // invoke in Java
   Foo.Companion.foo();
   Bar.Factory.bar();
   A.base();
   ```

   In Kotin

   ```kotlin
   class Foo {
      companion object {
          fun foo() = println("Hello")
   	}
   }
   
   class Bar {
       companion object Factory {
           fun bar() = println("Bar")
       }
   }
   
   class A {
         companion object {
           @JvmStatic
           fun base() = println("base kkk")
       
   }
   ```

3. 单例模式

   

```kotlin
class Single {

    companion object {
        val instance : Single by lazy(LazyThreadSafetyMode.SYNCHRONIZED) {
            Single()
        }
    }
}
```

https://bladecoder.medium.com/kotlin-singletons-with-argument-194ef06edd9e





