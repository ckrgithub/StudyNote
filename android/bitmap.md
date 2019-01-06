# bitmap内存大小
一张图片大小 = 宽 * 高 * 单个像素点所占字节数，则一张864x582的png图片大小是864x582x4=2011392，但bitmap大小是3465504。bitmap计算公式为bitmap.getWidth()*bitmap.getHeight()*单个像素点所占字节数。  
## 内存输出
```java
  ActivityManager activityManager = (ActivityManger)getSystemService(Context.ACTIVITY_SERVICE);
  ActivityManger.MemoryInfo memoryInfo = new ActivityManger.MemoryInfo();
  activityManger.getMemoryInfo(memoryInfo);
  Log.d(TAG,"AvailMemory:"+memoryInfo.availMem/1024/1024);
  Log.d(TAG,"lowMemory:"+memoryInfo.lowMemory);
  Log.d(TAG,"NativeHeapAllocatedSize:"+Debug.getNativeHeapAllocatedSize());
```
创建一张2G大小的bitmap:
```java
  Bitmap bitmap = Bitmap.createBitmap(1024,1024*512,Bitmap.Config.ARGB_8888);
```
在android8.0之前运行会OOM，而8.0之后则不会，因为8.0之前bitmap内存存放在Java heap，而8.0后则存放在Native heap.
## 内存回收
Bitmap其实占两部分对象，一个是java bitmap对象，还有一个是Native Bitamp对象，Java Bitamp对象由垃圾回收机制管理。在Android2.3.3之前必须手动调用recycle()方法释放Native内存。在8.0之后主动调用recycler方法，数据会立即释放，因为像素内存是在Native层分配的。但，在8.0之前，就算我们主动调用recycle方法，数据也不会立即释放，而是DeleteWeakGlobalRef交由Java GC来回收。**注意：以上所说的释放数据仅代表释放像素数据，并未释放Native层的Bitmap对象。**
而释放Native层Bitmap对象，都是监听Java层的bitmap是否被释放，一旦Java层bitmap对象释放则立即释放native层bitmap.
## 内存复用
* 被复用的Bitmap必须是Mutable
* 4.4之前，将要解码的图像必须是jpeg或png格式且和被复用的bitmap大小一样。
* 4.4之后，将要解码的图像的内存需要大于等于要复用的bitmap的内存
```java
  //不复用，消耗内存 32M
  Bitmap bitmap1=BitampFactory.decodeResource(getResources(),R.drawable.ic_launcher);
  Bitmap bitmap2=BitmapFactory.decodeResource(getResources(),R.drawable.ic_launcher);
  
  //复用，消耗内存 16M
  BitmapFactory.Options options = new BitmapFactory.Options();
  options.inMutable = true;
  Bitmap bitmap1=BitmapFactory.decodeResource(getResources(),R.drawable.ic_launcher,options);
  options.inBitmap = bitmap1;
  Bitmap bitmap2=BitmapFactory.decodeResource(getResources(),R.drawable.ic_launcher,options);
```
# 感谢
[图像处理](https://www.jianshu.com/p/8e8ad414237e)
