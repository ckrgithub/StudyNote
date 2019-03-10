# Gradle
gradle的一些有用的配置
```groovy
  //得到最近的一个tag版本号，可用作为App的versionName
  static def gitLatestTag(){
    try{
      def cmd='git describe --abbrev=0 --tags'
      def result= cmd.execute().text.trim()
      return result
    }catch(Exception e){
      return '1.0.0'
    }
  }
  //打release包时，将apk命名为ckr.apk输出
  android.applicationVariants.all{ variant ->
    variant.outputs.all{
      if(variant.buildType.name=='release'){
        outputFileName='ckr.apk'
      } 
    }
  }
```
