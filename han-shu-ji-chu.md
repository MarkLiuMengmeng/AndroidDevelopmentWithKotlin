---
description: >-
  在Java中，类是最基本的构建模块。而在Kotlin中，由于Kotlin支持函数式编程，函数才是最基本的构建模块，也就是说不需要类就可以完整构成一个程序或者库。本章介绍Kotlin中函数的特性和类型。
---

# 函数基础

## 函数声明和使用

我们在学习新的编程语言时，第一个程序往往是“Hello，World！”。在Kotlin中，只要一个函数就可以实现这个程序：

```kotlin
// SomeFile.kt
fun main(args: Array<String>) { // 声明了一个参数args，Kotlin_1.3.72版本之后将不再需要该参数
    println("Hello, World!") // 该函数等同于Java中的System.out.println
}
```

我们已经简单看到了一个函数是什么样子的，并且我们不用任何类就可以声明和使用它。一个函数由`fun`关键字、声明在括号里的参数和函数体组成，我们再来看一个有返回值的函数：

```kotlin
fun sum(a:Int,b:Int):Int{ // 与Java不同,返回值的类型定义在函数名称和参数表之后
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
// 打印结果: Hello. It is: Skr函数
```

## 函数的返回类型

在Kotlin中，所有函数都是有返回值的，如果没有指定函数的返回类型，它的返回值默认为`Unit`类型。为了演示起见，我们可以显式声明出来：

```kotlin
fun printSum(a: Int, b: Int): Unit {//在实际编码中，可以省略Unit声明
    val sum = a + b
    print(sum)
}
```

`Unit`对象等价于Java中的`void`，可以被视作为一个普通的对象，可以赋值给其他变量：

```kotlin
val v = printSum(12, 24)
println(v is Unit) // 打印结果: true
```

`Unit`是一个单例，所以下列三个命题均为真：

```kotlin
println(p is Unit) //打印结果: true
println(p == Unit) // 打印结果: true
println(p === Unit) // 打印结果: true
```

使用不带任何值的`return`语句即可返回`Unit`类型：

```kotlin
fun printSum(a: Int, b: Int) { // 没有显式指定返回类型的情况下就会隐式设为Unit类型
    if(a < 0 || b < 0) {
        return // 函数的返回语句
    }
    val sum = a + b
    print(sum)
    // 函数执行完毕会自然返回，我们不必再使用函数返回语句
}
```

如果我们指定了`Unit`之外的返回类型，那么需要显式地使用返回语句，否则将视为错误：

```kotlin
fun sumPositive(a: Int, b: Int): Int {
    if(a > 0 && b > 0) {
        return a + b
    }
    // 错误，if条件不满足时函数没有显式的返回语句
}
```

简单地加上一个返回语句即可修复此问题：

```kotlin
fun sumPositive(a: Int, b: Int): Int {
    if(a > 0 && b > 0) {
        return a + b
    }
    return -1
}
```

## 可变参数

有些时候，我们在生命函数时并不知道要处理的参数的具体数目，在这种情况下，我们可以使用`vararg`修饰符修饰一个参数，这样这个参数就可以接受任意数量的参数：

```kotlin
fun printSum(vararg numbers: Int) {
    val sum = numbers.sum()
    print(sum)
}
printSum(1,2,3,4) // 打印结果: 10
printSum(1) // 打印结果: 1
printSum() // 打印结果: 0
```

实参中所包含的数据可以在函数体内部视为一个数组，数组的类型对应着参数的数据类型，一般来说，数组的数据类型会是泛型类`Array<T>，`但在上例中我们使用的是`Int`类型的参数，Kotlin有一个优化过的数组类型`IntArray`，Kotlin会自动使用这个更佳的选择：

![](.gitbook/assets/chapter3_1.jpg)

我们仍然可以在可变参数之前声明普通的参数：

```kotlin
fun printAll(prefix: String, postfix: String, vararg texts: String){
    val allTexts = texts.joinToString(", ")
    println("$prefix$allTexts$postfix")
}
printAll("All texts: ","!" , "Hello", "World")
// 打印结果: All texts: Hello, World!
```

当我们使用可变参数时，可以一个一个地传参，也可以使用**延展操作符**（spread operator）一次性传进去整个数组：

```kotlin
printSum(1,2,3,4) // printSum函数定义参见本节开头，打印结果：10
val numbers = intArrayOf(1, 2, 3)
printSum(*numbers) // 打印结果：6
printSum(1,*numbers,2) // 打印结果：9
```

需要注意的是，一个函数只可以有一个可变参数。





