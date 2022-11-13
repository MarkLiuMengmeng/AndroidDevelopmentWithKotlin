---
description: >-
  在前面的章节，大多数的概念是Java开发者很熟悉的。本章中我们要介绍一个Java中没有的全新特性——扩展，Kotlin中非常吸引人的特性之一，使用好它将会为Android开发带来巨大的便利。
---

# 扩展函数与属性

## 扩展函数

所有大型的Java项目中，都会有许多工具类（`Utils`类），比如：`StringUtils`, `ScreenUtils`, and `ToastUtils`。这些工具类会把通用的功能代码抽离出来，方便我们使用，也方便测试。问题在于，Java在创建和使用这些类时支持得不好，比如我们要弹出一个`Toast`，通常用于显示一个提示，用Java我们会这么写：

```java
Toast.makeText(context, text, Toast.LENGTH_SHORT).show();
```

代码很基础，不过很可能大部分的开发者都曾经有过忘记调用`show`方法的经历，花费了大量时间去检查周边的逻辑条件定位它为什么没有正常运行。

为了避免我们犯这种低级错误，在Java中有一种解决办法是把它写成`Utils`类：

```java
public class ToastUtils {
    public static void toast(Context context, String text) {
        Toast.makeText(context, text, Toast.LENGTH_SHORT).show();
    }
}

// 使用
ToastUtils.toast(context, "Hello!");
```

不过，如果要使用这个工具类，你需要记住有这么一个方法，位于哪个类里。本质上来说，并没有比原始的写法更简单。更简单的方法？我们可以用Kotlin的扩展特性实现：

```kotlin
fun Context.toast(text: String) { // Context位于函数名之前，声明为Context的扩展函数
    Toast.makeText(this, text, LENGTH_LONG).show() // 在函数体中我们可以用this关键字引用Context实例
}
 // 使用
 context.toast("Hello!")
```

因为Activity是Context的子类，因此我们可以这样调用：

```kotlin
class MainActivity :Activity() {
    override fun onCreate(savedInstanceState: Bundle?){
        super.onCreate(savedInstanceState)
        toast("Hello!")
    }
}
```

上面举的例子中`Context`是作为扩展函数的接收者，我们可以使用`this`关键字访问接收者的实例，同样它也可以被隐式调用：

```kotlin
// 显示调用
fun Collection<Int>.dropPercent(percent: Double)
    = this.drop(floor(this.size * percent)
// 隐式调用
fun Collection<Int>.dropPercent(percent: Double) = drop(floor(this.size * percent)
```

扩展函数甚至可以定义在`Any?`类型上，这样我们就可以在任意类的实例中使用该函数：

```kotlin
fun Any?.logError(error: Throwable, message: String = "error") {
    Log.e(this?.javaClass?.simpleName ?: "null", message, error)
}

// 使用
user.logError(e, "Password Error") 
logError(e)
```

### 扩展函数的实现原理

扩展函数看起来可能相当神奇，不过背后的原理非常简单——在编译时扩展函数相当于一个静态方法，被扩展的类实例是这个静态方法的第一个参数。

比如我们之前定义过的`toast`扩展函数：

{% code title="ContextExt.kt" %}
```kotlin
fun Context.toast(text: String) {
    Toast.makeText(this, text, LENGTH_LONG).show()
}
```
{% endcode %}

当它编译后再反编译为Java时，看起来时这样的：

```java
public class ContextExtKt {
    public static void toast(Context receiver, String text) {
        Toast.makeText(receiver, text, Toast.LENGTH_SHORT).show();
    }
}
```

这也是为什么我们可以在Java代码中引用这个扩展函数的原因：

```java
// Java
ContextExtKt.toast(context, "Some toast")
```

这意味着，从JVM字节码的角度来看，扩展方法从未添加到`Context`类中。在编译期，所有使用扩展函数的代码均会被替换为静态方法调用。

因为扩展函数本质是一个静态函数，因此其他函数适用的特性也适用于扩展函数，比如`inline`修饰符、单表达式语法、命名参数等，这也带来了一些不太直观的问题，且看下回分解。

#### 扩展函数会覆写成员函数吗？

答案是不会。当一个类的成员函数和扩展函数拥有同样的方法名和参数时，以成员函数为准。

```
class A {
    fun test() {
        println("test from A")
    }
}
fun A.test() {
    println("test from Extension")
}
 A().test() // 打印: test from A
```

扩展函数不能更改原对象的行为，只允许增加额外的功能。这样的机制可以避免产生难以预料的行为。

#### 扩展函数中的类引用能否够提高可见性？

答案是不能。虽然扩展函数用起来像是原来的类中声明的一部分一样，但请记住它只是一个语法糖，仍然会受到可见性修饰符的限制，我们不能访问被扩展类中的`private`，`protected`修饰的元素，至于其他修饰符（`default`、`package`、`internal`），可见性仍然和普通对象的引用一样。

#### 扩展函数中的类拥有多态性吗？

答案是没有。扩展函数是在编译期被替换为静态方法的，因此实现多态性的基础不存在，多态实现依赖运行时的方法表，因此，扩展函数的行为以编译期确定的类型为准：

```java
 abstract class A
 class B: A()
 
 fun A.test() = println("test from A")
 fun B.test() = println("test from B")
 
 val b = B()
 b.test() // 打印: test from B
 (b as A).foo() // 打印: test from A
 val a: A = b
 a.foo() // 打印: test from A
```

这里的特性可能会引起问题，因为我们编写代码时常常转型，由于多态性在Java开发者中是非常深入人心的，所以可能会导致预期之外的行为。

为避免这一点，我们不应该同时为父类和子类增加相同的扩展方法，如果非要这样做，请在使用时留意要转型为对应的类型。同时，在开发一个library的时候，我们对外的API应该尽量避免增加扩展方法，以避免产生预期以外的行为或屏蔽引用者的实现。

#### 可以为伴随对象添加扩展函数吗？

答案是可以。前提是我们要在类中显式声明伴随对象，即使是空实现：

```kotlin
class A {
    companion object {}
}
fun A.companion.foo(){ print("function in companion") }
// 调用
A.foo()
```

#### 扩展函数可以帮助实现运算符重载吗？

答案是可以。运算符重载是Kotlin中最重要的特性之一，但需要声明满足要求的函数。很多时候我们使用的第三方库是没有满足这个条件的，比如RxJava中，我们常常用`CompositeDisposable`来管理订阅：

```kotlin
val subscriptions = CompositeDisposable()
subscriptions.add(repository
    .getAllCharacters(qualifiedSearchQuery)
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(this::charactersLoaded, view::showError))
```

如果我们用扩展函数定义了满足要求的运算符重载函数，就可以使用运算符重载特性：

```kotlin
operator fun CompositeDisposable.plusAssign(disposable: Disposable) {
    add(disposable)
}

// 使用
subscriptions += repository
    .getAllCharacters(qualifiedSearchQuery)
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(this::charactersLoaded, view::showError)
```

### 泛型扩展函数

在Android开发中，启动一个Activity是一个很普遍的行为，用Kotlin我们可能会这么写：

```kotlin
startActivity(Intent (this, SettingsActivity::class.java))
```

我们有时也在项目中会写关于启动Activity的工具类，如果我们用泛型扩展函数来写，事情会变得更简单：

```kotlin
inline fun <reified T : Any> Context.getIntent()
    = Intent(this, T::class.java)
inline fun <reified T : Any> Context.startActivity()
    = startActivity(getIntent<T>())
    
// 使用
startActivity<SettingsActivity>()

```

在开发工作中还有一项很常见的工作是序列化与反序列化JSON数据，以我们常用的Gson库来举例，经过泛型扩展后也更加易用：

```kotlin
inline fun Any.toJson() = globalGson.toJson(this)!!
inline fun <reified T : Any> String.fromJson()
    = globalGson.fromJson(this, T::class.java)

// 用法
val user = User("Mark", "Liu")
val json: String = user.toJson()
val userFromJson: User = json.fromJson<User>()


```

globalGson是一种常见做法，因为我们经常需要序列化与反序列化JSON数据，只定义和实例化一次是一种简单并有效的做法。

### 扩展函数在集合中的应用

集合接口在Kotlin中只定义了少量的方法，Kotlin标准库通过扩展为它们又增添了些有用的功能。

#### map、filter、flatMap函数

`map`函数提供了根据传入的函数参数改变集合元素的功能：

```kotlin
val list = listOf(1,2,3 ).map { it * 2 }
println(list) // 打印结果: [2, 4, 6]
```

`filter`函数只允许满足入参断言（predicate）的元素返回：

```kotlin
val list = listOf(1,2,3,4,5).map { it > 2 }
println(list) // 打印结果: [3, 4, 5]
```

`flatmap`函数将会返回一个给定`transform`函数生成的所有元素的单列表，经常用于合并多个集合列表：

```kotlin
val list = listOf(10, 20).flatMap { listOf(it, it+1, it + 2) }
println(list) // 打印结果: [10, 11, 12, 20, 21, 22]
```

让我们看下这些扩展函数的简易实现：

```kotlin
inline fun <T, R> Iterable<T>.map(transform: (T) -> R): List<R> {
    val destination = ArrayList<R>()
    for (item in this) destination.add(transform(item))
    return destination
}

inline fun <T> Iterable<T>.filter(predicate: (T) -> Boolean): List<T> {
    val destination = ArrayList<T>()
    for (item in this) if(predicate(item)) destination.add(item)
    return destination
}

inline fun <T, R> Iterable<T>.flatMap(transform: (T) -> Collection<R>):List<R> {
    val destination = ArrayList<R>()
    for (item in this) destination.addAll(transform(item))
    return destination
}
```

大多数Kotlin标准库的扩展函数的函数类型是内联，因为这样可以使lambda表达式使用起来更有效率。

#### forEach和onEach函数

`forEach`函数是`for`循环语句的替代选择：

```kotlin
listOf("A", "B", "C").forEach { print(it) } // 打印结果: ABC
```

自Kotlin1.1起，新增了一个相似的`onEach`函数，用于处理流过程中的遍历，常用于日志打印：

```kotlin
(1..10).filter { it % 3 == 0 }
    .onEach(::print) // 打印结果: 369
    .map { it / 3 }
    .forEach(::print) // 打印结果: 123
```

#### withIndex与indexed变体

有时，如何处理集合中的元素取决于元素的序号，我们可以使用`withIndex`函数返回包含序号的列表：

```kotlin
listOf(9,8,7,6).withIndex()
    .filter { (i, _) -> i % 2 == 0 } 
    .forEach { (i, v) -> print("$v at $i,") }
    // 打印结果: 9 at 0, 7 at 2,
```

我们也可以使用其他提供序号的变体：

```kotlin
val list1 = listOf(2, 2, 3, 3)
    .filterIndexed { index, _ -> index % 2 == 0 }
    println(list1) // 打印结果: [2, 3]
    
val list2 = listOf(10, 10, 10)
    .mapIndexed { index, i -> index * i }
    println(list2) // 打印结果: [0, 10, 20]
    
val list3 = listOf(1, 4, 9)
    .forEachIndexed { index, i -> print("$index: $i,") }
    println(list3) // 打印结果: 0: 1, 1: 4, 2: 9
```

#### sum、count、min、max和sorted等函数

`sum`函数用于计算所有元素的和，可用于`List<Int>`,  `List<Long>`,  `List<Short>`,  `List<Double>`,  `List<Float>` 和`List<Byte>`:

```kotlin
val sum = listOf(1,2,3,4).sum()
println(sum) // 打印结果: 10
```

有时我们如果想计算列表的某个属性值相加，比如用户分数的和，可能会这么做：

```kotlin
class User(val points: Int)
val users = listOf(User(10), User(1_000), User(10_000))

val points = users.map { it.points }. sum()
println(points) // 打印结果: 11010


```

实际上，我们不需要调用`map`函数处理集合，而使用`sumBy`函数指定一个适宜的选择器：

```kotlin
val points = users.sumBy { it.points }
println(points) // 打印结果: 11010
```

`sumBy`函数默认要求Int返回类型，对于`Double`数据类型的相加需要使用对应的`sumByDouble`函数。

`count`函数可以对元素进行计数：

```kotlin
val evens = (1..5).count { it % 2 == 1 }
val odds = (1..5).count { it % 2 == 0 }
println(evens) // 打印结果: 3
println(odds) // 打印结果: 2
val nums = (1..5).count()
println(nums) // 打印结果: 5
```

`min`和`max`函数可以返回列表中的最大最小值，前提是列表中的元素有自然顺序（实现了`Comparable<T>`接口）：

```kotlin
val list = listOf(4, 2, 5, 1)
println(list.min()) // 打印结果: 1
println(list.max()) // 打印结果: 5
println(listOf("koki", "adai", "bali", "mall").min()) // 打印结果: ada
```

`sorted`函数同样也需要有自然顺序，返回一个排序好的结果：

```kotlin
val strs = listOf("c", "d", "b", "a").sorted()
println(strs) // 打印结果: [a, b, c, d]
```

但是，如果遇到元素不可比较怎么办，下面举例三种解决方式：

```kotlin
// 1.按照元素中的某个可比较的属性处理
students.filter { it.passing }.sortedByDescending { it.grade }

val minByLen = listOf("ppp", "z", "as").minBy { it.length }
println(minByLen) // 打印结果: "z"

val maxByLen = listOf("ppp", "z", "as").maxBy { it.length }
println(maxByLen) // 打印结果: "ppp"

// 2.转换为字符串处理
val list = listOf(14, 31, 2)
print(list.sortedBy { "$it" }) // 打印结果: [14, 2, 31]

// 3.使用sortedWith函数指定一个比较器实现
val comparator = Comparator<String> { e1, e2 ->
    e2.length - e1.length
}
val minByLen = listOf("ppp", "z", "as").sortedWith(comparator)
println(minByLen) // 打印结果: [ppp, as, z]


```

而创建比较器，Kotlin标准库中也有对应的函数实现（`compareBy`、`compareByDescending`等）。另外有一个重要的函数`groupBy`，传入生成key的方法，返回一个对应的`Map`数：

```kotlin
val grouped = listOf("ala", "alan", "mulan", "malan")
    .groupBy { it.first() }
println(grouped) // 打印结果: {'a': ["ala", "alan"], "m": ["mulan", "malan"]}
```

让我们再看一个复杂的例子，给出一个学生列表，返回这些学生所在班级上成绩最好的学生：

```kotlin
class Student(val name: String, val classCode: String, val meanGrade:Float)

val students = listOf(
    Student("Homer", "1", 1.1F),
    Student("Carl", "2", 1.5F),
    Student("Donald", "2", 3.5F),
    Student("Alex", "3", 4.5F),
    Student("Marcin", "3", 5.0F),
    Student("Max", "1", 3.2F)
)

val bestInClass = students
    .groupBy { it.classCode }
    .map { (_, students) -> students.maxBy { it.meanGrade }!! }
    .map { it.name }
print(bestInClass) // 打印结果: [Max, Donald, Marcin]

```

## 扩展属性

我们还可以为某个类定义扩展属性:

```kotlin
val TextView.trimmedText:String
get() = text.toString.trim()

// 用法
textView.trimmedText
```

唯一的限制是，扩展属性不能有幕后字段（backing field）, 因此也没有对应的字段存储状态，下面的赋值无法完成：

```kotlin
val TextView.trimmedText:String = "test" // 错误
```
