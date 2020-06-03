---
description: >-
  Java是一门面向对象的语言，虽然Java8中引入了lambda表达式，我们可以在Android
  Studio中通过诸如Retrolambda这样的插件来使用它，但Kotlin支持更多更先进的函数式编程的特性，本章将介绍Kotlin中的这一部分。
---

# 函数式编程

## 函数类型

函数在Kotlin中是**一等公民**（first-class citizen），一等公民是一个编程语言术语，指的是一个**实体**（entity）支持所有的常规操作，这些操作一般指的是作为函数参数、作为函数返回值和赋值给变量。

由于Kotlin是静态类型语言，我们需要一种函数类型的记法来实现函数作为一等公民的操作，记法由函数的参数类型加上`->`和返回类型表示：

`参数类型(types of parameters)->返回类型return type`

```kotlin
lateinit var a: (Int) -> Int // 以Int(数据类型)为参数并返回Int的函数
lateinit var b: ()->Int // 不接收参数返回Int的函数
lateinit var c: (Int)->Unit // 以Int为参数不带返回类型的函数
```

