---
layout:     post
title:      "【Android】HttpsURLConnection實作憑證綁定的方法"
subtitle:   "HttpsURLConnection with certificate pinning."
date:       2018-05-19 13:23:12
author:     "Tabaco"
header-img: "img/in-post/post-httpsurlconnection-pinning/post-bg-httpsurlconnection-pinning.png"
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
在```Androidmanifest.xml```新增```android:networkSecurityConfig="@xml/network_security_config"```
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
接著進入~~業配~~主題(終於啊～)，這邊將展示如何作憑證綁定，詳細的程式碼可以參考[SSLPinning][i4]。

> 對OkHttp憑證榜定方式有興趣的可以看我另一篇文章：[OkHttp實作憑證綁定的方法][i5]

從[github][i3]網站上下載憑證，放在```assets```資料夾內
![](/img/in-post/post-httpsurlconnection-pinning/httpsurlconnection-pinning-1.png)

利用getAssets()取得憑證檔。
```java
private SSLContext getPinnedSSLContext() throws IOException {
    InputStream input = null;
    try {
        input = getActivity().getAssets().open("githubcom.crt");
        return PinnedSSLContextFactory.getSSLContext(input);
    } finally {
        if (null != input) {
            input.close();
        }
    }
}
```

建立憑證綁定的SSLContext實例。
```java
public class PinnedSSLContextFactory {

    private static final String TAG = "PinnedSSLContextFactory";

    /**
     * Creates a new SSLContext instance, loading the CA from the input stream.
     *
     * @param input InputStream with CA certificate.
     * @return The new SSLContext instance.
     */
    public static SSLContext getSSLContext(InputStream input) {
        try {
            Certificate ca = loadCertificate(input);
            KeyStore keyStore = createKeyStore(ca);
            TrustManager[] trustManagers = createTrustManager(keyStore);
            return createSSLContext(trustManagers);
        } catch (CertificateException e) {
            Log.e(TAG, "Failed to create certificate factory", e);
        } catch (KeyStoreException e) {
            Log.e(TAG, "Failed to get key store instance", e);
        } catch (KeyManagementException e) {
            Log.e(TAG, "Failed to initialize SSL Context", e);
        }
        return null;
    }

    /**
     * Loads CAs from an InputStream. Could be from a resource or ByteArrayInputStream or from
     * https://www.washington.edu/itconnect/security/ca/load-der.crt.
     *
     * @param input InputStream with CA certificate.
     * @return Certificate
     * @throws CertificateException If certificate factory could not be created.
     */
    private static Certificate loadCertificate(InputStream input) throws CertificateException {
        CertificateFactory cf = CertificateFactory.getInstance("X.509");
        return cf.generateCertificate(input);
    }

    /**
     * Creates a key store using the certificate.
     *
     * @param ca Certificate to trust
     * @return KeyStore containing our trusted CAs.
     * @throws KeyStoreException
     */
    private static KeyStore createKeyStore(Certificate ca) throws KeyStoreException {
        try {
            String keyStoreType = KeyStore.getDefaultType();
            KeyStore keyStore = KeyStore.getInstance(keyStoreType);
            keyStore.load(null, null);
            keyStore.setCertificateEntry("ca", ca);
            return keyStore;
        } catch (IOException e) {
            Log.e(TAG, "Could not load key store", e);
        } catch (NoSuchAlgorithmException e) {
            Log.e(TAG, "Could not load key store", e);
        } catch (CertificateException e) {
            Log.e(TAG, "Could not load key store", e);
        }
        return null;
    }

    /**
     * Creates a TrustManager that trusts the CAs in our KeyStore.
     *
     * @param keyStore Key store with certificates to trust.
     * @return TrustManager that trusts the CAs in our key store.
     * @throws KeyStoreException If initialization fails.
     */
    private static TrustManager[] createTrustManager(KeyStore keyStore) throws KeyStoreException {
        try {
            String tmfAlgorithm = TrustManagerFactory.getDefaultAlgorithm();
            TrustManagerFactory tmf = TrustManagerFactory.getInstance(tmfAlgorithm);
            tmf.init(keyStore);
            return tmf.getTrustManagers();
        } catch (NoSuchAlgorithmException e) {
            Log.e(TAG, "Failed to get trust manager factory with default algorithm", e);
        }
        return null;
    }

    /**
     * Creates an SSL Context that uses a specific trust manager.
     *
     * @param trustManagers Trust manager to use.
     * @return SSLContext that uses the trust manager.
     * @throws KeyManagementException
     */
    private static SSLContext createSSLContext(TrustManager[] trustManagers) throws
            KeyManagementException {
        try {
            SSLContext context = SSLContext.getInstance("TLS");
            context.init(null, trustManagers, null);
            return context;
        } catch (NoSuchAlgorithmException e) {
            Log.e(TAG, "Failed to initialize SSL context with TLS algorithm", e);
        }
        return null;
    }
}
```

最後透過openConnection建立連線。
```java
private String connect(URL url) throws IOException {
    URLConnection connection = url.openConnection();
    if (null != mSSLContext && connection instanceof HttpsURLConnection) {
        ((HttpsURLConnection) connection).setSSLSocketFactory(mSSLContext.getSocketFactory());
    }
    InputStream in = connection.getInputStream();
    return readStream(in);
}
```


[i1]: http://iforensicsblog.blogspot.tw/2017/11/blog-post_30.html
[i2]: https://developer.android.com/training/articles/security-ssl.html
[i3]: https://github.com
[i4]: https://github.com/7a6ac0/SSLPinning
[i5]: https://tabacowang.me/2018/10/12/okhttp-pinning/
