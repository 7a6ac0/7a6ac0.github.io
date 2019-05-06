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

本篇的Crash記錄主要是上傳到Firebase Crashlytics，而不上傳[Fabric][Fabric]，因為Crashlytics已經整合進Firebase，所以就不需要再連接[Fabric][Fabric]，有關其Firebase設定方式後續會寫一篇教學。(*ﾟ∀ﾟ*)

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

[Fabric]: https://get.fabric.io/
[Timber]: https://github.com/JakeWharton/timber