---
layout:     post
title:      "【Android】在Firebase上作Crashlytics的設定"
subtitle:   "Android crashlytics with Firebase"
date:       2019-05-07 06:09:11
author:     "Tabaco"
header-img: "img/in-post/post-android-crashlytics-with-firebase/post-bg-android-crashlytics-with-firebase.png"
header-mask: 0.3
catalog:    true
tags:
    - Android
    - Firebase
---

```Firebase Crashlytics```是一款輕量級的即時當機崩潰記錄器，可幫助我們跟踪，確定優先級並解決影響您應用質量的穩定性問題。當問題嚴重性突然增加時，它會發出警報。Crashlytics通過對App Crash進行分組並突出顯示導致App Crash的情況，從而為您節省故障排除時間。

本篇主要介紹Firebase Crashlytics的設定方法，如想要了解Android App如何設定崩潰記錄，可以參考我另一篇文章：[【Android】透過Crashlytics及Timber作Android logging][android-logging-with-crashlytics-and-timber]

1.  **新增Project**
    ```
    ```
    登入[Firebase][Firebase]，並新增一個APP Project。
    ![](/img/in-post/post-android-crashlytics-with-firebase/android-crashlytics-with-firebase-1.png)
    這裡我以我的[MVVMArch][MVVMArch]當作範例，將專案名稱填寫後可以選擇數據分析的位置，也有台灣可以選擇，在這使用預設就可以了。
    ![](/img/in-post/post-android-crashlytics-with-firebase/android-crashlytics-with-firebase-2.png)

2.  **新增應用程式**
    ```
    ```
    建立完成後需要新增一筆應用程式，如果是Iphone可以點選IOS來新增，這邊主要以Android為主，因此點選Android圖示來新增。
    ![](/img/in-post/post-android-crashlytics-with-firebase/android-crashlytics-with-firebase-3.png)
    填寫APP專案完整名稱及```偵錯簽署憑證 SHA-1```。
    ![](/img/in-post/post-android-crashlytics-with-firebase/android-crashlytics-with-firebase-4.png)
    至於```偵錯簽署憑證 SHA-1```如何取得，可以參考[Google官方教學][Authenticating]，或是從Android Studio取得Debug簽證 SHA-1值。
    ![](/img/in-post/post-android-crashlytics-with-firebase/android-crashlytics-with-firebase-5.png)
    執行signingReport後可以取得Debug憑證的SHA-1值。
    ![](/img/in-post/post-android-crashlytics-with-firebase/android-crashlytics-with-firebase-6.png)
    點選註冊應用程式後下載```google-services.json```設定檔，並放在```app```資料夾內。
    ![](/img/in-post/post-android-crashlytics-with-firebase/android-crashlytics-with-firebase-7.png)
    在APP專案內的```build.gradle```新增Firebase SDK。
    ![](/img/in-post/post-android-crashlytics-with-firebase/android-crashlytics-with-firebase-8.png)

3.  **設定Crashlytics**
    ```
    ```
    aa

[android-logging-with-crashlytics-and-timber]: https://tabacowang.me/2019/05/06/android-logging-with-crashlytics-and-timber/
[Firebase]: https://console.firebase.google.com/
[MVVMArch]: https://github.com/7a6ac0/MVVMArch
[Authenticating]: https://developers.google.com/android/guides/client-auth