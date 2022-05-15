---
description: >-
  泛型是一种编程风格，指的是类，函数，数据结构或算法可以以某种方式编写，从而可以让我们之后可以指定确切的类型。
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
虽然类型参数的名称可以随意指定，但我们在代码中经常能看到有以下常见的名称，是一种常用的命名惯例：

* &#x20;**？**：表示不确定的类型
* &#x20;**T (type)** ：表示具体的一个类型
* &#x20;**K ,V (key ,value)** ：分别代表键和值
* &#x20;**E (element)** ：代表元素
* &#x20;**N(Number)**：代表数字

良好的命名可以提高代码的可读性，相关的编程规范可参考：

[Oracle的泛型命名规范](https://docs.oracle.com/javase/tutorial/java/generics/types.html)

[Google的泛型命名规范](https://google.github.io/styleguide/javaguide.html#s5.2.8-type-variable-names)
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

![](.gitbook/assets/chapter6\_1.jpg)

定义协变和反协变，需要使用**型变修饰符**（variance modifiers）。

#### 型变修饰符

泛型在Kotlin中默认是不型变的，所以默认情况下，我们在函数的参数列表中或者变量中定义是什么类型就只能使用定义的类型：

```kotlin
public class Box<T> { }
fun sum(list: Box<Number>) { /* ... */ }
// 使用示例
sum(Box<Any>()) // 错误
sum(Box<Number>()) // 正确
sum(Box<Int>()) // 错误
```

即使`Int`是`Number`的子类型，也不能用在`sum`函数中；`Any`是`Number`类型的超类，也同样不能使用。我们可以使用型变修饰符来放松这个限制，在Java中我们使用**通配符**`?`（wildcard notation）来代表一个未知的类型，使用它来定义带有上界或者带有下界的参数类型。在Kotlin中，我们可以使用`in`和`out`修饰符同样可以达到相似的效果。

在Java中，一个带上界的通配符意味着可以接受上界类型及其所有的子类型：

```kotlin
//Java
public void sum(Box<? extends Number> list) { /* ... */ }
// 使用示例
sum(new Box<Any>()) // 错误
sum(new Box<Number>()) // 正确
sum(new Box<Int>()) // 正确
```

对应到Kotlin中，我们应该使用`out`修饰符，它用来表示协变行为，允许特定类型及其子类型参数化泛型：

```kotlin
//Kotlin
fun sum(list: Box<out Number>) { /* ... */ }
// 使用示例
sum(Box<Any>()) // 错误
sum(Box<Number>()) // 正确
sum(Box<Int>()) // 正确
```

在Java中，带有下界的通配符意味着可以接受指定类型及其超类：

```kotlin
//Java
public void sum(Box<? super Number> list) { /* ... */ }
// 使用示例
sum(new Box<Any>()) // 正确
sum(new Box<Number>()) // 正确
sum(new Box<Int>()) // 错误
```

对应到Kotlin中，我们应该使用`in`修饰符，它用来表示逆变行为，允许特定类型及其超类型参数化泛型：

```kotlin
//Kotlin
fun sum(list: Box<in Number>) { /* ... */ }
//使用示例
sum(Box<Any>()) // 正确
sum(Box<Number>()) // 正确
sum(Box<Int>()) // 错误
```

需要注意的是，修饰符`in`和`out`是不允许同时使用的。

我们还可以在两种不同的位置（声明端和使用端）定义型变。我们回顾一下我们前面提过的例子：

```kotlin
class Box<T>
open class Animal
class Dog : Animal()
var animalBox = Box<Animal>()
var dogBox = Box<Dog>()

animalBox = dogBox // 错误，类型不符
dogBox = animalBox // 错误，类型不符
```

有些时候，子类型参数化的泛型也能适用于我们的功能逻辑，那么我们需要完成子类型参数化的泛型对超类型参数化的泛型赋值，可以在使用端定义型变：

```kotlin
class Box<T>
open class Animal
class Dog : Animal()

var animal:Box<out Animal> = Box<Animal>() //使用端使用型变修饰符
animal = Box<Dog>() // 正确
```

我们在少量需要这样赋值时可以这样做，但如果我们有大量变量需要这样赋值，那么我们每个变量都要使用型变修饰符，并且要明确写出类型而不能使用类型推断，这显然增加了代码冗余。为了改善这一点，我们可以将使用端定义型变改为声明端：

```kotlin
class Box<out T> //声明端使用型变修饰符
open class Animal
class Dog : Animal()

var animal = Box<Animal>()
animal = Box<Dog>() // 正确
```

相对于在Java中PECS原则（_Producer `extends` and Consumer `super`_），Kotlin中应该叫POCI原则（_Producer `out` and Consumer `in`_）,

{% hint style="info" %}
[什么是PECS？](https://stackoverflow.com/questions/2723397/what-is-pecs-producer-extends-consumer-super)
{% endhint %}

根据这个原则，Kotlin做了更多，编译器会对类型参数的使用位置，也进行了限制。

泛型的类型参数的使用位置分为传入位（`in` position）和返回位（`out` position）：

```kotlin
// Kotlin
interface Stack<T> {
    fun push(t:T) // 在传入位使用泛型
    fun pop():T // 在返回位使用泛型
    fun swap(t:T):T // 在传入位和返回位使用泛型
    val last: T // 在返回位使用泛型
    Kotlinvar special: T // 在返回位使用泛型
}
```

经过型变的类型参数，仅能在对应位置使用，用英文名称更能说明它们之间的对应关系——`in` position对应着型变修饰符`in`：

```kotlin
// Kotlin
class ConsumerProducer<in T, out R> {
    fun consumeItemT(t: T): Unit { } 
    fun consumeItemR(r: R): Unit { } // 错误，在in position使用了out型变后的类型参数
    fun produceItemT(): T { // 错误，在out position使用了in型变后的类型参数
        // Return instance of type T
    }
    fun produceItemR(): R {
        //Return instance of type R
    }
}
```

我们已经知道Kotlin中默认的可见性修饰符是`public`，对泛型类型参数使用位置的限制也仅限于外部可见的成员，包括`protected`和`internal`，对`private`成员没有使用位置限制，以下代码可以通过编译：

```kotlin
// Kotlin
class ConsumerProducer<in T, out R> {
    private fun consumeItemT(t: T): Unit { }
    private fun consumeItemR(r: R): Unit { }
    private fun produceItemT(): T {
        // Return instance of type T
    }
    private fun produceItemR(): R {
        //Return instance of type R
    }
}
```

对类型参数使用位置限制的总结如下：

|                           |                 |                 |                 |
| ------------------------- | --------------- | --------------- | --------------- |
| 可见性修饰符                    | 不型变             | 协变（`in`修饰）      | 反协变（`out`修饰）    |
| public、protected、internal | in/out position | in position     | out position    |
| private                   | in/out position | in/out position | in/out position |

有个重要的例外是上述使用参数类型的位置限制中，不包含构造器，构造器永远是不型变的。

虽然构造器方法的可见性是`public`，以下使用仍是正确的：

```kotlin
// Kotlin
class Producer<out T>(t: T)
 // 示例
val stringProducer = Producer("A")
val anyProducer: Producer<Any> = stringProducer
```

一个重要的原因是构造器仅在创建类的实例时调用，在类的实例使用期间无法调用。

另一个原因是因为，Kotlin限制了此种情形仅能声明不变量的public字段，以确保类型安全：

```kotlin
// Kotlin
class Producer<out T>(val t: T) // 正确, 类型安全
class Producer<out T>(var t: T) // 错误, 类型不安全
class Producer<out T>(private var t:T) // 正确，类型安全，private修饰的变量外界无法访问
```

#### 集合型变

在Java中，数组是协变的。默认情况下，我们可以向一个`Object`数组赋值一个`String`数组：

```java
// Java
public class Computer {
     public Computer() {
         String[] stringArray = new String[]{"a", "b", "c"};
         printArray(stringArray); 
     }
     void printArray(Object[] array) {
         System.out.print(array);
     }
 }
```

在泛型出现前的早期版本Java中需要这样操作来处理不同情况下传参，但这样使用数组将会有潜在的运行时异常风险：

```java
// Java
public class Computer {
     public Computer() {
         Number[] numberArray = new Number[]{1, 2, 3};
         updateArray(numberArray);
     }
     void updateArray(Object[] array) {
         array[0] = "abc"; // 抛出java.lang.ArrayStoreException: java.lang.String
     }
 }
```

由于在Java中数组是协变的，编译器不会阻止编译通过，将这个错误遗留到了运行时。

而在Kotlin中，数组是不型变的，因此同样的写法将不会通过编译。

Kotlin标准库中有两种List接口，其一是`List`，是协变的，因为它不可被修改：

```kotlin
// Kotlin
fun printElements(list: List<Any>) {
    for(e in list) print(e)
}

val intList = listOf(1, 2, 3, 4)
val anyList = listOf<Any>(1, 'A')
printElements(intList) // 打印结果: 1234
printElements(anyList) // 打印结果: 1A
```

另一种是`MutableList`，它可以被修改，所以它是不协变的：

```
// Kotlin
fun addElement(mutableList: MutableList<Any>) {
    mutableList.add("Cat")
}

val mutableIntList = mutableListOf(1, 2, 3, 4)
val mutableAnyList = mutableListOf<Any>(1, 'A')
addElement(mutableIntList) // 错误: 类型不匹配
addElement(mutableAnyList)
```

### 类型擦除

类型擦除是进程移除了泛型类型（generic type）的类型参数（type argument），因此在运行时泛型类型会丢失它的类型参数信息。

类型擦除曾被引入JVM以解决Java字节码向后兼容问题，在Android平台上，Java和Kotlin都会编译为JVM字节码，因此它们都有类型擦除的弱点。

类型擦除会带来一些限制，比如在JVM语言中，我们无法以不同类型参数、相同泛型数据类型来重载方法：

```kotlin
// java.lang.ClassFormatError: Duplicate method name&signature...
 fun sum(ints: List<Int>) {
     println("Ints")
 }
 
 fun sum(strings: List<String>) {
     println("Ints")
 }
```

我们可以使用`JvmName`注解来重新制定JVM中生成的方法名称，从而解决这个问题：

{% code title="Test.kt" %}
```kotlin
@JvmName("intSum") 
fun sum(ints: List<Int>) {
    println("Ints")
}
fun sum(strings: List<String>) {
    println("Ints")
}
```
{% endcode %}

当这么做了之后，在Java端中使用这个方法就需要用它的新名字了：

```java
// Java
 TestKt.intSum(listOfInts);
```

有些时候，我们希望在运行时保留类型参数，这就需要用到重置类型参数（Reified type parameters）。

#### 重置类型参数

有些情况下，在运行时获取类型参数是有用的，例如检查传进来的数据类型，但由于类型擦除限制而无法做到：

```kotlin
fun <T> typeCheck(type: Any) {
    if(s is T){
        // 错误，无法检查擦除类型的实例: T
        println("The same types")
    } else {
        println("Different types")
    }
}
```

为了克服JVM中类型擦除带来的限制，Kotlin允许我们使用一种特殊的修饰符来在运行时保留类型参数——`reified`：

```kotlin
interface View
class HomeView: View
class ProfileView: View

inline fun <reified T> typeCheck(s: Any) {
    if(s is T){
        println("The same types")
    } else {
        println("Different types")
    }
}

// 使用
typeCheck<ProfileView>(ProfileView()) // 打印: The same types
typeCheck<HomeView>(ProfileView()) // 打印: Different types
typeCheck<View>(ProfileView()) // 打印: The same types
```

我们看到这个函数被声明为了内联函数，那是因为重置类型参数只能和内联函数一起使用，在编译期，Kotlin编译器会替换重置类型参数的`class`，从而避免了类型擦除。在字节码层面，重置类型参数会是一个运行时实际要执行的数据类型，或者是原生类型的包装类型。

重置数据类型提供了一种写方法的全新方式，我们如果想在Android中启动一个Activity，在Java中，我们会这么写：

```java
startActivity(Intent(this, ProductActivity::class.java));
```

而在Kotlin中，我们可以定义一个`startActivity`的工具方法，以更简单的方式导航：

```kotlin
inline fun <reified T : Activity> startActivity(context: Context) {
    context.startActivity(Intent(context, T::class.java))
}
 // 使用
 startActivity<MainActivity>(context)
```

需要注意的是，不能使用重置类型参数来创建一个类的实例（除非使用反射），原因是构造函数依赖于确定的类型，它永远不会被继承，对于一个类型参数所有可能的具体类型，没有一个确切的构造函数可以在这里安全的调用。

在Java中我们可以定义一个泛型类型的**原始类型**（raw type），但在Kotlin不能这样做，而需要使用**星号投影**（star-projection），来表示类型参数丢失或不重要：

```java
SimpleList<> // Java: 正确
SimpleList<> // Kotlin: 错误
SimpleList<*> // Kotlin: 正确
```
