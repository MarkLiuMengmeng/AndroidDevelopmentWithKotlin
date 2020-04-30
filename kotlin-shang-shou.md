---
description: Kotlin是一门非常出色的语言，能让Android开发更轻松，更简单。Kotlin将会改变你写代码和解决常见编程问题的方式。
---

# Kotlin上手

## 来和Kotlin认识一下

让我们看看Kotlin的一些优势：

* **安全**：Kotlin在可空性（nullability）和不变性（immutability）方面提供了更加安全的特性。Kotlin是静态类型语言，因此在编译时就知道每个表达式的类型，编译器可以验证我们尝试访问某个类实例的任何属性或方法。与之相似的是，Java也是静态类型的，但是与Java不同，Kotlin类型系统更严格（安全）。 我们必须明确告诉编译器给定的变量可以存储空值。 这样可以使程序在编译时甚至在编写代码时报出错误，而不是在运行时抛出NullPointerException。
* **易于调试**：使用Kotlin能在开发阶段更快地发现程序错误，这比在程序发布后产生崩溃损害用户体验好多了。Kotlin可以很方便地处理不可变的数据，例如它把可变集合（可读可写）和不可变集合（仅可读）区分开来，提供了不同的操作接口（在底层实现中，集合仍然是可变的）。
* **简洁**：Kotlin消除了大部分Java的冗余代码，同样的编程任务Kotlin需要的代码更少。因此，Kotlin写出的代码更具表现力，更容易阅读和理解。
* **互操作性**：Kotlin旨在与Java无缝并行工作（跨语言项目）。现有Java库和框架的生态系统与Kotlin一起使用而不会有任何性能损失。许多Java库甚至推出了Kotlin特定的版本，以符合Kotlin的习惯用法。 无需任何特殊语法Kotlin类可以在Java代码中直接实例化并透明地引用，反之亦然。 这使我们能够轻松地将Kotlin纳入现有的Android项目，与Java一起使用。
* **多功能性**：我们可以面向许多平台，包括移动应用程序 （Android甚至iOS）、服务器端应用程序（后端）、Web开发（前端 ）、数据科学、桌面应用程序，甚至构建系统（Gradle）。

当然，任何编程语言都需要有相应的工具支持才能真正变得趁手。有许多现代IDE支持Kotlin，例如Android Studio、IntelliJ Idea和Eclipse。Kotlin团队致力于使Kotlin插件变得越来越好。Kotlin大多数错误都已快速修复并且实现了社区要求的许多功能。

{% hint style="info" %}
Kotlin bug跟踪: [https://youtrack.jetbrains.com/issues/KT](https://youtrack.jetbrains.com/issues/KT) 

Kotlin slack频道: [http://slack.kotlinlang.org/](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up)
{% endhint %}

使用Kotlin进行Android程序开发变得越来越高效和愉快，Kotlin与JDK 6兼容，因此即使Android4之前的Android设备中，使用Kotlin创建的应用程序也可以安全运行。

Kotlin旨在通过结合过程式编程（procedural programming）和函数式编程（functional programming）的概念和元素以带来两全其美的编程体验。 它遵循许多准则，如每个Java开发人员的必读书籍——约书亚·布洛赫（Joshua Bloch）撰写的《Effective Java》。

最重要的是，Kotlin是开源的，因此我们可以查验该项目并积极参与到Kotlin项目的各个方面，例如Kotlin插件、编译器、文档或Kotlin语言本身。

## 来看看一些Kotlin语言的示例

