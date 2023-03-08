---
description: >-
  Kotlin非常看重设计模式，在前面我们已经了解到单例模式可以简化为object声明，使用高阶函数和函数类型简化观察者模式的实现。本章中我们将了解Kotlin的委托（Delegates）特性，并应用在我们常用的设计模式之中。
---

# 委托

## 类委派

Kotlin有一个叫做**类委派（class delegation）**的特性，它是**委派模式（Delegation pattern）**和**装饰器模式（Decorator pattern）**的重要实现途径，这两种设计模式在Java中存在很多年了，不过在Java中实现需要非常多的样板代码，而Kotlin作为一个对这些设计模式提供原生支持的语言，将所需的样板代码降到了最少。

### 委派模式(Delegation pattern)

在面向对象编程中，委派模式是实现继承的另一种方式，委派就是一个对象通过委派另一个对象来处理请求，而不是继承已有的对象。

如果我们使用Java风格的写法，为了支持多态，委派类和被委派类都需要实现所有被委派的方法和属性：

```kotlin
interface Player {
    fun playGame()
}

class RpgGamePlayer(val enemy: String) : Player {
    override fun playGame() {
        println("Killing $enemy")
    }
}

class WitcherPlayer(enemy: String) : Player {
    val player = RpgGamePlayer(enemy) 
    override fun playGame() {
        player.playGame() 
    }
}

// 使用
RpgGamePlayer("monsters").playGame() // 打印结果: Killing monsters
WitcherPlayer("monsters").playGame() // 打印结果: Killing monsters

```

之所以被称作是委派，是因为`WitcherPlayer`通过了一个`RpgGamePlayer`对象实例委派了定义在`Player`接口中的方法。使用继承也可以实现类似的效果：

```kotlin
class WitcherPlayer() : RpgGamePlayer()
```

初看起来，这两种实现方式差不多，使用继承甚至更简单一些。一方面，继承使用的更广，在Java中更常见，并且用于多种面向对象的设计模式之中；另一方面，有很多有影响力的书都推荐使用委派来替代继承，比如《Design Patterns》、《Effective Java》，下面是这种做法的一些基本论点：

* 在通常情况下，类并不是设计用来继承的。当我们覆写方法时，我们不知道所继承的类内部发生了怎样的变化，会影响哪些属性，对象、状态等，我们完全有可能因为不清楚类内部的逻辑，导致子类出现与父类不一致的行为。只有一小部分的类设计合理并有完备的文档来指引如何继承它们。
* 在Java中，可以委派多个类，但只能继承一个类。
* 通过接口，我们指定了想要委派的方法和属性，这符合**接口隔离原则（interface segregation principle）**，我们不应该向客户端暴露不必要的方法。
* 一些类是`final`类型的，我们只能够通过委派来使用它们的方法，实际上，不是设计为继承的类都应该是`final`的，Kotlin的设计者注意到了这点，所有Kotlin中的类默认是`final`的。这在设计公共库的时候更是如此，我们可以更改`final`类的实现而不必担心影响库的用户。

当然，使用委派也有一些负面影响：

* 我们需要单独创建一个接口来描述需要委派的方法。
* 我们不能访问`protected`修饰的方法和属性。

从上面的示例中还可以看到，使用委派需要写大量的样板代码，可能这是在Java中选择使用继承的另一个原因。不过，在Kotlin及众多的现代编程语言中，有更简单的方式实现委派模式：

```kotlin
class WitcherPlayer(enemy: String) : Player by RpgGamePlayer(enemy) {}
```

使用by关键字，可以通知编译器通过创建`RpgGamePlayer`对象委派`Player`接口中的所有方法，在`WitcherPlayer`构建时，被委派的`RpgGamePlayer`实例就会创建。还有一种方式时通过持有被委派对象的实例：

```kotlin
class WitcherPlayer(player: Player) : Player by player
```

这样做一个巨大的提升是：我们不用在自己实现被委派的方法，并且当方法的签名被改变的时候，我们也不需要做出修改，使得委派实现更容易维护了。

### 装饰器模式(Decorator pattern)

Kotlin的类委托另一个非常有用的场景是用来实现装饰器模式，装饰器模式（也称为包装器模式）是一种设计模式，可以向现有类添加行为，而无需使用继承。这种设计模式的经典结构可以用如下UML图来表示：

<figure><img src=".gitbook/assets/Decorator_UML_class_diagram.svg.png" alt=""><figcaption><p>装饰器模式的经典UML图实现，图片来自网络</p></figcaption></figure>

在Java世界中，最常见的一个装饰器模式的例子便是`InputStream`，这个类有许多被继承和装饰的其他类型，有些添加了缓存能力，有些添加了解压文件内容的能力，有些则提供了反序列化能力。让我们看一个例子——读取一个压缩文件的内容并将其转换为一个Java对象：

