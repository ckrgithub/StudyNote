# 一、android studio
## overview
* 基于gradle编译的灵活系统
* 快速及多特性的模拟器
* 同一的android开发环境
* 及时运行：没有编译新的apk，而是推送改变的东西到正在运行的app
* 代码模板和github集成
* 大量的测试工具和框架
* Lint工具：捕捉性能、可用性和版本兼容性等问题
* c++和ndk支持
* 内置支持google云
### 1.主窗口
![](https://developer.android.google.cn/studio/images/intro/main-window_2-2_2x.png)
其中包含：工具栏、导航栏、编辑窗口、工具窗口栏、工具窗口、状态栏
快捷键：
![](../img/keymap.png)
ctrl+e：搜索最近使用的文件
ctrl+f12：该文件的结构视图
ctrl+n：搜索类文件
ctrl+shift+n：搜索文件
alt+f7：查找当前选中的类、方法、字段、参数和语句被引用的地方
# 二、配置构建
### 1.
* 可定义、配置和扩展编译
* 构建多个apk基于相同的工程
* 跨源集重用代码及资源
### 2.applicationId
命名规则：
* 至少包含两个片段(一个.号以上)
* 每个片段必须以字母开始
* 所有字符必须是字母、数字或下划线
#### 3.添加依赖
* 图片
| gradle 3.0 | gradle 2.0 |
| ---------- | ---------- |
| ![](../img/gradle3.0.png) | ![](../img/gradle2.0.png) |

* 说明
|      指令      |     作用     | 说明 |
| -------------- | ------------| ---- |
| implementation | 不会对外暴露依赖；构建时间有所改进 | 1.moduleA依赖moduleB,moduleB使用implementation依赖moduleC，则moduleA访问不了moduleC；2.moduleC实现有所改变，则只会重新编译moduleC和moduleB |
| api            | 可以依赖传递；构建时间有所增加;等同complie | 1.moduleA依赖moduleB,moduleB使用implementation依赖moduleC，则moduleA可以访问moduleC；2.moduleC实现有所改变，则会编译所有使用moduleC的实现的模块，也就是moduleA/moduleB/moduleC都重新编译 |
| complieOnly | 编译时需要用到module，运行时是可选的；可减少apk大小；等同provided | 无 |
| runtimeOnly | 运行时需要用到模块；等同apk | 无 |
| annotationProcessor | 需要依赖注解库时用到；提供构建性能 | annotationProcessor 'com.jakewharton:butterknife-compiler:8.8.1' |
* 注解处理器
添加参数：
```groovy
  android {
    defaultConfig {
      javaCompileOptions {
        annotationProcessorOptions {
          argument "key1","value1"
          argument "key2", "value2"
        }
      }
    }
  }
```
不允许注解器错误检查
```groovy
  android {
    defaultConfig {
      javaCompileOptions {
        annotationProcessorOptioins {
          includeCompileClasspath false 
        }
      }
    }
  }
```
* 不包含依赖传递
```groovy
  dependencies {
    implementation("some-library") {
      exclude group: 'com.example.ckr',module: 'native'
    }
  }
```
#### 4.优化构建速度
* 避免编译不必要的资源
```groovy
  android {
    productFlavors {
      dev {
        resConfigs "en", "xxhdpi"//使用英文和xxhdpi目录下的图片
      }
    }
  }
```
* 使用静态依赖版本
如：gradle:2.+ 改为 gradle:2.3.3
* 允许离线模式
![](../img/offline.png)
* 使用webp格式图片
#### 5.配置构建变体
```groovy
  android {
    defaultConfig {
      manifestPlaceholders = [hostName: "com.ckr.github"]
    }
    buildTypes {
      release {
        minifyEnabled true
      }
      debug {
        applicationIdSuffix ".debug"
        debuggable true
      }
      staging {
        initWith debug    //initWith:允许复制另外一个build types配置的属性
        manifestPlaceholders = [hostName: "com.ckr.github.staging"]
        applicationIdSuffix ".debugStaging"
      }
    }
  }
```
* 配置product flavors
```groovy
  android {
    flavorDimensions "version"
    productFlavors {
      demo {
        dimension "version"
        applicationIdSuffix ".demo"
        versionNameSuffix "-demo"
      }
      full {
        dimension "version"
        applicationIdSuffix ".full"
        versionNameSuffix "-full"
      }
    }
  }
```
* 使用flavor dimensions合并多个product flavors
```groovy
  android {
    flavorDimensions "api","mode"
    productFlavors {
      demo {
        dimension "mode"
      }
      full {
        dimension "mode"
      }
      minApi24 {
        dimension "api"
        minSdkVersion 24
        versionNameSuffix "-minApi24"
      }
    }
  }
```
* 修改默认源集配置
```groovy
  android {
    main {
      java.srcDirs = ['other/java']  //把java默认资源目录'src/main/java'改为'other/java'
      res.srcDirs = ['other/res1','other/res2']  //把默认资源目录'src/main/res'改为'other/res1,other/res2'
      manifest.srcFile 'other/AndroidManifest.xml'
    }
  }
```
#### 6.压缩代码和资源
* mapping.txt:提供原始类和混淆类、方法和字段之间的转换
* seeds.txt:列举没有混淆的类和成员
* usage.txt:列举被移除的代码
* 解混淆：使用retrace.bat(sdk目录/tools/proguard/),如：retrace.bat -verbose mapping.txt obfuscated_trace.txt
#### 7.使用multidex
```groovy
android {
  defaultConfig {
    multiDexEnabled true
  }
  dependencies {
    implementation 'com.android.support:multidex:1.0.3'
  }
}
```
```java
  public class MyApp extends Application {
    @Override
    protected void attachBaseContext(Context context){
      super.attachBaseContext(context);
      MultiDex.install(this);
    }
  }
```
#### 8.分析apk
* Build > Analyze APK




# 感谢
[android官网](https://developer.android.google.cn/studio/build)
