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
# RN
[react native运行原理解析](https://blog.csdn.net/xiangzhihong8/article/details/52623852)

















