# Http学习
## 一、请求头
### 1.User-Agent
User-Agent是http协议中的一部分，属于头域的组成部分。它是个特殊字符串头，是一种向访问网站提供你所使用的浏览器类型及版本、操作系统及版本、浏览器内核等信息的标识。
通过这个标识，用户所访问的网站可以显示不同的排版，从而为用户提供更好的体验或者进行信息统计。java服务器通过以下代码获取：
```java
  String userType = request.getHeader("User-Agent");
```
