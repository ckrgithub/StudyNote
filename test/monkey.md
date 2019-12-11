# Monkey用法
monkey命令查找
```
adb shell monkey -help
```
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMyMDE1LmNuYmxvZ3MuY29tL2Jsb2cvMzY1Nzk5LzIwMTYwMS8zNjU3OTktMjAxNjAxMDQxMDU1MzgyMzEtNjY1MjM3ODU4LnBuZw)

* -p:指定启动App包名
* -c:指定启动的Activity的category类型
* -v:指定log详细程度，如：-v -v 提供比较详细信息；-v -v -v 提供最详细的信息 
* -s:指定伪随机数生成器的种子值，便于bug复现
* --ignore-crashes:忽略异常奔溃
* --ignore-timeouts:忽略anr
* --ignore-native-crashes:忽略native层代码的奔溃
* --ignore-security-exceptions:忽略一些许可错误
* --kill-procress-after-error:用于在发送错误后杀死进程
* --throttle:设置每个事件结束后延迟多少时间再继续下一个事件
* -f 加载monkey脚本文件进行测试，如：adb shell monkey -f sdcard/monkey.txt -v -v 500


## 基本语法
```adb
 adb shell monkey [options]
 //指定了seed值，每个事件之间休息300ms，执行了20个事件，然后将日志信息保存在了monkey.txt文件中
 adb shell monkey -p 包名  -s 123123 --throttle 300 -v -v 20 > d:\monkey.txt  
```

### 设备连接
adb devices：查看设备是否已连接

### 安装apk
adb install apk路径

### 所有app的内存使用情况
adb bugreport > bugreport.txt 

### App启动
-p:表示对象包，-v:表示反馈信息级别
```adb
adb shell monkey -p package -v 500
```

## monkey脚本
格式：
```
type= raw events
count= 1
speed= 1.0   
start data >>   
 
LaunchActivity(pkg_name, cl_name)

解析：
第一句到第三局就使用默认值，不需要改，其实这里设置是无效的，最终会采用命令行里的值；
start data >> 表示开始执行下面所有的命令行
LaunchActivity就是一个启动应用的命令(包名，activity名)
```
### 脚本命令
* LaunchActivity(pkg_name, cl_name)： 启动应用，第一个参数是包名，第二个是启动的activity名
* DispatchPointer(downtime,eventTime,action,x,y,xpressure,size,metastate,xPrecision,yPrecision,device,edgeFlags) ：向指定位置发送单个手势，相当于我们把手指按在某个点上；这个方法参数有12个，但是我们主要关注owntime,eventTime,action,x,y这么几个参数，x，y表示按下的坐标，可以通过上一篇文章UI Automator获取，这在你想测试点击某个具体view是很有用的
* DispatchPress(keycode)： 向系统发送一个固定的按键事件；例如home键，back键；参数是按键值 ，按键值可查看keycode
* UserWait：让脚本的执行暂停一段时间，做一个等待操作
* RotateScreen(rotationDegree, persist)： 翻转屏幕，第一个参数是旋转角度，第二个是旋转后是否停在当前位置
* Tap(x, y) ：单击事件，点击屏幕，参数是点击坐标
* Drag(xStart, yStart, xEnd, yEnd) ：在屏幕上滑动，坐标是从哪一点滑到哪一点
* LongPress()： 长按2s
* ProfileWait()： 等待5s
* PressAndHold(x, y, pressDuration) ：模拟长按 
* PinchZoom(x1Start, y1Start, x1End, y1End, x2Start, y2Start, x2End, y2End, stepCount)： 模拟缩放
* DispatchString(input)： 输入字符串
* RunCmd(cmd) ：执行shell命令，比如截图 screencap -p /data/local/tmp/tmp.png
* DispatchFlip(true/false) ：打开或者关闭软键盘
* UserWait(sleepTime) ：睡眠指定时间
* DeviceWakeUp() ：唤醒屏幕

如：
```
type= raw events
count= 1
speed= 1.0   
start data >>   
 
LaunchActivity(com.android.mangodialog,com.android.mangodialog.MainActivity);
UserWait(1000);
# 按下
DispatchPointer(0,0,0,400,500,0,0,0,0,0,0,0) 
# 抬起
DispatchPointer(0,0,1,400,500,0,0,0,0,0,0,0) 
UserWait(1000);
 
Tab(500,300);
UserWait(1000);
 
DispatchPress(KEYCODE_ENTER)
UserWait(1000);
 
DispatchPress(KEYCODE_BACK);
UserWait(1000);
 
RunCmd(screencap -p /sdcard/tmp.png);
UserWait(1000);
 
Drag(0, 0, 500, 500);
UserWait(1000);
 
RotateScreen(90,1)
UserWait(1000);
 
DispatchString(www.baidu.com);
UserWait(1000);
 
DispatchPress(KEYCODE_BACK);
UserWait(1000);

注：
脚本需要对照着MonkeySourceScript.java文件编写，这样才能知道方法需要多少个参数，参数有什么含义；同时要注意方法名是区分大小写的，其实最重要的是脚本的编写需要根据你的测试用例来写，一步步怎么操作就写上对应的方法
因为Monkey是运行在设备上的，所以需要将脚本先传到设备上，通过adb push d:\monkey.txt sdcard/monkey.txt 将文件推送到手机sd卡上
最后通过adb shell monkey -f sdcard/monkey.txt -v -v 1 执行脚本文件

```
# Monkey Server
* 启动Monkey Server：adb shell monkey --port 1080，开放设备的1080端口
* 连接Monkey Server：先输入 adb forward tcp:1080 tcp:1080，将本机端口（前）和设备端口（后）进行映射，再输入telnet 127.0.0.1 1080，这里是通过telnet连接本机的1080端口，就可连接到设备上的Monkey server，并且执行Server中的相关指令


# 感谢
[Android自动化测试之Monkey命令使用及monkey脚本编写](https://blog.csdn.net/qq_30993595/article/details/80748559)





