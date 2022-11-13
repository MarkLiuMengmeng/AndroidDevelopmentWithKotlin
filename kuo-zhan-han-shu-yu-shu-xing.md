---
description: >-
  在前面的章节，大多数的概念是Java开发者很熟悉的。本章中我们要介绍一个Java中没有的全新特性——扩展，Kotlin中非常吸引人的特性之一，使用好它将会为Android开发带来巨大的便利。
---

# 扩展函数与属性

## 扩展函数

所有大型的Java项目中，都会有许多工具类（`Utils`类），比如：`StringUtils`, `ScreenUtils`, and `ToastUtils`。这些工具类会把通用的功能代码抽离出来，方便我们使用，也方便测试。问题在于，Java在创建和使用这些类时支持得不好，比如我们要弹出一个`Toast`，通常用于显示一个提示，用Java我们会这么写：

```java
Toast.makeText(context, text, Toast.LENGTH_SHORT).show();
```

代码很基础，不过很可能大部分的开发者都曾经有过忘记调用`show`方法的经历，花费了大量时间去检查周边的逻辑条件定位它为什么没有正常运行。

为了避免我们犯这种低级错误，在Java中有一种解决办法是把它写成`Utils`类：

```java
public class ToastUtils {
    public static void toast(Context context, String text) {
        Toast.makeText(context, text, Toast.LENGTH_SHORT).show();
    }
}

// 使用
ToastUtils.toast(context, "Hello!");
```

不过，如果要使用这个工具类，你需要记住有这么一个方法，位于哪个类里。本质上来说，并没有比原始的写法更简单。更简单的方法？我们可以用Kotlin的扩展特性实现：

```kotlin
fun Context.toast(text: String) { // Context位于函数名之前，声明为Context的扩展函数
    Toast.makeText(this, text, LENGTH_LONG).show() // 在函数体中我们可以用this关键字引用Context实例
}
 // 使用
 context.toast("Hello!")
```

因为Activity是Context的子类，因此我们可以这样调用：

```kotlin
class MainActivity :Activity() {
    override fun onCreate(savedInstanceState: Bundle?){
        super.onCreate(savedInstanceState)
        toast("Hello!")
    }
}
```

上面举的例子中`Context`是作为扩展函数的接收者，我们可以使用`this`关键字访问接收者的实例，同样它也可以被隐式调用：

```kotlin
// 显示调用
fun Collection<Int>.dropPercent(percent: Double)
    = this.drop(floor(this.size * percent)
// 隐式调用
fun Collection<Int>.dropPercent(percent: Double) = drop(floor(this.size * percent)
```

扩展函数甚至可以定义在`Any?`类型上，这样我们就可以在任意类的实例中使用该函数：

```kotlin
fun Any?.logError(error: Throwable, message: String = "error") {
    Log.e(this?.javaClass?.simpleName ?: "null", message, error)
}

// 使用
user.logError(e, "Password Error") 
logError(e)
```

### 扩展函数的实现原理

扩展函数看起来可能相当神奇，不过背后的原理非常简单——在编译时扩展函数相当于一个静态方法，被扩展的类实例是这个静态方法的第一个参数。

比如我们之前定义过的`toast`扩展函数：

{% code title="ContextExt.kt" %}
```kotlin
fun Context.toast(text: String) {
    Toast.makeText(this, text, LENGTH_LONG).show()
}
```
{% endcode %}

当它编译后再反编译为Java时，看起来时这样的：

```java
public class ContextExtKt {
    public static void toast(Context receiver, String text) {
        Toast.makeText(receiver, text, Toast.LENGTH_SHORT).show();
    }
}
```

这也是为什么我们可以在Java代码中引用这个扩展函数的原因：

```java
// Java
ContextExtKt.toast(context, "Some toast")
```

这意味着，从JVM字节码的角度来看，扩展方法从未添加到`Context`类中。在编译期，所有使用扩展函数的代码均会被替换为静态方法调用。

因为扩展函数本质是一个静态函数，因此其他函数适用的特性也适用于扩展函数，比如`inline`修饰符、单表达式语法、命名参数等，这也带来了一些不太直观的问题，且看下回分解。

#### 扩展函数会覆写成员函数吗？

答案是不会。当一个类的成员函数和扩展函数拥有同样的方法名和参数时，以成员函数为准。

```
class A {
    fun test() {
        println("test from A")
    }
}
fun A.test() {
    println("test from Extension")
}
 A().test() // 打印: test from A
```

扩展函数不能更改原对象的行为，只允许增加额外的功能。这样的机制可以避免产生难以预料的行为。

#### 扩展函数中的类引用能否够提高可见性？

答案是不能。虽然扩展函数用起来像是原来的类中声明的一部分一样，但请记住它只是一个语法糖，仍然会受到可见性修饰符的限制，我们不能访问被扩展类中的`private`，`protected`修饰的元素，至于其他修饰符（`default`、`package`、`internal`），可见性仍然和普通对象的引用一样。

#### 扩展函数中的类拥有多态性吗？

答案是没有。扩展函数是在编译期被替换为静态方法的，因此实现多态性的基础不存在，多态实现依赖运行时的方法表，因此，扩展函数的行为以编译期确定的类型为准：

```java
 abstract class A
 class B: A()
 
 fun A.test() = println("test from A")
 fun B.test() = println("test from B")
 
 val b = B()
 b.test() // 打印: test from B
 (b as A).foo() // 打印: test from A
 val a: A = b
 a.foo() // 打印: test from A
```

这里的特性可能会引起问题，因为我们编写代码时常常转型，由于多态性在Java开发者中是非常深入人心的，所以可能会导致预期之外的行为。

为避免这一点，我们不应该同时为父类和子类增加相同的扩展方法，如果非要这样做，请在使用时留意要转型为对应的类型。同时，在开发一个library的时候，我们对外的API应该尽量避免增加扩展方法，以避免产生预期以外的行为或屏蔽引用者的实现。

#### 可以为伴随对象添加扩展函数吗？

答案是可以。前提是我们要在类中显式声明伴随对象，即使是空实现：

```kotlin
class A {
    companion object {}
}
fun A.companion.foo(){ print("function in companion") }
// 调用
A.foo()
```

#### 扩展函数可以帮助实现运算符重载吗？

答案是可以。运算符重载是Kotlin中最重要的特性之一，但需要声明满足要求的函数。很多时候我们使用的第三方库是没有满足这个条件的，比如RxJava中，我们常常用`CompositeDisposable`来管理订阅：

```kotlin
val subscriptions = CompositeDisposable()
subscriptions.add(repository
    .getAllCharacters(qualifiedSearchQuery)
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(this::charactersLoaded, view::showError))
```

如果我们用扩展函数定义了满足要求的运算符重载函数，就可以使用运算符重载特性：

```kotlin
operator fun CompositeDisposable.plusAssign(disposable: Disposable) {
    add(disposable)
}

// 使用
subscriptions += repository
    .getAllCharacters(qualifiedSearchQuery)
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(this::charactersLoaded, view::showError)
```

## 扩展属性



