---
layout: post
title: "Android 网络--我是怎么做的: Volley+OkHttp+Https"
summary:
comments: true
categories: Android
---

> Volley 已经发布很长时间了, 也已被广泛应用, 相关教程到处都是. 本文只说两个值得注意的地方.
> 本文讲解部分比较少, 请参阅提供的相关链接. 完整的实现代码在 [Github dodocat/AndroidNetworkDemo] 可能看起来比这里更清晰.
> 本文同步发布于简书: [](http://www.jianshu.com/p/e58161cbc3a4)

##  使用 OkHttp 作为传输层的实现.
Volley 默认根据 Android 系统版本使用不同的 Http 传输协议实现.
在 Android 2.3以下使用 `ApacheHttpStack ` 作为传输协议, 在 3.0 及以下使用 `HttpURLConnection` 作为传输层协议 (感谢评论中指正的朋友).

OkHttp 相较于其它的实现有以下的优点.
* 支持[SPDY](http://zh.wikipedia.org/wiki/SPDY)，允许连接同一主机的所有请求分享一个socket。
* 如果SPDY不可用，会使用连接池减少请求延迟。
* 使用GZIP压缩下载内容，且压缩操作对用户是透明的。
* 利用响应缓存来避免重复的网络请求。
* 当网络出现问题的时候，OKHttp会依然有效，它将从常见的连接问题当中恢复。
* 如果你的服务端有多个IP地址，当第一个地址连接失败时，OKHttp会尝试连接其他的地址，这对IPV4和IPV6以及寄宿在多个数据中心的服务而言，是非常有必要的。

因此使用 OkHttp 作为替代是好的选择.

1. 首先用 OkHttp 实现一个新的 `HurlStack` 用于构建 Volley 的 requestQueue.

``` java
public class OkHttpStack extends HurlStack {
    
    private OkHttpClient okHttpClient;

    /**
     * Create a OkHttpStack with default OkHttpClient.
     */
    public OkHttpStack() {
        this(new OkHttpClient());
    }

    /**
     * Create a OkHttpStack with a custom OkHttpClient
     * @param okHttpClient Custom OkHttpClient, NonNull
     */
    public OkHttpStack(OkHttpClient okHttpClient) {
        this.okHttpClient = okHttpClient;
    }

    @Override
    protected HttpURLConnection createConnection(URL url) throws IOException {
        OkUrlFactory okUrlFactory = new OkUrlFactory(okHttpClient);
        return okUrlFactory.open(url);
    }
}
```

1. 然后使用 OkHttpStack 创建新的 Volley requestQueue. 这样就行了.

``` java
requestQueue = Volley.newRequestQueue(getContext(), new OkHttpStack());
requestQueue.start();
```

# 使用 Https

作为一个有节操的开发者应该使用 Https 来保护用户的数据, Android 开发者网站上文章 [Security with HTTPS and SSL] 做了详尽的阐述.

OkHttp 自身是支持 Https 的. 参考文档 [OkHttp Https], 直接使用上面的 `OkHttpStack` 就可以了, 但是如果遇到服务器开发哥哥使用了自签名的证书(不要问我为什么要用自签名的), 就无法正常访问了.

 网上有很多文章给出的方案是提供一个什么事情都不做的`TrustManager` 跳过 `SSL` 的验证, 这样做很容受到攻击, Https 也就形同虚设了.

我采用的方案是将自签名的证书打包入 APK 加入信任.

好处:
* 应用难以逆向, 应用不再依赖系统的 trust store, 使得 Charles 抓包等工具失效. 要分析应用 API 必须反编译 APK.
* 不用额外购买证书, 省钱....

缺点:
* 证书部署灵活性降低, 一旦变更证书必须升级程序.

## 实现步骤
以最著名的自签名网站12306为例说明

* 导出证书

``` sh
echo | openssl s_client -connect kyfw.12306.cn:443 2>&1 |  sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > kyfw.12306.cn.pem
```

* 将证书转为 bks 格式
    下载最新的bcprov-jdk, 执行下面的命令. storepass 是导出密钥文件的密码.

``` sh
keytool -importcert -v \
    -trustcacerts \
    -alias 0 \
    -file <(openssl x509 -in kyfw.12306.cn.pem) \
    -keystore $CERTSTORE -storetype BKS \
    -providerclass org.bouncycastle.jce.provider.BouncyCastleProvider \
    -providerpath ./bcprov-jdk16-1.46.jar \
    -storepass asdfqaz
```

* 将导出的 kyfw.bks 文件放入 res/raw 文件夹下.

* 创建 `SelfSignSslOkHttpStack`

``` java
/**
 * A HttpStack implement witch can verify specified self-signed certification.
 */
public class SelfSignSslOkHttpStack extends HurlStack {

    private OkHttpClient okHttpClient;

    private Map<String, SSLSocketFactory> socketFactoryMap;

    /**
     * Create a OkHttpStack with default OkHttpClient.
     */
    public SelfSignSslOkHttpStack(Map<String, SSLSocketFactory> factoryMap) {
        this(new OkHttpClient(), factoryMap);
    }

    /**
     * Create a OkHttpStack with a custom OkHttpClient
     * @param okHttpClient Custom OkHttpClient, NonNull
     */
    public SelfSignSslOkHttpStack(OkHttpClient okHttpClient, Map<String, SSLSocketFactory> factoryMap) {
        this.okHttpClient = okHttpClient;
        this.socketFactoryMap = factoryMap;
    }

    @Override
    protected HttpURLConnection createConnection(URL url) throws IOException {
        if ("https".equals(url.getProtocol()) && socketFactoryMap.containsKey(url.getHost())) {
            HttpsURLConnection connection = (HttpsURLConnection) new OkUrlFactory(okHttpClient).open(url);
            connection.setSSLSocketFactory(socketFactoryMap.get(url.getHost()));
            return connection;
        } else {
            return  new OkUrlFactory(okHttpClient).open(url);
        }
    }
}
```

* 然后用 `SelfSignSslOkHttpStack` 创建 Volley 的 RequestQueue.

``` java
    String[] hosts = {"kyfw.12306.cn"};
    int[] certRes = {R.raw.kyfw};
    String[] certPass = {"asdfqaz"};
    socketFactoryMap = new Hashtable<>(hosts.length);

    for (int i = 0; i < certRes.length; i++) {
        int res = certRes[i];
        String password = certPass[i];
        SSLSocketFactory sslSocketFactory = createSSLSocketFactory(context, res, password);
        socketFactoryMap.put(hosts[i], sslSocketFactory);
    }

    HurlStack stack = new SelfSignSslOkHttpStack(socketFactoryMap);

    requestQueue = Volley.newRequestQueue(context, stack);
    requestQueue.start();
```

* 我们来试一试, 用上一步穿件的 RequestQueue 替换掉原来的, 然后发请求试试.

``` java
StringRequest request = new StringRequest(
    Request.Method.GET,
    "https://kyfw.12306.cn/otn/",
    new Response.Listener<String>() {
        @Override
        public void onResponse(String response) {
            responseContentTextView.setText(response);
        }
    },
    new Response.ErrorListener() {
        @Override
        public void onErrorResponse(VolleyError error) {
            responseContentTextView.setText(error.toString());
        }
    });
RequestManager.getInstance(this).addRequest(request, this);
```

1. done


[Volley]:http://developer.android.com/training/volley/index.html
[OkHttp]:http://square.github.io/okhttp/
[Gson]:https://github.com/google/gson

[Security with HTTPS and SSL]:https://developer.android.com/training/articles/security-ssl.html
[OkHttp Https]:https://github.com/square/okhttp/wiki/HTTPS
[Github dodocat/AndroidNetworkDemo]:https://github.com/dodocat/AndroidNetworkdemo
