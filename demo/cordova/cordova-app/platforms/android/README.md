# cordova学习
## cordovaActivity启动流流程
onCreate->loadConfig->解析config.xml->创建CordovaInterface->loadUrl
## config.xml
```java
config.xml:
<?xml version='1.0' encoding='utf-8'?>
<widget id="com.ckr.cordova" version="1.0.0" xmlns="http://www.w3.org/ns/widgets" xmlns:cdv="http://cordova.apache.org/ns/1.0">
    <feature name="Whitelist">
        <param name="android-package" value="org.apache.cordova.whitelist.WhitelistPlugin" />
        <param name="onload" value="true" />
    </feature>

    <name>CordovaApp</name>
    <description>
        A sample Apache Cordova application that responds to the deviceready event.
    </description>
    <author email="dev@cordova.apache.org" href="http://cordova.io">
        Apache Cordova Team
    </author>
    <content src="index.html" />
    <access origin="*" />
    <allow-intent href="http://*/*" />
    <allow-intent href="https://*/*" />
    <allow-intent href="tel:*" />
    <allow-intent href="sms:*" />
    <allow-intent href="mailto:*" />
    <allow-intent href="geo:*" />
    <allow-intent href="market:*" />
    <preference name="loglevel" value="DEBUG" />
</widget>

其中，onload标记：PluginManager初始化后该插件就被创建
   private void startupPlugins() {
        for (PluginEntry entry : entryMap.values()) {
            if (entry.onload) {//true
                getPlugin(entry.service);//创建插件
            } else {
                pluginMap.put(entry.service, null);
            }
        }
    }
```
## NativeToJsMessageQueue

