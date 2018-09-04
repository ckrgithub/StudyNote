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
### 发送插件
```java

    /**
     * Add a JavaScript statement to the list.
     */
    public void addPluginResult(PluginResult result, String callbackId) {
        if (callbackId == null) {
            LOG.e(LOG_TAG, "Got plugin result with no callbackId", new Throwable());
            return;
        }
        // Don't send anything if there is no result and there is no need to
        // clear the callbacks.
        boolean noResult = result.getStatus() == PluginResult.Status.NO_RESULT.ordinal();
        boolean keepCallback = result.getKeepCallback();
        if (noResult && keepCallback) {
            return;
        }
        JsMessage message = new JsMessage(result, callbackId);
        if (FORCE_ENCODE_USING_EVAL) {
            StringBuilder sb = new StringBuilder(message.calculateEncodedLength() + 50);
            message.encodeAsJsMessage(sb);
            message = new JsMessage(sb.toString());
        }

        enqueueMessage(message);
    }
```
## cordova.js
### 自执行函数
```javascript
(function(){
    //这里是块级作用域
})();
对比：
1.function foo(){}();//报错：Unexpected token
2.function foo(){}(1);//不会报错，但foo函数不会执行。
//等同于在一个function后面声明一个毫无关系的表达式：
function foo(){}
(1);
3.上面代码要实现的话，必须要实现赋值，如a=function(){}(),"a="这个告诉编译器这个是函数表达式，而不是函数的声明。因为函数表达式后面可以跟圆括号。
所以下面两段代码是等价的。
var a=function(i){
    alert(i);
}(5);//5
(function(i){
    alert(i);
})(5);//5

```
