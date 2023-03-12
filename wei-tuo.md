---
description: >-
  Kotlin非常看重设计模式，在前面我们已经了解到单例模式可以简化为object声明，使用高阶函数和函数类型简化观察者模式的实现。本章中我们将了解Kotlin的委托（Delegates）特性，并应用在我们常用的设计模式之中。
---

# 委托

## 类委托

Kotlin有一个叫做**类委托（class delegation）**的特性，它是**委托模式（Delegation pattern）**和**装饰器模式（Decorator pattern）**的重要实现途径，这两种设计模式在Java中存在很多年了，不过在Java中实现需要非常多的样板代码，而Kotlin作为一个对这些设计模式提供原生支持的语言，将所需的样板代码降到了最少。

### 委托模式(Delegation pattern)

在面向对象编程中，委托模式是实现继承的另一种方式，委托就是一个对象通过委托另一个对象来处理请求，而不是继承已有的对象。

如果我们使用Java风格的写法，为了支持多态，委托类和被委托类都需要实现所有被委托的方法和属性：

```kotlin
interface Player {
    fun playGame()
}

class RpgGamePlayer(val enemy: String) : Player {
    override fun playGame() {
        println("Killing $enemy")
    }
}

class WitcherPlayer(enemy: String) : Player {
    val player = RpgGamePlayer(enemy) 
    override fun playGame() {
        player.playGame() 
    }
}

// 使用
RpgGamePlayer("monsters").playGame() // 打印结果: Killing monsters
WitcherPlayer("monsters").playGame() // 打印结果: Killing monsters

```

之所以被称作是委托，是因为`WitcherPlayer`通过了一个`RpgGamePlayer`对象实例委托了定义在`Player`接口中的方法。使用继承也可以实现类似的效果：

```kotlin
class WitcherPlayer() : RpgGamePlayer()
```

初看起来，这两种实现方式差不多，使用继承甚至更简单一些。一方面，继承使用的更广，在Java中更常见，并且用于多种面向对象的设计模式之中；另一方面，有很多有影响力的书都推荐使用委托来替代继承，比如《Design Patterns》、《Effective Java》，下面是这种做法的一些基本论点：

* 在通常情况下，类并不是设计用来继承的。当我们覆写方法时，我们不知道所继承的类内部发生了怎样的变化，会影响哪些属性，对象、状态等，我们完全有可能因为不清楚类内部的逻辑，导致子类出现与父类不一致的行为。只有一小部分的类设计合理并有完备的文档来指引如何继承它们。
* 在Java中，可以委托多个类，但只能继承一个类。
* 通过接口，我们指定了想要委托的方法和属性，这符合**接口隔离原则（interface segregation principle）**，我们不应该向客户端暴露不必要的方法。
* 一些类是`final`类型的，我们只能够通过委托来使用它们的方法，实际上，不是设计为继承的类都应该是`final`的，Kotlin的设计者注意到了这点，所有Kotlin中的类默认是`final`的。这在设计公共库的时候更是如此，我们可以更改`final`类的实现而不必担心影响库的用户。

当然，使用委托也有一些负面影响：

* 我们需要单独创建一个接口来描述需要委托的方法。
* 我们不能访问`protected`修饰的方法和属性。

从上面的示例中还可以看到，使用委托需要写大量的样板代码，可能这是在Java中选择使用继承的另一个原因。不过，在Kotlin及众多的现代编程语言中，有更简单的方式实现委托模式：

```kotlin
class WitcherPlayer(enemy: String) : Player by RpgGamePlayer(enemy) {}
```

使用by关键字，可以通知编译器通过创建`RpgGamePlayer`对象委托`Player`接口中的所有方法，在`WitcherPlayer`构建时，被委托的`RpgGamePlayer`实例就会创建。还有一种方式时通过持有被委托对象的实例：

```kotlin
class WitcherPlayer(player: Player) : Player by player
```

这样做一个巨大的提升是：我们不用在自己实现被委托的方法，并且当方法的签名被改变的时候，我们也不需要做出修改，使得委托实现更容易维护了。

### 装饰器模式(Decorator pattern)

Kotlin的类委托另一个非常有用的场景是用来实现装饰器模式，装饰器模式（也称为包装器模式）是一种设计模式，可以向现有类添加行为，而无需使用继承。这种设计模式的经典结构可以用如下UML图来表示：

<figure><img src=".gitbook/assets/Decorator_UML_class_diagram.svg.png" alt=""><figcaption><p>装饰器模式的经典UML图实现，图片来自网络</p></figcaption></figure>

在Java世界中，最常见的一个装饰器模式的例子便是`InputStream`，这个类有许多被继承和装饰的其他类型，有些添加了缓存能力，有些添加了解压文件内容的能力，有些则提供了反序列化能力。让我们看一个例子——读取一个压缩文件的内容并将其转换为一个Java对象：

```java
FileInputStream fis = new FileInputStream("targetFile.gz"); // 创建一个简单流读取文件
BufferdInputStream bis = new BufferedInputStream(fis); // 创建一个包含缓冲功能的新流
GzipInputStream gis = new GzipInputStream(bis); // 创建一个新流使其拥有读Gzip压缩格式的能力
ObjectInputStream ois = new ObjectInputStream(gis); // 创建一个新流，用户有反序列化的能力，反序列化之前使用ObjectOutputStream输出的数据
TargetObject targetObject = (TargetObject)ois.readObject(); // 将数据最终反序列化为一个Java对象
```

我们可以看到使用装饰器模式我们可以随意指定我们想要的功能并能决定以什么样的顺序使用，相比较于继承来说要灵活许多。有些人认为可能将`InputStream`的所有功能放到一个大类里面，并且设计一些开关控制功能的开启和关闭要更好。这样一来使用可能会方便一些，不过它违反了**单一职责原则（single-responsibility principle）**，并且会把这个类的逻辑变得更复杂和难以理解。

既然装饰器模式被认为是一个好的实践，那么在Java代码中似乎不太容易见到。原因之一就是因为它实现起来并不简单，甚至有一些琐碎。而Kotlin语言可以改善这一点，假设我们要在列表的第一个位置添加一个静态元素，用委托可以这样做：

<pre class="language-kotlin"><code class="lang-kotlin"><strong>class ZeroElementListDecorator(val arrayAdapter: ListAdapter) : ListAdapter by arrayAdapter {
</strong>    override fun getCount(): Int = arrayAdapter.count + 1
    
    override fun getItem(position: Int): Any? = when {
        position == 0 -> null
        else -> arrayAdapter.getItem(position - 1)
    }
    
    override fun getView(position: Int, convertView: View?, parent:ViewGroup): View 
        = when {
            position == 0 -> parent.context.inflator.inflate(R.layout.null_element_layout, parent, false)
            else -> arrayAdapter.getView(position - 1, convertView, parent)
        }
    }
    
    override fun getItemId(position: Int): Long = when {
        position == 0 -> 0
        else -> arrayAdapter.getItemId(position - 1)
<strong>    }
</strong>}

// 使用
val arrayList = findViewById(R.id.list) as ListView
val list = listOf("A", "B", "C")
val arrayAdapter = ArrayAdapter(this, android.R.layout.simple_list_item_1, list)
arrayList.adapter = ZeroElementListDecorator(arrayAdapter)
</code></pre>

初看起来我们覆写了四个方法仍然不算特别简单，不过如果要用java写的话需要覆写十个以上。用Kotlin写就像在继承一个类一样，节省了很多重复的工作。

## 属性委托

Kotlin中不仅允许类委托也允许属性委托，我们接下来就看一下标准库中的属性委托，并尝试自定义的属性委托。

我们首先看一个属性委托的例子：

<pre class="language-kotlin"><code class="lang-kotlin">class User(val name: String, val surname: String)
var user: User by UserDelegate() // 通过实例UserDelegate来委托user属性
user = User("Mark", "Liu")
<strong>println(user.name)
</strong> 
<strong>class UserDelegate {
</strong>    operator fun getValue(thisRef: Any?, property: KProperty&#x3C;*>):User = readUserFromFile()
    operator fun setValue(thisRef: Any?, property: KProperty&#x3C;*>, user:User) {
        saveUserToFile(user)
    }
}
</code></pre>

`thisRef`代表的是使用这个委托类的上下文，如果仅在`Activity`中使用这个委托类，可以这样写：

```kotlin
class UserDelegate {
    operator fun getValue(thisRef: Activity, property: KProperty<*>):User 
        = thisRef.intent.getParcelableExtra("com.example.userkey")
}
```

如果需要在顶层文件中使用属性委托，那么`thisRef`的类型应该定义为`Nothing?`，因为这个参数将永远为`null`。

另一个参数`property`包含了被委托属性的元数据（属性名、数据类型等等）。

### 预定义的委托

Kotlin标准库中有许多非常趁手的属性委托，让我们看下它们是怎么实现的。

#### lazy

有些时候我们不希望一开始就初始化，而希望我们在第一次使用时进行初始化：

```kotlin
val someProperty by lazy { generate() }
```

`lazy`的实现：

```kotlin
public fun <T> lazy(initializer: () -> T): Lazy<T> = SynchronizedLazyImpl(initializer)
```

`SynchronizedLazyImpl`实现是线程安全的，某些场景下，我们也可以手动移除线程安全机制来提高效率：

```kotlin
val someProperty by lazy(LazyThreadSafetyMode.NONE) { generate() }
```

使用`lazy`委托有如下好处：

* 加快类加载的速度，提高应用的启动时间，因为初始化延后到了第一次使用时
* 一些值可能在应用的某些场景，或者用户特定的使用习惯下永远不会初始化，这样我们就节省了系统资源（内存，处理器时间片， 电量）

#### notNull

`notNull`委托的用法非常简单，作用是定义一个变量不为空，并且不在类构造的时候创建它：

```kotlin
var someProperty: SomeType by notNull()
```

达成这个目的的另一个方法是使用`lateinit`：

```kotlin
lateinit var someProperty: SomeType
```

`lateinit`和`notNull`的差异主要在于：

* `lateinit`比`notNull`执行效率高，所以能用`lateinit`的时候就不用`notNull`；
* `lateinit`不能用于原生数据类型和顶层属性。

如果我们在使用它们声明的变量时为空，则会抛出`IllegalStateException`，因此，只有在确定我们第一次使用时这个变量会被赋值时再使用。

请看`notNull`的实现：

```kotlin
public fun <T: Any> notNull(): ReadWriteProperty<Any?, T> = NotNullVar()

private class NotNullVar<T: Any>() : ReadWriteProperty<Any?, T> { // 私有类
    private var value: T? = null
    public override fun getValue(thisRef: Any?, property: KProperty<*>): T {
        return value ?: throw IllegalStateException("Property${property.name} should be initialized before get.") // 为空时会抛出异常，一场携带的消息与lateinit不同
    }
    public override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
        this.value = value
    }
}
```

#### observable

这个委托的作用是观测每次变量的变化（`setValue`被调用的时候），示例如下：

```kotlin
var name: String by Delegates.observable("Empty"){
    property, oldValue, newValue -> println("$oldValue -> $newValue") // property未使用，也允许使用_来替代
}

// 使用
name = "Mark" // 打印结果: Empty -> Mark
name = "Andecy" // 打印结果: Mark-> Andecy
name = "Andecy" // 打印结果: Andecy -> Andecy
```

在Android开发中，我们经常会监听ListView相关联的列表数据的变化，来刷新视图，这个场景可以用`observable`委托：

```kotlin
var list: List<LocalDate> by observable(list) { _, old, new ->
    if(new != old) notifyDataSetChanged()
}
```

#### vetoable

`vetoable`的功能和`observable`类似，区别在于它是在值被设置之前触发的，并且可以决定接纳或者拒绝这个值：

```kotlin
var list: List<String> by Delegates.vetoable(emptyList()){ _, old, new ->
    new.size > old.size // 只有在新值的大小大于旧值时才接受，如果不满足，变量的值将不变
}
```

#### Map类型的属性委托

请看示例：

```kotlin
class User(map: Map<String, Any>) { // map的键类型需要是String，用于和属性名关联，值的类型不限
    val name: String by map
    val kotlinProgrammer: Boolean by map
}
 
// 使用
val map: Map<String, Any> = mapOf( // 创建map，键的名字对应被委托的属性名称
    "name" to "Mark",
    "kotlinProgrammer" to true
)
 val user = User(map) 
 println(user.name) // 打印结果: Mark
 println(user.kotlinProgrammer) // 打印结果: true
```

使用`map`来委托属性，可以简化属性的取值并且易于扩展，在`map`类型数据向对象转换时非常方便。那么它是如何做到的呢？请看其简化版的实现：

```kotlin
operator fun <V, V1: V> Map<String, V>.getValue( // V是被委托的属性数据类型
    thisRef: Any?, // 委托可用的上下文
    property: KProperty<*>): V1 { // 返回类型，需要是V的子类型
    val key = property.name // 属性名是map的key
    val value = get(key)
    if (value == null && !containsKey(key)) {
        throw NoSuchElementException("Key ${property.name}is missing in the map.")
    } else {
        return value as V1
    }
}

operator fun <V> MutableMap<String, V>.setValue(
    thisRef: Any?,
    property: KProperty<*>,
    value: V) {
    put(property.name, value)
}

```

看了这些标准库中的委托后，相信已经对委托的一般实现方法和用法有了大致的了解。接下来我们看一些自定义委托的例子，看看委托是如何运用到Android开发中的。

### 自定义委托

上面的例子中`lazy`是不可变的，只能用于不可变量`val`，如果我们想自定义一个可变属性的延迟初始化委托，应该如何写呢？

我们首先回顾下属性委托的实现方法签名，以下根据只读属性委托和可读可写属性委托的两种情况，分别整理成了两个通用接口：

<pre class="language-kotlin"><code class="lang-kotlin"><strong>public interface ReadOnlyProperty&#x3C;in R, out T> {
</strong>    public operator fun getValue(thisRef: R, property: KProperty&#x3C;*>): T
}

public interface ReadWriteProperty&#x3C;in R, T> {
    public operator fun getValue(thisRef: R, property: KProperty&#x3C;*>): T
    public operator fun setValue(thisRef: R, property: KProperty&#x3C;*>, value: T)
}
</code></pre>

那么我们可以这样实现：

```kotlin
fun <T> mutableLazy(initializer: () -> T): ReadWriteProperty<Any?, T> =
    MutableLazy<T>(initializer) //委托的实现类
    
private class MutableLazy<T>(val initializer: () -> T) :
    ReadWriteProperty<Any?, T> { // 可读可写的属性
    private var value: T? = null // 存储变量
    private var initialized = false
    
    override fun getValue(thisRef: Any?, property: KProperty<*>): T { // 没有使用operator修饰因为已经在接口中修饰
        synchronized(this) {
            if (!initialized) {
                value = initializer()
            }
            return value as T
        }
    }
    override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
        synchronized(this) {
            this.value = value
            initialized = true
        }
    }
}

// 使用
var gameMode : GameMode by mutableLazy {
    getDefaultGameMode()
}
var mapConfiguration : MapConfiguration by mutableLazy {
    getSavedMapConfiguration()
}
var screenResolution : ScreenResolution by mutableLazy {
    getOptimalScreenResolutionForDevice()
}

```

#### 视图绑定

在采用**MVP（Model-View-Presenter）**架构的Android项目开发中，我们经常需要使用Presenter来改变视图，我们可能会写出下面的代码：

```kotlin
// 接口声明
interface MainView {
    fun getName(): String
    fun setName(name: String)
}

// Presenter的实现
override fun getName(): String {
    return nameView.text.toString()
}

override fun setName(name: String) {
    nameView.text = name
}

```

使用属性委托我们可以简化为如下代码：

```kotlin
// 接口声明
interface MainView {
    var name: String
}

// Presenter的实现
override var name: String by bindToText(R.id.textView)
```

`bindToText`是如何实现的呢？

```kotlin
fun Activity.bindToText(@IdRes viewId: Int ) 
    = object : ReadWriteProperty<Any?, String> {
    val textView by lazy { findViewById<TextView>(viewId) }
    override fun getValue(thisRef: Any?, property: KProperty<*>): String {
       return textView.text.toString()
    }
    override fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
       textView.text = value
    }
}
```

同理我们也可以实现一个改变视图可见性的委托：

```kotlin
fun Activity.bindToVisibility(@IdRes viewId: Int ) 
    = object :ReadWriteProperty<Any?, Boolean> {
        val view by lazy { findViewById(viewId) 
        override fun getValue(thisRef: Any?, property: KProperty<*>): Boolean {
            return view.visibility == View.VISIBLE
        }
        override fun setValue(thisRef: Any?,
            property: KProperty<*>, value: Boolean) {
            view.visibility = if(value) View.VISIBLE else View.GONE
        }
}
```

