# Http学习
## 一、请求头
### 1.User-Agent
User-Agent是http协议中的一部分，属于头域的组成部分。它是个特殊字符串头，是一种向访问网站提供你所使用的浏览器类型及版本、操作系统及版本、浏览器内核等信息的标识。
通过这个标识，用户所访问的网站可以显示不同的排版，从而为用户提供更好的体验或者进行信息统计。java服务器通过以下代码获取：
```java
  String userType = request.getHeader("User-Agent");
```
## http缓存处理
### 1.为什么要做缓存处理
减少服务器负荷，降低延迟提升用户体验。复杂的缓存策略会根据用户当前的网络情况采取不同的缓存策略，比如在2g网络很差的情况下，提高缓存使用的时间；
不同的应用、业务需求、接口所需要的缓存策略也会不一样，有点要保证数据的实时性，所以不能有缓存，有的你可以缓存5分钟，等待。你要根据具体情况所需
数据的时效性情况给出不同的方案。
### 2.retrofit+okhttp的缓存机制
在响应请求之后在data/data/<包名>/cache下建立一个response文件夹，保存缓存数据。
#### 缓存实现方式
先开启okhttp缓存
```java
  File httpCacheDir = new File(mContext.getCacheDir(),"responses");
  int cacheSize = 10*1024*1024;
  Cache cache = new Cache(httpCacheDir,cacheSize);
  OkHttpClient client = new OkHttpClient.Builder()
                                        .addInterceptor(mInterceptor)
                                        .cache(cache).build();
```
设置okhttp拦截器:主要是拦截操作，包括控制缓存的最大生命值，控制缓存的过期时间
* 通过CacheControl控制缓存数据
```java
  CacheControl.Builder cacheBuilder = new CacheControl.Builder();
  cacheBuilder.maxAge(0,TimeUnit.SECONDS);//最大生命时间
  cacheBuilder.maxStale(365,TimeUnit.DAYS);//过时时间
  CacheControl cacheControl = cacheBuilder.build();
```
* 设置拦截器
```java
  Request request = chain.request();
  if(!isNetworkAvailable(mContext)){
    request = request.newBuilder()
              .cacheControl(cacheControl)
              .build();
  }
  Response oriResponse = chain.proceed(request);
  if(isNetworkAvailable(mContext)){
    int maxAge = 60;
    return oriResponse.newBuilder()
                      .removeHeader("Pragma")
                      .header("Cache-Control","public ,max-age="+maxAge)
                      .build();
  }else{
    int maxStale = 60*60*24*28
    return oriResponse.newBuilder()
                      .removeHeader("Pragma")
                      .header("Cache-Control","public, only-if-cached, max-stale="+maxStale)
                      .build();
  }
```
如果.maxAge(0,TimeUnit.SECONDS)设置的时间比拦截器长是不起效果，如果设置比拦截器设置的时间短就会以这个时间为主。.maxStale(365,TimeUnit.DAYS)
设置的是过时时间。okhttp缓存分成两个来考虑，一个是为了请求时直接拿缓存省流量，一个是为了下次进入应用时可以直接拿缓存。
#### 注意
* 缓存是在每次网络请求之后，重新保存的，所以在超过缓存时间后，Retrofit会在检查到没缓存之后自动请求网络服务器数据
* 缓存数据也是需要网络下载的，所以在网络不好情况下，可能不能立即缓存

### 感谢
[okhttp缓存处理](https://werb.github.io/2016/07/29/%E4%BD%BF%E7%94%A8Retrofit2+OkHttp3%E5%AE%9E%E7%8E%B0%E7%BC%93%E5%AD%98%E5%A4%84%E7%90%86/)











