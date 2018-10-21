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

# VasSonic
一个轻量级的高性能的Hybrid框架，专注于提升页面`首屏加载速度`，完美支持静态直出和动态直出页面，兼容离线包等方案。
该框架使用终端应用层原生传输通道取代系统浏览器内核自身资源传输通道来请求页面主资源，在移动终端初始化的同时`并行请求`页面主资源并做到`流式拦截`，
减少传统方案上终端初始化耗时长导致页面主资源发起请求时机慢或传统并行方案下必须等待主资源完成下载才能交给内核加载的影响。
另外通过`客户端和服务端双方遵守VasSonic格式规范`(通过在html内增加注释代码区分模板和数据)，该框架做到智能地对页面内容进行`动态缓存`和
`增量更新`，减少对网络的依赖和数据传输的大小，大大提升h5页面加载速度。
## 页面规范约定
sonic将页面分割为不经常变化的`模板(template)和经常变化的数据块(data)`。页面将整个html通过VasSonic标签进行划分，包裹在标签中的内容为data，
标签外的内容为模板。
* VasSonic标签：
```html
<!-- sonicdiff-moduleName --> xxx <!-- sonicdiff-moduleName-end -->
```
上面整个模块称为一个数据块，moduleName表示数据块的tag，xxx是该tag对应的数据块的内容。如：
```html
  <html>
  <div id="data1">
    <!--sonicdiff-data1-->
    <p> id="partialRefresh">数据块1</p>
    <!--sonicdiff-data1-end-->
  </div>
  <div id="data2">
    <!--sonicdiff-data2-->
    <p id="data2">数据块2</p>
    <!--sonicdiff-data2-end-->
  </dive>
  </html>
```
将数据块抽离之后，通过数据块的tag进行占位，最终成为模板：
```html
  <html>
  <div id="data1">
    {data1}
  </div>
  <div id="data2">
    {data2}
  </div>
  </html>
```
## 请求规范约定
VasSonic为了支持区分客户端是否支持增量更新等功能，对头部字段进行了扩展
|字段|说明|请求头(y/n)|响应头(y/n)|
|---|---|---|---|
|accept-diff|表示终端是否支持VasSonic模式，true为支持，否则不支持|yes|no|
|If-noen-match|本地缓存的etag，给服务端判断是否命中304|yes|no|
|etag|页面内容的唯一标识(hash值)|no|yes|
|template-tag|模板唯一标识(hash值)，客户端使用本地校验或服务端使用判断是否模板有变更|no|yes|
|template-change|标记模板是否变更，客户端使用|no|yes|
|cache-offline|客户端使用，根据不同类型进行不同行为|no|yes|

* cache-offline字段说明
|字段|说明|
|---|---|
|true|缓存到磁盘并展示返回内容|
|false|展示返回内容，无需缓存到磁盘|
|store|缓存到磁盘，如果已经加载缓存，则下次加载，否则展示返回内容|
|http|容灾字段，如果http表示终端6小时之内不会采用sonic请求该url|
## 模式介绍
|模式|说明|条件|
|---|---|---|
|首次加载|本地没有缓存，即第一次加载页面|etag为空值或template_tag为空值|
|完全缓存|本地有缓存，且缓存内容跟服务器内容完全一样|etag一致|
|数据更新|本地有缓存，本地模板内容跟服务器模板内容一样，但数据块有变化|etag不一致且template_tag一致|
|模板更新|本地有缓存，缓存的模板内容跟服务器的模板内容不一样|etag不一致且template_tag不一致|

## 动态缓存
客户端第一次加载符合VasSonic规范的页面后，延迟几秒后，会将页面抽离成模板和数据并保存到本地。此时终端缓存目录下，
该页面将对应三个缓存文件xxx.html、xxx.template、xxx.data，其中xxx是该页面唯一标识(即sonicSessionId)。
对于非首次加载场景，VasSonic优先加载本地缓存，终端请求并计算差异数据块，最终通知页面刷新变化元素即可完成增量更新。
主要涉及两个问题：如何获取差异数据以及如何局部刷新。
* 获取数据：数据更新模式下，后台会在响应头部返回template-change=false等字段，同时响应包体返回的内容不再是完整的html,
而是全部的数据块。此时客户端通过将本地数据块和服务器返回的数据块进行diff对比，可以得到变化的数据块
* 局部刷新
得到数据块后，客户端只需通知页面设置的回调接口(getDiffDataCallback)进行界面元素更新即可。这里是javascript通信方式，
也可以自由定义(使用webView标准的javaScript通信方式或使用伪协议的方式)。页面收到终端的返回数据根据返回数据中的code字
段判断当前sonic模式。
## 首次加载秒开
VasSonic缓存更新等逻辑可以脱离webView执行，因此VasSonic可以随时进行预加载。
场景
* 重要的活动页面预埋，可以通过后台push方式提前预加载
* 预测用户的打开页面行为，提前对预测页面进行预加载
### 页面唯一标识：sessionId
通过sessionId来唯一标识一个url。默认拼接规则：
```
sessionId=[user_account_id+"_"]+MD5(URL.authority+URL.path+URL.sonic_remain_params);
```
|字段|说明|
|---|---|
|user_account_id|当前用户的账号，可选值，由SonicSessionConfig.isAccountRelated决定|
|MD5|MD5函数|
|URL.authority|当前url域名的认证机构，一般为null|
|URL.sonic_remian_params|当前URL的sonic保留字段，sonic_前缀或者sonic_remain_params=xxx|

```
  user_account_id="kid";
  url="http://xxx.yyy.com/app/index.html?id=2&type=1&sonic_remain_params=id";
  sessionId="[kid_]"+MD5["xxx.yyy.com/app/index.html"+"id=2"]
```
注意：在VasSonic框架中，相同sessionId对应的SonicSession仅能同时存在一个，主要出于两个原因考虑：
1.如果同时运行多个相关的SonicSession的话，会导致缓存管理复杂化，且容易出现相互覆盖问题  
2.如果出现同时打开同一个url，将url回退为标准webView流程执行即可







