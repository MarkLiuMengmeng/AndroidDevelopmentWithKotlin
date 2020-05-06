---
description: 积小流而成江海。这章我们介绍Kotlin语言的基础要素，或许单个要素看起来不是那么关键，但结合起来就是非常强大的语言结构。
---

# Kotlin基础

## 变量

在Kotlin中有两种变量：`var`（variable）和 `val`（value）。`var`是可变的，即可以在初始化赋值之后可以被重新赋值，等同于Java中的普通变量。

```text
fun main() {
    var fruit: String = "orange" //创建变量fruit并初始化赋值orange
    fruit = "banana" //将fruit变量重新赋值为banana
}
```

`val`是不可变的，即在初始化赋值之后不能被重新赋值，相当于Java中带`final`修饰符的变量。但它不能保证引用的对象中的值不会被修改（重新赋值）：

```text
class Fruit(var name:String)//类的声明会在第四章中说明
fun main() {
    val fruit = Fruit("orange")//创建变量fruit并初始化
    fruit = Fruit("banana")//错误，不能进行重新赋值
    fruit.name = "banana"//正确，引用中对象的值可以被修改
}
```

由于`val`变量可以有一个自定义getter，所以我们也不能保证每一次的访问就是返回的都是同一个对象：

```text
val random: Int
get() = Random().nextInt()//自定义getter将会在第四章中说明
```

{% hint style="info" %}
Kotlin是允许在文件级别定义变量和函数的，而Java必须要在类中定义。
{% endhint %}

## 类型推断

和Java中不同，Kotlin的变量类型声明在变量名之后：

```text
var fruit: String
```

虽然第一眼看起来很奇怪，但是这个结构是Kotlin类型推断的重要组成部分。类型推断就是编译器可以根据上下文来推断变量的数据类型，比如当变量声明和初始化写在一起时，我们可以省略类型声明。

```text
var fruit: String = "orange"//完整声明
var fruit = "orange"//省略类型声明
```

上面虽然类型声明被省略了，但数据类型会被隐式地设置为`String`，因为Kotlin是强类型的语言。在随后的使用中编译器会验证是否符合正确的数据类型：

```text
var fruit = "orange"
fruit = 10 //错误，fruit被推断为String类型，无法赋值int类型数据
```

我们可以通过显式声明两者都适用的数据类型使上面的赋值成立：

```text
var fruit: Any = "orange"
fruit = 10
```

`Any`相当于`Java`中的Object，处于继承层级的顶端。

编译器同样可以从函数进行类型推断：

```text
var total = sum(10, 20)
```

有时候我们自己并不确定编译器到底给这个变量设置了什么数据类型，这个类型可能是`Int`，也可能是`Double`或者`Float`，我们可以将插入符置于变量名上，按快捷键（Windows中的是**Shift + Ctrl + P**，macOS中的是**arrow key + control + P**）显示具体推断成何种数据类型：

![](.gitbook/assets/chapter2_1.jpg)

类型推断同样适用于泛型：

```text
var persons = listOf(personInstance1, personInstance2)
```

假设我们传入的是`Person`类，那么上面的推断类型就为`List<Person>`。



