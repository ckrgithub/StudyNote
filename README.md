# 升级功能测试
## 获取升级信息
* [https://api.dinghuo123.com/public/specialVersion?platform=Android&systemVersion=6.0&imei=865903038578968&isBrand=0&versionCode=312&versionName=3.19.0&userId=&userName=&dbid=](#url)
* [数据结构](#数据结构)
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
* [字段说明](#字段说明)
|field|type|description|
|-----|----|-----------|
|isAllowUpdate|boolean|是否允许更新|
