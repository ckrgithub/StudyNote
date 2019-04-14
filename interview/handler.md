# Handler机制
## 谈一谈handler机制
**问题分析**：在主线程运行耗时操作会报ANR异常，需要在子线程中做耗时操作，但谷歌规定子线程不能更改主UI，所以要通过Handler机制来进行线程间通信。
**答**：  
* 1.looper:一个线程可以产生一个looper对象，由他来管理此线程里的MessageQueue(消息队列)
* 2.Handler:可以构造handler对象来与Looper沟通，以便push新消息到MessageQueue里，或接收Looper从MessageQueue取出的消息。
* 3.线程：UIThread通常就是main Thread,而Android启动程序时，会替她建立一个MessageQueue.
