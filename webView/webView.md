# Webview
webView加载页面的方式：
* 加载网络页面
```java
  WebView webView = new WebView(context);
  webView.loadUrl("https://www.google.com");
```
* 加载本地页面
1.加载assets目录下的html页面：assets目录的页面，大多数可用来做页面数据的存储打包。
```java
  webView.loadUrl("file:///android_asset/ckr.html");
```
注意，file字段表示读取本地文件；android_asset表示读取当前应用的assets目录下的文件；ckr.html表示assets目录下的文件
2.加载缓存到本地的页面：主要做页面的离线缓存
```java
  File destFile = new File(getFilesDir().getAbsoluteFile(),"ckrcopy.html");
  InputStream in=null;
  try{
    in=getAssets().open("ckr.html");
    if(destFile.exists()){
      destFile.delete();
    }
    FileOutputStream out = new FileOutputStream(destFile);
    try{
      byte[] buffer = new byte[1024];
      int bytesLen;
      while((bytesLen=is.read(buffer))>=0){
        out.write(buffer,0,bytesLen);
      }
    }finally {
      out.flush();
      out.close();
    }
  }catch(IOException e){
    e.printStatckTrace();
  }
  String url="file://"+destFile.getAbsolutePath();
  webView.loadUrl(url);
```
## webView加载流程
初始化webView -> 请求页面 -> 下载数据 -> 解析html -> 请求js/css资源 -> dom渲染 -> 解析js执行 -> js请求数据 -> 解析渲染 -> 下载渲染图片
## webview加载慢
* 手机硬件：性能较差，会导致渲染速度慢
* js解析效率低：如果js文件较多、解析较复杂，就会导致渲染速度慢。
* 页面资源下载：一般加载一个h5页面，都会产生较多的网络请求，如图片、js文件、css文件等，需要将这些资源下载完后才能完成渲染
## 浏览器缓存机制
根据http协议头的Cache-Control(或Expires)和Last-Modified(或Etag)等字段来控制文件缓存的机制
* Cache-Control: 例如Cache-Control:max-age=24*60*60,表示缓存时长1天时间。如果一天内再次请求这个文件，浏览器不会发出请求，直接使用本地的缓存文件。
* Expires: 如Expires:Tue,20 Oct 2018 21:44:00 GMT,表示这个文件的过期时间是格林尼治时间2018.10.20 21:44:00。这个时间前不会再次发出请求。Expires和Cache-control同时出现时，Cache-Control优先级更高
* Last-Modified: 标识文件在服务器上的最新更新时间。下次请求时，如果文件缓存过期，浏览器通过If-Modified-Since字段带上这个时间，发送给服务器，由服务器比较时间戳来判断文件是否有修改。如果没修改，服务器返回304告诉浏览器继续使用缓存；如果有修改，返回200，同时返回最新的文件。
* Etag: 一个对文件进行标识的特征字串。在向服务器查询文件是否有更新时，浏览器通过If-None-Match字段八特征字串发送给服务器，由服务器和文件最新特征字串进行匹配，来判断文件是否有更新。没有更新返回304，有更新返回200。Etag和Last-Modified同时使用，只要满足一个条件，就认为文件没有更新。
## webView缓存模式
* LOAD_CACHE_ONLY: 不使用网络，只读取本地缓存数据。
* LOAD_DEFAULT: 根据cache-control决定是否从网络上取数据
* LOAD_NO_CACHE: 不使用缓存，只从网络获取数据
* LOAD_CACHE_ELSE_NETWORK: 只要本地有，无论是否过期，都使用缓存中的数据。本地没有缓存才从网络上获取
## 首屏速度优化
* webView预初始化：如在Application预先初始化WebView
* 预加载数据：在初始化webView同时，开始网络请求页面所需的数据
* 离线包：将h5的页面和资源进行打包后下发到客户端，并由客户端解压到本地存储。但更新机制复杂化

# RN
[react native运行原理解析](https://blog.csdn.net/xiangzhihong8/article/details/52623852)

# 解决方案
[腾讯VasSonic](https://mp.weixin.qq.com/s?__biz=MzA3NTYzODYzMg==&mid=2653579269&idx=2&sn=bb9822d2cd9b0dc79134bd9990220571&chksm=84b3ba02b3c43314aeb54375f3e729fcef5495ab39a64c94719e218f28caa1fc4dde50f7c92d&mpshare=1&scene=23&srcid=0825CYa3KqzW532zZLQwVnnH%23rd)  
[webView性能优化](https://tech.meituan.com/WebViewPerf.html)  
[页面框架优化参考点](https://stevesouders.com/hpws/rules.php)  
[h5秒开方案实现](https://juejin.im/post/5b94ca52e51d450e7d097f38)  
[H5秒开方案探索](https://mp.weixin.qq.com/s?__biz=MzA3NTYzODYzMg==&mid=2653579803&idx=1&sn=0a1f418f628a53e9262b97879f47593b&chksm=84b3b81cb3c4310a175b95264bf74a5be8abf39751ed0b21a353e68e6cf1846db11995bc2fec&mpshare=1&scene=23&srcid=08258x6omkd9AHgWWz81G80B%23rd)  
[首屏秒开优化方案探讨](https://mp.weixin.qq.com/s?__biz=MzA3NTYzODYzMg==&mid=2653579269&idx=1&sn=e3074cf75b824622bf96a2a488066e88&chksm=84b3ba02b3c43314f47db7c685de13089e1f61d9246211b27657c0649f6e7f2a3f53b22cc801&mpshare=1&scene=23&srcid=0825SymzNBsExO1hDt6YXpHA%23rd)  
[大众点评hybrid化建设](https://mp.weixin.qq.com/s?__biz=MzA3NTYzODYzMg==&mid=2653578296&idx=2&sn=03cc579cb7e016f8bfd4ba994b0a5947&chksm=84b3b63fb3c43f2956acad4f8d1c0a4d01e3692c829cca4fa0ad94fc554d7fe33e18db8e2d4c&mpshare=1&scene=23&srcid=08252SpCDqB2HymHiVSXA4eu%23rd)  
[如何让webView访问更快](https://my.oschina.net/yale8848/blog/1544298)  
[构建WebView缓存机制&资源加载方案](https://www.jianshu.com/p/5e7075f4875f)

[VasSonic wiki](https://github.com/Tencent/VasSonic/wiki)  
[Native、Hybrid、RN、Web App对比](https://www.cnblogs.com/dailc/p/5930238.html)














