---
description: >-
  s泛型是一种编程风格，指的是类，函数，数据结构或算法可以以某种方式编写，从而以后我们可以指定确切的类型。
  通常泛型提供类型安全性以及针对各种数据重用特定代码结构的能力。在本章，我们将了解泛型为什么会存在及如何定义泛型类、接口及函数。讨论如何在运行时处理泛型，以及泛型的继承关系，还有如何处理泛型的可空性。Kotlin中的泛型和Java很类似，不过Kotlin有一些新的改进，让我们开始吧～
---

# 泛型

## 为什么需要泛型？

我们在开发时经常会定义某一种特定数据类型（比如`Int`、`String`、`User`等）的集合，如果没有泛型，那么我们将不得不将每一种数据类型的集合类分开定义，比如`IntList`、`StringList`、`UserList`等等。我们可以想到，除了存储的数据类型以外，这些类将会有几乎一致的代码，这意味着我们要编写大量的重复代码，即使我们使用了继承来消除重复代码，仍然需要对每个类进行分别维护，这无疑是一个非常大的工作量。

在没有泛型的时候，我们可能会使用可以存储所有类型的通用集合（Java中的`Object`、Kotlin中的`Any?`），在取出数据的时候再进行数据类型的转换，这是一种解决方案，但是增加了冗余代码。

而泛型的出现将使这些问题迎刃而解，泛型在定义时可以用一个占位符代替实际类型，这个占位符叫做**类型参数**（type parameter）。下面是一个简单的泛型类定义：

```kotlin
class List<T> // T 是类型参数
```

`T`只是一个占位符，并不是固定的写法，换成别的字母声明也可以：

```kotlin
class List<E> // T换成E后，声明和前面的是等价的
```

{% hint style="info" %}
虽然类型参数的名称可以随意指定，但我们在代码中经常能看到有以下常见的名称，是一种编码规范：

* **？**：表示不确定的类型
*  **T \(type\)** ：表示具体的一个类型
*  **K ,V \(key ,value\)** ：分别代表键和值
*  **E \(element\)** ：代表元素
{% endhint %}

类型参数的含义是我们定义的类会使用一种特定的类型，但该类型会延后在创建时指定，这样一个`List`类可以初始化为多种数据类型：

```kotlin
var intLsit : List<Int>
var stringLsit : List<String>
var userLsit : List<User>
```

`List`类使用类型参数进行初始化，来指定`List`中可以存储哪种数据类型。

## 类型形参和类型实参

在函数基础那一章我们提到过函数的参数有形参（函数声明时定义的变量）和实参（函数调用时传入的变量值）之分。类似地，在泛型中也有类型形参（泛型定义时的类型参数）和类型实参（泛型初始化时指定的实际类型）之分。

我们可以把类型形参用在方法中，以确保传入我们希望的数据类型或返回一个我们期望的类型：

```kotlin
class List<T> {
    fun add(item:T) { 
        // code
    }
    fun get(intex: Int): T { 
        // code
    }
}
```

而传入的数据类型以及返回的数据类型取决于我们使用这些方法时的类型实参：

```kotlin
class User(val name: String)

val userList = List<User>()
userList.add(User("Mamun"))
println(userList.getItemAt(0).name) 
```

当我们使用类型实参指定好数据类型后，编译器会为我们进行类型检查，以确保所有的对象都是我们指定的数据类型，因此，当我们尝试传入其它数据类型时，编译器会报错：

```kotlin
val userList = List<User>()
userList.add(User("Mamun"))
userList.add(true) // 错误，已经指定为User类型后，不能接收Boolean类型参数
```

## 泛型约束

默认情况下，我们可以使用任何类型作为类型参数，同时我们也可以限定我们使用参数类型的范围，要做到这一点，需要定义**参数类型边界**（type parameter bound），最常见的**泛型约束**（generic constraints）是**上界**（upper bound），`Any?`是所有的类型参数的隐式上界，这就是为什么下面两种声明是等价的：

```kotlin
class List<T>
class List<T: Any?>
```

当我们显式地指定上界时就可以限定我们使用参数类型的范围：

```kotlin
class List<T: Number>
//使用示例
var numberList = List<Number>()
var intList = List<Int>()
var doubleList = List<Double>()
var stringList = List<String>() // 错误，不符合泛型约束
```

上面的例子中`Number`类是一个抽象类，是Kotlin中数字类型（如`Byte`, `Short`, `Int`, `Long`, `Float`, and `Double`）的超类。我们可以使用`Number`类及其所有子类作为类型参数，但是我们无法使用`String`类型，因为它不是`Number`类的子类，任何不符合指定类型的声明都会被IDE和编译器拒绝。类型参数同样有类型可空性的区分。

### 可空性

当我们定义了一个无界的类型参数时，我们可以指定可空类型和非空类型作为类型参数，因为上面我们提到无界的隐式上界就是`Any?`。有时，我们需要避免可空类型作为我们的类型参数，那么，我们需要显式声明一个非空类型的参数类型上界：

```kotlin
class View (val name:String)
class ViewGroup<T : View>

var viewGroupA: ViewGroup<View>
var viewGroupB: ViewGroup<View?> // 错误，该类不接收可空类型的参数
```

假设我们的`ViewGroup`类有一个取最后一个`View`的函数：

```kotlin
class ViewGroup<T : View>(private val list: List<T>) {
    fun last(): T = list.last()
}
```

我们如果给构造器传入一个空的列表，那么我们的程序将崩溃，因为当索引对应的列表元素不存在时，`last`方法会抛出一个异常：

```kotlin
val viewGroup = ViewGroup<View>(listOf())
//...
val view = viewGroup.last()//错误: java.util.NoSuchElementException: List is empty.
println(view.name)
```

相比于程序崩溃，我们可能更希望当列表为空时返回一个空值，Kotlin标准库中已经有对应的方法`lastOrNull`：

```kotlin
class ViewGroup<T : View>(private val list: List<T>) {
    fun lastOrNull(): T = list.lastOrNull() //错误，类型推断为可空类型，但函数返回类型要求不可空
}
```

但程序不会通过编译，因为推断类型与函数指定的返回类型不一致。为了解决这个冲突，我们可以在**使用端**（use-site）的类型参数加一个`?`来声明参数类型是可空的：

```kotlin
class ViewGroup<T : View>(private val list: List<T>) {
    fun lastOrNull(): T? = list.lastOrNull()
}
```

请注意`ViewGroup`的类型参数的上界是非空类型`View`，而使用`T?`意味着类型参数是否为空，返回类型都视为可空类型。让我们使用修改后的`ViewGroup`类：

```kotlin
val viewGroup = ViewGroup<View>(listOf())
//...
val view = viewGroup.lastOrNull()
println(view?.name) // 打印结果：null
```

同时，在使用端声明可空的类型参数不影响我们在**声明端**（declaration-site）声明可空的类型参数，使用端和声明端的类型参数的可空性都可分别指定：

```kotlin
class ViewGroup<T : View?>(private val list: List<T>) {
    fun lastOrNull(): T? = list.lastOrNull()
}

val viewGroup = ViewGroup(listOf(View("First"),null))
//...
val view = viewGroup.lastOrNull()
println(view?.name) // 打印结果：null
```

### 型变

在面向对象编程规范中，继承是很常见的概念，被继承的类是继承类的超类，继承类是被继承类的子类：

```kotlin
open class Animal(val name: String)
class Dog(name: String): Animal(name)
```

`Dog`类继承自`Animal`类，`Dog`类是`Animal`类的子类。这意味着我们可以在所有要求使用`Animal`类的地方使用`Dog`类：

```kotlin
fun present(animal: Animal) {
    println( "This is ${ animal. name } " )
}
present(Dog( "Pipi" )) // 打印结果: This is Pipi
```

**类型**（type）相比于**类**（class）而言，是一个更为广泛的概念，类型可由类或接口定义，或者由语言内置提供（原生数据类型）。每个Kotlin类（以`Dog`类为例）都至少有两种数据类型——可空类型（`Dog?`）和非空类型（`Dog`），泛型类（以`class Box<T>`）就更多了——`Box<Dog>`，`Box<Dog?>`，`Box<Animal>`，`Box<Box<Animal>>`等等。

上面我们所说的继承关系（`Dog`和`Animal`）放在泛型中又如何呢？在默认情况下，Kotlin中的泛型是**不型变**（invariant）的，这意味着，`Box<Dog>`类型和`Box<Animal>`类型没有继承关系:

```kotlin
class Box<T>
open class Animal
class Dog : Animal()
var animalBox = Box<Animal>()
var dogBox = Box<Dog>()

animalBox = dogBox // 错误，类型不符
dogBox = animalBox // 错误，类型不符
```

在Kotlin中，超类和子类的关系在应用到泛型上时可以得到保留（**协变**——covariant），也可以反转（**逆变**——contravariant），还可以忽略（**不型变**——invariant）。

协变的含义是，当`Dog`是`Animal`的子类时，`Box<Dog>`也是`Box<Animal>`的子类。

反协变的含义是，当`Dog`是`Animal`的子类时，`Box<Animal>`是`Box<Dog>`的子类。

可以用一个图描述这三种协变关系：

![](.gitbook/assets/chapter6_1.jpg)

定义协变和反协变，需要使用**型变修饰符**（variance modifiers）。

