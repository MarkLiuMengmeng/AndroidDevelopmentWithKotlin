---
description: >-
  Java是一门面向对象的语言，虽然Java8中引入了lambda表达式，我们可以在Android
  Studio中通过诸如Retrolambda这样的插件来使用它，但Kotlin支持更多更先进的函数式编程的特性，本章将介绍Kotlin中的这一部分。
---

# 函数式编程

## 函数类型

函数在Kotlin中是**一等公民**（first-class citizen），一等公民是一个编程语言术语，指的是一个**实体**（entity）支持所有的常规操作，这些操作一般指的是作为函数参数类型、作为函数返回类型和赋值给变量。

由于Kotlin是静态类型语言，我们需要一种函数类型的记法来实现函数作为一等公民的操作，记法由函数的参数类型列表加上`->`和返回类型表示：

`参数类型列表(types of parameters)->返回类型(return type)`

```kotlin
lateinit var a: (Int) -> Int // 以Int(数据类型)为参数并返回Int的函数
lateinit var b: ()->Int // 不接收参数返回Int的函数
lateinit var c: (Int)->Unit // 以Int为参数不带返回类型的函数
lateinit var d: (Int,Int)->String //以两个Int为参数并返回String的函数

fun printInt(value:Int) = println(value)

val c = ::printInt // 此操作符用来引用该函数
c(10) // 可以像调用函数那样使用这个变量，打印结果：10
```

上面我们说函数是一等公民，那么除了指定变量类型及赋值给变量之外，还可以作为函数参数类型和返回类型，我们直接将函数类型的语法替换到相应的参数类型或者返回类型即可：

```kotlin
lateinit var e: (String)->(Int)->Int //以String为参数，返回(Int)->Int类型
lateinit var f: (()->Int)->String //以()->Int为参数，返回String类型
```

函数类型也能应用在泛型中，我们可以将函数保存为一个待执行列表依次执行：

```kotlin
var todoList: List<() -> Unit> = // ...
for (task in todoList) task()
```

### 函数类型的底层实现

函数类型实际上是一个泛型接口的语法糖，让我们看一些例子：

* `()->Int`：对应的是`Function0<Int>`接口，之所以是`Function0`是因为它有0个参数，之所以是`Int`是因为返回类型是`Int`。
* `Int->Int`：对应的是`Function1<Int,Int>`接口，之所以是`Function1`是因为它有一个参数
* `()->(Int, Int)->String`：对应的是`Function0<Function2<Int,Int,String>>`。

所有的接口都只有一个方法`invoke`，同时它也是一个操作符，允许一个对象像函数那样使用。下面两种使用方式是等价的：

```kotlin
val a: (Int) -> Unit = //...
a(10) // 写法1
a.invoke(10) // 写法2
```

需要注意的是写法2可以结合使用安全调用操作符，当存储函数的变量可能为空时，我们最好这么使用它：

```kotlin
var a: ((Int) -> Int)? = null
if (false) a = fun(i: Int) = i * 2
print(a?.invoke(4)) // 打印结果: null
```

`Function`接口不存在于标准库中，它是一个**编译器合成类型**（synthetic compilergenerated type），在编译的时候才会生成，因此没有人为地限制参数的数目，同时也没有增加标准库的体量。

### 函数引用

有些时候我们需要将某个函数作为值来传递，我们在本节一开始已经用过一次，表示函数引用的语法是：

`::函数名(function name)`

来看一个简单的例子：

```kotlin
fun printInt(value:Int) = println(value)

val c = ::printInt
```

函数的引用使用的是反射，这就是为什么引用中还包含着函数的其他信息：

```kotlin
val c = ::printInt
val annotations = c.annotations
val parameters = c.parameters
println(annotations.size) // 打印结果: 0
println(parameters.size) // 打印结果: 1
```

我们还可以引用某个类的方法，其语法是：

`类型名(type name)::函数名(functionName)`

下面是一个示例：

```kotlin
val nonEmpty = listOf("A", "", "B", "").filter(String::isNotEmpty)
print(nonEmpty) // 打印结果: ["A", "B"]
```



## 匿名函数

匿名函数和普通函数的作用一样，区别是匿名函数定义的时候在`fun`关键字和参数列表声明之间没有函数名，一般把它当成对象来看待：

```kotlin
val a: (Int) -> Int = fun(i: Int) = i * 2 // 单表达式匿名函数
val b: ()->Int = fun(): Int { return 4 } // 带函数体的匿名函数
val c: (String)->Unit = fun(str: String){ println(str) }
//使用示例
// Usage
println(a(4)) // 打印结果: 8
println(b()) // 打印结果: 4
c("Kotlin") // 打印结果: Kotlin
```

我们提到过的Kotlin的类型推断在这里也能发挥作用，我们可以省略变量的类型声明：

```kotlin
var a = fun(i: Int) = i * 2
var b = fun(): Int { return 4 }
var c = fun(str: String){ println(str) }
```

甚至可以反过来使用：

```kotlin
val a: (Int) -> Int = fun(i) = i * 2 
val b: ()->Int = fun() = 4 //这里没有用函数体的花括号，如果用了将会推断为()->Unit类型
val c: (String)->Unit = fun(str){ println(str) }
```

让我们看一个在Android项目中使用匿名函数记录程序产生的异常的例子：

```kotlin
val TAG = "MainActivity"
val errorHandler = fun (error: Throwable) {
    if(BuildConfig.DEBUG) {
        Log.e(TAG, error.message, error)
    }
    toast(error.message)
    Crashlytics.logException(error)
}
// 使用示例
val error = Error("ExampleError")
errorHandler(error) // 输出结果: MainActivity: ExampleError
```

匿名函数提供了一种简单的方式像对象那样传递和使用一个函数，不过我们可以用lambda表达式更简单地实现相同的行为。

## Lambda表达式

最简单地定义匿名函数的方法是使用Kotlin的lambda表达式特性，Kotlin中的lambda表达式和Java8中的很相似，但最大的区别是Kotlin中的lambda表达式是[闭包](https://blog.csdn.net/wuwenxiang91322/article/details/9092569)（closure），允许我们在创建的上下文中修改变量，Java8中则不行。Kotlin中lambda表达式的记法如下：

`{ 参数列表(arguments) -> 函数体(function body) }`

lambda表达式会返回表达式的最后一句来作为`return`语句的替代：

* `{ 1 }`：没有参数返回1的lambda表达式，它的是`()->Int`。
* `{ s: String -> println(s) }`：以一个`String`类型为参数并打印出来lambda表达式，它的类型是 `(String)->Unit`。
* `{ a: Int, b: Int -> a + b }`：以两个`Int`类型为参数计算它们之和的lambda表达式，它的类型是`(Int, Int)->Int`。

我们在前一节定义的匿名函数同样可以使用lambda表达式定义：

```kotlin
var a: (Int) -> Int = { i: Int -> i * 2 }
var b: ()->Int = { 4 }
var c: (String)->Unit = { str: String -> println(str) }
```

由于默认情况下lambda表达式会采用最后一句作为返回值，因此除非和标签一起使用，否则不允许使用`return`语句：

```kotlin
var a: (Int) -> Int = { i: Int -> return i * 2 } // 错误，这里不允许使用return语句
var l: (Int) -> Int = l@ { i: Int -> return@l i * 2 } // 正确
```

我们可以写多行的lambda表达式：

```kotlin
val sum = { i: Int, j: Int ->
    println("calculate $i + $j")
    i + j // 该语句会作为返回值
}
```

上面的表达式非常简单，我们如果把多行的表达式写到一行，我们需要用分号隔开每条语句：

```kotlin
val sum = {i: Int, j: Int -> println("calculate $i + $j");i + j }
```

lambda表达式不仅可以使用传进来的参数，还可以访问创建上下文的数据：

```kotlin
var i = 1
val a: () -> Int = { ++i }
println (i) // 打印结果: 1
println (a()) // 打印结果: 2
println (i) // 打印结果: 2
```

Java8中的lambda表达式也可以引用上下文中的变量，但不允许改变（必须是final修饰的变量）。

Kotlin的类型推断机制同样适用于lambda表达式，我们可以指定变量的类型来推断lambda的参数类型，也可以指定lambda的参数类型推断变量的类型：

```kotlin
val c: (String)->Unit = { s -> println(s) } // 推断参数s为String类型
val c = { s: String -> println(s) } // 推断变量c的类型是(String)->Unit
```

### 单参数的隐式名称

如果lambda表达式仅有一个参数，我们可以省去lambda的参数声明并在函数体部分使用`it`关键字指代该参数：

```kotlin
val c: (String)->Unit = { println(it) } // 等同于{ s -> println(s) }
```

使用这项特性需要满足两个条件：

* lambda表达式仅有一个参数
* 参数类型可以从上下文中推断出来

隐式名称的使用可以使我们写出**LINQ**（Language Integrated Query）风格的代码，像下面这样：

```kotlin
val strings = arrayOf("a","b","abcde")
strings.filter { it.length == 5 }.map { it.toUpperCase() }// 从列表中筛选出长度为5的字符串并转换为大写
```

LINQ风格的语法在函数式编程语言中相当受欢迎，它可以使我们处理集合和字符串的代码变得非常精简。更多的示例将在第七章展开。

### 省略不使用的参数

有些时候我们并不需要lambda表达式中的所有参数，如果我们仍然将其写出，可能分散阅读者的注意力或者使阅读者尝试理解其意义。

假设我们有这样一个lambda表达式：

```kotlin
list.filterIndexed { index, value -> index % 2 == 0 }
```

Kotlin中引入了下划线记法，来替换不使用的参数，这样避免了我们关注无关的细节，也提升了可读性：

```kotlin
list.filterIndexed { index, _ -> index % 2 == 0 }
```

### 在参数列表中使用解构

在第四章中，我们已经看到了一个对象如何解构为多个变量：

```kotlin
data class Product(val name:String, val price:Double)
val productA = Product("Glove", 19.9)
var (name,price) = productA
```

自Kotlin1.1开始，我们可以在lambda表达式中的参数列表中使用解构声明。在使用时用括号括住想要解构的参数：

```kotlin
val showProductInfo: (Product) -> Unit = { (name, price) ->
    println("the price of $name is $price")
}
val productA = Product("Glove", 19.9)
showProductInfo(productA) // 打印结果： the price of Glove is 19.9
```

{% hint style="info" %}
我们在第四章中已经了解到，Kotlin中的解构声明是基于位置的，与TypeScript语言中的基于名称的解构声明不同，它们各有优缺点：

基于位置的解构声明对于属性的名称不敏感，对属性的排序敏感；

基于名称的解构声明对于属性的排序不敏感，对属性的名称敏感。
{% endhint %}

省略记法同样可以在解构声明中使用，也可以指定解构声明的类型及其中参数的类型，剩下的交给类型推断机制：

```kotlin
val f: (Product)->Unit = { (name, _) -> /* code */ }
val f = { (name, price): Product -> /* code */ }
val f = { (name:String, price:Double): Product -> /* code */ }
```

解构声明和省略记法的结合使用简化了数据对象的处理步骤，让我们可以用更少的代码更快地使用自己想要的数据。

## 高阶函数

**高阶函数**（Higher-order function）指的是以至少一个函数类型作为参数或者返回类型是参数类型的函数。Kotlin提供了对高阶函数的完整支持。

让我们先看两个简单函数，一个用来求和，一个用来求乘积：

```kotlin
fun sum(numbers: List<Int>): Int {
    var acc = 0
    for (num in numbers) {
        acc += num
    }
    return acc
}
fun multiply(numbers: List<Int>): Int {
    var acc = 1
    for (num in numbers) {
        acc *= num
    }
    return acc
}
//使用示例
val numbers = listOf(2,4,8)
println(numbers) // 打印结果：[2, 4, 8]
println(sum(numbers)) // 打印结果：14
println(multiply(numbers)) // 打印结果：64
```

我们可以看到它们之间非常相似，他们的名字、累加器（0或者1）和操作（+还是\*）有差别，整体结构还是非常相似的。如果遵循`DRY` \(Don't Repeat Yourself\) 原则，在项目中我们不应该有两块相似的代码。在面向对象的编程中，我们定义一个因使用对象不同而功能不同的函数比较容易，却比较难定义执行操作不同的相似函数（在这里，操作的不同指的是累加还是累乘）。

解决办法就是使用函数类型，我们把操作作为一个参数传进去，我们可以像这样抽出一个公共操作的方法：

```kotlin
private fun fold(
    numbers: List<Int>,
    start:Int,
    accumulator: (Int, Int) -> Int): Int {
    var acc = start
    for (num in numbers) {
        acc = accumulator(acc, num)
    }
    return acc
}

fun sum(numbers: List<Int>) = fold(numbers,0,{acc,num -> acc + num})
fun multiply(numbers: List<Int>) = fold(numbers,1,{acc,num -> acc * num})
//使用示例
val numbers = listOf(9,5,2,7)
println(numbers) // 打印结果：[9, 5, 2, 7]
println(sum(numbers)) // 打印结果：23
println(multiply(numbers)) // 打印结果：630
```

对于函数类型的参数，我们可以像使用别的普通参数那样使用它，比如我们可以使用可变参数关键字`vararg`修饰：

```kotlin
fun longOperation(vararg observers: ()->Unit) {
    //...
    for(o in observers) o()
}
//使用示例
longOperation({ notifyHeaderView() }, { notifyFooterView() })
```

这样的函数常用在观察者模式中，除此之外，高阶函数还有两种主要的应用场景：

* 为函数提供操作
* 线程操作后的回调

接下来我们详细看一下。

### 观察者模式

**观察者模式**（Observer pattern）是我们常用的一种设计模式，将对象分为观察者和被观察者，当被观察者状态发生改变时通知被观察者，观察者模式解除了两者之间的耦合，它们之间的依赖通过抽象而非具体实现。在Android开发中，观察者模式常见于视图的点击、触摸等事件监听之中，我们可以用lambda表达式写出没有更为简洁的监听实现：

```kotlin
button.setOnClickListener({ someOperation() })
```

我们如果用高阶函数来实现这个模式可以是这样的：

```kotlin
var listeners: List<()->Unit> = emptyList() // 使用一个空列表作为数据
fun addListener(listener: ()->Unit) {
    listeners += listener // 添加监听器
}
fun invokeListeners() {
    for( listener in listeners) listener() // 依次调用
}
```

### 为函数提供操作

正如我们本节一开始的例子所展示的，我们可以提取出公有的行为并进行重用，我们将操作作为参数传递进去，假设我们想从列表中按照一定条件筛选出一些元素，我们一般的做法是当判断条件命题为真的时候收集它们：

```kotlin
var visibleTasks = emptyList<Task>()
for (task in tasks) {
    if (task.active)
    visibleTasks += task
}
```

这种筛选操作是相当普遍的操作，我们可以使用高阶函数来写出所有根据命题真假筛选元素的操作：

```kotlin
fun <T> filter(list: List<T>, predicate: (T)->Boolean) {
    var resultList = emptyList<T>()
    for (elem in list) {
    if (predicate(elem))
        resultList += elem
    }
}
//使用示例
var visibleTasks = filter(tasks, { it.active })
```

### 线程操作后的回调

我们在执行一些耗时操作时，为了避免用户等待，我们经常要新开一个线程，并在操作结束后回调返回结果。我们同样可以用高阶函数做到这一点：

```kotlin
fun longOperationAsync(longOperation: ()->Unit, callback: ()->Unit) {
    Thread({ // 创建线程，将lambda表达式传入构造器
        longOperation() // 执行耗时操作的函数
        callback() // 回调函数
    }).start() // 启动线程
}
//使用示例
longOperationAsync(
        longOperation = { Thread.sleep(1000L) },
        callback = { print("After one second") }//一秒后打印，打印结果: After one second
    )
println("Now") // 立即打印, 打印结果: Now
```

实际上还有很多使用回调的可选项，比如RxJava，又或者使用经典的接口实现。

这三个场景是常见的使用高阶函数的场景，在实际开发中我们可以灵活运用减少代码中的冗余以便于维护。

### 和命名参数语法结合使用

让我们以一个从通过网络访问获取数据的例子来展示一下在Android项目中的应用：

```kotlin
fun getAndFillData(
    onStart: () -> Unit = {},// 网络访问开始前调用
    onFinish: () -> Unit = {})// 网络访问结束后调用
    {
    // ...
}
```

假设我们使用了下拉刷新，那么我们可以在上面的函数具体调用时控制进度条的展示：

```kotlin
getAndFillData(
    onStart = { view.swipeRefresh.isRefreshing = true } ,
    onFinish = { view.swipeRefresh.isRefreshing = false }
)
```

我们也可以让它静默刷新：

```kotlin
getAndFillData()
```

一般来说，我们在遇到多个有默认值的参数都应该使用命名参数语法，特别是像lambda表达式这样的函数类型，如果在传参时不加上参数名，我们不容易看出它的具体作用。

### lambda表达式处于参数列表末尾时的使用惯例

高阶函数在Kotlin中是非常重要的组成部分，因此有必要让它的使用更简单。如果函数的最后一个参数是函数，那么我们在使用lambda表达式传参时可以写在圆括号之外。我们先定义一个尾参数是函数的高阶函数：

```kotlin
fun longOperationAsync(timeout: Long, callback: ()->Unit) {
    // ...
}
```

对于这样的函数，我们在使用时就可以将作为尾参数的lambda表达式写在圆括号外：

```kotlin
longOperationAsync(1000) {
    hideProgress()
}
```

这看起来像一个外部的参数，这么使用可以帮助我们提升可读性，以Kotlin标准库创建线程的的`thread`函数为例，它的定义是这样的：

```kotlin
public fun thread(
    start: Boolean = true,
    isDaemon: Boolean = false,
    contextClassLoader: ClassLoader? = null,
    name: String? = null,
    priority: Int = -1,
    block: () -> Unit): Thread {
    // implementation
}
```

我们可以看到除`block`参数外，其它参数都有默认值，且block参数处于参数列表末尾，我们可以这样用：

```kotlin
thread { /* code */ }
```

这样看起来我们放在线程中执行的操作像是放在一个叫thread的代码块之中，这种使用惯例只是一个语法糖，但是我们可以看到这样创建线程比在Java之中简化了许多，清除了冗余代码之后，可读性也大大提高了。

在Android开发中，为了支持更多的设备以获取更多的用户群，我们开发的程序常常最低支持版本和目标版本是不一致的，因此我们常常会加上判断Android版本的代码来确保我们使用的特性在设备上是支持的：

```kotlin
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
    // Operations
}
```

如果我们把这样的代码写成高阶函数，并结合lambda处于参数列表末尾时的使用惯例，我们可以让这段代码可读性大大提高，同时提取成函数也便于我们代码重用：

```kotlin
fun ifSupportsLolipop(f:()->Unit) {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP){
        f()
    }
}
//使用示例
ifSupportsLollipop {
    // Operation
}
```

利用这个特性，我们甚至可以定义自己的控制结构。比如我们可以写出一个不断重复直到抛出错误控制结构：

```kotlin
fun repeatUntilError(code: ()->Unit): Throwable {
    while (true) {
        try {
            code()
        } catch (t: Throwable) {
            return t
        }
    }
}
//使用示例
val tooMuchAttemptsError = repeatUntilError {
    attemptLogin()
}
```

我们可以发挥想象，自定义任何想要的控制结构。

### Kotlin对Java中SAM的支持

高阶函数的使用在Kotlin中确实非常方便，但问题是Java中并不支持高阶函数，我们可能需要常常使用Kotlin和Java进行互操作。解决的方法是使用只有一个方法的接口，叫做**单抽象方法**（SAM，Single Abstract Method）或者称之为**函数接口**（functional interface）。对于SAM，Kotlin会自动生成一个接收那个唯一函数参数的构造器，称之为SAM构造器，这样我们就可以在Kotlin中使用高阶函数的特性了。

我们很熟悉的View类中的`OnClickListener`就是一个SAM，只有一个方法`onClick`。通过这一特性在Kotlin中使用`OnClickListener`，需要在接口名称之后传入一个**函数字面量**（function literal）作为参数，让我们比较一下它们的实现：

{% tabs %}
{% tab title="Java" %}
```java
button.setOnClickListener(new OnClickListener() {
    @Override public void onClick(View v) {
        // Operation
    }
});
```
{% endtab %}

{% tab title="Kotlin" %}
```kotlin
button.setOnClickListener(OnClickListener {
    // Operation
})
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
函数字面量指的是定义了一个未命名的函数的表达式，在Kotlin中有两种函数字面量：

* 匿名函数：`val a = fun() {}`
* lambda表达式：`val b = {}`
{% endhint %}

需要注意的是Kotlin编译器只为Java的SAM生成构造器，不会为Kotlin的SAM生成构造器，因为Kotlin已经原生支持函数类型了。但是SAM有比函数类型有优势的地方，它的函数参数是有名字的，在仅靠数据类型无法清楚表达参数含义的时候，缺少函数名会使代码难以阅读。

别担心，在函数类型之内的参数列表部分，我们依然可以使用为参数命名。以我们常用的设置列表中单个项的点击监听为例：

```kotlin
fun setOnItemClickListener(listener: (Int, View, View)->Unit) {
    // code
}
```

`listener`中的各个变量的意义不甚明朗，我们给它加上名字：

```kotlin
fun setOnItemClickListener(listener: (position: Int, view: View, parent: View)->Unit) {
    // code
}
```

现在的样子看起来，它和我们之前在Java中使用的方法就很相似了。

### 类型别名

自Kotlin1.1版本之后，新增了一个叫类型别名的特性，可以为已经存在的类型选择一个可选的名称：

```kotlin
data class User(val firstName: String, val lastName: String)
typealias Users = List<User>
```

原生数据类型也可使用该特性：

```kotlin
typealias Weight = Double
typealias Length = Float
```

类型别名必须定义在顶层，可以用可见性修饰符调整其作用域，默认的可见性是`public`，也就是说我们前面定义的类型别名可以在任何代码文件中使用。

需要注意的是类型别名仅仅是为了提高可读性，原来的类型可以与它进行任意交换赋值：

```kotlin
typealias Length = Float
var floatLength: Float = 180.08
val length: Length = floatLength
intLength = length
```

我们还可以用类型别名来缩短复杂泛型的名称，以提高它的可读性和一致性：

```kotlin
typealias Dictionary<V> = Map<String, V>
typealias Array2D<T> = Array<Array<T>>
```

类型别名还可以用于函数类型：

```kotlin
typealias Action<T> = (T) -> Unit
typealias OnElementClicked = (position: Int, view: View, parent:View)->Unit
```

类型接口命名过的函数类型还可以像接口那样被某个类实现：

```kotlin
typealias OnElementClicked = (position: Int, view: View, parent:View)->Unit
class MainActivity: Activity(), OnElementClicked {
    override fun invoke(position: Int, view: View, parent: View) {
        // ...
    }
}
```

我们经常需要给函数类型加上别名，主要的理由有：

* 别名通常比函数定义更短更易读
* 当我们重构改变函数定义时，使用别名会发生更少的修改
* 在函数类型作为参数声明时更加简洁

看到这里我们大概可以知道为何Kotlin中不需要通过定义SAM来使用高阶函数，因为所有SAM的优点我们可以通过使用类型别名加上命名参数实现。

## 内联函数

高阶函数具有非常强大的功能并且提高了代码的重用性，但是我们仍要考虑代码的效率问题。lambda表达式会被编译为类，而在Java中类的对象创建是开销较大的操作。

我们可以将函数标记为内联函数，在享有函数带来的好处的同时提升代码效率。内联函数的概念最早可追溯至C和C++，当一个函数被标记为内联函数时，在编译时编译器会将所有函数调用替换为函数体中的代码，它们将会被视为代码而不是函数，这会生成更多的字节码，换来的是运行时的效率提升。

让我们以一个测量执行时间的函数为例：

```kotlin
inline fun printExecutionTime(f: () -> Unit) {
    val startTime = System.currentTimeMillis()
    f()
    val endTime = System.currentTimeMillis()
    println("It took " + (endTime - startTime))
}

fun measureOperation() {
    printExecutionTime {
        someOperation()
    }
}
```

当我们对`measureOperation`函数进行编译时，`printExecutionTime`函数调用将被替换为实际的函数体：

```kotlin
fun measureOperation() {
    val startTime = System.currentTimeMillis()
    someOperation()
    val endTime = System.currentTimeMillis()
    println("It took " + (endTime - startTime))
}
```

由于省去了为lambda表达式创建类的开销，使用内联函数可以提升高阶函数的执行效率，因此推荐所有简短的高阶函数标记为内联。

不过使用内联函数也有不好的一面，除了我们上面提到的会产生更多的字节码之外，内联函数不能递归，也不能使用比内联函数更为严格的可见性修饰符修饰的函数，例如`public`修饰的内联函数不能使用`private`修饰的函数，因为这个`privete`修饰的函数有可能被注入到没有权限访问它的函数之中。

```kotlin
private fun someFun() {}
inline fun inlineFun() {
    someFun() // 错误
}
```

还有一个限制是当一个函数标记为内联之后，它的参数不能传递给非内联参数：

```kotlin
fun someFun(f: ()->Int) {
    //...
}
inline fun inlineFun(f: () -> Int) {
    someFun (f) // 错误
}
```

原因在于上面内联函数的参数`f`没有被创建，调用它的地方将被替换为它的函数体，而非内联函数不支持这种替换。解决的方法就是使用**非内联**（`noinline`）修饰符。

非内联修饰符修饰的函数将被视为普通函数，它的调用将不会用它的函数体来替代，使用它我们可以修正上面的错误：

```kotlin
fun someFun(f: ()->Int) {
    //...
}
inline fun inlineFun(noinline f: () -> Int) {
    someFun (f) // 正确
}
```

除了上面的传参的使用场景外，如果我们需要频繁调用一个lambda表达式而不想让代码膨胀过大，我们也应该使用`noinline`修饰符。不过，使用了noinline修饰符后自然也没有了性能的提升，因此如果所有的参数都标记为`noinline`，编译器会显示警告。一般来说，`noinline`修饰符只会用在内联函数的一部分参数上。

虽然内联函数会帮助我们提升性能，但我们可能不会频繁的使用它们，因为：

* 内联函数应当用在较小的函数上，如果我们标记的内联函数还调用了别的内联函数，那么编译后会产生相当庞大的字节码，导致编译时间变长，生成的代码变多。
* 内联函数无法使用更严格的修饰符修饰的函数，因此在大量需要`private`保护API的库中，它的使用会造成一些问题。

### 非局部返回

我们在前面已经提到过可以使用高阶函数来定义控制结构，比如我们前面的`ifSupportsLolipop`函数和`repeatUntilError`函数。让我们再来定义一个应用更普遍的`forEach`控制结构作为遍历的另一种方案：

```kotlin
fun forEach(list: List<Int>, body: (Int) -> Unit) {
    for (i in list) body(i)
}
// 使用示例
val list = listOf(1, 2, 3, 4, 5)
forEach(list) { print(it) } // 打印结果: 12345
```

这样实现最大的问题是我们无法返回到外部的函数当中去，让我们比较一下使用`for`和`forEach`实现的求最大界值：

{% tabs %}
{% tab title="使用for实现" %}
```kotlin
fun maxBounded(list: List<Int>, upperBound: Int, lowerBound: Int): Int {
    var currentMax = lowerBound
    for(i in list) {
        when {
            i > upperBound -> return upperBound
            i > currentMax -> currentMax = i
        }
    }
    return currentMax
}
```
{% endtab %}

{% tab title="使用forEach实现" %}
```kotlin
fun maxBounded(list: List<Int>, upperBound: Int, lowerBound: Int): Int {
    var currentMax = lowerBound
    forEach(list) { i->
        when {
            i > upperBound -> return upperBound //错误，此处不允许使用返回语句
            i > currentMax -> currentMax = i
        }
    }
    return currentMax
}
```
{% endtab %}
{% endtabs %}

同样的代码逻辑如果使用`forEach`实现就会产生错误，无法通过编译。原因是lambda表达式编译时会编译成一个类的匿名对象，其中包含有定义的函数，这样一来lambda表达式和外部不是同一个上下文，自然无法直接返回。

解决的方法之一是将`forEach`标记为内联，这样在编译时会将函数体直接替换而不会创建对象：

```kotlin
inline fun forEach(list: List<Int>, body: (Int) -> Unit) {
    for (i in list) body(i)
}

fun maxBounded(list: List<Int>, upperBound: Int, lowerBound: Int): Int {
    var currentMax = lowerBound
    forEach(list) { i->
        when {
            i > upperBound -> return upperBound //正确
            i > currentMax -> currentMax = i
        }
    }
    return currentMax
}
```

在内联函数的lambda表达式中使用的返回语句称为**非局部返回**（Non-local returns）。

### lambda表达式中带标签的返回

我们虽然可以定义自己的控制结构，但有些语句是不兼容我们的控制结构的，比如`continue`，这时候我们可以使用带标签返回来实现同样的效果。让我们直接来看带标签返回的示例：

```kotlin
inline fun <T> forEach(list: List<T>, body: (T) -> Unit) { // 使用泛型实现的forEach
    for (i in list) body(i)
}

fun printPositiveNums(nums: List<Int>) {
    forEach(nums) numberPrinter@ { // 定义标签
        if (it <= 0) return@numberPrinter // 指定一个标签，从lambda表达式中返回
        print(it)
    }
}
//使用示例
val list = listOf(1, -1, 2, 0, 3)
printPositiveNums(list) // 打印结果: 123
```

如果我们不想费心思为标签起名字，那么Kotlin提供了一项特性——**隐式标签**（implicit label）。如果一个lambda表达式定义为一个函数的参数，那么它就有隐式标签，标签名和所在函数的函数名一致，以上面的例子来说，隐式标签就是forEach：

```kotlin
fun printPositiveNums(nums: List<Int>) {
    forEach(nums)  { 
        if (it <= 0) return@forEach //使用隐式标签返回
        print(it)
    }
}
```

需要注意的是我们定义的`forEach`函数是内联的，那么我们也可以使用上面提到的非本地返回。不过，非本地返回是`printPositiveNums`函数的返回：

```kotlin
fun printPositiveNums(nums: List<Int>) {
    forEach(nums)  { // 2
        if (it <= 0) return
        print(it)
    }
}
//使用示例
val list = listOf(1, -1, 2, 0, 3)
printPositiveNums(list) // 打印结果: 1
```

### 跨内联修饰符

有些时候我们想在嵌套函数中使用内联，这时候编译器会提示我们错误，因为内敛函数允许非本地返回，而嵌套函数有可能在其他的上下文中执行时，这种操作不应该被允许。解决的办法是使用**跨内联修饰符**（crossinline modifier），通知编译器不允许非本地返回：

```kotlin
fun boo(f: () -> Unit) {
    //...
}

inline fun foo(crossinline f: () -> Unit) {
    boo { 
        println("A");
        f()
    }
}
```

让我们看一个在Android平台的例子，我们很多时候会需要在主线程进行一些操作，例如更新UI，那么我们可以利用`Looper`类来实现一个线程切换，将我们的操作切换到主线程执行。首先判断当前线程是否是主线程，如果是则执行，如果不是则创建一个包含主线程的handler然后将我们的操作`post`从而达到在主线程操作的目的：

```kotlin
inline fun runOnUiThread(crossinline action: () -> Unit) {
    val mainLooper = Looper.getMainLooper()
    if (Looper.myLooper() == mainLooper) {
        action()
    } else {
        Handler(mainLooper).post { action() }
    }
}
```

为了提高函数的执行效率，我们将其标记为内联，为了能使函数能在其他线程正常调用，我们在参数前加了跨内联修饰符。

使用跨内联修饰符可以让我们享受内联所带来的性能提升的同时，不影响函数在复杂上下文中的执行。

## 内联属性

自Kotlin1.1起，内联修饰符可以用在没有幕后字段的属性上。我们可以用于修饰整个属性，也可以用于修饰单个访问器。修饰整个属性，相当于该属性的每个访问器都标为内联，以下两种写法是等价的：

{% tabs %}
{% tab title="修饰单个访问器" %}
```kotlin
var viewIsVisible: Boolean
inline get() = findViewById(R.id.view).visibility == View.VISIBLE
inline set(value) {
    findViewById(R.id.view).visibility = if (value) View.VISIBLE else View.GONE
}
```
{% endtab %}

{% tab title="修饰整个属性" %}
```kotlin
inline var viewIsVisible: Boolean
get() = findViewById(R.id.view).visibility == View.VISIBLE
set(value) {
    indViewById(R.id.view).visibility = if (value) View.VISIBLE else View.GONE
}
```
{% endtab %}
{% endtabs %}

标记为内联的属性或属性的访问器，在编译时会将对应函数的调用替换为函数体：

{% tabs %}
{% tab title="代码中" %}
```kotlin
if (!viewIsVisible)
viewIsVisible = true
```
{% endtab %}

{% tab title="编译后" %}
```kotlin
if (!(findViewById(R.id.view).getVisibility() == View.VISIBLE))
{
    findViewById(R.id.view).setVisibility(true?View.VISIBLE:View.GONE);
}
```
{% endtab %}
{% endtabs %}

内联属性也会增加编译后的字节码量以换取性能的提升，对于大多数的属性来说，使用内联修饰符是有利的。

