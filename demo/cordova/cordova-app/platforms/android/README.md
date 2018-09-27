# cordova学习
## 一、cordovaActivity启动流流程
onCreate->loadConfig->解析config.xml->创建CordovaInterface->loadUrl
### 1.config.xml
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
### 2.NativeToJsMessageQueue
#### 发送插件
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
### 3.cordova.js
#### 自执行函数
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

## 二、自定义Cordova插件
1.自定义一个类，继承CordovaPlugin,并重写execute()方法。如：
```java
  public class TestPlugin extends CordovaPlugin {
    private static final String TAG="TestPlugin";

    @Override
    public void initialize(CordovaInterface cordova, CordovaWebView webView) {
        super.initialize(cordova, webView);
        Logd(TAG, "execute: cordova:"+cordova+",webView:"+webView);
    }

    @Override
    protected void pluginInitialize() {
        super.pluginInitialize();
    }

    @Override
    public boolean execute(String action, String rawArgs, CallbackContext callbackContext) throws JSONException {
        Logd(TAG, "execute: action:"+action+",rawArgs:"+rawArgs);
        return super.execute(action, rawArgs, callbackContext);
    }

    @Override
    public boolean execute(String action, JSONArray args, CallbackContext callbackContext) throws JSONException {
        Logd(TAG, "execute: action:"+action+",args:"+args);
        return super.execute(action, args, callbackContext);
    }

    @Override
    public boolean execute(String action, CordovaArgs args, CallbackContext callbackContext) throws JSONException {
        Logd(TAG, "execute: action:"+action+",cordovaArgs:"+args);
        return super.execute(action, args, callbackContext);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
    }
  }
```
2.在res目录下创建config.xml文件并配置插件feature及白名单
```java
  <?xml version="1.0" encoding="utf-8"?><!--情况cordova官网-->
<widget xmlns:cdv="http://cordova.apache.org/ns/1.0"
    id="io.cordova.hybrid"
    version="1.0.0"
    xmlns="http://www.w3.org/ns/widgets">
    <!--cordova自带的log输出级别配置-->
    <preference
        name="loglevel"
        value="DEBUG" />
    <!--定义应用程序的起始页.请看ConfigXmlParser的setStartUrl-->
    <content src="index.html" />
    <!--允许访问所有域名,请看http://cordova.axuer.com/docs/zh-cn/latest/reference/cordova-plugin-whitelist/-->
    <access origin="*" />
    <!--允许系统打印超链接url,请看http://cordova.axuer.com/docs/zh-cn/latest/reference/cordova-plugin-whitelist-->
    <allow-intent href="*://*.xxx.com/*" />
    <allow-intent href="tel:*" />
    <allow-intent href="sms:*" />
    <allow-intent href="mailto:*" />
    <allow-intent href="geo:*" />
    <platform name="android">
        <allow-intent href="market:*" />
    </platform>
    <!--允许webView可以导航到的url-->
    <allow-navigation href="*://*.xxx.com/*" />
    <!--白名单插件-->

    <!--测试插件testPlugin-->
    <feature name="TestPlugin"><!--name：service -->
        <!--value：插件类名-->
        <param
            name="android-package"
            value="com.ckr.baseframework.plugin.TestPlugin" />
    </feature>

    <preference
        name="AppendUserAgent"
        value="ydh-android/5.0.0" />

    <preference
        name="errorUrl"
        value="file:///android_asset/www/error.html" />

  </widget>
```
3.在assets/www目录下创建plugin文件夹，并添加testPlugin.js暴露该插件的方法
```javascript
  cordova.define("com.ckr.hybrid.TestPlugin",function(require,exports,module){
    var exec = require('cordova/exec');/*see to exec.js/androidExec(success, fail, service, action, args)*/
    module.exports={
        testNoArgs: function(sucCallback,errCallback,args){
            exec(sucCallback,errCallback,"TestPlugin","testNoArgs",[args]);
        },
        testWithArgs: function(sucCallback,errCallback,args){
            exec(sucCallback,errCallback,"TestPlugin","testWithArgs",[args]);
        },
    };
  });
```
接着，在cordova_plugins.js里声明插件对象
```javascript
  cordova.define('cordova/plugin_list', function(require, exports, module) {
    module.exports = [
        {
            "file":"plugin/testPlugin.js",
            "id": "com.ckr.hybrid.TestPlugin",
            "clobbers":[
                "hybrid.test"
            ]
        },
    ];
    module.exports.metadata = 
    // TOP OF METADATA
    {
      "cordova-plugin-whitelist": "1.3.3"
    };
    // BOTTOM OF METADATA
  });
```
4.在assets/www/js创建debug目录，及test.js文件
```javascript
  function testNoArgs(){
    hybrid.test.testNoArgs(function(args){
        alert("success:"+args);
    },function(err){
        alert("failure:"+err);
    });
  }
  function testWithArgs(){
      hybrid.test.testWithArgs(function(args){
          alert("success:"+args);
      },function(err){
          alert("failure:"+err);
      },'带参函数');
  }
  document.getElementById('testNoArgs').addEventListener('click',testNoArgs);
  document.getElementById('testWithArgs').addEventListener('click',testWithArgs);

```
接着，在index.html创建一个button,用于插件调试
```javascript
  <!DOCTYPE html>
  <html>
      <head>
          <!--
          Customize this policy to fit your own app's needs. For more guidance, see:
              https://github.com/apache/cordova-plugin-whitelist/blob/master/README.md#content-security-policy
          Some notes:
              * gap: is required only on iOS (when using UIWebView) and is needed for JS->native communication
              * https://ssl.gstatic.com is required only on Android and is needed for TalkBack to function properly
              * Disables use of inline scripts in order to mitigate risk of XSS vulnerabilities. To change this:
                  * Enable inline JS: add 'unsafe-inline' to default-src
          -->
          <meta http-equiv="Content-Security-Policy" content="default-src 'self' data: gap: https://ssl.gstatic.com 'unsafe-eval'; style-src 'self' 'unsafe-inline'; media-src *; img-src 'self' data: content:;">
          <meta name="format-detection" content="telephone=no">
          <meta name="msapplication-tap-highlight" content="no">
          <meta name="viewport" content="user-scalable=no, initial-scale=1, maximum-scale=1, minimum-scale=1, width=device-width">
          <link rel="stylesheet" type="text/css" href="css/index.css">
          <title>Hello World</title>
      </head>
      <body>
          <div class="app">
              <h1>Apache Cordova</h1>
              <div id="deviceready" class="blink">
                  <p class="event listening">Connecting to Device</p>
                  <p class="event received">Device is Ready</p>
              </div>
              <button id="testNoArgs">test</button>
              <button id="testWithArgs">带参test</button>
          </div>
          <script type="text/javascript" src="cordova.js"></script>
          <script type="text/javascript" src="js/index.js"></script>
          <script type="text/javascript" src="js/debug/test.js"></script>
      </body>
  </html>
```
最后，在TestPlugin实现业务逻辑
```java
  public class TestPlugin extends CordovaPlugin {
    private static final String TAG = "TestPlugin";


    @Override
    public void initialize(CordovaInterface cordova, CordovaWebView webView) {
        super.initialize(cordova, webView);
        Logd(TAG, "execute: cordova:" + cordova + ",webView:" + webView);

    }

    @Override
    protected void pluginInitialize() {
        super.pluginInitialize();
    }

    @Override
    public boolean execute(String action, CordovaArgs args, CallbackContext callbackContext) throws JSONException {
        Logd(TAG, "execute: action:" + action + ",cordovaArgs:" + args);
        return doInBackground(action, args, callbackContext);
    }

    private boolean doInBackground(String action, CordovaArgs args, CallbackContext callbackContext) {
        if (PluginAction.TEST_NO_ARGS.equals(action)) {
            callbackContext.success("回调成功");
            return true;
        } else if (PluginAction.TEST_WITH_ARGS.equals(action)) {
            callbackContext.error("回调失败");
            return true;
        }
        return false;
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
    }
  }
```
