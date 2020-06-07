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

