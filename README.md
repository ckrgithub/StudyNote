# 升级功能测试
## 一、获取升级信息
### 1.链接 
```
[url过长]:https://api.dinghuo123.com/public/specialVersion?platform=Android&systemVersion=6.0&imei=865903038578968&isBrand=0&versionCode=312&versionName=3.19.0&userId=&userName=&dbid=
```
### 2.数据结构
```
  {
    "code": 200,
    "message": "操作成功",
    "data": {
        "apkUrl": "https://s.beta.myapp.com/myapp/rdmexp/exp/file/comircloudydhagents_4111_4f702a8d-a797-493a-af64-f821e562d9bb.apk",
        "bundleVersion": "",
        "corpId": 0,
        "createTime": "2017-05-25 11:57:07",
        "fileSize": 1548960,
        "id": 4,
        "imei": "",
        "instanceId": 0,
        "isAllowUpdate": true,
        "isBrand": 0,
        "modifyTime": "",
        "netType": 1,
        "newFeature": "1.升级测试2.灰度升级",
        "platform": "",
        "shortBundleVersion": "",
        "systemVersion": "",
        "title": "全部账号升级",
        "updateMode": 2,
        "updateStrategy": 1,
        "userName": "",
        "versionCode": 450,
        "versionName": "4.12.0"
      }
  }
```
### 3.字段说明
|字段|类型|说明|作用|备注|
|---|---|----|----|----|
|isAllowUpdate|boolean|是否允许更新|false:不允许升级；true：允许升级，执行下一步|
|updateMode|int|升级模式|2：走bugly升级；否则，走服务器升级|[bugly升级只针对v3.19.0.312.apk下发策略](bugly升级只针对v3.19.0.312.apk下发策略)|
|netType|int|网络类型|1：只在wifi情况下提示升级；2:只在移动网络下提示升级；3：只在有网情况下提示升级|[只在走公司服务器升级流程才生效](只在走公司服务器升级流程才生效)|
|updateStrategy|int|升级策略|1：非强制性升级；2：强制性升级|[只在走公司服务器升级流程才生效](只在走公司服务器升级流程才生效)|
|isBrand|int|是否是品牌定制|0：不是；1：是|[只在走公司服务器升级流程才生效](只在走公司服务器升级流程才生效)|
|newFeature|string|升级信息说明|比如："1.修复bug\n2.升级测试"|[只在走公司服务器升级流程才生效](只在走公司服务器升级流程才生效)|
|title|string|升级标题文本|比如："App6.0.4"|[只在走公司服务器升级流程才生效](只在走公司服务器升级流程才生效)|
|apkUrl|string|链接|apk下载链接|[只在走公司服务器升级流程才生效](只在走公司服务器升级流程才生效)|

## 二、bugly升级
[图片](img/bugly.png)
### 基础配置
|字段|说明|
|---|---|
|策略下发条件| 指定原版本3.19.0.312|
|策略启动条件|手动启动|
|下发上限|3|
|激活上限|3|
|升级标题|App名+6.0.4.667|
|更新说明|  1.bug修复 2.升级测试|
