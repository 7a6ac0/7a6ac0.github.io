---
layout:     post
title:      "【Android】OkHttp實作憑證綁定的方法"
subtitle:   "OkHttp with certificate pinning."
date:       2018-10-12 13:23:12
author:     "Tabaco"
header-img: "img/in-post/post-okhttp-pinning/post-bg-okhttp-pinning.png"
header-mask: 0.3
catalog:    true
tags:
    - Android
    - Java
---

## APP憑證綁定
什麼是```憑證綁定(Certificate Pinning)```？簡單的來說，憑證綁定是防止攻擊者使用假憑證進行中間人攻擊的一種安全機制。換言之，若未確實做到憑證綁定，則有心人士便可利用假憑證嗅探傳輸中的加密內容，以MITM(Man-in-the-middle)手法攔截敏感資訊。

參考資料：
* [鑒真數位Blog][i1]
* [https://developer.android.com/training/articles/security-ssl.html][i2]

## OkHttp憑證榜定
> 之前有介紹過HttpsUrlConnection憑證榜定的方式，有興趣的話可以到參考：[HttpsURLConnection實作憑證綁定的方法][i5]

這裡將介紹如何實作OkHttp憑證榜定，詳細的程式碼可以參考[SSLPinningOkHttp][i3]
先建立[CertificatePinner][i4]的實例，再新增憑證參數即可。
```java
CertificatePinner certPinner = new CertificatePinner.Builder()
    .add("github.com",
        "sha256/pL1+qb9HTMRZJmuC/bB/ZI9d302BYrrqiVuRyW+DGrU=")
    .add("github.com",
        "sha256/RRM1dGqnDFsCJXBTHky16vi1obOlCgFFn/yOhI/y+ho=")
    .build();
```
接著將`CertificatePinner`的實例加入`OkHttpClient`內，之後如果可以正常連線代表已經有憑證榜定了。
```java
private String connect(String url) throws IOException {
    OkHttpClient okHttpClient = new OkHttpClient.Builder()
        .certificatePinner(mCertPinner)
        .build();

    Request request = new Request.Builder()
                .url(url)
                .build();

    Response response = okHttpClient.newCall(request).execute();

    if(response.isSuccessful())
        return "ok";
    return null;
}
```

是不是比HttpsUrlConnection簡單多了。

[i1]: http://iforensicsblog.blogspot.tw/2017/11/blog-post_30.html
[i2]: https://developer.android.com/training/articles/security-ssl.html
[i3]: https://github.com/7a6ac0/SSLPinningOkHttp
[i4]: https://square.github.io/okhttp/3.x/okhttp/okhttp3/CertificatePinner.html
[i5]: https://tabacowang.me/2018/05/19/httpsurlconnection-pinning/