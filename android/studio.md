# Apk分析器
> 内容：  
> 查看文件和大小信息  
> 查看AndroidManifest.xml  
> 查看dex文件  
> 过滤dex文件树视图  

## 查看文件和大小
Apk是遵循zip文件格式的文件。Apk分析器可以显示原始大小和下载大小。其中，**原始大小**：解压缩后的大小，**下载大小**：估计压缩过后代大小

![](https://developer.android.google.cn/studio/images/build/apk-file-sizes_2x.png)

## 查看AndroidManifest
如果你的项目包括多个AndroidManifest.xml文件，它们会被合并到一个单独的文件。这个文件在APK中是一个二进制的文件

![](https://developer.android.google.cn/studio/images/build/apk-manifest-error_2x.png)

## 查看dex文件
类、包、总引用和声明计数在查看器中提供，这可以帮助决定是否使用multidex或如何删除依赖项，以使其低于64k dex限制

![](https://developer.android.google.cn/studio/images/build/apk-over-64k-limit_2x.png)

## 过滤dex文件树视图

## apkanalyzer命令行
`apkanalyzer`:包含在Android SDK Tools包中，位于android_sdk/tools/bin/apkanalyzer
### 语法
```
  apkanalyzer [global-options] subject verb [options] apk-file [apk-file2]
```
subject:是你想要查找的内容，可以是整个apk或apk的一部分
* `apk`:分析apk文件属性，如：application ID,version code,versin name
* `files`:分析apk文件中的文件
* `manifest`:分析清掉文件
* `dex`:分析dex文件
* `resources`:查看text、image和string资源
```
  apkanalyzer -h apk file-size myapk.apk
```
#### Global options
|选项|描述|
|--human-readable|人类可读格式|

### Commands
|Apk文件属性|描述|
|apk summary myapk.apk|输出application ID，version code,version name,如`com.ckr.app 1 1.0`|
|apk file-size myapk.apk|输出apk文件总大小|
|apk download-size myapk.apk|输出压缩过后的文件大小|
|apk compare [options] myapk.apk myapk2.apk|比较两个apk文件，options有：`--different-only`,`--files-only`,`--patch-size`|

### Dex
|dex文件信息|描述|
|dex list myapk.apk|输出apk中的dex文件列表|
|dex references [--files path] [--files path2] myapk.apk|输出指定dex文件的方法引用数|






















