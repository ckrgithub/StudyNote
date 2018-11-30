# hook
## 动态代理
* 静态代理  
```java
  //接口
  public interface Shopping {
    Object[] doShopping(long money);
  }
  //原始实现
  public class ShoppingImpl implements Shopping {
    @Override
    public Object[] doShopping(long money){
      System.out.println(String.format("花了%s块钱",money));
      return new Object[]{"鞋子","衣服","零食"};
    }
  }
  //代理实现
  public class ShoppingProxy implements Shopping {
    Shopping mShopping;
    ShoppingProxy(Shopping shopping){
      mShopping=shopping;
    }
    
    @Override
    public Object[] doShopping(long money){
      //先黑点钱
      long readCost =(long)(money*0.5);
      System.out.println(String.format("花了%s块钱",readCost));
      //帮忙买东西
      Object[] things=mShopping.doShopping(readCost);
      //偷梁换柱
      if(things!=null&&things.length>1){
        things[0]="被掉包的东西！";
      }
      return things;
    }
  }
```
* 动态代理
传统的静态代理模式需要为每个需要代理的类写一个代理类。动态代理：JDK提供了动态代理方式，可以简单理解为JVM可以在运行时帮我们动态生成一系列的代理类。
```java
  public static void main(String[] args){
    Shopping shopping = new ShoppingImpl();
    //正常购物
    System.out.println(Arrays.toString(shopping.doShopping(100)));
    //招代理
    shopping =(Shopping)Proxy.newProxyInstance(Shopping.class.getClassLoader(),shopping.getClass().getInterfaces(),new ShoppingHandler(shopping));
    System.out.println(Arrays.toString(shopping.doShopping(100)));
  }
```
* 代理Hook
如果我们自己创建代理对象，然后把原始对象替换为我们的代理对象，就可以修改参数，替换返回值，我们称之为Hook.
首先我们得找到被Hook的对象，称之为Hook点；什么样的对象比较好Hook?自然容易找到的对象。什么样对象容易找到？静态变量和单例；在一个进程之内，静态变量和
单例变量是相对不容易发生变化的，因此非常容易定位。而普通的对象则要么无法标记，要么容易改变。
分析startActivity的调用链：Context.startActivity(Activity.startActivity的调用链与之不同)，由于Context的实现实际上是ContextImpl;
```java
  @Override
  public void startActivity(Intent intent,Bundle options){
    warnIfCallingFromSystemProcess();
    if((intent.getFlags()&Intent.FLAG_ACTIVITY_NEW_TASK)==0){
      throw new AndroidRuntimeException("Calling startActivity() from outside of an Activity.context requires the FLAG_ACTIVITY_NEW_TASK flag.Is this really what you want?");
    }
    mMainThread.getInstrumentation().execStartActivity(getOuterContext(),mMainThread.getApplicationThread(),null,(Activity)null,intent,-1,options);
  }
```
实际上使用了ActivityThread类的Instrumentation成员的execStartActivity方法；注意到ActivityThread实际上是主线程，而主线程一个进程只有一个，因此这是个比较好的Hook点。
```java
  //先获取当前ActivityThread对象
  Class<?> activityThreadClass = Class.forName("android.app.ActivityThread");
  Method currentActThreadMethod = activityThreadClass.getDeclaredMethod("currentActivityThread");
  currentActThreadMethod.setAccessible(true);
  Object currentActThread=currentActThreadMethod.invoke(null);
```
拿到currentActThread之后，我们需要修改它的mInstrumentation这个字段为我们代理对象，我们先实现这个代理对象，由于JDK动态代理只支持接口，而这个
Instrumentation是一个类，没办法，我们只能手动写静态代理类，覆盖掉原始的方法即可
```java
  public class EvilInstrumentation extends Instrumentation {
    private static final String TAG = "EvilInstrumentation";
    Instrumentation mBase;
    public EvilInstrumentation(Instrumentation base){
      mBase=base;
    }
    public ActivityResult execStartActivity(Context who,IBinder contextThread,IBinder token,Activity target,Intent intent,
      int requestCode,Bundle options){
      try{
        Method execStartActivity = Instrumentation.class.getDeclaredMethod("execStartActivity",Context.class,IBinder.class,IBinder.class,Activity.class,Intent.class,
        int.class,Bundle.class);
        return (ActivityResult)execStartActivity.invoke(mBase,who,contextThread,token,target,intent,requestCode,options);
      }catch(Exception e){
        throw new RuntimeException("do not support!");
      }
    }
  }
```
有了代理对象，就可以偷梁换柱：
```java
  public static void attachContext() throws Exception{
    Class<?> actThreadClass=Class.forName("android.app.ActivityThread");
    Method curActThreadMethod=actThreadClass.getDeclaredMethod("currentActivityThread");
    curActThreadMethod.setAccessible(true);
    Object curActThread=curActThreadMethod.invoke(null);
    //拿到原始mInstrumentation字段
    Field mInstrumentationField=actThreadClass.getDeclaredField("mInstrumentation");
    mInstrumentationField.setAccessible(true);
    Instrumentation mInstrumentation=(Instrumentation)mInstrumentationField.get(curActThread);
    //创建代理对象
    Instrumentation evilInstrumentation=new EvilInstrumentation(mInstrumentation);
    //偷梁换柱
    mInstrumentationField.set(curActThread,evilInstrumentation);
  }
```
## Binder Hook
Android系统通过Binder机制给应用程序提供了一系列的系统服务，如：ActivityManagerService,ClipboardManager,AudioManager
系统各个远程service对象都是以Binder形式存在，而这些Binder有一个管理者，就是ServiceManager.
```java
  //获取系统服务
  ActivityManager am = (ActivityManager)context.getSystemService(Context.ACTIVITY_SERVICE);
  
  public Object getSystemService(String name){
    ServiceFetcher fetcher = SYSTEM_SERVICE_MAP.get(name);
    return fetcher==null?null:fetcher.getService(this);
  }
  
  registerService(ACCOUNT_SERVICE,new ServiceFetcher(){
    public Object createService(ContextImpl ctx){
      IBinder b=ServiceManager.getService(ACCOUNT_SERVICE);//获取原始IBinder对象
      IAccountManager service = IAccountManager.Stub.asInterface(b);//转换为Service接口
      return new AccountManager(ctx,service);
    }
  });
```
* asInterface过程
```java
  public static android.content.IClipboard asInterface(android.os.IBinder obj){
    if(obj==null){
      return null;
    }
    android.os.IInterface iin=obj.queryLocalInterface(DESCRIPTOR);//Hook点
    if(iin!=null&&iin instanceOf android.content.IClipboard){
      return (android.content.IClipboard)iin
    }
    return new android.content.IClipboard.Stub.Proxy(obj);
  }
```
先查看进程是否存在这个Binder对象，如果有直接返回；如果不存在，则创建一个代理对象，让代理对象委托驱动完成跨进程调用。
观察这个方法，前面的那个if语句肯定动不了手脚；最好一句也无从下手，要修改asInterface方法的返回值，唯一能做的：
```java
  android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
```
* getService过程
```java
  IBinder b= ServiceManager.getService("service_name");
```
我们希望修改这个getService方法的返回值，让这个方法返回一个我们伪造过的IBinder对象;这样，我们可以在伪造的IBinder对象的queryLoaclInterface
方法做处理。getService是静态方法，没办法拦截，也没办法获取到这个静态方法的局部变量。
```java
  public static IBinder getService(String name){
    try{
      IBinder service=sCache.get(name);
      if(service!=null){
        return service;
      }else{
        return getIServiceManager().getService(name);
      }
    }catch(RemoteException e){
      e.printStackTrace();
    }
  }
```
天无绝人之路！ServiceManager为了避免每次都进行跨进程通信，把这些Binder代理对象缓存在map里.我们可以替换map里面的内容为Hook过的IBinder对象.
总结：  
1.首先伪造一个系统服务对象，让asInterface能够返回我们伪造对象而不是原始的系统服务对象  
2.伪造一个IBinder对象，主要修改它的queryLocalInterface方法，让它返回我们伪造的系统服务对象；然后把这个伪造对象放置ServiceManager的缓存map里  
通过Binder机制优先查找本地Binder对象的这个特性达到Hook掉系统服务对象。因此queryLocalInterface失去它原本的意义(只查找本地Binder对象，没有本地对象
返回null).由于我们接管了asInterface这个方法的全部，我们伪造的系统服务对象不能只是拥有本地Binder对象的能力，还要有Binder代理对象操纵驱动的能力.
### Hook系统剪切板服务


## 感谢
[维术的博客](http://weishu.me/archives/)
