# 混淆使用
## 混淆启动
```Groovy
buildTypes {
  release {
    minifyEnabled true  //启动混淆
    shrinkResources true //优化未使用的资源文件
    zipAlignEnabled true //zip优化安装包大小
    //混淆文件
    proguardFiles getDefaultProguardFile('proguard-android.txt'),'proguard-rules.pro'
  }
}
```
## 混淆通用模板
```
 # 指定代码的压缩级别
 -optimizationpasses 5
 
 -ignorewarnings #忽略警告
 -verbose #混淆时是否记录日志(混淆后生产映射文件map类名 -> 转化后类名的映射)
 -dontpreverify #不预校验
 -dontoptimize #不优化输入类的文件
 -dontshrink #该选项 表示 不启用压缩  混淆时是否做预校验(可去掉加快混淆速度)
 
 -dontusemixedcaseclassnames #混淆时不会产生形形色色的类名，是否使用大小写混合
 -dontskipnonpubliclibraryclasses #不跳过jars中的非public classes
 
 -keepattributes Exceptions #解决AGPBI警告
 -keepattributes Exceptions,InnerClasses
 -keepattributes EnclosingMethod
 -keepattributes SourceFile,LineNumberTable #运行抛出异常时保留代码行号
 -keepattributes Signature #过滤泛型
 #过滤注解
 -keepattributes *Annotation* 
 -keep class * extends java.lang.annotation.Annotation { *; }
 -keep interface * extends java.lang.annotaion.Annotation { *; }
 
 #继承四大组件、application...不进行混淆
 -keep public class * extends android.app.Application
 -keep public class * extends android.app.Activity
 -keep public class * extends android.app.Fragment
 -keep public class * extends android.app.Service
 -keep public class * extends android.content.BroadcastReceiver
 -keep public class * extends android.content.ContentProvider
 -keep public class * extends android.app.backup.BackupAgentHelper
 -keep public class * extends android.preference.Preference
 -keep public class * extends android.support.v4.**
 -keep public class * extends com.android.vending.licensing.ILicensingService
 -keep public class * extends android.support.multidex.MultiDexApplication
 -keep public class * extends android.view.View
 -keep class andrid.support.** { *; }
 #保留View的set/get方法
 -keepclassmembers public class * extends android.view.View {
  void set*(***);
  *** get*();
 }
 -keep public class * extends android.view.View {
  *** get*();
  void set*(***);
  public <init>(android.content.Context);
  public <init>(android.content.Context,android.util.AttributeSet);
  public <init>(android.content.Context,android.util.AttributeSet,int);
 }
 #保持指定规则的方法不被混淆(如：onClick方法)
 -keepclassmembers class * extends android.app.Activity {
  public void *(android.view.View);
 }
 # 对于带有回调函数onXXEvent的，不能被混淆
 -keepclassmembers class * {
  void *(*Event);
 }
 #保持R文件不被混淆
 -keep class **.R$* { *; }
 #不混淆R类及其所有内部static类中的所有static变量字段,$是用来分割内嵌类与其母体的标志
 -keep public class **.R$*{
  public static final int *;
 }
 -keepclassmembers class **.R$* {
  public static <fields>;
 }
 
 #过滤Js
 -keepattributes *JavascriptInterface*
 #保护webView对html页面的api不被混淆
 -keep class **.Webview2JsInterface { *; }
 -keepclassmembers class * extends android.webkit.WebViewClient {
  public void *(android.webkit.WebView,java.lang.String,android.graphics.Bitmap);
  public void boolean *(android.webkit.WebView,java.lang.String)
 }
 -keepclassmembers class * extends android.webkit.WebChromeClient {
  public void *(android.webkit.WebView,java.lang.String);
 }
 
 #保持自定义控件类不被混淆,指定构造方法不去混淆
 -keepclasseswithmembers class * {
  public <init>(android.content.Context,android.util.AttributeSet);
 }
 -keepclasseswithmemebers class * {
  public <init>(android.content.Context,android.util.AttributeSet,int);
 }
 
 #保持枚举不被混淆
 -keepclassmembers enum * {
  public static **[] values();
  public static ** valueOf(java.lang.String);
 }
 #保持native方法不被混淆
 -keepclasseswithmemebernames class * {
  native <methods>;
 }
 #保持Parcelable不被混淆
 -keep class * implements android.or.Parcelable {
  public static final android.os.Parcelable$Creator *;
 }
 -keep public class * implements java.io.Serializable {
  public *;
 }
 #不混淆Serializable接口的子类中指定的某些成员变量和方法
 -keepclassmembers class * implements java.io.Serializable {
  static final long serialVersionUID;
  private static final java.io.ObjectStreamField[] serialPersistentFields;
  !static !transient <fields>;
  private void writeObject(java.io.ObjectOutputStream);
  private void readObject(java.io.ObjectInputStream);
  java.lang.Object writeReplace();
  java.lang.Object readResolve();
 }
 
 -keepclassmemebers class * {
  public <init> (org.json.JSONObject);
 }
 #v7不混淆
 -keep class android.support.v7.** { *; }
 -keep interface android.support.v7.** { *; }
 -dontwarn android.support.v7.**
 #support-design
 -dontwarn android.support.design.**
 -keep class android.support.design.** { *; }
 -keep interface android.support.design.** { *; }
 -keep public class android.support.design.R$* { *; }
 #support v7 appcompat
 -dontwarn android.support.v7.**
 -keep class android.support.v7.** { *; }
 -keep class android.support.v7.internal.** { *; }
 -keep interface android.support.v7.internal.** { *; }
 -keep public class android.support.v7.widget.** { *; }
 -keep public class android.support.v7.internal.widget.** { *; }
 -keep public class android.support.v7.internal.view.menu.** { *; }
 #support v4
 -dontwarn android.support.v4.**
 -keep class android.support.v4.app.** { *; }
 -keep interface android.support.v4.app.** { *; }
 -keep class android.support.v4.** { *; }
 -keep public class * extends android.support.v4.view.ActionProvider {
  public <init>(android.content.Context);
 }
 #support v7 cardview
 -keep class android.support.v7.widget.RoundRectDrawable { *; }
 
 -dontwarn andorid.net.http.**
 -keep class org.apache.http.** { *; }

```

# 感谢
[博客1](https://blog.csdn.net/ouyang_peng/article/details/73088090)
[博客2](https://www.jianshu.com/p/a1c7d0d5f194)
