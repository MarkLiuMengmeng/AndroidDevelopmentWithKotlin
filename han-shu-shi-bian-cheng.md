---
description: >-
  Java是一门面向对象的语言，虽然Java8中引入了lambda表达式，我们可以在Android
  Studio中通过诸如Retrolambda这样的插件来使用它，但Kotlin支持更多更先进的函数式编程的特性，本章将介绍Kotlin中的这一部分。
---

# 函数式编程

## 函数类型

函数在Kotlin中是**一等公民**（first-class citizen），一等公民是一个编程语言术语，指的是一个**实体**（entity）支持所有的常规操作，这些操作一般指的是作为函数参数类型、作为函数返回类型和赋值给变量。

由于Kotlin是静态类型语言，我们需要一种函数类型的记法来实现函数作为一等公民的操作，记法由函数的参数类型列表加上`->`和返回类型表示：

`参数类型列表(types of parameters)->返回类型return type`

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

`Function`接口不存在于标准库中，它是一个**编译器合成类型**（synthetic compilergenerated type），在编译的时候才会生成，因此没有人为地限制参数的数目，同时也没有增加标准库的体量。



