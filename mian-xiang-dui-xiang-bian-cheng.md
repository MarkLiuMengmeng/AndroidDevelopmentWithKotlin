---
description: >-
  Kotlin提供了对面向对象编程（OOP）的完整支持。面向对象程序设计以对象为核心，该方法认为程序由一系列对象组成，类是对现实世界的抽象，对象是类的实例。本章将介绍Kotlin中面向对象编程相关的部分，主要是类和对象，Kotlin简化了数据模型类的定义和操作，还支持Java中不支持的操作符重载以及Java8才引入的接口默认实现。
---

# 面向对象编程

## 类

Kotlin中的类和Java中的类很类似，不过Kotlin可以用更简洁的语法完成。

类的定义使用`class`关键字。与Java不同，实例化类不再需要使用`new`关键字：

```kotlin
class User 
val user = User()
```

由于Kotlin和Java良好的互操作性，如果在Java中使用Kotlin文件中定义的类，使用`new`关键字就可以了：

```java
//Java
User user = new User()
```

实例化类是否使用`new`关键字取决于使用的位置（在Java还是Kotlin文件）而不是类定义的位置。

## 属性

一个**属性**（Property）由一个**幕后字段**（Backing field）和它的**访问器**（Accessor）组成。访问器指的是getter和setter。在Kotlin中，属性可以定义在顶层（文件中），也可以作为一个类的成员。

```kotlin
//Test.kt
val name:String //定义在顶层的属性
```

在Java中，我们考虑到类的**封装**（Encapsulation），常见的做法是把字段设为私有（`private`），把访问器（getter和setter）设为公有。

{% hint style="info" %}
Java中getter和setter的约定：

我们假设属性定义是 `private String name`;定义在`User`类中：

* **getter**：一个不带任何参数的方法，方法名是属性名加上`get`前缀（布尔类型属性的前缀是`is`），如`user.getName()`
* **setter**：带有一个参数的方法，方法名是属性名加上`set`前缀，如`user.setName(String name)`
{% endhint %}

让我们在一个例子中详细看一下，假设我们需要定义一个数据模型类User，可能需要从外部API（后端）或者数据库中取得数据，有两个成员属性`name`和`age。`下面是Java类的定义：

```kotlin
public class User {
    private int age;
    private String name;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public int getAge() {
        return age;
    }
    
    public void setAge(int age) {
        this.age = age;
    }
    
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
}
```

我们可以看到仅仅有两个属性的类，代码还是不少。虽然许多IDE可以使用快捷键来生成getter和setter，但较多的代码会影响我们的阅读效率，较多的方法增加了重构中出错的可能性。让我们看看使用Kotlin对这个类的等价定义：

```kotlin
class User {
    var name: String
    var age: Int
    constructor(name: String, age: Int) {//等价于Java中的构造器
        this.name = name
        this.age = age
    }
}
```

你或许会奇怪，上面并没有定义getter和setter方法，怎么说它们是等价的呢。实际上，Kotlin编译器会为我们自动生成默认的getter和setter方法，我们无需写任何代码。

上面我们定义的构造器是**次构造器**（Secondary Constructor）。相对应的，自然有**主构造器**（Primary constructor），它定义在类头中。一个类可以有一个主构造器和多个次构造器，我们可以使用主构造器语法重新定义上面的类，代码会少一点点：

```kotlin
class User constructor(name: String, age: Int) {
    var name: String
    var age: Int
    
    init {
        this.name = name
        this.age = age
        println("User instance created")
    }
}
```

上面我们使用`init`关键字在**初始化块**（Initializer block）中进行初始化操作，初始化块会在类创建时执行，从而完成我们想要的赋值等操作。我们如果不需要其他操作（如上面例子中的打印语句），我们可以直接复制给变量来简化代码：

```kotlin
class User constructor(name: String, age: Int) {
    var name: String = name
    var age: Int = age
}
```

我们看到，构造器中参数的名字和属性的名字是一样的。是的，我们还能直接在主构造器中加上变量修饰符缩短代码：

```kotlin
class User constructor (var name: String, var age: Int)
```

如果主构造器没有任何注解（如`@Inject`），没有**可见性修饰符**（Visibility modifier）。这样`constructor`关键字也可以省略：

```kotlin
class User (var name: String, var age: Int)
```

考虑到可读性和减少代码合并时产生的冲突，我们可以在每个参数间进行换行来优化一下代码：

```kotlin
class User(
    var name: String,
    var age: Int
)
```

很难相信，这和一开始我们声明的Java编写的`User`类是等价的，但是Kotlin做到了，Kotlin编译器替我们完成了剩下的工作。这样的代码适用于很多在面向对象编程时使用到的简单类，包含了类的名称、变量类型和名称、构造器和访问器，而且更易读、更易维护。

{% hint style="info" %}
我们知道，Kotlin中的变量有可变类型（`var`）变量和不可变类型（`val`）变量，在类中`var`修饰的字段会自动生成默认的getter和setter，而`val`修饰的字段仅会生成默认的getter，因为val变量初始化后不能赋值。
{% endhint %}

{% hint style="info" %}
Java和Kotlin的属性访问语法

**Java**：通过属性对应的getter或者setter方法访问，如`user.getName()`，`user.setName("Mamun")`。

**Kotlin**：直接使用`.`来访问属性，如`val name = user.name`，`user.name = "Mamun"`。虽然看起来好像失去了封装性，但实际上Kotlin使用的是getter和setter，所以Kotlin相当于在语言层面内置了面向对象的封装。并且在访问属性的同时可以使用自增和自减操作符，如`user.age++`。

由于Kotlin和Java有良好的互操作性，和上节提到的类的实例化语法类似，使用的地点（Java文件还是Kotlin文件中）决定你应该使用哪种语法。
{% endhint %}

### 自定义getter和setter

有些时候我们会希望在访问属性时进行一些额外的操作，比如验证一些值的正确性、打印一些日志等，这些都可以通过自定义getter和setter实现。

我们知道年龄如果为负数是不合理的，让我们给上面的`User`类中的`age`属性加上负值验证：

```kotlin
class User(var name: String,age:Int) {
    var age = age
    get() {
        println("getter value retrieved")
        return field
    }
    set(value) {
        field = if (value < 0) 0 else value
        println("setter new value assigned $field")
    }
}
//使用示例
val user = User("Mamun",24)
var age = user.age //打印结果：getter value retrieved
user.age = 22 //打印结果：setter new value assigned 22
```

上面自定义的getter和setter仅对`age`属性生效，对`name`属性是不生效的。我们可以看到我们用到了一个特殊的变量`field`，它代表的是对应的幕后字段，在这里我们不能直接使用`age`，因为这样会死循环调用getter。

我们可以看到Kotlin中编写代码时幕后字段和它的getter和setter是放到一起的，这样我们可以很容易地读懂代码逻辑和进行维护。而在Java中我们常常把字段声明在类的顶部，把getter和setter声明在类的底部，我们经常不能在一页中看清楚他们的逻辑。

{% hint style="info" %}
直接（自定义getter和setter）或间接（自动生成的getter和setter）使用`field`字段的，这样的属性才会有幕后字段，否则（getter和setter都是使用的其他变量）没有幕后字段。
{% endhint %}

### 延迟初始化属性

有些时候，我们知道这个变量不会为空，但是声明时也不能直接给它赋值。比如常用的存储Android视图引用的变量：

```kotlin
class MainActivity : AppCompatActivity() {
    private var button: Button? = null
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        button = findViewById(R.id.button) as Button
    }
}
```

`button`变量在声明的时候，布局文件尚未初始化，此时不能给它赋值，为了完成在`onCreate`方法的赋值操作，我们必须声明它为可空类型。这样做是很不方便的，可空类型变量在使用时每次都要进行非空检查或者使用安全调用操作符，而视图变量是一个使用频率又相当高。

这时候我们可以使用`lateinit`修饰符来修饰该变量，告诉编译器它在使用之前不会为空，它的初始化被推迟了，这样我们可以把上面的变量声明为非空类型的：

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var button: Button
    
    override fun onCreate(savedInstanceState: Bundle?) {
        button = findViewById(R.id.button) as Button
    }
}
```

这时候确保它非空就是我们程序员的责任了，如果在未被初始化时访问它，会抛出**UninitializedPropertyAccessException**。

延迟初始化的应用场景很常见，不仅出现在视图相关的变量中，还经常出现在依赖注入库、单元测试库等的使用中：

```kotlin
@Inject lateinit var volumeManager: VolumeManager // 使用依赖注入库Dagger
@Mock lateinit var mockEventBus: EventBus // 使用单元测试库Mockito
```

### 注解属性

我们用Kotlin写了一个属性，Kotlin就会为我们生成很多配套的JVM字节码（`privte`字段、getter和setter）。有些时候有些库的注解处理器或基于反射实现的库要求我们的字段设置为`public`。如单元测试库JUnit，它要求我们`@Rule`注解必须加在字段或者getter方法上：

```kotlin
@Rule
val activityRule = ActivityTestRule(MainActivity::class.Java) //无法识别
```

使用Kotlin写这段代码JUnit是无法识别的，我们需要使用注解`@JvmField`来指明这是一个Java字段：

```kotlin
@JvmField @Rule
val activityRule = ActivityTestRule(MainActivity::class.Java)
```

注解`@JvmField`的使用有一定限制：

* 必须有幕后字段
* 可见性不能为`private`
* 没有`open`、`const`、`override`这些修饰符修饰

我们也可以直接注解在getter方法上：

```kotlin
@get:Rule //注解在默认getter方法上
val activityRule = ActivityTestRule(MainActivity::class.Java)

val activityRule//注解在自定义getter方法上
@Rule get() = ActivityTestRule(MainActivity::class.java)
```

### 内联属性

我们可以使用`inline`修饰符来优化属性的调用，前提是没有幕后字段的属性：

```kotlin
inline val now: Long
    get() {
        return System.currentTimeMillis()
    }
```

在编译时，每个`now`属性的调用都会被替换为实际值：

```kotlin
System.currentTimeMillis()
```

内联属性会节省一些额外的创建开销，并能在编写代码时提高可读性。

## 构造器

通过前面我们大概了解到，我们可以定义不带构造器的类，也可以定义带一个主构造器的类，还可以定义一个带主构造器和若干次构造器的类。

需要注意的是，主构造器一个类只能声明一个，属性声明仅能在主构造器中进行，如果需要在次构造器中使用属性，那就需要先在类中声明：

```kotlin
class User(var name: String) {
    var age:Int? = null
    
    constructor(name:String,age:Int):this(name){
        this.age = age
    }
}
```

我们看到上面的例子中，次构造器使用的`this`关键字来调用主构造器。在主构造器定义的类中，必须隐式或者显式地调用主构造器。显式调用主构造器是指直接调用主构造器，隐式调用是指次构造器调用主构造器。

如果子类没有构造器而父类有非空的构造器，则每个次构造器都要使用`super`关键字来初始化基类：

```kotlin
class ProductView : View {
    constructor(ctx: Context) : super(ctx)
    constructor(ctx: Context, attrs : AttributeSet) : super(ctx, attrs)
    constructor(context: Context?, attrs : AttributeSet?, defStyleAttr:
                Int) : super(context, attrs, defStyleAttr)
}
```

构造器默认的可见性是`public`，我们想设置构造器的可见性时，直接在构造器关键字`constructor`前加上可见性修饰符：

```kotlin
class User private constructor()
```

当我们使用像Dagger这样的依赖注入库来注解构造器时也是如此：

```kotlin
class User @Inject constructor()
```

它们也可以一起使用：

```kotlin
class User @Inject private constructor()
```

