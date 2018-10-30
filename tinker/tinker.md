# Tinker
Tinker是一个支持不需要重新安装apk情况下进行dex、库和资源更新的andriod热修复解决方案库。
## 使用指南
### 添加插件依赖
```groovy
  buildscript {
    dependencies {
      classpath "com.tencent.bugly:tinker-support:1.1.2"
    }
  }
```
注意：自tinker-support 1.0.3版本起无需再配置tinker插件的classpath
### 集成sdk
```groovy
  apply plugin: 'tinker-support.gradle'
  android {
    defaultConfig {
      ndk {
        //设置支持so库
        abiFilters 'armabi','x86','armabi-v7a','x86_64','arm64-v8a'
      }
    }
  }
  dependencies {
    compile "com.android.support:multidex:1.0.1"
    compile 'com.tencent.bugly:crashreport_upgrade:1.3.5'
    compile 'com.tencent.tinker:tinker-android-lib:1.9.6'
    compile 'com.tencent.bugly:nativecrashreport:lastest.release'
  }
```
tinker-support.gradle文件：
```groovy
  apply plugin: 'com.tencent.bugly.tinker-support'
  //此处填写每次构建生产的基准包目录
  def bakPath = file("${buildDir}/bakApk/")
  //此处填写每次构建生产的基准包文件
  def baseApkDir = "app_xxx"
  
  tinkerSupport {
    //开启tinker-support插件，默认值true
    enable = true
    //自定归档目录，默认值当前module的子目录tinker
    autoBackupApkDir = "${bakPath}"
    
    //是否启用覆盖tinkerPatch配置功能，默认值false
    //开启后，tinkerPatch配置不生效，即无需添加tinkerPatch
    overrideTinkerPatchConfiguration=true
    
    //编译补丁包，必须指定基准版本的apk
    baseApk="${bakPath}/${baseApkDir}/app-release.apk"
    
    //对应tinker插件applyMapping
    baseApkProguardMapping = "${bakPath}/${baseApkDir}/app-release-mapping.txt"
    //对应tinker插件的applyResourceMapping
    baseApkResourceMapping="${bakPath}/${baseApkDir}/app-release-R.txt"
    
    //构建基准包和补丁包都要指定不同的tinkerId,并且必须保持唯一性
    tinkerId ="base-1.0.1"
    
    //构建多渠道补丁时使用
    // buildAllFlavorsDir="${bakPath}/${baseApkDir}"
    
    //是否启用加固模式，默认为false
    // isProtectedApp = true
    
    //是否开启反射Application模式
    enableProxyApplication= false
    
    //是否支持新增非export的Activity(注意：设置为true才能修改AndroidManifest文件)
    supportHotplugComponent=true
  }
  //一般，无需对下面的参数做任何修改
  tinkerPatch {
    ignoreWarning =false
    useSign =true
    dex{
      dexMode="jar"
      pattern=["classes*.dex"]
      loader=[]
    }
    lib {
      pattern=["lib/*/*.so"]
    }
    res {
      pattern =["res/*","r/*","assets/*","resources.arsc","AndroidManifest.xml"]
      ignoreChange=[]
      largeModSize =100
    }
    packageConfig {}
    sevenZip {
      zipArtifact="com.tencent.mm:SevenZip:1.1.10"
    }
    buildConfig {
      keepDexApply = false
    }
  }
```
|参数|默认值|描述|
|---|---|---|
|ignoreWarnig|false|如果出现以下的情况，且ignoreWarning为false,我们将中断编译。1.minSdkVersion小于14，但是dexMode的值为"raw"2.新编译的安装包出现新增的四大组件3.定义在dex.loader用于加载补丁的类不在main dex中4.定义在dex.loader用于加载补丁的类出现修改5.resources.arsc改变，但没有使用applyResourceMapping编译|
|useSign|true|在运行过程中，需要验证基准apk与补丁包的签名是否一致|
|dexMode|jar|对于raw模式：我们将会保持输入dex的格式；对于jar模式，我们将会把输入dex重新压缩封装到jar|
|pattern|[]|需要处理dex路径，支持*、?通配符，必须使用/分割。路径是相对安装包，如：assets/...|
|loader|[]|它定义哪些类在加载补丁包的时候会用到。这些类是通过Tinker无法修改的类，也是一定要放在main dex的类。这里需要定义的类有：1.自定义的Application类；2.Tinker库中用于加载补丁包的部分类，即com.tencent.tinker.loader.*;3.如果自定义了TinkerLoader,需要将它以及它引用的所有类也加入loader中；4.其他一些不希望被更改的类。如：BaseBuildeInfo类。5.使用1.7.6版本后参数1、2会自动填写|
|ignoreChange|[]|若满足ignoreChange的pattern，在编译时会忽略该文件的新增、删除和修改。|
|largeModSize|100|对于修改的资源，如果大于largeModSize，将使用bsdiff算法。这可以降低补丁包的大小，但是会增加合成时的复杂度。默认值100kb|
|sevenZip||7zip路径配置项，执行前提是useSign为true|
|zipArtifact|null|将自动根据机器属性获得对应7za运行文件|

### 自定义Application
```java
  public class CkrApp extends TinkerApplication {
    public CkrApp(){
      super(7,"com.ckr.CkrAppLike","com.tencent.tinker.loader.TinkerLoader",false);
    }
  }
```
|参数|说明|
|---|---|
|tinkerFlags|表示Tinker支持的类型dex only、library only or all support,默认Tinker_enable_all|
|delegateClassName|填写自定义的ApplicationLike|
|loaderClassName|Tinker加载器|
|tinkerLoadVerifyFlag|加载dex或者lib是否验证md5|  
```java
 public class CkrAppLike extends DefaultApplicationLike {
   public SampleApplicationLike(Application application, int tinkerFlags,
            boolean tinkerLoadVerifyFlag, long applicationStartElapsedTime,
            long applicationStartMillisTime, Intent tinkerResultIntent) {
        super(application, tinkerFlags, tinkerLoadVerifyFlag, applicationStartElapsedTime, applicationStartMillisTime, tinkerResultIntent);
    }
    
    @Override
    public void onCreate(){
      super.onCreate();
      Bugly.init(getApplication,"1234567",false);
    }
    @TargetApi(Build.VERSION_CODES.ICE_CREAM_SANDWICH)
    @Override
    public void onBaseContextAttached(Context base){
      super.onBaseContextAttached(base);
      MultiDex.install(base);
      
      //安装tinker
      Beta.installTinker(this);
    }
    
    @TargetApi(Build.VERSION_CODES.ICE_CREAM_SANDWICH)
    public void registerActivityLifecyclerCallback(ActivityLifecycleCallbacks callbacks){
      getApplication().registerActivityLifecycleCallbacks(callbacks);
    }
 }
```
### AndroidManifest.xml配置
```xml
  <uses-permission android:name="android.permission.READ_PHONE_STATE" />
  <uses-permission android:name="android.permission.INTERNET" />
  <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
  <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
  <uses-permission android:name="android.permission.READ_LOGS" />
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
  
  <activity
    android:name="com.tencent.bugly.beta.ui.BetaActivity"
    android:configChanges="keyboardHidden|orientation|screenSize|locale"
    android:theme="@android:style/Theme.Translucent" />
    
  <provider
    android:name=".utils.BuglyFileProvider"
    android:authorities="${applicationId}.fileProvider"
    android:exported="false"
    android:grantUriPermissions="true"
    tools:replace="name,authorities,exported,grantUriPermissions">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/provider_paths"
        tools:replace="name,resource"/>
  </provider>
```
### 混淆配置
```java
  -dontwarn com.tencent.bugly.**
  -keep public class com.tencent.bugly.**{*;}
  # tinker混淆规则
  -dontwarn com.tencent.tinker.**
  -keep class com.tencent.tinker.** { *; }
```
# 使用指南
## 完整接入流程
* 打基准包安装并上报联网(注：填写唯一的tinkerId)
* 对基准包的bug修复(可以java代码变更或资源变更)
* 修改基准包路径、修改补丁包tinkerId、mapping文件路径(如果开启混淆需要配置)、resId文件路径
* 执行buildTinkerPatchRelease打Release版本补丁包
* 选择app/build/outputs/patch目录下的补丁包并上传(注：不要选择tinkerPatch目录下的补丁包，不然上传会有问题)
* 编辑下发补丁规则，点击立即下发
* 杀死进程并重启基准包，请求补丁策略(sdk会自动下载补丁并合成)
* 再次重启基准包，校验补丁应用结果
* 查看页面，查看激活数据的变化

## 编译基准包
* 1.配置
![](https://bugly.qq.com/docs/img/hotfix/android/Snip20170113_3.png?v=20181014122344)

强调：基线版本配置一个唯一的tinkerId,而这个基线版本能够应用补丁的前提是集成过热更新sdk,并启动上报过联网，这样我们后台将会这个tinkerId对应到一个目标版本，如：thinkId="bugly_1.0.0"对应一个目标版本是1.0.0，基于这个版本打的补丁包就能匹配到目标版本。
* 2.编译基准包
![](https://bugly.qq.com/docs/img/hotfix/android/Snip20170113_4.png?v=20181014122344)

build/outputs/bakApk路径下生成每次编译基准包、混淆配置文件、资源id文件，如：
![](https://bugly.qq.com/docs/img/hotfix/android/Snip20170209_2.png?v=20181014122344)

实际应用中，请注意保存线上发布版本的基准apk包、mapping文件、R.txt文件，如果线上版本有bug，就可以借助我们tinker-support插件进行补丁包的生成。
我们每次冷启动都会请求补丁策略，会上报当前版本号和tinkerId,这样我们后台就能将这个唯一的tinkerId对应到一个版本。
## 根据基线版本生成补丁包
![](https://bugly.qq.com/docs/img/hotfix/android/Snip20170113_12.png?v=20181014122344)

执行构建补丁包：
![](https://bugly.qq.com/docs/img/hotfix/android/Snip20170209_4.png?v=20181014122344)

补丁版本怎么匹配目标版本，可以双击patch包
![](https://bugly.qq.com/docs/img/hotfix/android/Snip20161215_54.png?v=20181014122344)  
![]()

## 上传补丁包到平台
![](https://bugly.qq.com/docs/img/hotfix/android/1479199765798.png?v=20181014122344)




