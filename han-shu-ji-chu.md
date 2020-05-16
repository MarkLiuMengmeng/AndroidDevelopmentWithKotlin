---
description: >-
  在Java中，类是最基本的构建模块。而在Kotlin中，由于Kotlin支持函数式编程，函数才是最基本的构建模块，也就是说不需要类就可以完整构成一个程序或者库。本章介绍Kotlin中函数的特性和类型。
---

# 函数基础

## 函数声明和使用

我们在学习新的编程语言时，第一个程序往往是“Hello，World！”。在Kotlin中，只要一个函数就可以实现这个程序：

```kotlin
// SomeFile.kt
fun main(args: Array<String>) { // 声明了一个参数args，Kotlin 1.3.72版本之后将不再需要该参数
    println("Hello, World!") // 该函数等同于Java中的System.out.println
}
```

我们已经简单看到了一个函数是什么样子的，并且我们不用任何类就可以声明和使用它。一个函数由`fun`关键字、声明在括号里的参数和函数体组成，我们再来看一个有返回值的函数：

```kotlin
fun sum(a:Int,b:Int):Int{ // 返回值的类型定义在函数名称和参数表之后
    return a + b
}
```

{% hint style="info" %}
在Java中，我们把类似的结构称作**方法**（Method），现在我们称作**函数**（Function）。它们之间的区别是这样的：

* 函数是指依照函数名调用的一段代码。而方法是和类（对象）的实例所关联的一个函数，有时又叫它成员函数。
* 简而言之，类内部的函数被称为方法。 一般说来，在Java中只有方法，但是学者有时会说静态Java方法实际上是函数。 在Kotlin中，我们定义函数可以不与任何对象相关联。
{% endhint %}

Kotlin调用函数的语法和Java等大部分编程语言类似：

```kotlin
val total = sum (12,24)//调用sum函数赋值给total变量
```

## 参数

在函数的声明中，每个参数都必须明确指定其类型。所有参数都被定义为不可变类型（只读）且无法被定义为可变的，因为可变类型的参数容易出错且在Java中经常被程序员误用。如果我们确实需要一个可变类型变量，那么可以在函数体中声明一个具有相同名称局部变量来作为影子变量：

```kotlin
fun findDuplicates(list: List<Int>): List<Int> {
    var list = list.sorted()
    //...
}
```

我们可以这么做，但会显示警号，因为这样的重复命名让我们难以定位问题并降低了代码的可读性。一个更好的办法是用数据的内容来命名参数，用数据的目的来命名变量，这两者在大多数情况下是不同的：

```kotlin
fun findDuplicates(originList<Int>): List<Int> {
    var sortedList = originList.sorted()
    //...
}
```

{% hint style="info" %}
我们一直在使用参数这个术语，实际上它是有两种类型的：**形参**（Parameters）和**实参**（Arguments）。以我们上节末尾使用的`sum`函数来说：

* 形参指的是在函数声明中定义的变量，指的是`sum`函数声明时声明的`a`和`b`。
* 实参指的是在调用函数时传进去的实际值，指的是在调用`sum`函数时传入的`12`和`24`.
{% endhint %}

和Java中一样，实参是可以传入形参的子类型的。如果我们声明形参类型为`Any?`，那么我们可以传入任何值：

```kotlin
fun show(arg: Any?) {
    println("Hello. It is: $arg")
}
show(null)
// 打印结果: Hello. It is: null
show(1)
// 打印结果: Hello. It is: 1
show("Skr")
// 打印结果: Hello. It is: Skr
```

