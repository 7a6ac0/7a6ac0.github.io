---
layout:     post
title:      "【Android】透過Crashlytics及Timber作Android logging"
subtitle:   "Android logging with Crashlytics and Timber"
date:       2019-05-06 12:11:46
author:     "Tabaco"
header-img: "img/in-post/post-android-logging-with-crashlytics-and-timber/post-bg-android-logging-with-crashlytics-and-timber.png"
header-mask: 0.3
catalog:    true
tags:
    - Android
    - Kotlin
    - Firebase
---

開發Android APP時常常會遇到執行到一半APP就Crash，這時候就需要去檢查Log訊息看看錯誤訊息是在哪發生的，開發人員可以從Log訊息快速地Debug，而最常使用的方式：
```java
	Log.e(TAG, "A message about something weird")
```
為了取得更詳細的訊息，可以在```try and catch```發生```exception```時印出錯誤訊息。
```java
	try {
		. . .
	}catch(exception: Exception) {
		exception.printStackTrace()
	}
```
在開發階段此種debug方式非常方便，但是如果APP已經上架Google Play，並且錯誤訊息是發生在各個User Device，開發人員無法透過以上方式取得錯誤訊息作Debug。

## Firebase Crashlytics
```Firebase Crashlytics```是一款輕量級的即時當機崩潰記錄器，可幫助我們跟踪，確定優先級並解決影響您應用質量的穩定性問題。當問題嚴重性突然增加時，它會發出警報。Crashlytics通過對App Crash進行分組並突出顯示導致App Crash的情況，從而為您節省故障排除時間。

本篇的崩潰記錄主要是上傳到Firebase Crashlytics，而不上傳[Fabric][Fabric]，因為Crashlytics已經整合進Firebase，所以就不需要再連接[Fabric][Fabric]，有關其Firebase設定方式後續會寫一篇教學。(*ﾟ∀ﾟ*)

## Crashlytics SDK
在APP內要使用Crashlytics回報崩潰記錄前需要作些設定。在Project等級的```build.gradle```內新增Crashlytics repositories及dependencies：
```gradle
buildscript {
    repositories {
        // ...

        // Add repository
        maven {
           url 'https://maven.fabric.io/public'
        }
    }
    dependencies {
        // ...

        // Check for v3.1.2 or higher
        classpath 'com.google.gms.google-services:4.2.0'

        // Add dependency
        classpath 'io.fabric.tools:gradle:1.28.1'
    }
}

allprojects {
    // ...
    repositories {
       // ...

       // Check that you have the following line (if not, add it):
       google()  // Google's Maven repository
       // ...
    }
}
```
在app等級的```build.gradle```新增Crashlytics dependencies：
```gradle
apply plugin: 'com.android.application'
apply plugin: 'io.fabric'
apply plugin: 'com.google.gms.google-services'

dependencies {
    // ...

    // Check for v11.4.2 or higher
    implementation 'com.google.firebase:firebase-core:16.0.8'

    // Add dependency
    implementation 'com.crashlytics.sdk.android:crashlytics:2.10.0'
}
```
## Timber SDK
[Timber][Timber]是一個輕量級的第三方函式庫，能夠幫助開發者更便利地使用Android Log。

在app等級的```build.gradle```加上dependency：
```gradle
implementation 'com.jakewharton.timber:timber:4.7.1'
```
接著可以在程式任何一個地方使用```Timber.d("Message")```印出Debug訊息，```Timber.e("Error Message")```印出Error訊息。

不知道大家有沒有發現Timber不需要像Log一樣要傳入TAG參數，因為在呼叫Timber時會自動取得目前的class name作為TAG，這一點真的是很便利。

## Crashlytics and Timber
原本需要在每個程式碼的```exception```加上下列三行程式碼才算完成崩潰記錄，假如有許多地方都要作崩潰記錄，每次都要加上這三行實在令人崩潰啊啊啊(／‵Д′)／~ ╧╧
```java
try {
    // throw exception
    throwException()
} catch (e: Exception) {
    // Crashlytics
    Crashlytics.logException(e)

    // Firebase Crash Reporting
    FirebaseCrash.logcat(Log.ERROR, TAG, e.printStackTrace())
    FirebaseCrash.report(e)
}
```
而改用```Timber```的話，只需要加上一行程式碼就可以完成崩潰記錄。
```java
try {
    // throw exception
    throwException()
} catch (e: Exception) {
    Timber.e(e.printStackTrace())
}
```
為了要使用Timber回報崩潰記錄，需要在APP內的Application Class中呼叫```Timber.plant```來初始化Timber。其中可以透過```BuildConfig.DEBUG```判斷現在APP是Debug還是Release。

Debug版則使用Timber內建的```Timber.DebugTree()```即可，而Release版則需要額外新增CrashlyticsTree類別作記錄。
```java
class MainApplication : Application() {

    override fun onCreate() {
        super.onCreate()

        if (BuildConfig.DEBUG) {
            Timber.plant(Timber.DebugTree())
        } else {
            Timber.plant(CrashlyticsTree())
        }
    }
}
```
接著新增CrashlyticsTree類別來回報Release版的崩潰記錄。
```java
class CrashlyticsTree : Timber.Tree() {

    override fun log(priority: Int, tag: String?, message: String, t: Throwable?) {
        if (priority == Log.VERBOSE || priority == Log.DEBUG || priority == Log.INFO) {
            return
        }

        Crashlytics.setInt(CRASHLYTICS_KEY_PRIORITY, priority)
        Crashlytics.setString(CRASHLYTICS_KEY_TAG, tag)
        Crashlytics.setString(CRASHLYTICS_KEY_MESSAGE, message)

        if (t == null) {
            Crashlytics.logException(Exception(message))
        } else {
            Crashlytics.logException(t)
        }
    }

    companion object {
        private val CRASHLYTICS_KEY_PRIORITY = "priority"
        private val CRASHLYTICS_KEY_TAG = "tag"
        private val CRASHLYTICS_KEY_MESSAGE = "message"
    }
}
```
最後就可以在Firebase內的Crashlytics頁面檢查全部的崩潰記錄。

[Fabric]: https://get.fabric.io/
[Timber]: https://github.com/JakeWharton/timber