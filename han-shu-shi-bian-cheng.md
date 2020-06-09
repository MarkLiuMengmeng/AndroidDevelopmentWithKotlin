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

c = ::printInt // 此操作符将会在稍后讲解
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

主要注意的是写法2可以结合使用安全调用操作符，当存储函数的变量可能为空时，我们最好这么使用它：

```kotlin
var a: ((Int) -> Int)? = null
if (false) a = fun(i: Int) = i * 2
print(a?.invoke(4)) // 打印结果: null
```

`Function`接口不存在于标准库中，它是一个**编译器合成类型**（synthetic compilergenerated type），在编译的时候才会生成，因此没有人为地限制参数的数目，同时也没有增加标准库的体量。

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

这三个场景是常见的使用高阶函数的场景，在实际开发中我们可以灵活运用减少代码中的冗余，便于维护。

