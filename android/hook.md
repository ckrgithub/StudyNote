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
1.伪造剪切版服务对象
```java
  public class BinderHookHandler implements InvocationHandler {
    //原始Service对象
    Object base;
    public BinderHookHandler(IBinder base,Class<?> stubClass){
      try{
        Method asInterfaceMethod = stubClass.getDeclaredMethod("asInterface",IBinder.class);
        //IClipboard.Stub.asInterface(base);
        this.base = asInterfaceMethod.invoke(null,base);
      }catch(Exception e){
        throw new RuntimeException("hooked failed!");
      }
    }
    @TargetApi(Build.VERSION_CODES.HONEYCOMB)
    @Override
    public Object invoke(Object proxy,Method method,Object[] args) throws Throwable{
      //把剪切板的内容替换为"you are hooked"
      if("getPrimaryClip".equals(method.getName())){
        return ClipData.newPlainText(null,"you are hooked");
      }
      //欺骗系统，使之认为剪切板上一直有内容
      if("hasPrimaryClip".equals(method.getName())){
        return true;
      }
      return method.invoke(base,args);
    }
  }
```
注意，我们拿到原始IBiner对象之后，如果我们希望使用被Hook之前的系统服务，并不能直接使用这个IBinder对象，而是需要使用asInterface方法将它
转换为IClipboard接口；因为getService方法返回的IBinder实际上是一个裸Binder代理对象，只有与驱动打交道的能力，并不能独立工作；asInterface()
方法返回的IClipboard.Stub.Proxy类的对象通过操纵这个裸BinderProxy对象从而实现了IClipboard接口定义的操作。  
2.伪造IBinder对象  
上一步，我们已经伪造好了系统服务对象，现在要做的是想办法让asInterface方法返回我们伪造的对象。
```java
  public class BinderProxyHandler implements InvocationHandler {
    //绝大部分，这是一个BinderProxy对象
    //只有当Service和我们在同一个进程时才是Binder本地对象
    IBinder base;
    Class<?> stub;
    Class<?> mInterface;
    public BinderProxyHandler(IBinder base){
      this.base=base;
      try{
        stub = Class.forName("android.content.IClipboard$Stub");
        mInterface = Class.forName("android.content.IClipboard");
      }catch(Exception e){
        e.printStatchTrace();
      }
      @Override
      public Object invoke(Object proxy,Method method,Object[] args) throws Throwable{
        if("quearyLocalInterface".equals(method.getName())){
            //这里返回真正被Hook掉的Service接口，queryLocalInterface就不是原本的意思了
            return Proxy.newProxyInstance(proxy.getClass().getClassLoader(),
              //asInterface时会检测是否是特定类型的接口然后进行强制转换
              new Class[] {IBinder.class,IInterface.class,this.mInterface},
              new BinderHookHandler(base,stub);
            );
            return method.invoke(base,args);
         }
      }
    }
  }
```
使用动态代理伪造一个跟原始IBinder一模一样的对象，然后在这个伪造的IBinder对象的queryLocalInterface方法返回我们第一步创建的
伪造的系统服务对象。  
3.替换ServiceManager的IBinder对象  
使用反射修改ServiceManager类里面缓存的Binder对象，使得getService方法返回我们伪造的IBinder对象，进而asInterface方法使用
伪造IBinder对象的queryLocalInterface方法返回我们伪造的系统服务对象。
```java
  Class<?> serviceManager = Class.forName("android.os.ServiceManager");
  Method getService = serviceManager.getDeclaredMethod("getService",String.class);
  //ServiceManager管理原始的Clipboard Binder对象
  IBinder rawBinder = (IBinder)getService.invoke(null,CLIPBOARD_SERVICE);
  //Hook掉这个Binder代理对象的queryLocalInterface方法
  IBinder hookBinder = (IBinder) Proxy.newProxyInstance(serviceManager.getClassLoader(),
    new Class<?>[]{IBinder.class},
    new BinderProxyHandler(rawBinder);
   );
   //把这个Hook过的Binder对象放进ServiceManager的cache里面
   Field cacheField = serviceManger.getDeclaredField("sCache");
   cacheField.setAccessible(true);
   Map<String,IBinder> cache = (Map) cacheField.get(null);
   cache.put(CLIPBOARD_SERVICE,hookBinder);
```
## Hook机制之AMS&PMS
* ActivityManagerService:  
1.startActivity最终调用了AMS的startActivity系列方法，实现了Activity的启动；Activity生命周期回调，也在AMS中完成；  
2.startService,bindService最终调用到AMS的startService和bindService方法
3.动态广播的注册和接收在AMS中完成(静态广播在PMS中完成)
4.getContentResolver最终从AMS的getContentProvider获取到ContentProvider
而PMS则完成诸如权限校验(checkPermission,checkUidPermission),Apk meta信息获取(getApplicationInfo等)，四大组件信息获取(query系列方法)等重要功能
### AMS获取过程
startActivity两种方式：  
1.直接调用Context类的startActivity方法；这种方法启动的activity没有activity栈，因此不能以standard方法启动，必须加上FLAG_ACTIVITY_NEW_TASK这个flag  
2.调用被Activity类重载过的startActivity方法，通常在我们的activity中直接调用这个方法就是这种形式
* Context.startActivity
Context是一个抽象类，Activity,Service等并没有直接继承Context，而是继承了ContextWrapper;ContextWrapper的实现:
```java
  @Override
  public void startActivity(Intent intent){
    mBase.startActivity(intent);
  }
```
其中mBase的实现是ContextImpl类。所以，Context.startActivity最终使用了ContextImpl里面的方法：
```java
  public void startActivity(Intent intent,Bundle options){
    warnIfCallingFromSystemProcess();
    if((intent.getFlags&Intent.FLAG_ACTIVITY_NEW_TASK) ==0){
      throw new AndroidRuntimeException(
        "Calling startActivity() from outside of an Activity "
        +"context requires the FLAG_ACTIVITY_NEW_TASK flag."
        +" Is this really what you want?"
      );
    }
    mMainThread.getInstrumentation().execStartActivity(getOunterContext(),mMainThread.getApplicationThread,null,(Activity)null,intent,-1,options);
  }
```
代码相当简单，我们知道了两件事：
1.我们知道在Service等非Activity的Context里启动Activity为什么需要添加FLAG_ACTIVITY_NEW_TASK
2.startActivity使用了Instrumentation类的execStartActivity方法；
```java
  public ActivityResult execStartActivity(
    Context who,IBinder contextThread,IBinder token,Activity target,Intent intent,int requestCode,Bundle options){
      try{
        intent.migrateExtraStreamToClipData();
        intent.prepareToLeaveProcess();
        int result = ActivityManagerNative.getDefault().startActivity(whoThread,who.getBasePackageName(),intent,intent.resolveTypeIfNeeded(who.getContentResolver()),token,target!=null?target.mEmbeddedID:null,requestCode,0,null,null,options);
        checkStartActivityResult(result,intent);
      }catch(RemoteException e){
        
      }
      return null;
    }
  ;
```
### Activity.startActivity
这个startActivity通过若干次调用辗转到startActivityForResult这个方法：
```java
  Instrumentation.ActivityResult ar = mInstrumentation.execStartActivity(this,mMainThread.getApplicationThread(),mToken,this,intent,requestCode,options);
```
### Hook AMS
ActivityManagerNative实际上是ActivityManagerService这个远程对象的Binder代理对象；每次需要与AMS打交道时，需要借助这个代理对象通过驱动进而完成IPC调用。
```java
  static public IActivityManager getDefault(){
    return gDefault.get();
  }
```
gDefault这个静态变量定义如：
```java
  private static final Singleton<IActivityManager> gDefault = new Singleton<IActivityManager>(
    protected IActivityManager create(){
      IBinder b = ServiceManager.getService("activity");
      IActivityManager am = IActivityManager.Stub.asInterface(b);
      return am;
    } 
  );
```
Hook掉AMS:
```java
  Class<?> actManagerNativeClass = Class.forName("android.app.ActivityManagerNative");
  //获取gDefault字段，想办法替换它
  Field gDefaultField=actMangerNativeClass.getDeclaredField("gDefault");
  gDefaultField.setAccessible(true);
  Object gDefault = gDefaultField.get(null);
  //4.x以上的gDefault是一个android.util.Singleton对象
  Class<?> singleton = Class.forName("android.util.Singleton");
  Field mInstanceField = singleton.getDeclaredField("mInstance");
  mInstanceField.setAccessible(true);
  //ActivityManagerNative的gDefault对象里原始的IActivityManger对象
  Object rawActManager = mInstanceField.get(gDefault);
  //创建一个这个对象的代理，然后替换这个字段
  Class<?> iActivityManagerInterface=Class.forName("android.app.IActivityManager");
  Object proxy=Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(),
    new Class<?>[] {iActivityManagerInterface},new IActivityMangerHandler(rawActManager)
  );
  mInstanceField.set(gDefault,proxy);
```
Android Framework层对于四大组件处理，调用AMS服务的时候，全部都是通过使用这种方式
### PMS获取过程
PMS的获取也通过Context完成，具体是getPackageManager这个方法。
```java
  public PackageManger getPackageManager(){
    if(mPackageManager!=null){
      return mPackagerManager;
    }
    IPackageManager pm = ActivityThread.getPackageManager();
    if(pm!=null){
      return (mPackageManager=new ApplicationPackageManager(this,pm));
    }
    return null;
  }
```
真正的PMS的代理对象在ActivityThread里面。ContextImpl通过ApplicationPackageManager对它进行一层包装
```java
  public static IPackageManager getPackageManager(){
    if(sPackageManger!=null){
      return sPackageManger;
    }
    IBinder b=ServiceManager.getService("package");
    sPackageManager=IPackageManager.Stub.asInterface(b);
    return sPackageManger;
  }
```
### Hook PMS
```java
  //获取全局ActivityThread对象
  Class<?> actThreadClass = Class.forName("android.app.ActivityThread");
  Method curActThreadMethod = actThreadClass.getDeclaredMethod("currentActivityThread");
  Object curActThread = curActThreadMethod.invoke(null);
  //获取ActivityThread里原始的sPackageManager
  Field sPackageMangaerField = actThreadClass.getDeclaredField("sPackageManager");
  sPackageManagerField.setAccessible(true);
  Object sPackageManager = sPackageManagerField.get(curActThread);
  //hook掉原始对象
  Class<?> iPackageManagerInterface = Class.forName("android.content.pm.IPackageManager");
  Object proxy = Proxy.newProxyInstance(iPackageManagerInterface.getClassLoader(),
    new Class<?>[]{iPackageManagerInterface},
    new HookHandler(sPackageManager)
  );
  //替换掉ActivityThread里的sPackageManager字段
  sPackageManagerField.set(curActThread,proxy);
  //替换掉ApplicationPackageManager里的mPM对象
  PackageManger pm = context.getPackageManager();
  Field mField = pm.getClass().getDeclaredField("mPM");
  mField.setAccessible(true);
  mField.set(pm,proxy);
```
注意：Context实现类里没有使用静态全局变量来保存PMS的代理对象，而是每拥有一个Context的实例就持有一个PMS代理对象引用。所以如果想要完全
Hook住PMS，需要精确控制整个进程内部创建的Context对象。由此可知，静态变量和单例都是良好的Hook点。
以上Hook仅仅使用反射和动态代理技术，更加强大的Hook机制可以进行字节码编织，如J2EE使用了cglib和asm进行AOP编程；而android上现有的插件框架
还是加载编译时代码，采用动态生成类的技术理论上也是可行的。


## 感谢
[维术的博客](http://weishu.me/archives/)
