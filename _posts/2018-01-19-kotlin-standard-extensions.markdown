---
layout:     post
title:      "【Kotlin】Kotlin基礎：Standard Extension Functions"
subtitle:   "Kotlin Basic：Standard Extension Functions"
date:       2018-01-18 17:20:00
author:     "Tabaco"
header-img: "img/in-post/post-kotlin-standard-extensions/post-bg-kotlin-standard-extensions.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Android
    - Kotlin
---

## 前言
> Kotlin 的 extension 是用來為現成的 class 加入新 method 和 attribute。Extension 是用來取代 Java programming 時常寫的 Utils static method。Extension 比 Utils 較佳是 extension 有 IDE 提示和用法較為自然。

舉個例子，在Java內要在已存在的 class 新增 extension function，你可能會這樣寫：
```java
class Util {
  public static final boolean isNumeric(String receiver) {
    return reveiver.matches("\\d+");
  }
}
...

String myString = ...;

if(Util.isNumeric(myString)) ...
```

而Kotlin則可以用下面的程式碼來新增 extension function：
```kotlin
fun String.isNumeric(): Boolean {
  return this.matches("\\d+".toRegex())
}
...

val myString = ...

if(myString.isNumeric()) ...

```

是不是感覺簡單多了！

在Kotlin也有許多內建的 extension function，剛開始學習Kotlin時會對程式內使用`run`、`apply`、`let`、`also`、`with`感到很困惑，所以在這篇文章內會介紹以上用法，讓大家可以快速駕馭Kotlin。

## Run
```kotlin
public inline fun <T, R> T.run(block: T.() -> R): R = block()
```
`run` 可以在任何型態 `T` 執行，並返回最後一行的返回值或者指定return表達式。

思考看看底下這個範例：
```kotlin
val generator = PasswordGenerator()
generator.seed = "someString"
generator.hash = {s -> someHash(s)}
generator.hashRepititions = 1000

val password: Password = generator.generate()
```
當建立PasswordGenerator需要對多個attribute作設定，因此還沒使用`run`之前需要打好幾次`generator`，所以可以用`run`修改成以下程式碼：
```kotlin
val password: Password = PasswordGenerator().run {
       seed = "someString"
       hash = {s -> someHash(s)}
       hashRepetitions = 1000

       generate()
   }
```
## apply
```kotlin
public inline fun <T> T.apply(block: T.() -> Unit): T { block(); return this }
```
`apply`的用法跟`run`很像，但有一點不同的是`apply`的返回值永遠都為自身對象，在函式區塊內可以用`this`代表自身對象。
用以下範例示範如何使用apply建立一個Intent：
```kotlin
// 普通建立Intent方法
fun createIntent(intentData: String, intentAction: String): Intent {
    val intent = Intent()
    intent.action = intentAction
    intent.data=Uri.parse(intentData)
    return intent
}

// 使用apply建立Intent方法
fun createIntent(intentData: String, intentAction: String) =
        Intent().apply { action = intentAction }
                .apply { data = Uri.parse(intentData) }
```
## let
```kotlin
public inline fun <T, R> T.let(block: (T) -> R): R = block(this)
```
`let`將會返回最後一行的返回值或者指定return表達式，在函式區塊內使用`it`代表該對象，可以使用`let`作`null`檢查。

範例程式碼：
```kotlin
val original = "abc"

original.let {
    println("The original String is $it") // "abc"
    it.reversed() 
}.let {
    println("The reverse String is $it") // "cba"
    it.length  
}.let {
    println("The length of the String is $it") // 3
}
```
`let`的返回值是函式區塊內的最後一個對象，它的值和類型都可以被改變，如上列程式碼中第一次`let`返回值為String型態的`cba`，第二次`let`返回值則變更為Int型態的數值`3`。
## also
```kotlin
public inline fun <T> T.also(block: (T) -> Unit): T { block(this); return this }
```
Kotlin 1.1版才新增的 extension function，`also`將會返回值永遠都為自身對象，在函式區塊內使用`it`代表該對象。
```kotlin
val original = "abc"

original.also {
    println("The original String is $it") // "abc"
    it.reversed() 
}.also {
    println("The reverse String is ${it}") // "abc"
    it.length  
}.also {
    println("The length of the String is ${it}") // "abc"
}
```
從上列程式碼得知不管調用多少次`also`都是返回original原本的String型態。
## with
```kotlin
public inline fun <T, R> with(receiver: T, block: T.() -> R): R = receiver.block()
```
with函式跟前面幾個函式使用方法有點不同，因為它並不是以 extension 的形式存在的，在函式區塊內可以透過`this`指定該對象，`with`將會返回最後一行的返回值或者指定return表達式。

範例：
```kotlin
val a = with("string") {
    println(this)
    3
}
println(a)
```
執行結果：
```kotlin
string
3
```
## 參考資料
* [Kotlin Basics: Standard Extension Functions](https://lmller.github.io/kotlin-standard-extensions)
* [Standard.kt](https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/src/kotlin/util/Standard.kt)
* [Mastering Kotlin standard functions: run, with, let, also and apply](https://medium.com/@elye.project/mastering-kotlin-standard-functions-run-with-let-also-and-apply-9cd334b0ef84)