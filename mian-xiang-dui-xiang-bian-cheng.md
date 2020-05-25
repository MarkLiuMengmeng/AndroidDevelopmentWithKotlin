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

### 构造器参数和属性

我们已经知道在主构造器中使用`var`或者`val`关键字可以声明属性，如果不使用所声明的仅仅是构造器参数，构造器参数仅仅在构造器中使用，用于类的创建，创建之后，无法像访问类的属性那样访问它：

```kotlin
class User(var name:String, age:Int)
val user = User("Mamun", 27)
println(user.name) // 打印结果：Mamun
println(user.age) // 错误，age只是一个构造器参数
```

下面这个表格列出了他们之间的主要区别：

| 类的声明 | 是否生成getter | 是否生成setter | 类型 |
| :--- | :--- | :--- | :--- |
| `class User(name:String)` | 否 | 否 | 构造器参数 |
| `class User(var name:String)` | 是 | 是 | 属性 |
| `class User(val name:String)` | 是 | 否 | 属性 |

### 构造器的默认参数

如果一个类有很多参数，但有的参数是可选的，为了满足我们多样化的创建需求，在Java中我们会用很多设计模式：

* **伸缩式构造器模式**（Telescoping constructor pattern）:设置一系列的构造器，每个构造器依次增加一个参数。现在已经不推荐这种方式了，因为当参数较多时，代码会难以维护且难以阅读，由于参数的顺序是固定的，在传参时你仍然可能传入你并不设置的参数值。在Android中，仍有少量的类在使用这种方式，比如`android.view.View`类：

```kotlin
val view = View(context)
val view = View(context, attributeSet)
val view = View(context, attributeSet, defStyleAttr)
```

* **JavaBean模式**（JavaBeans pattern）：使用无参构造方法创建对象，通过属性的setter设置参数。这种方式比较适合自动化的赋值，因为手动编写代码时容易遗漏设置操作：

```kotlin
val user = User()
user.setName("Mamun")
user.setAge(27)
```

* **建造者模式**（Builder pattern）：使用建造者对象来逐步设置参数，每一步的设置都会返回当前生成的对象，最后使用一个无参的`build`方法完成对象的构造。建造者模式兼顾了伸缩式构造器模式的安全性和JavaBean模式的可读性，但是由于多了一个辅助对象的开销，某些情况下的性能会受到影响：

```kotlin
Retrofit retrofit = new Retrofit.Builder()
                                .baseUrl("https://api.github.com/")
                                .build()
```

好消息是，Kotlin引入了默认参数和命名参数语法，默认参数给了我们传参的可选性，命名参数语法给了我们传参顺序的灵活性，结合这两个特性，我们可以把类的创建工作变得非常简单，我们可以像调用一个函数那样完成类的创建：

```kotlin
class User(var name: String = "", var age: Int = 1, val gender:String = "Unkown")

val user = User()
val user1 = User("Mamun",18)
val user2 = User(gender = "Female")
```

## 接口

在Kotlin中使用`interface`关键字定义接口：

```kotlin
interface PhoneNumProvider{
    fun validatePhoneNum()
}
```

Kotlin中不使用Java中的`implement`关键字，而是使用`:`来实现接口：

```kotlin
class User:PhoneNumProvider {
    override fun validatePhoneNum() {
        //phoneNumber validation
    }
}
```

Kotlin的接口中还可以定义属性：

```kotlin
interface PhoneNumProvider {
    val phoneNum: String
    fun validatePhoneNum()
}
```

实现接口的类必须实现所有的属性和方法：

```kotlin
class User() : PhoneNumProvider {
    override val phoneNum: String = "1234567"
    override fun validatePhoneNum() {
        //phoneNumber validation
    }
}
```

**重写**（`override`）属性可以放在主构造器中：

```kotlin
class User(override val phoneNum: String) : PhoneNumProvider {
    override fun validatePhoneNum() {
        //phoneNumber validation
    }
}
```

和Java8引入的接口默认方法类似，Kotlin也允许接口中定义的方法有默认实现：

```kotlin
interface PhoneNumProvider {
    val phoneNum: String
    val phoneNumLength:Int
    get()= phoneNum.length // 属性get方法的默认实现
    
    fun validatePhoneNum() = phoneNumLength == 11 // 普通方法的默认实现，返回类型被推断为Boolean
}
```

对于默认实现的限制是不能使用幕后字段，因为接口是无状态的。默认实现的意义在于可以将子类中可能重复定义的方法提取出来，使代码更易维护且更具可读性。

和Java中类似，Kotlin中的类只能继承一个类，可以实现多个接口，继承类和实现接口都使用`:`，它们之间用`,`隔开：

```kotlin
open class Human
interface PhoneNumProvider {
    val phoneNum: String
    
    fun validatePhoneNum()
}

interface EmailProvider{
    val email:String
    
    fun validateEmail()
}

class User(override val phoneNum: String,override val email:String):Human(),PhoneNumProvider,EmailProvider {
    override fun validatePhoneNum(){
        //phoneNumber validation
    }
    override fun validateEmail(){
        //email validation
    }
}
```

既然一个类可以实现多个接口，那么如果两个接口中有一样签名的方法该如何呢？假设我们有两个有相同方法的接口：

```kotlin
interface A{
    fun test(){
        print("A")
    }
}
interface B{
    fun test(){
        print("B")
    }
}
```

答案是复写一个方法即可：

```kotlin
class Tester:A,B{
    override fun test(){
        println("tester")
    }
}
```

打印结果以实现的子类为准，因为越靠近继承链底端的类越具体，所复写方法的优先级越高：

```kotlin
Tester().test() // 打印结果：tester
```

如果想使用父类的方法可以使用`super`关键字加方括号引用类名：

```kotlin
class Tester:A,B{
    override fun test(){
        super<A>.test()
        super<B>.test()
    }
}
//使用示例
Tester().test() // 打印结果：AB
```

## 继承

我们已经知道所有类型的父类型是`Any`，等同于Java中的`Object`。每个类都隐式或显式地继承`Any`：

```kotlin
class Animal // 隐式继承Any
class Animal:Any // 显式继承Any
```

与Java不同的是，在Kotlin中每个类默认加上了`final`修饰符。由于多态的存在，参数可以传指定类型的子类型，意料之外的子类型传值可能会引起意外的行为。所以Kotlin引入了这个特性来增强继承方面的安全性。

和Java一样，Kotlin也是单继承的。但继承不再使用`extends`关键字，而是`:`

```kotlin
class Animal
class Tiger:Animal() //错误，final修饰的类无法被继承 
```

我们要使一个类可以被继承，要显式地加上`open`修饰符，它就像`final`的反义词：

```kotlin
open class Animal
class Tiger:Animal() //正确
```

需要注意的是，类中的属性和方法默认也是`final`的，我们需要给每一个想要重写的成员加上`open`关键字：

```kotlin
open class Animal(){
    open var name:String = "Unkown"
    open fun eat(){}
}
class Tiger:Animal(){
    override var name:String = "Tiger"
    override fun eat(){
        println("Tiger is eating")
    }
}
```

在上面`Animal`类中我们并不想实现`eat`方法，只是定义了一个行为，把它声明为抽象类是更好的方案。声明抽象成员和抽象类需要使用`abstract`关键字：

```kotlin
abstract class Animal(){
    abstract var name:String // 用了abstract之后，不初始化变量
    abstract fun eat() // 用了abstract之后，没有方法体
}
```

这样修改之后，前面例子中的`Tiger`类依然是正确的。被`abstract`修饰的类和成员默认是`open`的，因为它们需要继承来实现具体的操作。

{% hint style="info" %}
**抽象类**（abstract class）和**接口**（interface）：

**抽象类**：抽象类是使用`abstract`修饰符修饰的类，它只能用作基类不能实例化。

* 抽象类中可以有抽象成员也可以有普通成员
* 如果一个子类实现了父类（抽象类）的所有抽象方法，那么该子类可以不必是抽象类，否则就依然是抽象类
* 抽象方法没有方法体，抽象属性不初始化变量

**接口**：接口使用`interface`修饰符修饰，它也不能被实例化。

* 一个类只能继承一个类，但是可以实现多个接口
* 接口中的成员默认均是抽象的
{% endhint %}

## JvmOverloads注解

我们前面在构造器一节提到Android中的`View`类使用的是伸缩式构造器模式。如果有时我们想自定义视图的话，我们的构造器可能是这样声明的：

```kotlin
class CustomView : View {
    constructor(context: Context?) : this(context, null)
    constructor(context: Context?, attrs: AttributeSet?) :this(context, attrs, 0)
    constructor(context: Context?, attrs: AttributeSet?, defStyleAttr:Int) : 
                super(context, attrs, defStyleAttr) {
        //...
    }
}
```

这样的写法可以看到有许多冗余的代码，Kotlin的解决方案是使用`@JvmOverload`注解：

```kotlin
class CustomView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr)
```

该注解会告知编译器生成额外的JVM字节码构造器重载：

```kotlin
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

我们可以看到Kotlin在提升与Java的互操作性方面做出的努力。

## 数据类

我们在开发工作经常会创建一些用于存储数据的类（存储来自数据库或者服务器的数据）。在Java中我们需要写出许多getter和setter，还经常会重写`hashCode`、`equals`等方法。虽然Android Studio可以生成其中的大部分代码，但维护工作仍然是一个问题。

我们已经了解到Kotlin会为我们自动生成getter和setter方法。Kotlin实际上做了更多，我们可以使用`data`关键字来修饰一个类：

```kotlin
data class Product(var name: String, var price: Double)
```

这样修饰后的数据类编译器会为我们自动生成相应的`equals`、`hashCode`、`toString`、`copy`和多个`componentN` 方法。需要注意的是，data关键字不能和`abstract`、`inner`、和`sealed`一起使用。

让我们具体来看一下这些方法。

### equals和hashCode方法

在处理数据类时，我们经常需要比较两个实例是否数据相等**据**，意思是验证它们是否包含同样的数据而非验证它们是否是同一个实例。常见的做法是使用`equals`方法：

```kotlin
product.equals(product1)
```

`equals`方法内部实现用到了`hashCode`方法，它们之间有一个**合约**（Contract）用法：

* 如果两个对象相等，则它们必须具有相同的哈希码。
* 如果两个对象具有相同的哈希码，则它们可以相等也可以不相等（因为我们可以根据需要比较其它字段）。

所以，我们重写`equals`方法就必须重写`hashCode`方法，否则会出现意料之外的行为。如果我们只想重写`hashCode`方法，`equals`方法则不必重写。

重写`hashCode`方法的一般做法是让两个相等（相等根据具体的需求来定，完全相等或是结构相等甚至别的需求）的对象有一样的哈希码。

我们来看一下上面`Product`类的Java实现：

```kotlin
public class Product {
    private String name;
    private Double price;
    
    public Product(String name, Double price) {
        this.name = name;
        this.price = price;
    }
    
    @Override
    public int hashCode() {
        int result = name != null ? name.hashCode() : 0;
        result = result + (price != null ? price.hashCode() : 0);
        return result;
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (o == null || getClass() != o.getClass()) {
            return false;
        }
        Product product = (Product) o;
        if (name != null ? !name.equals(product.name) :product.name != null) {
            return false;
        }
        return price != null ? price.equals(product.price) :product.price == null;
    }
    
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public Double getPrice() {
        return price;
    }
    public void setPrice(Double price) {
        this.price = price;
    }
}
```

在很早之前，使用Java需要手动编写所有的代码。现在，像Android Studio这样的现代IDE可以自动为我们生成这样的代码，但我们仍需要维护这些代码以确保它们在修改或重构之后能够正常工作。

在Kotlin中，这个问题将迎刃而解，下面这个类的定义包含上面Java实现的所有方法，仅仅需要在普通类声明前加上`data`关键字：

```kotlin
data class Product(var name: String, var price: Double)
```

我们在使用**结构相等操作符**（Structural equality operator）`==`时，始终在幕后调用的是`equals`方法，有了如上定义之后：

```kotlin
val productA = Product("Glove", 19.9)
val productB = Product("Glove", 19.9)
val productC = Product("Earphone", 29.9)
print(productA == productA) // 打印结果: true
print(productA == productB) // 打印结果: true
print(productB == productA) // 打印结果: true
print(productA == productC) // 打印结果: false
print(productB == productC) // 打印结果: false
```

默认情况下，编译器生成的`equals`方法比较的是声明在主构造器中的属性，这适用于大多数情况。如果我们需要自定义实现，我们可以重写`equals`方法，编译器不会再生成默认实现。

### toString 方法

默认生成的方法中含有所有包含在主构造器中的属性信息：

```kotlin
data class Product(var name:String, var price:Double)
val productA = Product("Glove", 19.9)
println(productA) // 打印结果：Product(name=Glove, price=19.9)
```

相比于普通类`toString`实现中的类名+内存地址（Product@28a418fc），我们可以在控制台或者日志中输出更易读的信息。

### copy 方法

默认情况下，数据类还会生成适当的copy方法，用于创建某个类的副本：

```kotlin
data class Product(var name:String, var price:Double)

val productA = Product("Glove", 19.9)
println(productA) // 打印结果：Product(name=Glove, price=19.9)
val productB = productA.copy()
println(productB) // 打印结果：Product(name=Glove, price=19.9)
```

`copy`方法中含有主构造器中对应的所有参数，我们还可以结合命名参数语法修改任意参数：

```kotlin
val productA = Product("Glove", 19.9)
val productB = productA.copy(name = "Earphone")
val productC = productA.copy(name = "Earphone",price = 39.9)
println(productA) // 打印结果：Product(name=Glove, price=19.9)
println(productB) // 打印结果：Product(name=Earphone, price=19.9)
println(productC) // 打印结果：Product(name=Earphone, price=39.9)
```

`copy`方法产生的副本是浅拷贝（只复制当前对象，不复制对象中的对象），拷贝前后两个对象中的对象仍是共用一个：

```kotlin
data class Producer(val name:String,var address:String)
data class Product(val name:String, val price:Double, val producer:Producer)

val productA = Product("Glove", 19.9,Producer("A","China"))
val productB = productA.copy()
productA.producer.address = "America"
println(productA) // 打印结果：Product(name=Glove, price=19.9, producer=Producer(name=A, address=America))
println(productB) // 打印结果：Product(name=Glove, price=19.9, producer=Producer(name=A, address=America))
```

在属性声明中使用不可变型变量`val`，然后通过其副本修改数据，这样做的好处是自由地在多线程操作中共享数据，以防止可变型变量`var`在多线程操作中潜在的错误共享错误。

### componentN 方法与解构声明



