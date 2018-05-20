---
layout:     post
title:      "【Android】HttpsURLConnection實作憑證綁定的方法"
subtitle:   "HttpsURLConnection with certificate pinning."
date:       2018-05-19 13:23:12
author:     "Tabaco"
header-img: "img/in-post/post-exoplayer-kotlin/post-bg-exoplayer-kotlin.png"
header-mask: 0.3
catalog:    true
tags:
    - Android
    - Java
---

## APP憑證綁定
什麼是```憑證綁定(Certificate Pinning)```？簡單的來說，憑證綁定是防止攻擊者使用假憑證進行中間人攻擊的一種安全機制。換言之，若未確實做到憑證綁定，則有心人士便可利用假憑證嗅探傳輸中的加密內容，以MITM(Man-in-the-middle)手法攔截敏感資訊。

Ref: [鑒真數位Blog][i1]、[https://developer.android.com/training/articles/security-ssl.html][i2]

## 網路安全性設定
Android N(API 24)以後可在APP資源內新增安全性設定檔以防止MITM攻擊，但此方法只侷限在API 24以後Android版本，API 24以前仍然需要使用憑證綁定方式防止MITM攻擊。
新增```res/xml/network_security_config.xml```，這邊我以[github][i3]作為例子。
```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config>
        <domain includeSubdomains="true">github.com</domain>
        <pin digest="SHA-256">pL1+qb9HTMRZJmuC/bB/ZI9d302BYrrqiVuRyW+DGrU=</pin>
        <!-- backup pin -->
        <pin digest="SHA-256">RRM1dGqnDFsCJXBTHky16vi1obOlCgFFn/yOhI/y+ho=</pin>
    </domain-config>
</network-security-config>
```
接著在```Androidmanifest.xml```新增```android:networkSecurityConfig="@xml/network_security_config"```
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="me.tabacowang.sslpinning">
    <uses-permission android:name="android.permission.INTERNET"/>
    <application

        ...

        android:networkSecurityConfig="@xml/network_security_config">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

## HttpsURLConnection憑證綁定
接著進入主題(終於啊～，這邊將展示如何作憑證綁定，詳細的程式碼可以參考[SSLPinning][i4]。



[i1]: http://iforensicsblog.blogspot.tw/2017/11/blog-post_30.html
[i2]: https://developer.android.com/training/articles/security-ssl.html
[i3]: https://github.com
[i4]: https://github.com/7a6ac0/SSLPinning
