# Glide
初始化流程：
* 得到注解生成的GeneratedAppGlideModule
* 解析Manifest配置的并对应GlideModule的meta-data
* 调用GlideModule中的applyOptions函数
* 创建glide对象
* 调用GlideModule中的registerComponents()
```java
  private static void initializeGlide(@NonNull Context context,@NonNull GlideBuilder buider){
    Context applicationContext=context.getApplicationContext();
    GeneratedAppGlideModule annotationGeneratedModule=getAnnotationGeneratedGlideModules();
    List<GlideModule> manifestModules=Collections.emptyList();
    if(annotationGeneratedModule==null|| annotationGeneratedModule.isManifestParsingEnabled()){
      manifestModules=new ManifestParser(applicationContext).parse();
    }
    if(annotationGeneratedModule!=null&&!annotationGeneratedModule.getExcludedModuleClasses().isEmpty()){
      Set<Class<?>> excludedModuleClasses=annotationGeneratedModule.getExcludeModuleClasses();
      Iterator<GlideModule> iterator=manifestModules.iterator();
      while(iterator.hasNext()){
        GlideModule current=iterator.next();
        if(!excludedModuleClasses.contains(current.getClass())){
          continue;
        }
        if(Log.isLoggable(TAG,Log.DEBUG)){
          Log.d(TAG,"AppGlideModule excludes manifest GlideModule: "+current);
        }
        iterator.remove();
      }
    }
    if(Log.isLoggable(TAG,Log.DEBUG)){
      for(GlideModule glideModule: manifestModules){
        Log.d(TAG,"Discovered GlideModule from manifest: "+glideModule.getClass());
      }
    }
    RequestManagerRetriever.RequestManagerFactory factory=annotationGeneratedModule!=null?annotationGeneratedModule.getRequestManagerFactory():null;
    builder.setRequestManagerFactory(factory);
    for(GlideModule module: manifestModules){
      module.applyOptions(applicationContext,builder);
    }
    if(annotationGeneratedModule!=null){
     annotationGeneratedModule.applyOptions(applicationContext,builder); 
    }
    Glide glide=builder.build(applicationContext);
    for(GlideModule module:manifestModules){
      module.registerComponents(applicationContext,glide,glide.registry);
    }
    if(annotationGeneratedModule!=null){
      annotationGeneratedModule.registerComponents(applicationContext,glide,glide.registry);
    }
    applicationContext.registerComponentCallbacks(glide);
    Glide.glide=glide;
  }
  
  private static GeneratedAppGlideModule getAnnotationGeneratedGlideModules(){
    GeneratedAppGlideModule result=null;
    try{
      
    }catch(ClassNotFoundException e){
      
    }catch(InstantiationException e){
      throwIncorrectGlideModule(e);
    }catch(IllegalAccessException e){
      throwIncorrectGlideModule(e);
    }catch(NoSuchMethodException e){
      throwIncorrectGlideModule(e);
    }catch(InvocationTargetException e){
      throwIncorrectGlideModule(e);
    }
    return result;
  }
  
  private static void throwIncorrectGlideModule(Exception e){
    throw new IllegalStateException("GeneratedAppGlideModuleImpl is implemented incorrectly."
      +"If you are manaully implemented this class,remove your implememtation.The Annotation"
      +"processor will generate a correct implementation",e);
  }
```
Manifest解析
```java
  public final class ManifestParser{
    private static final String TAG="ManifestParser";
    private static final String GLIDE_MODULE_VALUE="GlideModule";
    private final Context context;
    public ManifestParser(Context context){
      this.context=context;
    }
    public List<GlideModule> parse(){
      List<GlideModule> modules =new ArrayList<>();
      try{
        ApplicationInfo appInfo=context.getPackageManager().getApplicationInfo(context.getPackageName(),PackageManager.GET_META_DATA);
        if(appInfo.metaData==null){
          return modules;
        }
        for(String key:appInfo.metaData.keySet()){
          if(GLIDE_MODULE_VALUE.equals(appInfo.metaData.get(key))){
            modules.add(parseModule(key));
          }
        }
      }catch(PackageManager.NameNotFoundException e){
        throw new RuntimeException("Unable to find metadata to parse GlideModules", e);
      }
      return modules;
    }
    private static GlideModule parseModule(String className){
      Class<?> clazz;
      try{
        clazz=Class.forName(className);
      }catch(ClassNotFoundException e){
        throw new IllegalArgumentException("Unable to find GlideModule implementation",e);
      }
      Object module=null;
      try{
        module=clazz.getDeclaredConstructor().newInstance();
      }catch(InstantiationException e){
        throw new RuntimeException("Unable to instantiate GlideModule implementation for "+clazz,e);
      }catch(IllegalAccessException){
        throw new RuntimeException("Unable to instantiate GlideModule implementation for "+clazz,e);
      }catch(NoSuchMethodException e){
        throw new RuntimeException("Unable to instantiate GlideModule implementation for "+clazz,e);
      }catch(InvocationTargetException e){
        throw new RuntimeException("Unable to instantiate GlideModule implementation for "+clazz,e);
      }
      if(!(module instanceof GlideModule)){
        throw new RuntimeException("Expected instanceof GlideModule,but found "+module);
      }
      return (GlideModule)module;
    }
  }
```
## GlideBuilder
创建
* 原数据缓存GlideExecutor
* 磁盘缓存GlideExecutor
* 动画GlideExecutor
* 内存大小计算器memorySizeCalculator
* 网络状态监听器DefaultConnectivityMonitorFactory
* 位图池大小，大于0，则创建LruBitmapPool；否则，创建BitmapPoolAdapter

```java
  public final class GlideBuilder{
    private final Map<Class<?>,TransitionOptions<?,?>> defaultTransitionOptions=new ArrayMap<>();
    private Engine engine;
    private BitmapPool bitmapPool;
    private ArrayPool arrayPool;
    private MemoryCache memoryCache;
    private GlideExecutor sourceExecutor;
    private GlideExecutor diskCacheExecutor;
    private DiskCache.Factory diskCacheFactory;
    private MemorySizeCalculator memorySizeCalculator;
    private ConnectivityMonitorFactory connectivityMonitorFactory;
    private RequestOptions defaultRequestOptions=new RequestOptions();
    @Nullable
    private RequestManagerFactory requestManagerFactory;
    private GlideExecutor animationExecutor;
    private boolean isActiveResourceRetentionAllowed;
    @Nullable
    private List<RequestListener<Object>> defaultRequestListeners;
    
    @NonNull
    public GlideBuilder setBitmapPool(@Nullable BitmapPool bitmapPool){
      this.bitmapPool=bitmapPool;
      return this;
    }
    @NonNull
    public GlideBuilder setArrayPool(@Nullable ArrayPool arrayPool){
      this.arrayPool=arrayPool;
      return this;
    }
    @NonNull
    public GlideBuilder setMemoryCache(@Nullable MemoryCache memoryCache){
      this.memoryCache=memoryCache;
      return this;
    }
    @NonNull
    public GlideBuilder setDiskCache(@Nullable DiskCache.Factory diskCacheFactory){
      this.diskCacheFactory=diskCacheFactory;
      return this;
    }
    @NonNull
    public GlideBuilder setSourceExecutor(@Nullable GlideExecutor service){
      this.sourceExecutor=service;
      return this;
    }
    @NonNull
    public GlideBuilder setDiskCacheExecutor(@Nullable GlideExecutor service){
      this.diskCacheExecutor =service;
      return this;
    }
    @NonNull
    public GlideBuilder setAnimationExecutor(@Nullable GlideExecutor service){
      this.animationExecutor=service;
      return this;
    }
    @NonNull
    public GlideBuilder setDefaultRequestOptions(@Nullable RequestOptions requestOptions){
      this.defaultRequestOptions=requestOptions;
      return this;
    }
    @NonNull
    public <T> GlideBuilder setDefaultTransitionOptions(@NonNull Class<T> clazz,@Nullable TransitionOptions<?,T> options){
      defaultTransitionOptions.put(clazz,options);
      return this;
    }
    @NonNull
    public GlideBuilder setMemorySizeCalculator(@NonNull MemorySizeCalculator.Builder builder){
      return setMemorySizeCalculator(builder.build());
    }
    @NonNull
    public GlideBuilder setMemorySizeCalculator(@Nullable MemeorySizeCalculator calculator){
      this.memorySizeCalculator = calculator;
      return this;
    }
    @NonNull
    public GlideBuilder setConnectivityMonitorFactory(@Nullable ConnectivityMonitorFactory factory){
      this.connectivityMonitorFactory=factory;
      return this;
    }
    @NonNull
    public GlideBuilder setIsActiveResourceRetentionAllowed(boolean isActiveResourceRetentionAllowed){
      this.isActiveRetentionAllowed=isActiveResourceRetentionAllowed;
      return this;
    }
    @NonNull
    public GlideBuilder addGlobalRequestListener(@NonNull RequestListener<Object> listener){
      if(defaultRequestListeners==null){
        defaultRequestListeners=new ArrayList<>();
      }
      defaultRequestListeners.add(listener);
      return this;
    }
    void setRequestManagerFactory(@Nullable RequestManagerFactory factory){
      this.requestManagerFactory=factory;
    }
    GlideBuilder setEngine(Engine engine){
      this.engine=engine;
      return this;
    }
    @NonNull
    Glide build(){
      if(sourceExecutor==null){
        sourceExecutor=GlideExecutor.newSourceExecutor();
      }
      if(diskCacheExecutor==null){
        diskCacheExecutor=GlideExecutor.newDiskCacheExecutor();
      }
      if(animationExecutor==null){
        animationExecutor=GlideExecutor.newAnimationExecutor();
      }
      if(memorySizeCalculator==null){
        memorySzieCalculator=new MemorySizeCalculator.Builder(context).build();
      }
      if(connectivityMonitorFactory==null){
        connectivityMonitorFactory=new DefaultConnectivityMonitorFactory();
      }
      if(bitmapPool==null){
        int size=memorySizeCalculator.getBitmapPoolSize();
        if(size>0){
          bitmapPool=new LruBitmapPool(size);
        }else{
          bitmapPool=new BitmapPoolAdapter();
        }
      }
      if(arrayPool==null){
        arrayPool=new LruArrayPool(memorySizeCalculator.getArrayPoolSizeInBytes());
      }
      if(memoryCache==null){
        memoryCache=new LruResourceCache(memorySizeCalculator.getMemoryCacheSize());
      }
      if(diskCacheFactory==null){
        diskCacheFactory=new InternalCacheDiskCacheFactory(context);
      }
      if(engine==null){
        engine=new Engine(memoryCache,diskCacheFactory,diskCacheExecutor,sourceExecutor
          ,GlideExecutor.newUnlimitedSourceExecutor(),GlideExecutor.newAnimationExecutor(),isActiveResourceRetentionAllowed);
      }
      if(defaultRequestListeners==null){
        defaultRequestListeners=Collections.emptyList();
      }else{
        defaultRequestListeners=Collections.unmodifiableList(defaultRequestListeners);
      }
      RequestManagerRetriever requestMangerRetriever=new RequestManagerRetriever(requestMangerFactory);
      return new Glide(context,engine,memoryCache,bitmapPool,arrayPool,requestManagerRetriever,connectivityMonitorFactory,
        logLevel,defaultRequestOptions.lock(),defaultTransitionOptions,defaultRequestListeners);
    }
  }
```
### GlideExecutor
* 磁盘缓存线程池：默认构建一个核心线程数和一个最大线程数
* 原数据线程池：默认构建最优核心线程数和最优最大线程数
* 无限制原数据线程池：默认构建0个核心线程数和Integer.MAX_VALUE个最大线程数
* 动画线程池：默认构建0个核心线程数和2个/1个最大线程数

```java
  public final class GlideExecutor implements ExecutorService{
    private static final String DEFAULT_SOURCE_EXECUTOR_NAME="source";
    private static final String DEFAULT_DISK_CACHE_EXECUTOR_NAME="disk-cache";
    private static final int DEFAULT_DISK_CACHE_EXECUTOR_THREADS=1;
    private static final String TAG="GlideExecutor";
    private staic final String SOURCE_UNLIMITED_EXECUTOR_NAME="source-unlimited";
    private static final String ANIMATION_EXECUTOR_NAME="animatioin";
    private static final long KEEP_ALIVE_TIME_MS=TimeUnit.SECONDS.toMillis(10);
    private static final int MAXIMUM_AUTOMATIC_THREAD_COUNT=4;
    private static volatile int bestThreadCount;
    private final ExecutorService delegate;
    
    public static GlideExecutor newDiskCacheExecutor(){
      return newDiskCacheExecutor(DEFAULT_DISK_CACHE_EXECUTOR_THREADS,DEFAULT_DISK_CACHE_EXECUTOR_NAME,UncaughtThrowableStrategy.DEFAULT);
    }
    public static GlideExecutor newDiskCacheExecutor(UncaughtThrowableStrategy uncaughtThrowableStrategy){
      return newDiskCacheExecutor(DEFAULT_DISK_CACHE_EXECUTOR_THREADS,DEFAULT_DISK_CAHCE_EXECUTOR_NAME,uncaughtThrowableStrategy);
    }
    public static GlideExecutor newDiskCacheExecutor(int threadCount,String name,UncaughtThrowableStrategy uncaughtThrowableStrategy){
      return new GlideExecutor(new TheadPoolExecutor(threadCount,threadCount,0,TimeUnit.MILLISECONDS
        ,new PriorityBlockingQueue<Runnable>(),new DefaultThreadFactory(name,uncaughtThrowableStrategy,true)));
    }
    public static GlideExecutor newSourceExecutor(){
      return newSourceExecutor(calculateBestThreadCount(),DEFAULT_SOURCE_EXECUTOR_NAME,UncaughtThrowableStrategy.DEFAULT);
    }
    public static GlideExecutor newSourceExecutor(UncaughtThrowableStrategy uncaughtThrowableStrategy){
      return newSourceExecutor(calculateBestThreadCount(),DEFAULT_SOURCE_EXECUTOR_NAME,uncaughtThrowableStrategy);
    }
    public static GlideExecutor newSourceExecutor(int threadCount,String name,UncaughtThrowableStrategy uncaughtThrowableStrategy){
      return new GlideExecutor(new ThreadPoolExecutor(threadCount,threadCount,0,TimeUnit.MILLISECONDS
        ,new PriorityBlockingQueue<Runnable>(),new DefaultThreadFactory(name,uncaughtThrowableStrategy,false)));
    }
    public static GlideExecutor newUnlimitedSourceExecutor(){
      return new GlideExecutor(new ThreadPoolExecutor(0,Integer.MAX_VALUE,KEEP_ALIVE_TIME_MS,TimeUnit.MILLISECONDS
        ,new SynchronousQueue<Runnable>(),new DefaultThreadFactory(SOURCE_UNLIMITED_EXECUTOR_NAME,UncaughtThrowableStrategy.DEFAULT,false)));
    }
    public static GlideExecutor newAnimationExecutor(){
      int bestThreadCount=calculateBestThreadCount();
      int maximumPoolSize=bestThreadCount>=4?2:1;
      return newAnimationExecutor(maximumPoolSize,UncaughtThrowableStrategy.DEFAULT);
    }
    public static GlideExecutor newAnimationExecutor(int threadCount,UncaughtThrowableStrategy uncaughtThrowableStrategy){
      return new GlideExecutor(new ThreadPoolExecutor(0,threadCount,KEEP_ALIVE_TIME_MS,TimeUnit.MILLISECONDS
        ,new PriorityBlockingQueue<Runnable>(),new DefaultThreadFactory(ANIMATION_EXECUTOR_NAME,uncaughtThrowableStrategy,true)));
    }
    GlideExecutor(ExecutorService delegate){
      this.delegate=delegate;
    }
    @Override
    public void execute(@NonNull Runnable command){
      delegate.execute(command);
    }
    @NonNull
    @Override
    public Future<?> submit(@NonNull Runnable task){
      delegate.submit(task);
    }
    @NonNull
    @Override
    public <T>List<Future<T>> invokeAll(@NonNull Collection<? extends Callable<T>> tasks) throws InterruptedException{
      delegate.invokeAll(tasks);
    }
    @NonNull
    @Override
    public <T> List<Future<T>> invokeAll(@NonNull Collection<? extends Callable<T>> tasks,long timeout,@NonNull TimeUnit unit) throws InterruptedException{
      return delegate.invokeAll(tasks,timeout,unit);
    }
    @NonNull
    @Override
    public <T> T invokeAny(@NonNull Collection<? extends Callable<T>> tasks) throws InterruptedException,ExecutionException{
      return delegate.invokeAny(tasks);
    }
    @Override
    public <T> T invokeAny(@NonNull Collection<? extends Callable<T>> tasks,long timeout,@NonNull TimeUnit unit) throws InterruptedException,ExecutionException,TimeoutException{
    return delegate.invokeAny(tasks,timeout,unit);
    }
    @NonNull
    @Override
    public <T> Fucture<T> submit(@NonNull Runnable task,T result){
      return delegate.submit(task,result);
    }
    @Override
    public <T> Future<T> submit(@NonNull Callable<T> task){
      return delegate.submit(task);
    }
    @Override
    public void shutdown(){
      delegate.shutdown();
    }
    @NonNull
    @Override
    public List<Runnable> shutdownNow(){
      return delegate.shutdownNow();
    }
    @Override
    public boolean isShutdown(){
      return delegate.isShutdown();
    }
    @Override
    public boolean isTerminated(){
      return delegate.isTerminated();
    }
    @Override
    public boolean awaitTermination(long timeout,@NonNull TimeUnit unit) throws InterruptedException{
      return delegate.awaitTermination(timeout,unit);
    }
    @Override
    public String toString(){
      return delegate.toString();
    }
    public static int calculateBestThreadCount(){
      if(bestThreadCount==0){
        bestThreadCount=Max.min(MAXIMUM_AUTOMATIC_THREAD_COUNT,RuntimeCompat.availableProcessors());
      }
      return bestThreadCount;
    }
    public interface UncaughtThrowableStrategy{
      UncaughtThrowableStartegy IGNORE=new UncaughtThrowableStrategy(){
        @Override
        public void handle(Throwable t){
          
        }
      };
      UncaughtThrowableStrategy LOG=new UncaughtThrowableStrategy(){
        @Override
        public void handle(Throwable t){
          if(t!=null&&Log.isLoggable(TAG,Log.ERROR)){
            Log.e(TAG,"Request threw uncaught throwble",t);
          }
        }
      };
      UncaughtThrowableStrategy THROW=new UncaughtThrowableStrategy(){
        @Override
        public void handle(Throwable t){
          if(t!=null){
            throw new RuntimeException("Request threw uncaught throwable",t);
          }
        }
      };
      UncaughtThrowableStrategy DEFAULT=LOG;
      void handle(Throwable t);
    }
    private static final class DefaultThreadFactory implements ThreadFactory{
      private static final int DEFAULT_PRIORITY=android.os.Process.THREAD_PRIORITY_BACKGROUND
        +android.os.Process.THREAD_PRIORITY_MODE_FAVORABLE;
      private final String name;
      @Synthetic final UncaughtThrowableStrategy uncaughtThrowableStrategy;
      @Synthetic final boolean preventNetWorkOpretions;
      private int threadNum;
      DefaultTheadFactory(String name,UncaughtThrowableStrategy uncaughtThrowableStrategy,boolean prevetNetworkOperations){
        this.name=name;
        this.uncaughtThrowableStrategy=uncaughtThrowableStrategy;
        this.preventNetworkOperations=preventNetworkOperations;
      }
      @Override
      public synchronized Thread newThead(@NonNull Runnable runnable){
        final Thread result=new Thead(runnable,"glide-"+name+"-thread-"+threadNum){
          @Override
          public void run(){
            android.os.Process.setThreadPriority(DEFAULT_PRIORITY);
            if(preventNetworkOperations){
              StrictMode.setThreadPolicy(new ThreadPolicy.Builder().detectNetwork().penaltyDeath().build());
            }
            try{
              super.run();
            }catch(){
              uncaughtThrowableStrategy.handle(t);
            }
          }
        };
        threadNum++;
        return result;
      }
    }
  }
```
RuntimeCompat：可用进程数
```java
  final class RuntimeCompat{
    private static final String TAG="GlideRuntimeCompat";
    private static final String CPU_NAME_REGEX="cpu[0-9]+";
    private static final String CPU_LOCATION="/sys/devices/system/cpu/";
    private RuntimeCompat(){
      
    }
    static int availableProcessors(){
      int cpus=Runtime.getRuntime().availableProcessors();
      if(Build.VERSION.SDK_INT <17){
        cpus=Math.max(getCoreCountPre17(),cpus);
      }
      return cpus;
    }
    private static int getCoreCountPre17(){
      File[] cpus=null;
      //重写线程策略，允许磁盘读操作
      //https://github.com/bumptech/glide/issues/1170
      ThreadPolicy originalPolicy = StrictMode.allowThreadDiskReads();
      try{
        File cpuInfo=new File(CPU_LOCATION);
        final Pattern cpuNamePattern = Pattern.compile(CPU_NAME_REGEX);
        cups=cupInfo.listFiles(new FilenameFilter(){
          @Override
          public boolean accept(File file,String s){
            return cpuNamePattern.match(s).matches();
          }
        });
      }catch(Throwable t){
        
      }finally{
        StrictMode.setThreadPolicy(originalPolicy);
      }
      return Math.max(1,cups!=null?cups.length:0);
    }
  }
```
### MemorySizeCalcultor
内存大小计算器:
* 位图池大小bitmapPoolSize: 4或1张全屏图片大小
* 内存缓存大小memoryCacheSize: 2张全屏图大小
* 数组池大小arrayPoolSize: 低端设备，数组池内存分配等于高端设备一半  2MB
```java
  public final class MemorySizeCalculator{
    private static final String TAG="MemorySizeCalculator";
    static final int BYTES_PER_ARGB_8888_PIXEL=4;
    private static final int LOW_MEMORY_BYTE_ARRAY_POOL_DIVISOR=2;
    private final int bitmapPoolSize;
    private final int memoryCacheSize;
    private final Context context;
    private final int arrayPoolSize;
    interface ScreenDimensions{
      int getWidthPixels();
      int getHeightPixels();
    }
    MemorySizeCalculator(MemorySizeCalculator.Builder builder){
      this.context=builder.context;
      //低端设备，数组池内存分配等于高端设备一半  2MB
      arrayPoolSize=isLowMemoryDevice(builder.activityManager)?builder.arrayPoolSizeBytes/LOW_MEMORY_BYTE_ARRAY_POOL_DIVISOR:builder.arrayPoolSizeBytes;
      //0.4f  0.33f
      int maxSize=getMaxSize(builder.activityManager,builder.maxSizeMultiplier,builder.lowMemoryMaxSizeMultiplier);
      int widthPixels=builder.screenDimensions.getWidthPixels();
      int heightPixels=builder.screenDimensions.getHeightPixels();
      int screenSize=widthPixels*heightPixels*BYTES_PER_ARGB_8888_PIXEL;
      //4:1(4或1张全屏图片大小)
      int targetBitmapPoolSize=Math.round(screenSize*builder.bitmapPoolScreens);
      //2(2张全屏图大小)
      int targetMemoryCacheSize=Math.round(screenSize*builder.memoryCacheScreens);
      int availableSize=maxSize-arrayPoolSize;
      if(targetMemoryCacheSize+targetBitmapPoolSize<=availableSize){
        memoryCacheSize=targetMemoryCahceSize;
        bitmapPoolSize=targetBitmapPoolSize;
      }else{
        float part=avialableSize/(builder.bitmapPoolScreens+builder.memoryCacheScreens);
        memoryCachSize=Math.round(part*builder.memoryCacheScreens);
        bitmapPoolSize=Math.round(part*builder.bitmapPoolScreens);
      }
    }
    @TargetApi(Builder.VERSION_CODES.KITKAT)
    static boolean isLowMemoryDevice(ActivityManager activityManager){
      if(Build.VERSION.SDK_INT>=Build.VERSION_CODES.KITKAT){
        return activityManager.isLowRamDevice();
      }else{
        return true;
      }
    }
    private static int getMaxSize(ActivityManager activityManager,float maxSizeMultiplier,float lowMemoryMaxSizeMultiplier){
      final int memroyClassBytes=activityManager.getMemoryClass()*1024*1024;
      final boolean isLowMemoryDevice=isLowMemoryDevice(activityManager);
      return Math.round(memoryClassBytes*(isLowMemoryDevice?lowMemoryMaxSizeMultiplier:maxSizeMultiplier));
    }
    public int getMemoryCacheSize(){
      return memoryCacheSize;
    }
    public int getBitmapPoolSize(){
      return bitmapPoolSize;
    }
    public int getArrayPoolSizeInBytes(){
      return arrayPoolSize;
    }
    privte String toMb(int bytes){
      return Formatter.formatFileSize(context,bytes);
    }
    
    public static final class Builder{
      static final int MEMORY_CACHE_TARGET_SCREENS=2;
      /**
      * On Android O+,we use {@link android.graphics.Bitmap.Config#HEADWARE} for all reasonably sized images unless we are creating thumbnails for the first time.As a result, the Bitmap pool is much less important on O than it was on previous versions.
      */
      static final int BITMAP_POOL_TARGET_SCREENS=Build.VERSION.SDK_INT<Build.VERSION_CODES.O?4:1;
      static final float MAX_SIZE_MULTIPLIER=0.4f;
      static final float LOW_MEMORY_MAX_SIZE_MULTIPLIER=0.33f;
      //4MB
      static final int ARRAY_POOL_SIZE_BYTES=4*1024*1024;
      final Context context;
      ActivityManager activityManager;
      ScreenDimensions screenDimensions;
      float memoryCacheScreens=MEMORY_CACHE_TARGET_SCREENS;
      float bitmapPoolScreens=BITMAP_POOL_TARGET_SCREENS;
      float maxSizeMultiplier=MAX_SIZE_MULTIPLIER;
      float lowMemoryMaxSizeMultiplier=LOW_MEMORY_MAX_SIZE_MULTIPLIER;
      int arrayPoolSizeBytes=ARRAY_POOL_SIZE_BYTES;
      
      public Builder(Context context){
        this.context=context;
        activityManager=(ActivityManager)context.getSystemService(Context.ACTIVITY_SERVICE);
        screenDimensions=new DisplayMetricsScreenDimensions(context.getResources().getDisplayMetrics());
        //On Android O+ Bitmaps are allocated natively,ART is much more efficient at managing garbage and we rely heavily on HEADWARE Bitmaps,making Bitmap re-use much less important.
        if(Build.VERSION.SDK_INT>=Build.VERSION_CODES.O && isLowMemoryDevice(activityManager)){
          bitmapPoolScreens=0;
        }
        public Builder setMemoryCacheScreens(float memoryCacheScreens){
          Preconditions.checkArgument(memroyCacheScreens>=0,"Memory cache screens must be greater than or equal to 0");
          this.memoryCacheScreens=memoryCacheScreens;
          return this;
        }
        public Builder setBitmapPoolScreens(float bitmapPoolScreens){
          Preconditions.checkArgument(bitmapPoolScreens>=0,"Bitmap pool screens must be greater than or equal to 0");
          this.bitmapPoolScreens=bitmapPoolScreens;
          return this;
        }
        public Builder setMaxSizeMultiplier(float maxSizeMultiplier){
          Preconditions.checkArgument(maxSizeMultiplier>=0&& maxSizeMultiplier<=1,"Size multiplier must be between 0 and 1");
          this.maxSizeMultiplier = maxSizeMultiplier;
          return this;
        }
        public Builder setLowMemoryMaxSizeMultiplier(float lowMemoryMaxSizeMultiplier){
          Preconditions.checkArgument(lowMemoryMaxSizeMultiplier>=0&&lowMemoryMaxSizeMultiplier<=1);
          this.lowMemoryMaxSizeMultiplier=lowMemoryMaxSizeMultiplier;
          return this;
        }
        public Builder setArrayPoolSize(int arrayPoolSizeBytes){
          this.arrayPoolSizeBytes=arrayPoolSizeBytes;
          return this;
        }
        Builder setActivityManager(ActivityManager activityManager){
          this.activityManager=activityManager;
          return this;
        }
        Builder setScreenDimensions(ScreenDimensions screenDimensions){
          this.screenDimensions=screenDimensions;
          return this;
        }
        public MemorySizeCalculator build(){
          return new MemorySizeCalculator(this);
        }
      }
    }
    private static final class DisplayMetricsScreenDimensions implements ScreenDimensions{
      private final DisplayMetrics displayMetrics;
      DisplayMetricsScreenDimensions(DisplayMetrics displayMetrics){
        this.displayMetrics=displayMetrics;
      }
      @Override
      public int getWidthPixels(){
        return displayMetrics.widthPixels;
      }
      @Override
      public int getHeightPixels(){
        return displayMetrics.heightPixels;
      }
    }
  }
```
### DefaultConnectivityMonitorFactory
```java
  public class DefaultConnectivityMonitorFactory implements ConnectivityMonitorFactory{
    private static final String TAG="ConectivityMonitor";
    private static final String METWORK_PERMISSION="android.permission.ACCESS_NETWORK_STATE";
    
    @NonNull
    @Override
    public ConnectivityMonitor build(@NonNull Context context,@NonNull ConnectivityMonitor.ConnectivityListener listener){
      int permissionResult =ContextCompat.checkSelfPermission(context,NETWORK_PERMISSION);
      boolean hasPermission=permissionResult==PackageManager.PERMISSION_GRANTED;
      return hasPermission?new DefaulConnectivityMonitor(context,listener):new NullConnectivityMonitor();
    }
  }
```
### LruBitmapPool
* LruPoolStrategy：默认创建位图大小与位图配置策略SizeConfigStrategy
```java
  public class LruBitmapPool implements BitmapPool{
    private static final String TAG="LruBitmapPool";
    private static final Bitmap.Config.DEFAULT_CONFIG=Bitmap.Config.ARAG_8888;
    private final LruPoolStrategy strategy;
    private final Set<Bitmap.Config> allowedConfigs;
    private final long initialMaxSize;
    private final BitmapTracker tracker;
    private long maxSize;
    private long currentSize;
    private int hits;
    private int misses;
    private int puts;
    privatte int evictions;
    LruBitmapPool(long maxSize,LruPoolStrategy strategy,Set<Bitmap.Config> allowConfigs){
      this.initialMaxSize=maxSize;
      this.maxSize=maxSize;
      this.strategy=strategy;
      this.allowedConfigs=allowedConfigs;
      this.tracker=new NullBitmapTracker();
    }
    public LruBitmapPool(long maxSize){
      this(maxSize,getDefaultStrategy(),getDefaultAllowedConfigs());
    }
    public LruBitmapPool(long maxSize,Set<Bitmap.Config> allowedConfigs){
      this(maxSize,getDefaultStrategy(),allowedConfigs);
    }
    @Override
    public long getMaxSize(){
      return maxSize;
    }
    @Override
    public synchronized void setSizeMultiplier(float sizeMultiplier){
      maxSize=Math.round(initialMaxSize*sizeMultiplier);
      evict();
    }
    @Override
    public synchronized void put(Bitmap bitmap){
      if(bitmap==null){
        throw new NullPointerException("Bitmap must not be null");
      }
      if(bitmap.isRecycled()){
        throw new IllegalStateException("Cannot pool recycled bitmap");
      }
      if(!bitmap.isMutable()||strategy.getSize(bitmap)>maxSize||!allowedConfigs.contains(bitmap.getConfig())){
        bitmap.recycle();
        return;
      }
      final int size=strategy.getSize(bitmap);
      strategy.put(bitmap);
      tracker.add(bitmap);
      puts++;
      currentSize+=size;
      dump();
      evict();
    }
    private void evict(){
      trimToSize(maxSize);
    }
    @Override
    @NonNull
    public Bitmap get(int width,int height,Bitmap.Config config){
      Bitmap result=getDirtyOrNull(width,height,config);
      if(result!=null){
        result.eraseColor(Color.TRANSPARENT);
      }else{
        result=createBitmap(width,height,config);
      }
      return result;
    }
    @NonNull
    @Override
    public Bitmap getDirty(int width,int height,Bitmap.Config config){
      Bitmap result=getDirtyOrNull(width,height,config);
      if(result==null){
        result=createBitmap(width,height,config);
      }
      return result;
    }
    @NonNull
    private static Bitmap createBitmap(int width,int height,@Nullable Bitmap.Config config){
      return Bitmap.createBitmap(width,height,config!=null?config:DEFAULT_CONFIG);
    }
    @TargetApi(Build.VERSION_CODES.O)
    private static void assertNotHardwareConfig(Bitmap.Config config){
      if(Build.VERISON.SDK_INT<Build.VERSION_CODES.O){
        return;
      }
      if(config==Bitmap.Config.HARDWARE){
        throw new IllegalArgumentException("Connot create a mutalbe Bitmap with config: "+config+". Consider setting Downsample#ALLOW_HARDWARE_CONFIG to false in your RequestOptions and/or in GlideBuilder.setDefaultRequestOptions");
      }
    }
    @Nullable
    private synchronized Bitmap getDirtyOrNull(int width,int height,@Nullable Bitmap.Config config){
      assertNotHardwareConfig(config);
      final Bitmap result=strategy.get(width,height,config!=null?config:DEFAULT_CONFIG);
      if(result==null){
        misses++;
      }else{
        hits++;
        currentSize-=strategy.getSize(result);
        tracker.remove(result);
        normalize(result);
      }
      dump();
      return result;
    }
    private static void normalize(Bitmap bitmap){
      bitmap.setHasAlpha(true);
      maybeSetPreMultiplied(bitmap);
    }
    @TargetApi(Build.VERSION_CODES.KITKAT)
    private static void maybeSetPreMultiplied(Bitmap bitmap){
      if(Build.VERSION_CODES.SDK_INT>=Build.VERSION_CODES.KITKAT){
        bitmap.setPremultiplied(true);
      }
    }
    @Override
    public void clearMemory(){
      trimToSize(0);
    }
    @Override
    public void trimMemory(int level){
      if(level>=android.content.ComponentCallbacks2.TRIM_MEMORY_BACKGROUND){
        clearMemory();
      }else if(level>=android.content.ComponentCallback2.TRIM_MEMORY_UI_HIDDEN||level==android.content.ComponentCallback2.TRIM_MEMORY_RUNNING_CRITICAL){
        trimToSize(getMaxSize/2);
      }
    }
    private synchronized void trimToSize(long size){
      while(currentSize>size){
        final Bitmap removed=strategy.removeLast();
        if(removed==null){
          currentSize=0;
          return
        }
        tracker.remove(removed);
        currentSize-=strategy.getSize(removed);
        evictions++;
        dump();
        removed.recycle();
      }
    }
    private void dump(){
      if(Log.isLoggable(TAG,Log.VERBOSE)){
        dumpUnchecked();
      }
    }
    private void dumpUnchecked(){
      Log.v(TAG,"Hits="+hits+",misses="+misses+",puts="+puts+",evictions="+evictions+",currentSize="+currentSize+",maxSize="+masSize+"\nStrategy="+strategy);
    }
    
    private static LurPoolStrategy getDefaultStrategy(){
      final LruPoolStrategy strategy;
      if(Build.VERSION.SDK_INT>=Build.VERSION_CODES.KITKAT){
        strategy=new SizeConfigStrategy();
      }else{
        strategy=new AttributeStrategy();
      }
      return strategy;
    }
    @TargetApi(Build.VERSION_CODES.O)
    private static Set<Bitmap.Config> getDefaultAllowedConfigs(){
      Set<Bitmap.Config> configs=new HashSet<>(Arrays.asList(Bitmap.Config.values()));
      if(Build.VERSION.SDK_INT>=Build.VERSION_CODES.KITKAT){
        //GIFs 占坑
        configs.add(null);
      }
      if(Build.VERSION.SDK_INT>=Build.VERSION_CODES.O){
        configs.remove(Bitmap.Config.HARDWARE);
      }
      return Collections.unmodifiableSet(configs);
    }
    private interface BitmapTracker{
      void add(Bitmap bitmap);
      void remove(Bitmap bitmap);
    }
    private static class ThrowingBitmapTracker implements BitmapTracker{
      private final Set<Bitmap> bitmaps=Collections.synchronizedSet(new HashSet<Bitmap>());
      @Override
      public void add(Bitmap bitmap){
        if(bitmaps.contains(bitmap)){
          throw new IllegalStateException("Cannot add already added bitmap:"+bitmap+" ["+bitmap.getWidth()+"x"+bitmap.getHeight()+"]");
        }
        bitmaps.add(bitmap);
      }
      @Override
      public void remove(Bitmap bitmap){
        if(!bitmaps.contains(bitmap)){
          throw new IllegalStateException("Connot remove bitmap not in tracker");
        }
        bitmaps.remove(bitmap);
      }
    }
    private static final class NullBitmapTracker implements BitmapTracker{
      NullBitmapTracker(){}
      @Override
      public void add(Bitmap bitmap){}
      @Override
      public void remove(Bitmap bitmap){}
    }
  }
```
SizeConfigStrategy:重用bitmap字节大小的位图，提高了位图池命中率
* RGBA_F16_IN_CONFIGS数组:{Bitmap.Config.ARGB_8888,null,Bitmap.Config.RGBA_F16}
* ARGB_8888_IN_CONFIGS数组:{Bitmap.Config.ARGB_8888,null}
* RGB_565_IN_CONFIGS数组:{Bitmap.Config.RGB_565}
* ARGB_4444_IN_CONFIG数组:{Bitmap.Config.ARGB_4444}
* ALPHA_8_IN_CONFIG数组:{Bitmap.Config.ALPHA_8}  
> Bitmap.Config:描述像素如何存储，将影响质量(颜色深度)和透明/半透明颜色。
> ALPHA_8:每个像素都存储为一个半透明通道，不存储颜色信息。在这种配置下，每个像素需要一个字节的内存
> RGB_565:每个像素存储为2字节，只有rgb通道被编码：红色存储为5位精度(32个可能值)，绿色存储为6位精度(64个可能值)，蓝色存储为5位精度。当使用不需要高颜色保真度的不透明位图时，这种配置可能很有用
> ARGB_4444:因为此配置质量较差，推荐使用ARGB_8888
> ARGB_8888:每个像素被存储为4个字节。每个通道存储为8位(256个可能值)
> RGBA_F16:每个像素被存储为8个字节。每个通道存储为高精度的浮点值。这个配置适合宽色域和HDR内容
> HEADWARE:位图仅存储在图形内存中的特殊配置。这个配置的位图不可变，在屏幕上绘制是最优的
* 查找最优Key:
```java
  @RequiresApi(Build.VERSION_CODES.KITKAT)
  public class SizeConfigStrategy implements LruPoolStrategy{
    private static final int MAX_SIZE_MULTIPLE=8;
    private static fianl Bitmap.Config[] ARGB_8888_IN_CONFIGS;
    static{
      Bitmap.Config[] result=new Bitmap.Config[]{
        Bitmap.Config.ARGB_8888,
        null,
      };
      if(Build.VERSION.SDK_INT>=Build.VERSION_CODES.O){
        result=Arrays.copyOf(result,result.length+1);
      }
      ARGB_8888_IN_CONFIGS=result;
    }
    private static final Bitmap.Config[] RGBA_F16_IN_CONFIGS=ARGB_8888_IN_CONFIGS;
    private static final Bitmap.COnfig[] RGB_565_IN_CONFIGS=new Bitmap.Config[]{Bitmap.Config.RGB_565};
    private static final Bitmap.Config[] ARGB_4444_IN_CONFIGS=new Bitmap.Config[]{Bitmap.Config.ARGB_4444};
    private static final Bitmap.Config[] ALPHA_8_IN_CONFIGS=new Bitmap.Config[]{Bitmap.Config.ALPHA_8};
    
    private final KeyPool keyPool=new KeyPool();
    //类似：LinkedHashMap,
    private final GroupedLinkedMap<Key,Bitmap> groupedMap=new GroupedLinkedMap<>();
    private final Map<Bitmap.Config,NavigableMap<Integer,Integer>> sortedSizes=new HashMap<>();
    
    @Override
    public void put(Bitmap bitmap){
      int size=Util.getBitmapByteSize(bitmap);
      Key key=keyPool.get(size,bitmap.getConfig());
      groupedMap.put(key,bitmap);
      NavigalbeMap<Integer,Integer> sizes=getSizesForConfig(bitmap.getConfig());
      Integer current=sizes.get(key.size);
      sizes.put(key.size,current==null?1:current+1);
    }
    @Override
    @Nullable
    public Bitmap get(int width,int height,Bitmap.Config config){
      int size=Util.getBitmapByteSize(width,height,config);
      Key bestKey=findBestKey(size,config);
      Bitmap result=groupedMap.get(bestKey);
      if(result!=null){
        decrementBitmapOfSize(bestKey.size,result);
        result.reconfigure(width,height,config);
      }
      return result;
    }
    private Key findBestKey(int size,Bitmap.Config config){
      Key result=keyPool.get(size,config);
      for(Bitmap.Config possibleConfig:getInConfigs(config)){
        NavigableMap<Integer,Integer> sizesForPossibleConfig=getSizesForConfig(possibleConfig);
        Integer possibleSize=sizesForPossibleConfig.ceilingKey(size);
        if(possibleSize!=null&&possibleSize<=size*MAX_SIZE_MULTIPLE){
          if(possibleSize!=size||(possibleConfig==null?config!=null:!possibleConfig.equals(config))){
            keyPool.offer(result);
            result=keyPool.get(possibleSize,possibleConfig);
          }
          break;
        }
      }
      return result;
    }
    @Override
    @Nullable
    public Bitmap removeLast(){
      Bitmap removed=groupedMap.removeLast();
      if(removed!=null){
        int removedSize=Util.getBitmapByteSize(removed);
        decrementBitmapOfSize(removedSize,removed);
      }
      return removed;
    }
    private void decrementBitmapOfSize(Integer size,Bitmap removed){
      Bitmap.Config config=removed.getConfig();
      NavigableMap<Integer,Integer> sizes=getSizesForConfig(config);
      Integer current=sizes.get(size);
      if(current==null){
        throw new IllegalStateException("Tried to decrement empty size,size:"+size+:",removed:"+logBitmap(removed)+",this:"+this);
      }
      if(current==1){
        sizes.remove(size);
      }else{
        sizes.put(size,current-1);
      }
    }
    private NavigableMap<Integer,Integer> getSizesForConfig(Bitmap.Config config){
      NavigableMap<Integer,Integer> sizes=sortedSizes.get(config);
      if(sizes==null){
        sizes=new TreeMap<>();
        sortedSizes.put(config.sizes);
      }
      return sizes;
    }
    @Override
    public String logBitmap(Bitmap bitmap){
      int size=Util.getBitmapByteSize(bitmap);
      return getBitmapString(size,bitmap.getConfig());
    }
    @Override
    public String logBitmap(int width,int height,Bitmap.Config config){
      int size=Util.getBitmapByteSize(width,height,config);
      return getBitmapString(size,config);
    }
    @Override
    public int getSize(Bitmap bitmap){
      return Util.getBitmapByteSize(bitmap);
    }
    @Override
    public String toString(){
      StringBuilder sb=new StringBuilder().append("SizeConfigStrategy{groupedMap=").append(groupedMap).append(",sortedSizes=(");
      for(Map.Entry<Bitmap.Config,NavigableMap<Integer,Integer>> entry: sortedSizes.entrySet()){
        sb.append(entry.getKey()).append('[').append(entry.getValue()).append("], ");
      }
      if(!sortedSizes.isEmpty()){
        sb.replace(sb.length()-2,sb.length(),"");
      }
      return sb.append(")}").toString();
    }
    //**Key池**
    static class KeyPool extends BaseKeyPool<Key>{
      public Key get(int size,Bitmap.Config config){
        Key result=get();
        result.init(size,config);
        return result;
      }
      @Override
      protected Key create(){
        return new Key(this);
      }
    }
    //**Key**:保存Bimap.Config及对应的大小
    static final class Key implements Poolable{
      private final KeyPool pool;
      private Bitmap.Config config;
      public Key(KeyPool pool){
        this.pool=pool;
      }
      Key(KeyPool pool,int size,Bitmap.Config config){
        this(pool);
        init(size,config);
      }
      public void init(int size,Bitmap.Config config){
        this.size=size;
        this.config=config;
      }
      @Override
      public void offer(){
        pool.offer(this);
      }
      @Override
      public String toString(){
        return getBitmapString(size,config);
      }
      @Ovrride
      public boolean equals(Object o){
        if(o instanceof Key){
          Key other=(Key)o;
          return size==other.size&&Util.bothNullOrEqual(config,other.config);
        }
        return false;
      }
      @Override
      public int hashCode(){
        int result=size;
        result=31*result+(config!=null?config.hashCode():0);
        return result;
      }
    }
    static String getBimapString(int size,Bitmap.Config config){
      return "["+size+"]("+config+")";
    }
    private static Bitmap.Config[] getInConfigs(Bitmap.Config requested){
      if(Build.VERSION.SDK_INT>=Build.VERSION_CODES.O){
        if(Bitmap.Config.RGBA_F16.equals(requested)){
          return RGBA_F16_IN_CONFIGS;
        }
      }
      switch(requested){
        case ARGB_8888:
          return ARGB_8888_IN_CONFIGS;
        case RGB_565:
          return RGB_565_IN_CONFIGS;
        case ARGB_4444:
          return ARGB_4444_IN_CONFIGS;
        case ALPHA_8:
          return ALPHA_8_IN_CONFIGS;
        default:
          return new Bitmap.Config[]{requested};
      }
    }
  }
```
AttributeStrategy:重用宽度、高度、Bitmap.Config都一样的bitmap
```java
  class AttributeStrategy implements LruPoolStrategy{
    private final KeyPool keyPool=new KeyPool();
    private final GroupedLinkedMap<Key,Bitmap> groupedMap=new GroupedLinkedMap<>();
    @Override
    public void put(Bitmap bitmap){
      final Key key=keyPool.get(bitmap.getWidth(),bitmap.getHeight(),bitmap.getConfig());
      groupedMap.put(key,map);
    }
    @Override
    public Bitmap get(int width,int height,Bitmap.Config config){
      final Key key=keyPool.get(width,height,config);
      return groupedMap.get(key);
    }
    @Override
    public Bitmap removeLast(){
      return groupedMap.removeLast();
    }
    @Override
    public String logBitmap(Bitmap bitmap){
      return getBitmapString(bitmap);
    }
    @Override
    public String logBitmap(int width,int height,Bitmap.Config config){
      return getBitmapString(width,height,config);
    }
    @Override
    public int getSize(Bitmap bitmap){
      return Util.getBitmapByteSize(bitmap);
    }
    @Override
    public String toString(){
      return "AttributeStrategy:\n"+groupedMap;
    }
    private static String getBitmapString(Bitmap bitmap){
      return getBitmapString(bitmap.getWidth(),bitmap.getHeight(),bitmap.getConfig());
    }
    static String getBitmapString(int width,int height,Bitmap.Config config){
      return "["+width+"x"+height+"],"+config;
    }
    static class KeyPool extends BaseKeyPool<Key>{
      Key get(int width,int height,Bitmap.Config config){
        Key result=get();
        result.init(width,height,config);
        return result;
      }
      @Override
      protected Key create(){
        return new Key(this);
      }
    }
    static class Key implements Poolable{
      private final KeyPool pool;
      private int width;
      private int height;
      private Bitmap.Config config;
      public Key(KeyPool pool){
        this.pool=pool;
      }
      public void init(int width,int height,Bitmap.Config config){
        this.width=width;
        this.height=height;
        this.config=config;
      }
      public boolean equals(Object o){
        if(o instanceof Key){
          Key other=(Key)o;
          return width==other.width&&height==other.height&&config==other.config;
        }
        return false;
      }
      @Override
      public int hashCode(){
        int result=width;
        result=31*result+height;
        result=31*result+(config!=null?config.hashCode():0);
        return result;
      }
      @Override
      public String toString(){
        return getBitmapString(width,height,config);
      }
      @Override
      public void offer(){
        pool.offer(this);
      }
    }
  }
```
BaseKeyPool：创建长度为20的队列
```java
  abstract class BaseKeyPool<T extends Poolable>{
    private static final int MAX_SIZE=20;
    private final Queue<T> keyPool=Util.createQueue(MAX_SIZE);
    
    T get(){
      T result=keyPool.poll();
      if(result==null){
        result=create();
      }
      return result;
    }
    public void offer(T key){
      if(keyPool.size()<MAX_SIZE){
        keyPool.offer(key);
      }
    }
    abstract T create();
  }
```
Util
```java
  public final class Util{
    private static final int HASH_MULTIPLIER=31;
    private static final int HASH_ACCUMULATOR=17;
    private static final char[] HEX_CHAR_ARRAY="0123456789abcdef".toCharArray();
    private static final char[] SHA_256_CHARS=new char[64]
    private Uitl(){}
    @NonNull
    public static String sha256BytesToHex(@NonNull byte[] bytes){
      sychronized(SHA_256_CHARS){
        return bytesToHex(bytes,SHA_256_CHARS);
      }
    }
    //字节数组转为16进制的字符串
    @NonNull
    private static String bytesToHex(@NonNull byte[] bytes,@NonNull char[] hexChars){
      int v;
      for(int j=0;j<bytes.length;j++){
        v=bytes[j]&0xFF;
        hexChars[j*2]=HEX_CHAR_ARRAY[V>>>4];
        hexChars[j*2+1]=HEX_CHAR_ARRAY[V&0x0F];
      }
      return new String(hexChars);
    }
    @Deprecated
    public static int getSize(@NonNull Bitmap bitmap){
      return getBitmapByteSize(bitmap);
    }
    @TargetApi(Build.VERSION_CODES.KITKAT)
    public static int getBitmapByteSize(@NonNull Bitmap bitmap){
      if(bitmap.isRecycled()){
        throw new IllegalStateException("Cannot obtain size for recycled Bitmap:"+bitmap+"["+bitmap.getWidth()+"x"+bitmap.getHeight()+"]"+bitmap.getConfig());
      }
      if(Build.VERSION.SDK_INT>=Build.VERSION_CODES.KITKAT){
        try{
          return bitmap.getAllocationByteCount();
        }catch(NullPointerException e){
          
        }
      }
      return bitmap.getHeight*bitmap.getRowBytes();
    }
    private static int getBytesPerPixel(@Nullable Bitmap.Config config){
      if(config==null){
        config=Bitmap.Config.ARGB_8888;
      }
      int bytesPerPixel;
      switch(config){
        case ALPHA_8:
          bytesPerPixel=1;
          break;
        case RGB_565:
        case ARGB_4444:
          bytesPerPixel=2;
          break;
        case RGBA_F16:
          bytesPerPixel=8;
          break;
        case ARGB_8888:
        default:
          bytesPerPixel=4;
          break;
      }
      return bytesPerPixel;
    }
    public static boolean isValidDimensions(int width,int height){
      return isValidDimension(width)&&isValidDimension(height);
    }
    private static boolean isValidDimensions(int dimen){
      return dimen>0||dimen==Target.SIZE_ORIGINAL;
    }
    public static void assetMainThread(){
      if(!isOnMainThread()){
        throw new IllegalArgumentException("You must call this method on the main thread");
      }
    }
    public static void assertBackgroundThread(){
      if(!isOnBackgroundThread()){
        throw new IllegalArgumentException("You must call this method on a background thread");
      }
    }
    public static boolean isOnMainThread(){
      return Looper.myLooper()==Looper.getMainLooper();
    }
    public static boolean isOnBackgroundThread(){
      return !isOnMainThread();
    }
    @NonNull
    public static <T> Queue<T> createQueue(int size){
      return new ArrayDeque<>(size);
    }
    @NonNull
    public static <T> List<T> getSnapshot(@NonNull Collection<T> other){
      List<T> result=new ArrayList<>(other.size());
      for(T item: other){
        if(item!=null){
          result.add(item);
        }
      }
      return result;
    }
    public static boolean bothNullOrEqual(@Nullable Object a,@Nullable Object b){
      return a==null?b==null:a.equals(b);
    }
    public static boolean bothModelsNullEquivalentOrEquals(@Nullable Object a,@Nullable Object b){
      if(a==null){
        return b==null;
      }
      if(a istanceof Model){
        return ((Model)a).isEquivalentTo(b);
      }
      return a.equals(b);
    }
    public static int hashCode(int value){
      return hashCode(value,HASH_ACCUMULATOR);
    }
    public static int hashCode(int value,int accumulator){
      return accumulator*HASH_MULTIPLIER+value;
    }
    public static int hashCode(float value){
      return hashCode(value,HASH_ACCUMULATOR);
    }
    public static int hashCode(float value,int accumulator){
      return hashCode(Float.floatToIntBits(value),accumulator);
    }
    public static int hashCode(@Nullable Object object,int accumulator){
      return hashCode(object==null?0:object.hashCode(),accumulator);
    }
    public static int hashCode(boolean value,int accumulator){
      return hashCode(value?1:0,accumulator);
    }
    public static int hashCode(boolean value){
      return hashCode(value,HASH_ACCUMULATOR);
    }
  }
```
GroupedLinkedMap:类似LinkedHashMap，思想：找到LRU位图大小，而不是LRU位图对象。当需要减少缓存大小时，我们可以从最近最少使用的位图大小中国删除位图。

```java
  class GroupedLinkedMap<K extends Poolable,V>{
    private final LinkedEntry<K,V> head=new LinkedEntry<>();
    private final Map<K,LinkedEntry<K,V>> keyToEntry = new HashMap<>();
    
    public void put(K key,V value){
      LinkedEntry<K,V> entry=keyToEntry.get(key);
      if(entry==null){
        entry=new LinkedEntry<>(key);//下一个==上一个==当前
        makeTail(entry);
        keyToEntry.put(key,entry);
      }else{
        key.offer();
      }
      entry.add(value);
    }
    @Nullable
    public V get(K key){
      LinkedEntry<K,V> entry=keyToEntry.get(key);
      if(entry==null){
        entry=new LinkedEntry<>(key);
        keyToEntry.put(key,entry);
      }else{
        key.offer();
      }
      makeHead(entry);
      return entry.removeLast();
    }
    @Nullable
    public V removeLast(){
      LinkedEntry<K,V> last = head.prev;
      while(!last.equals(head)){
        V removed = last.removeLast();
        if(removed!=null){
          return removed;
        }else{
          removeEntry(last);
          keyToEntry.remove(last.key);
          last.key.offer();
        }
        last=last.prev;
      }
      return null;
    }
    @Override
    public String toString(){
      StringBuilder sb=new StringBuilder("GroupedLinkedMap(");
      LinkedEntry<K,V> current=head.next;
      boolean hadAtLeastOneItem=false;
      while(!current.equals(head)){
        hadAtLeastOneItem=true;
        sb.append("{").append(current.key).append(":").append(current.size()).append("}, ");
        current=current.next;
      }
      if(hadAtLeastOneItem){
        sb.delete(sb.length()-2,sb.length());
      }
      return sb.append(")").toString();
    }
    private void makeHead(LinkedEntry<K,V> entry){
      removeEntry(entry);
      entry.prev=head;
      entry.next=head.next;
      updateEntry(entry);
    }
    private void makeTail(LinkedEntry<K,V> entry){
      removeEntry(entry);
      entry.prev=head.prev;
      entry.next=head;
      updateEntry(entry);
    }
    private static <K,V> void updateEntry(LinkedEntry<K,V> entry){
      entry.next.prev=entry;
      entry.prev.next=entry;
    }
    private static <K,V> void removeEntry(LinkedEntry<K,V> entry){
      entry.prev.next=entry.next;
      entry.next.prev=entry.prev;
    }
    //**链表条目**：创建一个key为null的链表头
    private static class LinkedEntry<K,V>{
      @Synthetic final K key;
      private List<V> values;
      LinkedEntry<K,V> next;
      LinkedEntry<K,V> prev;
      
      LinkedEntry(){
        this(null);
      }
      LinkedEntry(K key){
        next=prev=this;
        this.key=key;
      }
      @Nullable
      public V removeLast(){
        final int valueSize=size();
        return valueSize>0?values.remove(valueSize-1):null;
      }
      public int size(){
        return values!=null?values.szie():0;
      }
      public void add(V value){
        if(values==null){
          values=new ArrayList<>();
        }
        values.add(value);
      }
    }
  }
  
```
LruArrayPool:使用最近最少使用策略维持一个固定大小的数组池
```java
  public final class LruArrayPool implements ArrayPool {
    //4MB
    private static final int DEFUALT_SIZE=4*1024*1024;
    //int数组的最大倍数可以大于从池中返回的请求大小
    static final int MAX_OVER_SIZE_MULTIPLE=8;
    //用于计算单个字节数组可能消耗总池的最大百分比
    private static final int SINGLE_ARRAY_MAX_SIZE_DIVISOR=2;
    private final GroupedLinkedMap<Key,Object> groupedMap=new GroupedLinkedMap<>();
    private final KeyPool keyPool=new KeyPool();
    private final Map<Class<?>,NavigableMap<Integer,Integer>> sortedSizes=new HashMap<>();
    private final Map<Class<?>,ArrayAdapterInterface<?>> adapters=new HashMap<>();
    private final int maxSize;
    private int currentSize;
    
    public LruArrayPool(){
      maxSize=DEFAULT_SIZE;
    }
    public LruArrayPool(int maxSize){
      this.maxSize=maxSize;
    }
    @Deprecated
    @Override
    public <T> void put(T array,Class<T> arrayClass){
      put(array);
    }
    @Override
    public synchronized <T> void put(T array){
      Class<T> arrayClass=(Class<T>)array.getClass();
      ArrayAdapterInterface<T> arrayAdapter=getAdapterFromType(arrayClass);
      int size=arrayAdapter.getArrayLength(array);
      int arrayBytes=size*arrayAdapter.getElementSizeInBytes();
      if(!isSmallEnoughForReuse(arrayBytes)){
        return;
      }
      Key key=keyPool.get(size,arrayClass);
      groupedMap.put(key,array);
      NavigableMap<Integer,Integer> sizes=getSizesForAdapter(arrayClass);
      Integer current =sizes.get(key.size);
      sizes.put(key.size,current==null?1:current+1);
      currentSize+=arrayBytes;
      evict();
    }
    @Override
    public synchronized <T> T getExact(int size,Class<T> arrayClass){
      Key key=keyPool.get(size,arrayClass);
      return getForKey(key,arrayClasss);
    }
    @Override
    public synchronized <T> T get(int size,Class<T> arrayClass){
      Integer possibleSize=getSizesForAdapter(arrayClass).ceilingKey(size);
      final Key key;
      if(mayFillRequest(szie,possibleSize)){
        key=keyPool.get(possibleSize,arrayClass);
      }else{
        key=keyPool.get(size,arrayClass);
      }
      return getForKey(key,arrayClass);
    }
    private <T> T getForKey(Key key,Class<T> arrayClass){
      ArrayAdapterInterface<T> arrayAdapter=getAdapterFromType(arrayClass);
      T result=getArrayForKey(key);
      if(result!=null){
        currentSize-=arrayAdapter.getArrayLength(result)*arrayAdapter.getElementSizeInBytes();
        decrementArrayOfSize(arrayAdapter.getArrayLength(result),arrayClass);
      }
      if(resutl==null){
        if(Log.isLoggable(arrayAdapter.getTag(),Log.VERBOSE)){
          Log.v(arrayAdpate.getTag(),"Allocated "+key.size+" bytes");
        }
        result=arrayAdapter.newArray(key.size);
      }
      return result;
    }
    @Nullable
    private <T> T getArrayForKey(Key key){
      return (T) groupedMap.get(key);
    }
    private boolean isSmallEnoughForReuse(int byteSize){
      return byteSize <=maxSize/SINGLE_ARRAY_MAX_SIZE_DIVISOR;
    }
    private boolean mayFillRequest(int requestedSize,Integer actualSize){
      return actualSize!=null&&(isNoMoreThanHalfFull()||actualSize<=(MAX_OVER_SIZE_MULTIPLE*requestedSize));
    }
    private boolean isNoMoreThanHalfFull(){
      return currentSize==0||(maxSize/currentSize>=2);
    }
    @Override
    public synchronized void clearMemory(){
      evictToSize(0);
    }
    @Override
    public synchronized void trimMemory(int level){
      if(level>=android.content.ComponentCallbacks2.TRIM_MEMORY_BACKGROUND){
        clearMemory();
      }else if(level>=android.content.ComponentCallbacks2.TRIM_MEMORY_UI_HIDDEN||level==android.content.ComponentCallbacks2.TRIM_MEMORY_RUNNING_CRITICAL){
        evictToSize(maxSize/2);
      }
    }
    private void evict(){
      evictToSize(maxSize);
    }
    private void evictToSize(int size){
      while(currentSize>size){
        Object evicted=groupedMap.removeLast();
        Preconditions.checkNotNull(evicted);
        ArrayAdapterInterface<Object> arrayAdapter=getAdapterFromObject(evicted);
        currentSize-=arrayAdapter.getArrayLength(evicted)*arrayAdapter.getElementSizeInBytes();
        decrementArrayOfSize(arrayAdapter.getArrayLength(evicted),evicted.getClass());
      }
    }
    private void decrementArrayOfSize(int size,Class<?> arrayClass){
      NavigableMap<Integer,Integer> sizes=getSizesForAdapter(arrayClass);
      Integer current = size.get(size);
      if(current==null){
        throw new NullPointerException("Tried to decrement empty size, size: "+size+",this:"+this);
      }else{
        if(current.intValue==1){
          sizes.remove(Integer.valueOf(size));
        }else{
          sizes.put(Integer.valueOf(size),Integer.valueOf(current.intValue()-1));
        }
      }
    }
    private NavigableMap<Integer,Integer> getSizesForAdapter(Class<?> arrayClass){
       NavigableMap<Integer,Integer> sizes=(NavigableMap)this.sortedSizes.get(arrayClass);
       if(sizes==null){
        sizes=new TreeMap();
        this.sortedSizes.put(arrayClass,sizes);
       }
       return (NavigableMap)sizes;
    }
    private <T> ArrayAdapterInterface<T> getAdapterFromObject(T object){
      return this.getAdapterFromType(object.getClass());
    }
    private <T> ArrayAdapterInterface<T> getAdapterFromType(Class<T> arrayPoolClass){
      ArrayAdapterInterface<?> adapter=(ArrayAdapterInterface)this.adapters.get(arrayPoolClass);
      if(adpater==null){
        if(arrayPoolClass.equals(int[].class)){
          adapter = new IntegerArrayAdapter();
        }else{
          if(!arrayPoolClass.equals(byte[].class)){
            throw new IllegalArgumentException("No array pool found for:"+arrayPoolClass.getSimpleName());;
          }
          adapter=new ByteArrayAdapter();
        }
        this.adapters.put(arrayPoolClass,adapter);
      }
      return (ArrayAdapterInterface)adapter;
    }
    int getCurrentSize(){
      int currentSize=0;
      Iterator var2=this.sortedSizes.keySet().iterator();
      while(var2.hasNext()){
        Class<?> type=(Class)var2.next();
        Integer size;
        ArrayAdapterInterface adapter;
        for(Iterator var4=((NavigableMap)this.sortedSizes.get(type)).keySet().iterator();var4.hasNext();currentSize+=size.intValue()*((Integer)((NavigableMap)this.sortedSizes.get(type)).get(size)).intValue()*adapter.getElementSizeInBytes()){
          size=(Integer)var4.next();
          adapter=this.getAdapterFromType(type);
        }
      }
      return currentSize;
    }
    private static final class Key implements Poolable{
      private final LruArrayPool.KeyPool pool;
      int size;
      private Class<?> arrayClass;
      Key(LruArrayPool.KeyPool pool){
        this.pool=pool;
      }
      void init(int length,Class<?> arrayClass){
        this.size=length;
        this.arrayClass=arrayClass;
      }
      public boolean equals(Object o){
        if(!(o instanceof LruArrayPool.Key)){
          return false;
        }else{
          LruArrayPool.Key other=(LruArrayPool.Key)o;
          return this.size==other.size&&this.arrayClass==other.arrayClass;
        }
      }
      public String toString(){
        return "Key{size="+this.size+"array="+this.arrayClass+"}";
      }
      public void offer(){
        this.pool.offer(this);
      }
      public int hashCode(){
        int result=this.size;
        result=31*result+(this.arrayClass!=null?this.arrayClass.hashCode():0);
        return result;
      }
    }
    private static final class KeyPool extends BaseKeyPool<LruArrayPool.Key>{
      KeyPool(){
      }
      LruArrayPool.Key get(int size,Class<?> arrayClass){
        LruArrayPool.Key result=(LruArrayPool.Key) this.get();
        result.init(size,arrayClass);
        return result;
      }
      protected LruArrayPool.Key create(){
        return new LruArrayPool.Key(this);
      }
    }
  }
```
IntegerArrayAdapter
```java
  public final class IntegerArrayAdapter implements ArrayAdapterInterface<int[]>{
    private static final String TAG="IntegerArrayPool";
    @Override
    public String getTag(){
      return TAG;
    }
    @Override
    public int getArrayLength(int[] array){
      return array.length;
    }
    @Override
    public int[] newArray(int length){
      return new int[length];
    }
    @Override
    public int getElementSizeInBytes(){
      return 4;
    }
  }
```
ByteArrayAdapter
```java
  public final class ByteArrayAdapter implements ArrayAdapterInterface<byte[]>{
    private static final String TAG="ByteArrayPool";
    @Override
    public String getTag(){
      return TAG;
    }
    @Override
    public int getArrayLength(byte[] array){
      return array.length;
    }
    @Override
    public byte[] newArray(int length){
      return new byte[length];
    }
    @Override
    public int getElementSizeInBytes(){
      return 1;
    }
  }
```
LruResourceCache:使用最近最少策略的内存缓存
```java
  public class LruResourceCache extends LruCache<Key,Resource<?>> implements MemoryCache{
    private ResourceRemovedListener listener;
    public LruResourceCache(long size){
      super(size);
    }
    @Override
    public void setResourceRemovedListener(@NonNull ResourceRemovedLisnter listener){
      this.listener = listener;
    }
    @Override
    protected void onItemEvicted(@NonNull Key key,@Nullable Resource<?> item){
      if(listener!=null&&item!=null){
        listener.onResourceRemoved(item);
      }
    }
    @Override
    protected int getSize(@Nullable Resource<?> item){
      if(item==null){
        return super.getSize(null);
      }else{
        return item.getSize();
      }
    }
    @Override
    public void trimMemory(int level){
      if(level>=android.content.ComponentCallbacks2.TRIM_MEMORY_BACKGROUND){
        clearMemory();
      }else if(level>=android.content.ComponentCallbacks2.TRIM_MEMORY_UI_HIDDEN||level==android.content.ComponentCallbacks2.TRIM_MEMRORY_RUNNING_CRITICAL){
        trimToSize(getMaxSize()/2);
      }
    }
  }
```
LruCache:使用最近最少策略回收item,每个item分配一个大小
```java
  public class LruCache<T,Y>{
    private final Map<T,Y> cache = new LinkedHashMap<>(100,0.75f,true);
    private final long initialMaxSize;
    private long maxSize;
    private long currentSize;
    public LruCache(long size){
      this.initialMaxSize=size;
      this.maxSize=size;
    }
    public synchronized void setSizeMultiplier(float multiplier){
      if(multiplier<0){
        throw new IllegalArgumentException("Multiplier must be >=0");
      }
      maxSize=Math.round(initialMaxSize*multiplier);
      evict();
    }
    protected int getSize(Y item){
      return 1;
    }
    protected synchronized int getCount(){
      return cache.size();
    }
    protected void onItemEvicted(@NonNull T key,@Nullable Y item){
    }
    public synchronized long getMaxSize(){
      return maxSize;
    }
    public synchronized long getCurrentSize(){
      return currentSize;
    }
    public synchronized boolean contains(@NonNull T key){
      return cache.containsKey(key);
    }
    @Nullable
    public synchronized Y get(@NonNull T key){
      return cache.get(key);
    }
    @Nullable
    public synchronized Y put(@NonNull T key,@Nullable Y item){
      final int itemSize=getSize(item);
      if(itemSize>=maxSize){
        onItemEvicted(key,item);
        return null;
      }
      if(item!=null){
        currentSize+=itemSize;
      }
      @Nullable final Y old = cache.put(key,item);
      if(old!=null){
        currentSize-=getSize(old);
        if(!old.equals(item)){
          onItemEvicted(key,old);
        }
      }
      evict();
      return old;
    }
    @Nullable
    public synchronized Y remove(@NonNull T key){
      final Y value = cache.remove(key);
      if(value!=null){
        currentSize-=getSize(value);
      }
      return value;
    }
    public void clearMemory(){
      trimToSize(0);
    }
    protected synchronized void trimToSize(long size){
      Map.Entry<T,Y> last;
      Iterator<Map.Entry<T,Y>> cacheIterator;
      while(currentSize>size){
        cacheIterator=cache.entrySet().iterator();
        last=cacheIterator.next();
        final Y toRemove = last.getValue();
        currentSize-=getSize(toRemove);
        final T key=last.getKey();
        cacheIterator.remove();
        onItemEvicted(key,toRemove);
      }
    }
    private void evict(){
      trimToSize(maxSize);
    }
    
  }
```
InternalCacheDiskCacheFactory:创建缓存大小为250M、缓存目录为"image_manager_disk_cache"
```java
  public fianl class InternalCacheDiskCacheFactory extends DiskLruCacheFactory{
    public InternalCacheDiskCacheFactory(Context context){
      this(context,DiskCache.Factory.DEFUALT_DISK_CACHE_DIR,DiskCache.Factory.DEFAULT_DISK_CACHE_SIZE);
    }
    public InternalCacheDiskCacheFactory(Context context,long diskCacheSize){
      this(context,DiskCache.Factory.DEFAULT_DISK_CACHE_DIR,diskCacheSize);
    }
    public InternalCacheDiskCacheFactory(final Context context,final String diskCacheName,long diskCacheSize){
      super(new CacheDirectoryGetter(){
        @Override
        public File getCacheDirectory(){
          File cacheDirectory=context.getCacheDir();
          if(cacheDirectory==null){
            return null;
          }
          if(diskCacheName!=null){
            return new File(cacheDirectory,diskCacheName);
          }
          return cacheDirectory;
        }
      },diskCacheSize);
    }
  }
```
DiskLruCacheFactory
```java
DiskLruCacheFactory implements DiskCache.Factory{
  private final long diskCacheSize;
  private final CacheDirectoryGetter cacheDirectoryGetter;
  public interface CacheDirectoryGetter{
    File getCacheDirectory();
  }
  public DiskLruCacheFactory(final String diskCacheFolder,long diskCacheSize){
    this(new CacheDirectoryGetter(){
      @Override
      public File getCacheDirectory(){
        return new File(diskCacheFolder);
      }
    },diskCacheSize);
  }
  public DiskLruCacheFactory(final String diskCacheFolder,final String diskCacheName,long diskCacheSize){
    this(new CacheDirectoryGetter(){
      @Override
      public File getCacheDirectory(){
        return new File(diskCacheFolder,diskCacheName);
      }
    },diskCacheSize);
  }
  public DiskLruCacheFactory(CacheDirectoryGetter cacheDirectoryGetter,long diskCacheSize){
    this.diskCacheSize=diskCacheSize;
    this.cacheDirectoryGetter=cacheDirectoryGetter;
  }
  @Override
  public DiskCache build(){
    File cacheDir = cacheDirectoryGetter.getCacheDirectory();
    if(cacheDir==null){
      return null;
    }
    if(!cacheDir.mkdirs()&&(!cacheDir.exists()||!cacheDir.isDirectory())){
      return null;
    }
    return DiskLruCacheWrapper.create(cacheDir,diskCacheSize);
  }
  
}
```
DiskCache
```java
  public interface DiskCache{
    interface Factory{
      int DEFAULT_DISK_CACHE_SIZE=250*1024*1024;
      String DEFAULT_DISK_CACHE_DIR="image_manager_disk_cahce";
      @Nullable
      DiskCache build();
    }
    interface Writer{
      boolean write(@NonNull File file);
    }
    @Nullable
    File get(Key key);
    void put(Key key,Writer writer);
    void delete(Key key);
    void clear();
  }
```
DiskLruCacheWrapper
```java
  public class DiskLruCacheWrapper implements DiskCache{
    private static final String TAG="DiskLurCacheWrapper";
    private static final int APP_VERSION=1;
    private static final int VALUE_COUNT=1;
    private static DiskLruCacheWrapper wrapper;
    private final SafeKeyGenerator safeKeyGenerator;
    private final File directory;
    private final long maxSize;
    private final DiskCacheWriteLocker writeLocker=new DiskCacheWriteLocker();
    private DiskLruCache diskLruCache;
    @Deprecated
    public static synchronized DiskCache get(File directory,long maxSize){
      if(wrapper==null){
        wrapper=new DiskLruCacheWrapper(directory,maxSize);
      }
      return wrapper;
    }
    public static DiskCache create(Fiel directory,long maxSize){
      return new DiskLruCacheWrapper(directory,maxSize);
    }
    @Deprecated
    protected DiskLruCacheWrapper(File directory,long maxSize){
      this.directory=directory;
      this.maxSize=maxSize;
      this.safeKeyGenerator=new SafeKeyGenerator();
    }
    private synchronized DiskLruCache getDiskCache() throws IOException{
      if(diskLruCache==null){
        diskLruCache=DiskLruCache.open(directory,APP_VERSION,VALUE_COUNT,maxSize);
      }
      return diskLruCache;
    }
    @Override
    public File get(Key key){
      String safeKey = safeKeyGenerator.getSafeKey(key);
      File result=null;
      try{
        final DiskLruCache.Value value=getDiskCache().get(safeKey);
        if(value!=null){
          result=value.getFile(0);
        }
      }catch(IOException e){   
      }
      return result;
    }
    @Override
    public void put(Key key,Writer writer){
      String safeKey=safeKeyGenerator.getSafeKey(key);
      writeLocker.acquire(safeKey);
      try{
        try{
          DiskLruCache diskCache=getDiskCache();
          Value current=diskCache.get(safeKey);
          if(current!=null){
            return;
          }
          DiskLruCache.Editor editor=diskCache.edit(safeKey);
          if(editor==null){
            throw new IllegalStateException("Had two simultaneous puts for: "+safeKey);
          }
          try{
            File file = editor.getFile(0);
            if(writer.write(file)){
              editor.commit();
            }
          }finally{
            editor.abortUnlessCommitted();
          }
        }catch(IOException e){
        }
      }finally{
        writeLocker.release(safeKey);
      }
    }
    public void delete(Key key){
      String safeKey=safeKeyGenerator.getSafeKey(key);
      try{
        getDiskCache().remove(safeKey);
      }catech(IOException e){
        
      }
    }
    @Override
    public synchronized void clear(){
      try{
        getDiskCache().delete();
      }catch(IOException e){
        
      }finally{
        resetDiskCache();
      }
    }
    private synchronized void resetDiskCache(){
      diskLruCache=null;
    }
  }
```
SafeKeyGenerator:生成并缓存安全唯一的文件名
```java
  public class SafeKeyGenerator{
    private final LruChace<Key,String> loadIdToSafeHash=new LruCache<>(1000);
    private final Pools.Pool<PoolableDigestContainer> digestPool=FactoryPools.threadSafe(10,
      new FactoryPools.Factory<PoolableDigestContainer>(){
        @Override
        public PoolableDigestContainer create(){
          try{
            return new PoolableDigestContainer(MessageDigest.getInstance("SHA-256"));
          }catch(NoSuchAlgorithmException e){
            thrwo new RuntimeException(e);
          }
        }
      });
    public String getSafeKey(Key key){
      String safeKey;
      synchronized(loadIdToSafeHash){
        safeKey=loadIdToSafeHash.get(key);
      }
      if(safeKey==null){
        safeKey=calculateHexStringDigest(key);
      }
      synchronized(loadIdToSafeHash){
        loadIdToSafeHash.put(key,safeKey);
      }
      return safeKey;
    }
    private String calculateHexStringDigest(Key key){
      PoolableDigestContainer container = Preconditions.checkNotNull(digestPool.acquire());
      try{
        key.updateDiskCacheKey(container.messageDigest);
        return Util.sha256BytesToHex(container.messageDigest.digest());
      }finally{
        digestPool.release(container);
      }
    }
    private static final class PoolableDigestContainer implements FactoryPools.Poolable{
      final MessageDigest messageDigest;
      private final StateVerifier stateVerifier = StateVerifier.newInstance();
      PoolableDigestContainer(MessageDigest messageDigest){
        this.messageDigest=messageDigest;
      }
      @NonNull
      @Override
      public StateVerifier getVerifier(){
        return stateVerifier;
      }
    }
    
  }
```
DiskCacheWriteLocker
```java
  final class DiskCacheWriteLocker{
    private final Map<String,WriteLock> locks=new HashMap<>();
    private final WriteLockPool writeLockPool=new WriteLockPool();
    void acquire(String safeKey){
      WriteLock writeLock;
      synchronized(this){
        writeLock=locks.get(safeKey);
        if(writeLock==null){
          writeLock=writeLockPool.obtain();
          locks.put(safeKey,writeLock);
        }
        writeLock.interestedThreads++;
      }
      writeLock.lock.lock();
    }
    void release(String safeKey){
      WriteLock writeLock;
      synchronized(this){
        writeLock=Preconditions.checkNotNull(lock.get(safeKey));
        if(writeLock.interestedThreads<1){
          throw new IllegalStateException("Cannot release a lock that is not held,safeKey: "+safeKey+",interestedThreads: "+writeLock.interestedThreads);
        }
        writeLock.interestedThreads--;
        if(writeLock.interestedThreads==0){
          WriteLock removed=locks.remove(saveKey);
          if(!removed.equals(writeLock)){
            throw new IllegalStateException("Removed the wrong lock,expected to remove: "+writeLock+",but actually removed:"+removed+",safeKey: "+safeKey);
          }
          writeLockPool.offer(removed);
        }
      }
      writeLock.lock.unlock();
    }
    private static class WriteLock{
      final Lock lock = new ReentrantLock();
      int interestedThreads;
      WriteLock(){}
    }
    private static class WriteLockPool{
      private static final int MAX_POOL_SIZE=10;
      private final Queue<WriteLock> pool=new ArrayDeque<>();
      WriteLockPool(){}
      WriteLock obtain(){
        WriteLock result;
        synchronized(pool){
          result=pool.poll();
        }
        if(result==null){
          result=new WriteLock();
        }
        return result;
      }
      void offer(WriteLock writeLock){
        synchronized(pool){
          if(pool.size()<MAX_POOL_SIZE){
            pool.offer(writeLock);
          }
        }
      }
    }
  }
```
FactoryPools
```java
  public final class FactoryPools{
    private static final String TAG="FactoryPools";
    private static final int DEFAULT_POOL_SIZE=20;
    private static final Resetter<Object> EMPTY_RESETTER=new Resetter<Object>(){
      @Override
      public void reset(@NonNull Object object){
        
      }
    };
    private FactoryPools(){}
    @NonNull
    public static <T extends Poolable> Pool<T> simple(int size,@NonNull Factory<T> factory){
      return build(new SimplePool<T>(size),factory);
    }
    @NonNull
    public static <T extends Poolable> Pool<T> threadSafe(int size,@NonNull Factory<T> factory){
      return build(new SynchronizedPool<T>(size),factory);
    }
    @NonNull
    public static <T> Pool<List<T>> threadSafeList(){
      return threadSafeList(DEFAULT_POOL_SIZE);
    }
    @NonNull
    public static <T> Pool<List<T>> threadSafeList(int size){
      return build(new SynchronizedPool<List<T>>(size),new Factory<List<T>>(){
        @NonNull
        @Override
        public List<T> create(){
          return new ArrayList<>();
        }
      },new Resetter<List<T>>(){
        @Override
        public void reset(@NonNull List<T> object){
          object.clear();
        }
      });
    }
    @NonNull
    private static <T extends Poolable> Pool<T> build(@NonNull Pool<T> pool,@NonNull Factory<T> factory){
      return build(pool,factory,FactoryPools.<T>emptyResetter());
    }
    @NonNull
    private static <T>Pool<T> build(@NonNull Pool<T> pool,@NonNull Factory<T> factory,@NonNull Resetter<T> resetter){
      return new FactoryPool<>(pool,factory,resetter);
    }
    @NonNull
    private static <T> Resetter<T> emptyResetter(){
      return (Resetter<T>)EMPTY_RESETTER;
    }
    public interface Factory<T>{
      T create();
    }
    public interface Resetter<T>{
      void reset(@NonNull T object);
    }
    public interface Poolable{
      @NonNull
      StateVerifier getVerifier();
    }
    private static final class FactoryPool<T> implements Pool<T>{
      private final Factory<T> factory;
      private final Resetter<T> resetter;
      private final Pool<T> pool;
      FactoryPool(@NonNull Pool<T> pool,@NonNull Factory<T> factory,@NonNull Resetter<T> resetter){
        this.pool=pool;
        this.factory=factory;
        this.resetter=resetter;
      }
      @Override
      public T acquire(){
        T result=pool.acquire();
        if(result==null){
          result=factory.create();
        }
        if(result instanceof Poolable){
          ((Poolable)result).getVerifier().setRecycled(false);
        }
        return result;
      }
      @Override
      public boolean release(@NonNull T instance){
        if(instance instanceof Poolable){
          ((Poolable)instance).getVerifier().setRecycled(true);
        }
        resetter.reset(instance);
        return pool.release(instance);
      }
    }
    
  }
```
Pools:对象池
```java
  public final class Pools{
    public interface Pool<T>{
      @Nullable
      T acquire();
      boolean release(@NonNull T instance);
    }
    private Pools(){}
    //非同步对象池
    public static class SimplePool<T> implements Pool<T>{
      private final Object[] mPool;
      private int mPoolSize;
      public SimplePool(int maxPoolSize){
        if(maxPoolSzie<=0){
          throw new IllegalArgumentException("The max pool size must be > 0");
        }
        mPool=new Object[maxPoolSize];
      }
      @Override
      public T acquire(){
        if(mPoolSize>0){
          final int lastPooledIndex=mPoolSize-1;
          T instance=(T)mPool[lastPooledIndex];
          mPool[lastPooledIndex]=null;
          mPoolSize--;
          return instance;
        }
        return null;
      }
      @Override
      public boolean release(@NonNull T instance){
        if(isInPool(instance)){
          throw new IllegalStateException("Already in the pool!");
        }
        if(mPoolSize<mPool.length){
          mPool[mPoolSize]=instance;
          mPoolSize++;
          return true;
        }
        return false;
      }
      private boolean isInPool(@NonNull T instance){
        for(int i=0;i<mPoolSize;i++){
          if(mPool[i]==instance){
            return true;
          }
        }
        return false;
      }
    }
    //同步对象池
    public static class SynchronizedPool<T>  extends SimplePool<T>{
      private final Object mLock=new Oject();
      public SynchronizedPool(int maxPoolSize){
        super(maxPoolSize);
      }
      @Override
      public T acquire(){
        synchronized(mLock){
          return super.acquire();
        }
      }
      @Override
      public boolean release(@NonNull T element){
        synchronized(mLock){
          return super.release(element);
        }
      }
    }
  }
```
DiskLruCache:每个缓存都有一个字符串键和固定数量的值。每个key必须匹配正则表达式[a-z0-9_-]{1,120}。多进程同时使用同一个缓存目录则回报错。
```java
  public final class DiskLruCache implements Closeable{
    static final String JOURNAL_FILE="journal";
    static final String JOURNAL_FILE_TEMP="journal.tmp";
    static final String JOURNAL_FILE_BACKUP="journal.bkp";
    static final String MAGIC="libcore.io.DiskLruCache";
    static final String VERSION_1="1";
    static final long ANY_SEQUENCE_NUMBER=-1;
    private static final String CLEAN="CLEAN";
    private static final String DIRTY="DIRTY";
    private static final String REMOVE="REMOVE";
    private static final String READ="READ";
    
    private final File directory;
    private final File journalFile;
    private final File journalFileTmp;
    private final File journalFileBackup;
    private final int appVersion;
    private long maxSize;
    private final int valueCount;
    private long size=0;
    private Writer journalWriter;
    private final LinkedHashMap<String,Entry> lruEntries=new LinkedHashMap<>(0,0.75f,true);
    private int redundantOpCount;
    private long nextSequenceNumber=0;
    
    final ThreadPoolExecutor executorService=new ThreadPoolExecutor(0,1,60L,TimeUnit.SECONDS,new LinkedBlockingQueue<Runnable>(),new DiskLruCacheThreadFactory());
    private final Callable<Void> cleanupCallable= new Callable<Void>(){
      public Void call() throws Exception{
        synchronized(DiskLruCache.this){
          if(journalWriter==null){
            return null;
          }
          trimToSize();
          if(journalRebuildRequired()){
            rebuildJournal();
            redundantOpCount=0;
          }
        }
        return null;
      }
    };
    private DiskLruCache(File directory,int appVersion,int valueCount,long maxSize){
      this.directory=directory;
      this.appVersion=appVersion;
      this.journalFile=new File(directory,JOURNAL_FILE);
      this.journalFileTmp=new File(directory,JOURNAL_FILE_TEMP);
      this.journalFileBackup=new File(directory,JOURNAL_FILE_BACKUP);
      this.valueCount=valueCount;
      this.maxSize=maxSize;
    }
    public static DiskLruCache open(File directory,int appVersion,int valueCount,long maxSize) throws IOException{
      if(maxSize<=0){
        throw new IllegalArgumentException("maxSize<=0");
      }
      if(valueCount<=0){
        throw new IllegalArgumentException("valueCount<=0");
      }
      File backupFile=new File(directory,JOURNAL_FILE_BACKUP);
      if(backupFile.exists()){
        File journalFile=new File(directory,JOURNAL_FILE);
        if(journalFile.exists()){
          backupFile.delete();
        }else{
          renameTo(backupFile,journalFile,false);
        }
      }
      DiskLruCache cache=new DiskLruCache(directory,appVersion,valueCount,maxSize);
      if(cache.journalFile.exists()){
        try{
          cache.readJournal();
          cache.processJournal();
          return cache;
        }catch(IOException journalIsCorrupt){
          cache.delete();
        }
      }
      directory.mkdirs();
      cache=new DiskLruCache(directory,appVersion,valueCount,maxSize);
      cache.rebuildJournal();
      return cache;
    }
    private void readJournal()throws IOException{
      StrictLineReader reader=new StrictLineReader(new FileInputStream(journalFile),Util.US_ASCII);
      try{
        String magic=reader.readLine();
        String version=reader.readLine();
        String appVersionString=reader.readLine();
        String valueCountString=reader.readLine();
        String blank=reader.readLine();
        if(!MAGIC.equals(magic)||!VERSION_1.equals(version)||!Integer.toString(appVersion).equals(appVersionString)||!Integer.toString(valueCount).equals(valueCountString)||!"".equals(blank)){
          throw new IOException("unexpected journal header: ["+magic+", "+version+", "+valueCountString+", "+black+"]");
        }
        int lineCount=0;
        while(true){
          try{
            readJournalLine(reader.readLine);
            lineCount++;
          }catch(EOFException endOfJournal){
            break;
          }
        }
        redundantOpCount=lineCount-lruEntries.size();
        if(reader.hasUnterminatedLine()){
          rebuildJournal();
        }else{
          journalWriter=new BufferedWriter(new OutputStreamWriter(new FileOutputStream(journalFile,true),Util.US_ASCII));
        }
      }finally{
        Util.cloesQuietly(reader);
      }
    }
    private void readJournalLine(String line)throws IOException{
      int firstSpace=line.indexOf(" ");
      if(firstSpace==-1){
        throw new Exception("unexpected journal line: "+line);
      }
      int keyBegin=firstSpace+1;
      int secondSpace=line.indexOf(" ",keyBegin);
      final String key;
      if(secondSpace==-1){
        key=line.substring(keyBegin);
        if(firstSpace==REMOVE.length()&&line.statsWith(REMOVE)){
          lruEntries.remove(key);
          retrun;
        }
      }else{
        key=line.substring(keyBegin,secondSpace);
      }
      Entry entry=lruEntries.get(key);
      if(entry==null){
        entry=new Entry(key);
        lruEntries.put(key,entry);
      }
      if(secondSpace!=-1&&firstSpace==CLEAN.length()&&line.startsWith(CLEAN)){
        String[] parts=line.substring(secondSpace+1).split(" ");
        entry.readable=true;
        entry.currentEditor=null;
        entry.setLengths(parts);
      }else if(secondSpace==-1&&firstSpace==DIRTY.length()&&line.startsWith(DIRTY)){
        entry.currentEditor=new Editor(entry);
      }else if(secondSpace==-1&&firstSpace==READ.length()&&line.startsWith(READ)){
        
      }else{
        throw new IOException("unexpected journal line: "+line);
      }
    }
    private void processJournal() throws IOException{
      deleteIfExists(journalFileTmp);
      for(Iterator<Entry> i =lruEntries.values().iterator();i.hasNext();){
        Entry entry=i.next();
        if(entry.currentEditor==null){
          for(int t=0;t<valueCount;t++){
            size+=entry.lengths[t]
          }
        }else{
          entry.currentEditor=null;
          for(int t=0;t<valueCount;t++){
            deleteIfExists(entry.getCleanFile(t));
            deleteIfExists(entry.getDirtyFile(t));
          }
          i.remove();
        }
      }
    }
    private synchronized void rebuildJournal() throws IOException{
      if(journalWriter!=null){
        journalWriter.close();
      }
      Writer writer=new BufferedWriter(new OutputStreamWriter(new FileOutputStream(journalFileTmp),Util.US_ASCII));
      try{
        writer.write(MAGIC);
        writer.write("\n");
        writer.write(VERSION_1);
        writer.write("\n");
        writer.write(Integer.toString(appVersion));
        writer.write("\n");
        writer.write(Integer.toString(valueCount));
        writer.write("\n");
        writer.write("\n");
        for(Entry entry:lruEntries.values()){
          if(entry.currentEditor!=null){
            writer.write(DIRTY+" "+entry.key+"\n");
          }else{
            writer.write(CLEAN+" "+entry.key+entry.getLengths()+"\n");
          }
        }
      }finally{
        writer.close();
      }
      if(journalFile.exists()){
        renameTo(journalFile,journalFileBackup,true);
      }
      renameTo(journalFileTmp,journalFile,false);
      journalFileBackup.delete();
      journalWriter=new BufferredWriter(new OutputStreamWriter(new FileOutputStream(journalFile,true),Util.US_ASCII));
    }
    private static void deleteIfExists(File file) throws IOException{
      if(file.exists()&&!file.delete()){
        throw new IOException();
      }
    }
    private static void renameTo(File from,File to,boolean deleteDestination) throws IOException{ 
      if(deleteDestination){
        deleteIfExists(to);
      }
      if(!from.renameTo(to)){
        throw new IOException();
      }
    }
    public synchronized Value get(String key) throws IOException{
      checkNotClosed();
      Entry entry=lruEntries.get(key);
      if(entry==null){
        return null;
      }
      if(!entry.readable){
        return null;
      }
      for(File file:entry.cleanFiles){
        if(!file.exists()){
          return null;
        }
      }
      redundantOpCount++;
      journalWriter.append(READ);
      journalWriter.append(" ");
      journalWriter.append(key);
      journalWriter.append("\n");
      if(journalRebuildRequired()){
        executorService.submit(cleanupCallable);
      }
      return new Value(key,entry.sequenceNumber,entry.cleamFiles,entry.lengths);
    }
    public Editor edit(String key) throws IOException{
      return edit(key,ANY_SEQUENCE_NUMBER);
    }
    private synchronized Editor edit(String key,long expectedSequenceNumber) throws IOException{
      checkNotCloased();
      Entry entry=lruEntries.get(key);
      if(expectedSequenceNumber!=ANY_SEQUENCE_NUMBER&&(entry==null||entry.sequenceNumber!=expectedSequenceNumber)){
        return null;
      }
      if(entry==null){
        entry=new Entry(key);
        lruEntries.put(key,entry);
      }else if(entry.currentEditor!=null){
        return null;
      }
      Editor editor=new Editor(entry);
      entry.currentEditor=editor;
      journalWriter.append(DIRTY);
      journalWriter.append(" ");
      journalWriter.append(key);
      journalWriter.append("\n");
      journalWriter.flush();
      return editor;
    }
    public File getDirectory(){
      return directory;
    }
    public synchronized long getMaxSize(){
      return maxSize;
    }
    public synchronized void setMaxSize(long maxSize){
      this.maxSize=maxSize;
      executorService.submit(cleanupCallable);
    }
    public synchronized long size(){
      return size;
    }
    private synchronized void completeEdit(Editor editor,boolean success) throws IOException{
      Entry enty=editor.entry;
      if(entry.currentEditor!=editor){
        throw new IllegalStateException();
      }
      if(success&&!entry.readable){
        for(int i=0;i<valueCount;i++){
          if(!editor.written[i]){
            editor.abort();
            throw new IllegalStateException("Newly created entry did not create value for index "+i);
          }
          if(!entry..getDirtyFile(i).exists()){
            editor.abort();
            return;
          }
        }
      }
      for(int i=0;i<valueCount;i++){
        File dirty=entry.getDirtyFile(i);
        if(success){
          if(dirty.exists()){
            File clean=entry.getCleanFile(i);
            dirty.renameTo(clean);
            long oldLength=entry.lengths[i];
            long newLength=clean.length();
            entry.lengths[i]=newLength;
            size=size-oldLength+newLength;
          }
        }else{
          deleteIfExists(dirty);
        }
      }
      redundantOpCount++;
      entry.currentEditor=null;
      if(entry.readable|success){
        entry.readable=true;
        journalWriter.append(CLEAN);
        journalWriter.append(" ");
        journalWriter.append(entry.key);
        journalWriter.append(entry.getLengths());
        journalWriter.append("\n");
        if(success){
          entry.sequenceNumber=nextSequenceNumber++;
        }
      }else{
        lruEntries.remove(entry.key);
        journalWriter.append(REMOVE);
        journalWriter.append(" ");
        journalWriter.append(entry.key);
        journalWriter.append("\n");
      }
      journalWriter.flush();
      if(size>maxSize||journalRebuildRequired()){
        executorService.submit(cleanupCallable);
      }
    }
    private boolean journalRebuildRequired(){
      final int redundantOpCompactThreshold=2000;
      return redundantOpCount>=redundantOpCompactThreshold&&redundantOpCount>=lruEntries.size();
    }
    public synchronized boolean remove(String key) throws IOException{
      checkNotCloased();
      Entry entry=lruEntries.get(key);
      if(entry==null||entry.currentEditor!=null){
        return false;
      }
      for(int i=0;i<valueCount;i++){
        File file=entry.getCleanFile(i);
        if(file.exists()&&!file.delete()){
          throw new IOException("failed to delete "+file);
        }
        size-=entry.lengths[i];
        entry.lengths[i]=0;
      }
      redundantOpCount++;
      journalWriter.append(REMOVE);
      journalWriter.append(" ");
      journalWriter.append(key);
      journalWriter.append("\n");
      lruEntries.remove(key);
      if(journalRebuildRequired()){
        executorService.submit(cleanupCallable);
      }
      return true;
    }
    public synchronized boolean isClosed(){
      return journalWriter==null;
    }
    private void checkNotClosed(){
      if(journalWriter==null){
        throw new IllegalStateException("cache is closed");
      }
    }
    public synchronized void flush() throws IOException{
      chekcNotClosed();
      trimToSize();
      journalWriter.flush();
    }
    public synchronized void close() throws IOException{
      if(journalWriter ==null){
        return;
      }
      for(Entry entry:new ArrayList<Entry>(lruEntries.values())){
        if(entry.currentEditor!=null){
          entry.currentEditor.abort();
        }
      }
      trimToSize();
      journalWriter.close();
      journalWriter=null;
    }
    private void trimToSize() throws IOException{
      while(size>maxSize){
        Map.Entry<String,Entry> toEvict=lruEntries.entrySet().iterator().next();
        remove(toEvict.getKey());
      }
    }
    public void delete() throws IOException{
      close();
      Util.deleteContents(directory);
    }
    private static String inputStreamToString(InputStream in) throws IOException{
      return Util.readFully(new InputStreamReader(in,Util.UTF_8));
    }
    public final class Value{
      private final String key;
      private final long sequenceNumber;
      private final long[] lengths;
      private final File[] files;
      private Value(String key,long sequenceNumber,File[] files,long[] lengths){
       this.key=key;
       this.sequenceNumber=sequenceNumber;
       this.files=files;
       this.lengths=lengths;
      }
      public Editor edit()throws IOException{
        return DiskLruCache.this.edit(key,sequenceNumber);
      }
      public File getFile(int index){
        return files[index];
      }
      public String getString(int index)throws IOException{
        InputStream is=new FileInputStream(files[index]);
        return inputStreamToString(is);
      }
      public long getLength(int index){
        return lengths[index];
      }
    }
    public final class Editor{
      private final Entry entry;
      private final boolean[] written;
      private boolean committed;
      private Editor(Entry entry){
        this.entry=entry;
        this.written=(entry.readable)?null:new boolean[valueCount];
      }
      private InputStream newInputStream(int index) throws IOException{
        synchronized(DiskLruCache.this){
          if(entry.currentEditor!=this){
            throw new IllegalStateException();
          }
          if(!entry.readable){
            return null;
          }
          try{
            return new FileInputStream(entry.getCleanFile(index));
          }catch(FileNotFoundException e){
            return null;
          }
        }
      }
      public String getString(int index) throws IOException{
        InputStream in = newInputStream(index);
        return in!=null?inputStreamToString(in):null;
      }
      public File getFile(int index) throws IOException{
        synchronized(DiskLruCache.this){
          if(entry.currentEditor!=this){
            throw new IllegalStateException();
          }
          if(!entry.readable){
            written[index]=true;
          }
          File dirtyFile=entry.getDirtyFile(index);
          if(!directory.exists()){
            directory.mkdirs();
          }
          return dirtyFile;
        }
      }
      public void set(int index,String value) throws IOException{
        Writer writer=null;
        try{
          OutputStream os=new FileOutputStream(getFile(index));
          writer=new OutputStreamWriter(os,Util.UTF_8);
          writer.write(value);
        }finally{
          Util.closeQuietly(writer);
        }
      }
      public void commit() throws IOException{
        completeEdit(this,true);
        committed=true;
      }
      public void abort() throws IOException{
        completeEdit(this,false);
      }
      public void abortUnlessCommited(){
        if(!committed){
          try{
            abort();
          }catch(IOException ignored){
            
          }
        }
      }
    }
    private final class Entry{
      private final String key;
      private final long[] lengths;
      File[] cleanFiles;
      File[] dirtyFiles;
      private boolean readable;
      private Editor currentEditor;
      private long sequenceNumber;
      private Entry(String key){
        this.key=key;
        this.lengths=new long[valueCount];
        cleanFiles=new File[valueCount];
        dirtyFiles=new File[valueCount];
        StringBuilder fileBuilder=new StringBuilder(key).append(" ");
        int truncateTo=fileBuilder.length();
        for(int i=0;i<valueCount;i++){
          fileBuilder.append(i);
          cleanFiles[i]=new File(directory,fileBuilder.toString());
          fileBuilder.append(".tmp");
          dirtyFiles[i]=new File(directory,fileBuilder.toString());
          fileBuilder.setLength(truncateTo);
        }
      }
      public String getLengths() throws IOException{
        StringBuilder result=new StringBuilder();
        for(long size:lengths){
          result.append(" ").append(size);
        }
        return result.toString();
      }
      private void setLengths(String[] strings) throws IOException{
        if(strings.length!=valueCount){
          throw invalidLengths(strings);
        }
        try{
          for(int i=0;i<strings.length;i++){
            lengths[i]=Long.parseLong(strings[i]);
          }
        }catch(NumberFormatException e){
          throw invalidLengths(strings);
        }
      }
      private IOException invalidLengths(Strings[] strings) throws IOException{
        throw new IOException("unexpected journal line: "+java.util.Arrays.toString(strings));
      }
      public File getCleanFile(int i){
        return cleanFiles[i];
      }
      public File getDirtyFile(int i){
        return dirtyFiles[i];
      }
    }
    private static final class DiskLruCacheThreadFactory implements ThreadFactory{
      @Override
      public synchronized Thread newThread(Runnable runnable){
        Thread result=new Thread(runnable,"glide-disk-lru-cache-thread");
        result.setPriority(Thread.MIN_PRIORITY);
        return result;
      }
    }
  }
```
StrictLineReader:用于读取严格有行组成的输入，比如：基于行缓存条目或缓存日志，该类使用不同的输入结束报告和更严格的行定义。
```java
  class StrictLineReader implements Closeable{
    private static final byte CR=(byte)"\r";
    private static final byte LF=(byte)"\n";
    private final InputStream in;
    private final Charset charset;
    private byte[] buf;
    private int pos;
    private int end;
    public StrictLineReader(InputStream in,Charset charset){
      this(in,8*1024,charset);
    }
    public StrictLineReader(InputStream in,int capacity,Charset charset){
      if(in==null||charset==null){
        throw new NullPointerException();
      }
      if(capacity<0){
        throw new IllegalArgumentException("capacity <=0");
      }
      if(!(charset.equals(Util.US_ASCII))){
        throw new IllegalArgumentException("Unsupported encoding");
      }
      this.in=in;
      this.charset=charset;
      buf=new byte[capacity];
    }
    public void close() throws IOException{
      synchronized(in){
        if(buf!=null){
          buf=null;
          in.close();
        }
      }
    }
    public String readLine() throws IOException{
      synchronized(in){
        if(buf==null){
          throw new IOException("LineReader is closed");
        }
        if(pos>=end){
          fillBuf();
        }
        for(int i=pos;i!=end;++i){
          if(buf[i]==LF){
            int lineEnd=(i!=pos&&buf[i-1]==CR)?i-1:i;
            String res=new String(buf,pos,lineEnd-pos,charset.name());
            pos=i+1;
            return res;
          }
        }
        ByteArrayOutputStream out =new ByteArrayOutputStream(end-pos+80){
          @Override
          public String toString(){
            int length=(count>0&&buf[count-1]==CR)?count-1:count;
            try{
              return new String(buf,0,length,charset.name());
            }catch(UnsupportedEncodingException e){
              throw new AssertionError(e);
            }
          }
        };
        while(true){
          out.write(buf,pos,end-pos);
          end=-1;
          fillBuf();
          for(int i=pos;i!=end;++i){
            if(buf[i]==LF){
              if(i!=pos){
                out.write(buf,pos,i-pos);
              }
              pos=i+1;
              return out.toString();
            }
          }
        }
      }
    }
    public boolean hasUnterminatedLine(){
      return end==-1;
    }
    private void fillBuf() throws IOException{
      int result=in.read(buf,0,buf.length);
      if(result==-1){
        throw new EOFException();
      }
      pos=0;
      end=result;
    }
  }
```
## Engine
负责开启加载和管理活动资源和缓存资源
```java
  public class Engine implements EngineJobListener,MemoryCache.ResourceRemovedListener,EngineResource.ResourceListener{
    private static final String TAG="Engine";
    private static final int JOB_POOL_SIZE=150;
    private static fiinal boolean VERBOSE_IS_LOGGABLE=Log.isLoggable(TAG,Log.VERBOSSE);
    private final Jobs jobs;
    private final EngineKeyFactory keyFactory;
    private final MemoryCache cache;
    private final EngineJobFactory engineJobFactory;
    private final ResourceRecycler resourceRecycler;
    private final LazyDiskCacheProvider diskCacheProvider;
    private final DecodeJobFactory decodeJobFactory;
    private final ActiveResources activeResources;
    public Engine(MemoryCache memoryCache,DiskCache.Factory diskCacheFactory,GlideExecutor diskCacheExecutor,GlideExecutor sourceExecutor,GlideExecutor sourceUnlimitedExecutor,GlideExecutor animationExecutor,boolean isActiveResourceRetentionAllowed){
      this(memoryCache,diskCacheFactory,diskCacheExecutor,sourceExecutor,sourceUnlimiteExecutor,animationExecutor,null,null,null,null,null,null,isActiveResourceRetentionAllowed);
    }
    Engine(MemoryCache cache,DiskCache.Factory diskCacheFactory,GlideExecutor diskCacheExecutor,GlideExecutor sourceExecutor,GlideExecutor sourceUnlimitedExecutor,GlideExecutor animationExecutor,Jobs jobs,EngineKeyFactory keyFactory,ActiveResources activeResources,EngineJobFactory engineJobFactory,DecodeJobFactory decodeJobFactory,ResourceRecycler resourceRecycler,boolean isActiveResourceRetetionAllowed){
      this.cache=cache;
      this.diskCacheProvider=new LazyDiskCacheProvider(diskCacheFactory);
      if(activeResources==null){
        activeResources=new ActiveResources(isActiveResourceRetentionAllowed);
      }
      this.activeResources=activeResources;
      activeResources.setListener(this);
      if(keyFactory==null){
        keyFactory=new EngineKeyFactory();
      }
      this.keyFactory=keyFactory;
      if(jobs==null){
        jobs=new Jobs();
      }
      this.jobs=jobs;
      if(engineJobFactory==null){
        engineJobFactory=new EngineJobFactory(diskCacheExecutor,sourceExecutor,sourceUnlimitedExecutor,animationExecutor,this);
      }
      this.engineJobFactory=engineJobFactory;
      if(decodeJobFactory==null){
        decodeJobFactory=new DecodeJobFactory(diskCacheProvider);
      }
      this.decodeJobFactory=decodeJobFactory;
      if(resourceRecycler==null){
        resourceRecycler=new ResourceRecycler();
      }
      this.resourceRecycler=resourceRecycler;
      cache.setResourceRemovedListener(this);
    }
    public <R> LoadStatus load(GlideContext glideContext,Object model,Key signature,int width,int height,Class<?> resourceClass,Class<R> transcodeClass,Priority priority,DiskCacheStrategy diskCacheStrategy,Map<Class<?>,Transformation<?>> transformations,boolean isTransformationRequired,boolean isScaleOnlyOrNoTransform,Options options,boolean isMemoryCacheable,boolean isScaleOnlyOrNoTransform,Options options,boolean isMemoryCachealbe,boolean useUnlimitedSourceExecutorPool,boolean useAnimationPool,boolean onlyRetrieveFromCache,ResourceCallback cb){
      Util.assertMainThread();
      long startTime=VERBOSE_IS_LOGGABLE?LogTime.getLogTime():0;
      EngineKey key=keyFactory.buildKey(model,signature,width,height,transformations,resourceClass,transcodeClass,options);
      EngineResource<?> active=loadFromActiveResources(key,isMemoryCacheable);
      if(active!=null){
        cb.onResourceReady(active,DataSource.MEMORY_CACHE);
        return null;
      }
      EngineResource<?> cached=loadFromCache(key,isMemoryCacheable);
      if(cached!=null){
        cb.onResourceReady(cached,DataSource.MEMORY_CACHE);
        return null;
      }
      EngineJob<?> current=jobs.get(key,onlyRetrieveFromCache);
      if(current!=null){
        current.addCallback(cb);
        return new LoadStatus(cb,current);
      }
      EngineJob<R> engineJob=engineJobFactory.build(key,isMemoryCacheable,useUnlimitedSourceExecutorPool,useAnimationPool,onleyRetrieveFromCache);
      DecodeJob<R> decodeJob=decodeJobFactory.build(glideContext,model,key,signature,width,height,resourceClass,transcodeClass,priority,diskCacheStrategy,transformations,isTransformationRequired,isScaleOnlyOrNoTransform,onlyRetrieveFromCache,options,engineJob);
      jobs.put(key,engineJob);
      engineJob.addCallback(cb);
      engineJob.start(decodeJob);
      return new LoadStatus(cb,engineJob);
    }
    private EngineResource<?> laodFromActiveResources(Key key,boolean isMemoryCacheable){
      if(!isMemoryCacheable){
        return null;
      }
      EngineResource<?> active=activeResources.get(key);
      if(active!=null){
        active.acquire();
      }
      return active;
    }
    private EngineResource<?> loadFromCache(Key key,boolean isMemoryCacheable){
      if(!isMemoryCacheable){
        return null;
      }
      EngineResource<?> cached=getEngineResourceFromCache(key);
      if(cache!=null){
        cache.acquire();
        activeResources.activate(key,cached);
      }
      return cached;
    }
    private EngineResource<?> getEngineResourceFromCache(Key key){
      Resource<?> cached=cache.remove(key);
      final EngineResource<?> result;
      if(cache==null){
        result=null;
      }else if(cached instanceof EngineResource){
        result=(EngineResource<?>) cached;
      }else{
        result=new EnigneResource<>(cached,true,true);
      }
      return result;
    }
    public void release(Resource<?> resource){
      Util.assertMainThread();
      if(resource instanceof EngineResource){
        ((EngineResource<?>)resource).release();
      }else{
        throw new IllegalArgumentException("Cannot release anything but an EngineResource");
      }
    }
    @Override
    public void onEngineJobComplete(EngineJob<?> engineJob,Key key,EngineResource<?> resource){
      Util.assertMainThread();
      if(resource!=null){
        resource.setResourceListener(key,this);
        if(resource.isCacheable()){
          activeResources.activate(key,resource);
        }
      }
      jobs.removeIfCurrent(key,engineJob);
    }
    @Override
    public void onEngineJobCancelled(EngineJob<?> engineJob,Key key){
      Util.assertMainThread();
      jobs.removeIfCurrent(key,engineJob);
    }
    @Override
    public void onResourceRemoved(@NonNull final Resource<?> resource){
      Util.assertMainThread();
      resourceRecycler.recycler(resource);
    }
    @Override
    public void onResourceReleased(Key cacheKey,EngineResource<?> resource){
      Util.assertMainThread();
      activeResources.deactivate(cacheKey);
      if(resource.isCacheable()){
        cache.put(cacheKey,resource);
      }else{
        resourceRecyler.recycle(resource);
      }
    }
    public void clearDiskCache(){
      diskCacheProvider.getDiskCache().clear();
    }
    public void shutdown(){
      engineJobFactory.shutdown();
      diskCacheProvider.clearDiskCacheIfCreated();
      activeResources.shutdown();
    }
    public static class LoadStatus{
      private final EngineJob<?> engineJob;
      private final ResourceCallback cb;
      LoadStatus(ResourceCallback cb,EngineJob<?> engineJob){
        this.cb=cb;
        this.engineJob=engineJob;
      }
      public void cancel(){
        engineJob.removeCallback(cb);
      }
    }
    private static class LazyDiskCacheProvider implements DecodeJob.DiskCacheProvider{
      private final DiskCache.Factory factory;
      private volatile DiskCache diskCache;
      LazyDiskCacheProvider(DiskCache.Factory factory){
        this.factory=factory;
      }
      synchronized void clearDiskCacheIfCreated(){
        if(diskCache==null){
          return;
        }
        diskCache.clear();
      }
      @Override
      public DiskCache getDiskCache(){
        if(diskCache==null){
          synchronized(this){
            if(diskCache==null){
              diskCache=factory.build();
            }
            if(diskCache==null){
              diskCache=new DiskCacheAdapter();
            }
          }
        }
        return diskCache;
      }
    }
    static class DecodeJobFactory{
      final DecodeJob.DiskCacheProvider diskCacheProvider;
      final Pools.Pool<DecodeJob<?>> pool=FactoryPools.simple(JOB_POOL_SIZE,new FactoryPools.Factory<DecodeJob<?>>(){
        @Override
        public DecodeJob<?> create(){
          return new DecodeJob<>(diskCacheProvider,pool);
        }
      });
      private int creationOrder;
      DecodeJobFactory(DecodeJob.DiskCacheProvder diskCacheProvider){
        this.diskCacheProvider=diskCacheProvider;
      }
      <R> DecodeJob<R> build(GlideContext glideContext,Object model,EngineKey loadKey,Key signature,int width,int height,Class<?> resourceClass,Class<R> transcodeClass,Priority priority,DiskCacheStrategy diskCacheStrategy,Map<Class<?>,Transformation<?>> transformations,boolean isTransformationRequired,boolean isScaleOnlyOrNoTransform,boolean onlyRetrieveFromCache,Options options,DecodeJob.Callback<R> callback){
        DecodeJob<R> result=Preconditions.checkNotNull((DecodeJob<R>) pool.acquire());
        return result.init(glideContext,model,loadKey,signature,width,height,resourceClass,transcodeClass,priority,diskCacheStrategy,transformations,isTransformationRequired,isScaleOnlyOrNoTransform,onlyRetrieveFromCache,options,callback,createOrder++);
      }
    }
    static class EngineJobFactory{
      final GlideExecutor diskCacheExecutor;
      final GlideExecutor sourceExecutor;
      final GlideExecutor sourceUnlimitedExecutor;
      final GlideExecutor animationExecutor;
      final EngineJobListener listener;
      final Pools.Pool<EngineJob<?>> pool=FactoryPools.simple(JOB_POOL_SIZE,new FactoryPools.Factory<EngineJob<?>>(){
        @Override
        public EngineJob<?> create(){
          return new EngineJob<>(diskCacheExecutor,sourceExecutor,sourceUnlimitedExecutor,animationExecutor,listener,pool);
        }
      });
      EngineJobFactory(GlideExecutor diskCacheExecutor,GlideExecutor sourceExecutor,GlideExecutor sourceUnlimitedExecutor,GlideExecutor animationExecutor,EngineJobLisenter listener){
        this.diskCacheExecutor=diskCacheExecutor;
        this.sourceExecutor=sourceExecutor;
        this.sourceUnlimitedExecutor=sourceUnlimitedExecutor;
        this.animationExecutor=animationExecutor;
        this.listener=listener;
      }
      void shutdown(){
        shutdownAndAwaitTermination(diskCacheExecutor);
        shutdownAndAwaitTermination(sourceExecutor);
        shutdownAndAwaitTermination(sourceUnlimitedExecutor);
        shutdownAndAwaitTermination(animationExecutor);
      }
      <R> EngineJob<R> build(Key key,
        boolean isMemoryCacheable,
        boolean useUnlimitedSourceGeneratorPool,
        boolean useAnimationPool,
        boolean onlyRetrieveFromCahce){
        EngineJob<R> result=Preconditions.checkNotNull((EngineJob<R>)pool.acquire());
        return result.init(key,isMemoryCacheable,useUnlimitedSourceGeneratorPool,useAnimationPool,onlyRetrieveFromCache);
      }
      private static void shutdownAndAwaitTermination(ExecutorService pool){
        long shutdownSeconds=5;
        pool.shutdown();
        try{
          if(!pool.awaitTermination(shutdownSeconds,TimeUnit.SECONDS)){
            pool.shutdownNow();
            if(!pool.awaitTermination(shutdownSeconds,TimeUnit.SECONDS)){
              throw new RuntimeException("Failed to shutdown");
            }
          }
        }catch(InterruptedException e){
          throw new RuntimeException(e);
        }
      }
    }
  }
```
Jobs:
```java
  final class Jobs{
    private final Map<Key,EngineJob<?>> jobs=new HashMap<>();
    private final Map<Key,EngineJob<?>> onlyCacheJobs=new HashMap<>();
    Map<Key,EngineJob<?>> getAll(){
      return Collections.unmodifiableMap(jobs);
    }
    EngineJob<?> get(Key key,boolean onlyRetrieveFromCache){
      return getJobMap(onlyRetrieveFromCache).get(key);
    }
    void put(Key key,EngineJob<?> job){
      getJobMap(job.onlyRetrieveFromCache()).put(key,job);
    }
    void removeIfCurrent(Key key,EngineJob<?> expected){
      Map<Key,EngineJob<?>> jobMap=getJobMap(expected.onlyRetrieveFromCache);
      if(expected.equals(jobMap.get(key))){
        jobMap.remove(key);
      }
    }
    private Map<Key,EngineJob<?>> getJobMap(boolean onlyRetrieveFromCache){
      return onlyRetrieveFromCache?onlyCacheJobs:jobs;
    }
  }
```
EngineJob:为加载添加回调和移除回调，当加载完成后通知回调
```java
  class EngineJob<R> implements DecodeJob.Callback<R>,Poolable{
    private static final EngineResourceFactory DEFAULT_FACTORY=new EngineResourceFactory();
    private static final Handler MAIN_THREAD_HANDLER=new Handler(Looper.getMainLooper,new MainThreadCallback());
    private static final int MSG_COMPLETE=1;
    private static final int MSG_EXCEPTIOIN=2;
    private static final int MSG_CHACELLED=3;
    private final List<ResourceCallback> cbs=new ArrayList<>(2);
    private final StateVerifier stateVerifier=StateVerifier.newInstance();
    private final Pools.Pool<EngineJob<?>> pool;
    private final EngineResourceFactory engineResourceFactory;
    private final EngineJobListener listener;
    private final GlideExecutor diskCacheExecutor;
    private final GlideExecutor sourceExecutor;
    private final GlideExecutor sourceUnlimitedExecutor;
    private final GlideExecutor animationExecutor;
    private Key key;
    private boolean isCacheable;
    private boolean useUnlimitedSourceGeneratorPool;
    private boolean useAnimationPool;
    private boolean onlyRetrieveFromCache;
    private Resource<?> resource;
    private DataSource dataSource;
    private boolean hasResource;
    private GlideException exception;
    private boolean hasLoadFailed;
    private List<ResourceCallback> ignoreCallbacks;
    private EngineResource<?> engineResource;
    private DecodeJob<R> decodeJob;
    private volatile boolean isCancelled;
    EngineJob(GlideExecutor diskCacheExecutor,GlideExecutor sourceExecutor,GlideExecutor sourceUnlimitedExecutor,EngineJobListener listener,Pools.Pool<EngineJob<?>> pool){
      this(diskCacheExecutor,sourceExecutor,sourceUnlimitedExecutor,animationExecutor,listener,pool,DEFAULT_FACTORY);
    }
    EngineJob(GlideExecutor diskCacheExecutor,GlideExecutor sourceExecutor,GlideExecutor sourceUnlimitedExecutor,GlideExecutor animationExecutor,EngineJobListener listener,Pools.Pool<EngineJob<?>> pool,EngineResourceFactory engineResourceFactory){
      this.diskCacheExecutor=diskCacheExecutor;
      this.sourceExecutor=sourceExecutor;
      this.sourceUnlimitedExecutor=sourceUnlimitedExecutor;
      this.animationExecutor=animationExecutor;
      this.listener=listener;
      this.pool=pool;
      this.engineResourceFactory=engineResourceFactory;
    }
    EngineJob<R> init(Key key,boolean isCacheable,boolean useUnlinitedSourceGeneratorPool,boolean useAnimationPool,boolean onlyRetrieveFromCache){
      this.key=key;
      this.isCacheable=isCacheable;
      this.useUnlimitedSourceGeneratorPool=useUnlimitedSourceGeneratorPool;
      this.useAnimationPool=useAnimationPool;
      this.onlyRetrieveFromCache=onlyRetrieveFromCache;
      return this;
    }
    public void start(DecodeJob<R> decodeJob){
      this.decodeJob=decodeJob;
      GlideExecutor executor=decodeJob.willDecodeFromCache()?diskCacheExecutor:getActiveSourceExecutor();
      executor.execute(decodeJob);
    }
    void addCallback(ResourceCallback cb){
      Util.assertMainThread();
      stateVerifier.throwIfRecycled();
      if(hasResource){
        cb.onResourceReady(engineResource,dataSource);
      }else if(hasLoadFailed){
        cb.onLoadFailed(exception);
      }else{
        cbs.add(cb);
      }
    }
    void removeCallback(ResourceCallback cb){
      Util.assertMainThread();
      stateVerifier.throwIfRecycled();
      if(hasResource||hasLoadFailed){
        addIgnoredCallback(cb);
      }else{
        cbs.remove(cb);
        if(cbs.isEmpty()){
          cancel();
        }
      }
    }
    boolean onlyRetrieveFromCache(){
      return onlyRetrieveFromCache;
    }
    private GlideExecutor getActiveSourceExecutor(){
      return useUnlimitedSourceGeneratorPool?sourceUnlimitedExecutor:(useAnimationPool?animationExecutor:sourceExecutor);
    }
    private void addIgnoredCallback(ResourceCallback cb){
      if(ignoredCallbacks==null){
        ignoredCallbacks=new ArrayList<>(2);
      }
      if(!ignoredCallbacks.contains(cb)){
        ignoredCallbacks.add(cb);
      }
    }
    private boolean isInIgnoredCallbacks(ResourceCallback cb){
      return ignoredCallbacks!=null&&ignoredCallbacks.contains(cb);
    }
    void cancel(){
      if(hasLoadFailed||hasResource||isCancelled){
        return ;
      }
      isCancelled=true;
      decodeJob.cancel();
      listener.onEngineJobCancelled(this,key);
    }
    boolean isCancelled(){
      return isCancelled;
    }
    void handleResultOnMainThread(){
      stateVerifier.throwIfRecycled();
      if(isCancelled){
        resource.recycle();
        release(false);
        return;
      }else if(cbs.isEmpty()){
        throw new IllegalStateException("Received a resource without any callbacks to notify");
      }else if(hasResource){
        throw new IllegalStateException("Already have resource");
      }
      engineResource=engineResouceFactory.build(resource,isCacheable);\
      hasResource=true;
      engineResource.acquire();
      listener.onEngineJobComplete(this,key,engineResource);
      for(int i=0,size=cbs.size();i<size;i++){
        ResourceCallback cb=cbs.get(i);
        if(!isIngnoredCallbacks(cb)){
          engineResource.acquire();
          cb.onResourceReady(engineResource,dataSource);
        }
      }
      engineResource.release();
      release(false);
    }
    void handleCancelledOnMainThread(){
      stateVerifier.throwIfRecycled();
      if(!isCancelled){
        throw new IllegalStateException("Not cancelled");
      }
      listener.onEngineJobCancelled(this,key);
      release(false);
    }
    private void release(boolean isRemoveFromQueue){
      Util.assertMainThread();
      cbs.clear();
      key=null;
      engineResource=null;
      resource=null;
      if(ignoredCallbacks!=null){
        ignoredCallbacks.clear();
      }
      hasLoadFailed=false;
      isCancelled=false;
      hasResource=false;
      decodeJob.release(isRemovedFromQueue);
      decodeJob=null;
      exception=null;
      dataSource=null;
      pool.release(this);
    }
    @Override
    public void onResourceReady(Resource<R> resource,DataSource dataSource){
      this.resource=resource;
      this.dataSource=dataSource;
      MAIN_THREA_HANDLER.obtainMessage(MSG_COMPLETE,this).sendToTarger();
    }
    @Override
    public void onLoadFailed(GlideExeption e){
      this.exception=e;
      MAIN_THREAD_HANDLER.obtainMessage(MSG_EXCEPTION,this).sendToTarget();
    }
    @Override
    public void reschedule(DecodeJob<?> job){
      getActiveSourceExecutor().execute(job);
    }
    void handleExceptionOnMainThread(){
      stateVerifier.throwIfRecycled();
      if(isCancelled){
        release(false);
        return;
      }else if(cbs.isEmpty()){
        throw new IllegalStateException("Received an exception without any callbacks to notify");
      }else if(hasLoadFailed){
        throw new IllegalStateException("Already failed once");
      }
      hasLoadFailed=true;
      listener.onEngineJobComplete(this,key,null);
      for(ResourceCallback cb:cbs){
        if(!isInIgnoredCallbacks(cb)){
          cb.onLoadFailed(exception);
        }
      }
      release(false);
    }
    @Override
    public StateVerifier getVerifier(){
      return stateVerifier;
    }
    static class EngineResourceFactory(){
      public <R> EngineResource<R> build(Resource<R> resource,boolean isMemoryCacheable){
        return new EngineResource<>(resource,isMemoryCacheable,true);
      }
    }
    private static class MainThreadCallback implements Handler.Callback{
      MainThreadCallback(){}
      @Override
      public boolean handleMessage(Message message){
        EngineJob<?> job=(EngineJob<?>)message.obj;
        switch(message.what){
          case MSG_COMPLETE:
            job.handleResultOnMainThread();
            break;
          case MSG_EXCEPTION:
            job.handleExceptionOnMainThread();
            break;
          case MSG_CANCELLED:
            job.handleCacelledOnMainThread();
            break;
          default:
            throw new IllegalStateException("Unrecognized message: "+message.what);
        }
        return true;
      }
    }
  }
```
EngineKeyFactory
```java
  class EngineKeyFactory{
    EngineKey buildKey(Object model,Key signature,int width,int height,Map<Class<?>,Transformation<?>> transformations,Class<?> resourceClass,Class<?> transcodeClass,Options options){
      return new EngineKey(model,signature,width,height,transformations,resourceClass,transcodeClass,options);
    }
  }
```
EngineKey:用于多路复用加载的内存中唯一缓存键
```java
  class EngineKey implements Key{
    private final Object model;
    private final int width;
    private final int height;
    private final Class<?> resourceClass;
    private final Class<?> transcodeClass;
    private final Key signature;
    private final Map<Class<?>,Transformation<?>> transformations;
    private final Options options;
    private int hashCode;
    EngineKey(Object model,Key signature,int width,int height,Map<Class<?>,Transformation<?>> transformation,Class<?> resourceClass,CLass<?> transcodeClass,Options options){
      this.model=Preconditions.checkNotNull(model);
      this.signature=Preconditioins.checkNotNull(signature,"Signature must not be null");
      this.width=width;
      this.height=height;
      this.transformations=Preconditions.checkNotNull(transformations);
      this.resourceClass=Preconditions.checkNotNull(resourceClass,"Resource class must not be null");
      this.transcodeClass=Preconditions.checkNotNull(transcodeClass,"Transcode class must not be null");
      this.options=Preconditions.checkNotNull(options);
    }
    @Override
    public boolean equals(Object o){
      if(o instanceof EngineKey){
        EngineKey other=(EngineKey)o;
        return model.equals(other.model)&&signature.equals(other.signature)&&height==other.height&&width==other.width&&transformations.equals(other.transformations)&&resourceClass.equals(other.resourceClass)&&transcodeClass.equals(other.transcodeClass)&&options.equals(other.options);
      }
      return false;
    }
    @Override
    public int hashCode(){
      if(hashCode==0){
        hashCode=model.hashCode;
        hashCode=31*hashCode+signature.hashCode();
        hashCode=31*hashCode+width;
        hashCode=31*hashCode+height;
        hashCode=31*hashCode+transformations.hashCode();
        hashCode=31*hashCode+resourceClass.hashCode();
        hashCode=31*hashCode+transcodeClass.hashCode();
        hashCode=31*hashCode+options.hashCode();
      }
      return hashCode;
    }
    @Override
    public void updateDiskCacheKey(@NonNull MessageDigest messageDigest){
      throw new UnsupportedOprationException();
    }
  }
```
ResourceRecycler:可以安全地回收循环递归的资源
```java
  class ResourceRecycler{
    private boolean isRecycling;
    private final Handler handler=new Handler(Looper.getMainLooper(),new ResourceRecyclerCallback());
    void recycle(Resource<?> resource){
      Util.assertMainThread();
      if(isRecycling){
        handler.obtainMessage(ResourceRecyclerCallback.RECYCLE_RESOURCE,resource).sendToTarget();
      }else{
        isRecyling=true;
        resource.recycle();
        isRecycling=false;
      }
    }
    private static final class ResourceRecyclerCallback implements Handler.Callback{
      static final int RECYCLE_RESOURCE=1;
      ResourceRecyclerCallback(){}
      @Override
      public boolean handleMessage(Message message){
        if(message.what==RECYCLE_RESOURCE){
          Resource<?> resource=(Resource<?>)message.obj;
          resource.recycle();
          return true;
        }
        return false;
      }
    }
  }
```
ActiveResources:
```java
final class ActiveResources{
  private static final int MSG_CLEAN_REF=1;
  private final boolean isActiveResourceRetentionAllowed;
  private final Handler mainHandler=new Handler(Looper.getMainLooper(),new Callback(){
    @Override
    public boolean handleMessage(Message msg){
      if(msg.what==MSG_CLEAN_REF){
        cleanupActiveReference((ResourceWeakReference)msg.obj);
        return true;
      }
      return false;
    }
  });
  final Map<Key,ResourceWeakReference> activeEngineResources=new HashMap<>();
  private ResourceListener listener;
  @Nullable
  private ReferenceQueue<EngineResource<?>> resourceReferenceQueue;
  private Thread cleanReferenceQueueThread;
  private volatile boolean isShutdown;
  private volatile DequeuedResourceCallback cb;
  ActiveResources(boolean isActiveResourceRetentionAllowed){
    this.isActiveResourceRetentionAllowed=isActiveResourceRetentionAllowed;
  }
  void setListener(ResourceListener listener){
    this.listener=listener;
  }
  void activate(Key key,EngineResource<?> resource){
    ResourceWeakReference toPut=new ResourceWeakReference(key,resource,getReferenceQueue(),isActiveResourceRetentionAllowed);
    ResourceWeakReference removed=activeEngineResources.put(key,toPut);
    if(removed!=null){
      removed.reset();
    }
  }
  void deactivate(Key key){
    ResourceWeakReference removed=activeEngineResources.remove(key);
    if(removed!=null){
      removed.reset();
    }
  }
  @Nullable EngineResource<?> get(Key key){
    ResourceWeakReference activeRef=activeEngineResources.get(key);
    if(activeRef==null){
      return null;
    }
    EngineResource<?> active=activeRef.get();
    if(active==null){
      cleanupActiveReference(activeRef);
    }
    return active;
  }
  void cleanupActiveReference(@NonNull ResourceWeakReference ref){
    Util.assertMainThread();
    activeEngineResources.remove(ref.key);
    if(!ref.isCacheable||ref.resource==null){
      return;
    }
    EngineResource<?> newResource=new EngineResource<>(ref.source,true,false);
    newResource.setResourceListener(ref.key,listener);
    listener.onResourceReleased(ref.key,newResource);
  }
  private ReferenceQueue<EngineResource<?>> getReferenceQueue(){
    if(resourceReferenceQueue==null){
      resourceReferenceQueue=new ReferneceQueue<>();
      cleanReferenceQueueThread=new Thread(new Runnable(){
        @Override
        public void run(){
          Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
          cleanReferenceQueue();
        }
      },"glide-active-resources");
      cleanReferenceQueueThread.start();
    }
    return resourceReferenceQueue;
  }
  void cleanReferenceQueue(){
    while(!isShutdown){
      try{
        ResourceWeakReference ref=(ResourceWeakReference)resourceReferenceQueue.remove();
        mainHandler.obtainMessage(MSG_CLEAN_REF,ref).sendToTarget();
        DequeuedResourceCallback current=cb;
        if(current!=null){
          current.onResourceDequeued();
        }
      }catch(InterruptedException e){
        Thread.currentThread().interrupt();
      }
    }
  }
  void setDequeuedResourceCallback(DequeuedResourceCallback cb){
    this.cb=cb;
  }
  interface DequeuedResourceCallback{
    void onResourceDequeued();
  }
  void shutdown(){
    isShutdown=true;
    if(cleanReferenceQueueThread==null){
      return;
    }
    cleanReferenceQueueThread.interrupt();
    try{
      cleanReferenceQueueThread.join(TimeUnit.SECONDS.toMillis(5));
      if(cleanReferenceQueueThread.isActive()){
        throw new RuntimeException("Failed to join in time");
      }
    }catch(InterruptedException e){
      Thread.currentThread().interrupt();
    }
  }
  static final class ResourceWeakReference extends WeakReference<EngineResource<?>>{
    final Key key;
    final boolean isCacheable;
    Resource<?> resource;
    ResourceWeakReference(@NonNull Key key,@NonNull EngineResource<?> referent,@NonNull ReferenceQueue<? super EnigeResource<?>> queue,boolean isActiveResourceRetentionAllowed){
      super(referent,queue);
      this.key=Preconditions.checkNotNull(key);
      this.resource=referent.isCacheable()&&isActiveResourceRetentionAllowed?Preconditions.checkNotNull(referent.getResource()):null;
      isCacheable=referent.isCacheable();
    }
    void reset(){
      resource=null;
      clear();
    }
  }
}
```
DecodeJob:解码来自缓存数据或原资源的资源，并应用于转换和转码。
```java
  class DecodeJob<R> implements DataFectcherGenerator.FetcherReadyCallback,Runnable,Comparable<DecodeJob<?>>,Poolable{
    private static final String TAG="DecodeJob";
    private final DecodeHelper<R> decodeHelper = new DecodeHelper<>();
    private final List<Throwable> throwables=new ArrayList<>();
    private final StateVerifier stateVerifier=StateVerifier.newInstance();
    private final Pools.Pool<DecodeJob<?>> pool;
    private final DeferredEncodeManager<?> deferredEncodeManager = new DeferredEncodeManager<>();
    private final ReleaseManager releaseManager=new ReleaseManager();
    private GlideContext glideContext;
    private Key signature;
    private Priority priority;
    private EngineKey loadKey;
    private int width;
    private int height;
    private DiskCacheStrategy diskCacheStrategy;
    private Options options;
    private Callback<R> callback;
    private int order;
    private Stage stage;
    private RunReason runReason;
    private long startFetchTime;
    private boolean onlyRetrieveFromCache;
    private Object model;
    private Thread currentThread;
    private Key currentSourceKey;
    private Key currentAttemptingKey;
    private Object currentData;
    private DataSource currentDataSource;
    private DataFetcher<?> currentFetcher;
    private volatile DataFetcherGenerator currentGenrator;
    private volatile boolean isCallbackNotified;
    private volatile boolean isCancelled;
    
    DecodeJob(DiskCacheProvider diskCacheProvider,Pools.Pool<DecodeJob<?>> pool){
      this.diskCacheProvider=diskCacheProvider;
      this.pool=pool;
    }
    DecodeJob<R> init(GlideContext glideContext,Object model,EngineKey loadKey,Key signature,int width,int height,Class<?> resourceClass,Class<R> transcodeClass,Priority priority,DiskCacheStrategy diskCacheStrategy,Map<Class<?>,Transformation<?>> transformations,boolean isTransformationRequired,boolean isScaleOnlyOrNoTransform,boolean onlyRetrieveFromCache,Options options,Callback<R> callback,int order){
      decodeHelper.init(glideContext,model,signature,width,height,diskCacheStrategy,resourceClass,reanscodeClass,priority,options,transformations,isTransformationRequired,isScaleOnlyOrNoTransform,diskCacheProvider);
      this.gildeContext=glideContext;
      this.signature=signature;
      this.priority=priority;
      this.loadKey=loadKey;
      this.width=width;
      this.height=height;
      this.diskCacheStrategy=diskCacheStrategy;
      this.onlyRetrieveFromCache=onlyRetrieveFromCache;
      this.options=options;
      this.callback=callback;
      this.order=order;
      this.runReason=RunReason.INITIALIZE;
      this.model=model;
      return this;
    }
    boolean willDecodeFromCache(){
      Stage firstStage=getNextStage(Stage.INITIALIZE);
      return firstStage==Stage.RESOURCE_CACHE||firstStage==Stage.DATA_CACHE；
    }
    void release(boolean isRemovedFromQueue){
      if(releaseManager.release(isRemovedFromQueue)){
        releaseInternal();
      }
    }
    private void onEncodeComplete(){
      if(releaseManager.onEncodeComplete()){
        releaseInternal();
      }
    }
    private void onLoadFailed(){
      if(releaseManager.onFailed()){
        releaseInternal();
      }
    }
    private void releaseInternal(){
      releaseManager.reset();
      deferredEncodeManager.clear();
      decodeHelper.clear();
      isCallbackNotified=false;
      glideContext=null;
      signature=null;
      options=null;
      priority=null;
      stage=null;
      currentGenerator=null;
      currentThread=null;
      currentSourceKey=null;
      currentData=null;
      currentDataSource=null;
      startFetchTime=0L;
      isCancelled=false;
      model=null;
      throwables.clear();
      pool.release(this);
    }
    @Override
    public int compareTo(@NonNull DecodeJob<?> other){
      int result=getPriority()-other.getPriority();
      if(result==0){
        result=other-other.order;
      }
      return result;
    }
    private int getPriority(){
      return priority.ordinal();
    }
    public void cancel(){
      isCancelled=true;
      DataFetcherGenerator local=currentGenerator;
      if(local!=null){
        local.cancel();
      }
    }
    @Override
    public void run(){
      GlideTrace.beginSectionFormat("DecodeJob#run(model=%s)",model);
      DataFetcher<?> localFetcher=currentFetcher;
      try{
        if(isCancelled){
          notifyFailed();
          return;
        }
        runWrapped();
      }catch(Throwable t){
        if(stage!=Stage.ENCODE){
          throwables.add(t);
          notifyFailed();
        }
        if(!isCancelled){
          throw t;
        }
      }finally{
        if(localFetcher!=null){
          localFetcher.cleanup();
        }
        GlideTrace.endSection();
      }
    }
    private void runWrapped(){
      switch(runReason){
        case INITIALIZE:
          stage=getNextStage(Stage.INITIALIZE);
          currentGenerator=getNextGenerator();
          runGenerators();
          break;
        case SWITCH_TO_SOURCE_SERVICE:
          runGenerators();
          break;
        case DECODE_DATA:
          decodeFromRetrievedData();
          break;
        default:
          throw new IllegalStateException("Unrecognized run reason:  "+runReason);
      }
    }
    private DataFetcherGenerator getNextGenerator(){
      switch(stage){
        case RESOURCE_CACHE:
          return new ResourceCacheGenerator(decodeHelper,this);
        case DATA_CACHE:
          return new DataCacheGenerator(decodeHelper,this);
        case SOURCE:
          return new SourceGenerator(decodeHelper,this);
        case FINISHED:
          return null;
        default:
          throw new IllegalStateException("Unrecognized stage::  "+stage);
      }
    }
    private void runGenerator(){
      currentThread=Thread.currentThread();
      startFetchTime=LogTime.getLogTime();
      boolean isStarted=false;
      while(!isCancelled&&currentGenerator!=null&&!(isStarted=currentGenerator.startNext())){
        stage=getNextStage(stage);
        currentGenerator=getNextGenerator();
        if(stage==Stage.SOURCE){
          reschedule();
          return;
        }
      }
      if((stage==Stage.FINISHED||isCancelled)&&!isStarted){
        notifyFailed();
      }
    }
    private void notifyFailed(){
      setNotifiedOrThrow();
      GlideException e=new GlideException("Failed to load resource",new ArrayList<>(throwables));
      callback.onLoadFailed(e);
      onLoadFailed();
    }
    private void notifyComplete(Resource<R> resource,DataSource dataSource){
      setNotifiedOrThrow();
      callback.onResourceReady(resource,dataSource);
    }
    private void setNotifyOrThrow(){
      stateVerfier.throwIfRecycled();
      if(isCallbackNotified){
        throw new IllegalStateException("Already notified")
      }
      isCallbackNotified=true;
    }
    private Stage getNextStage(Stage current){
      switch(current){
        case INITIALIZE:
          return diskCacheStrategy.decodeCacheResource()?Stage.RESOURCE_CACHE:getNextStage(Stage.RESOURCE_CACHE);
        case RESOURCE_CACHE:
          return diskCacheStrategy.decodeCacheData()?Stage.DATA_CACHE:getNextStage(Stage.DATA_CACHE);
        case DATA_CACHE:
          return onlyRetrieveFromCache?Stage.FINISHED:Stage.SOURCE;
        case SOURCE:
        case FINISHED:
          return Stage.FINISHED;
        default:
          throw new IllegalArgumentException("Unrecognized stage:  "+current);
      }
    }
    @Override
    public void reschedule(){
      runReason=RunReason.SWTICH_TO_SOURCE_SERVICE;
      callback.reschedule(this);
    }
    @Override
    public void onDataFetcherReady(Key sourceKey,Object data,DataFetcher<?> fetcher,DataSource dataSource,Key attemptedKey){
      this.currentSourceKey=sourceKey;
      this.currentData=data;
      this.currentFetcher=fetcher;
      this.currentDataSource=dataSource;
      this.currentAttemptingKey=attemptedKey;
      if(Thread.currentThread()!=currentThread){
        runReason=RunReason.DECODE_DATA;
        callback.reschedule(this);
      }else{
        GlideTrace.beginSection("DecodeJob decodeFromRetrievedData");
        try{
          decodeFromRetrievedData();
        }finally{
          GlideTrace.endSection();
        }
      }
    }
    @Override
    public void onDataFetcherFailed(Key attemptedKey,Exception e,DataFetcher<?> fetcher,DataSource dataSource){
      fetcher.cleanup();
      GlideException exception=new GlideException("Fetching data failed",e);
      exception.setLoggingDetails(attemptedKey,dataSource,fetcher.getDataClass());
      throwables.add(exception);
      if(Thread.currentThread()!=currentThread){
        runReason=RunReason.SWITCH_TO_SOURCE_SERVICE;
        callback.reschedult(this);
      }else{
        runGenerators();
      }
    }
    private void decodeFromRetrievedData(){
      Resource<R> resource=null;
      try{
        resource=decodeFromData(currentFetcher,currentData,currentDataSource);
      }catch(GlideExeption e){
        e.setLoggingDetails(currentAttemptingKey,currentDataSource);
        throwables.add(e);
      }
      if(resource!=null){
        notifyEncodeAndRelease(resource,currentDataSource);
      }else{
        runGenerators();
      }
    }
    private void notifyEncodeAndRelease(Resource<R> resource,DataSource dataSource){
      if(resource instanceof Initializable){
        ((Initializable)resource).initialize();
      }
      Resource<R> result=resource;
      LockedResource<R> lockedResource=null;
      if(deferredEncodeManager.hasResourceToEncode()){
        lockedResource=LockedResource.obtain(resource);
        result=lockedResource;
      }
      notifyComplete(result,dataSource);
      stage=Stage.ENCODE;
      try{
        if(deferredEncodeManager.hasResourceToEncode()){
          deferredEncodeManager.encode(diskCacheProvider,options);
        }
      }finally{
        if(lockedResource!=null){
          lockedResource.unlock();
        }
      }
      onEncodeComplete();
    }
    private <Data> Resource<R> decodeFromData(DataFetcher<?> fetcher,Data data,DataSource dataSource)throws GlideExeption{
      try{
        if(data==null){
          return null;
        }
        long startTime=LogTime.getLogTime();
        Resource<R> result=decodeFromFetcher(data,dataSource);
        return result;
      }finally{
        fetcher.cleanup();
      }
    }
    private <Data> Resource<R> decodeFromFetcher(Data data,DataSource dataSource)throws GlideExeption{
      LoadPath<Data,?,R> path=decodeHelper.getLoadPath((Class<Data>)data.getClass());
      return runLoadPath(data,dataSource,path);
    @NonNull
    private Options getOptionsWithHardwareConfig(DataSource dataSource){
      Options options=this.options;
      if(Build.VERSION.SDK_INI<Build.VERSION_CODES.O){
        return options;
      }
      boolean isHardwareConfigSafe=dataSource==DataSource.RESOURCE_DISK_CACHE||decodeHelper.isScaleOnlyOrNoTransform();
      Boolean isHardwareConfigAllowed=options.get(Downsampler.ALLOW_HEADWARE_CONFIG);
      if(isHardwareConfigAllowed!=null&&(!isHardwareConfigAllowed||isHardwareConfigSafe)){
        return options;
      }
      options=new Options();
      options.putAll(this.options);
      options.set(Downsampler.ALLOW_HADWARE_CONFIG,isHardwareConfigSafe);
      return options;
    }
    private <Data,ResourceType> Resource<R> runLoadPath(Data data,DataSource dataSource,LoadPath<Data,ResourceType,R> path) throws GlideException{
      Options options=getOptionsWithHardwareConfig(dataSource);
      DataRewinder<Data> rewinder=glideContext.getRegistry().getRewinder(data);
      try{
        return path.load(rewinder,options,width,height,new DecodeCallback<ResourceType>(dataSource));
      }finally{
        rewinder.cleanup();
      }
    }
    private void logWithTimeAndKey(String message,long startTime){
      logWithTimeAndKey(message,startTime,null);
    }
    private void logWithTimeAndKey(String message,long startTime,String extraArgs){
      
    }
    @NonNull
    @Override
    public StateVerifier getVerifier(){
      return stateVerifier;
    }
    @NonNull
    <Z> Resource<Z> onResourceDecoded(DataSource dataSource,@NonNull Resource<Z> decoded){
      Class<Z> resourceSubClass=(Class<Z>)decoded.get().getClass();
      Transformation<Z> appliedTransformation=null;
      Resource<Z> transformed=decode;
      if(dataSource!=DataSource.RESOURCE_DISK_CACHE){
        appliedTransformation=decodeHelper.getTransformation(resourceSubClass);
        transformed=appliedTransformation.transform(glideContext,decoded,width,height);
      }
      if(!decoded.equals(transformed)){
        decoded.recyle();
      }
      final EncodeStrategy encodeStrategy;
      final ResourceEncoder<Z> encoder;
      if(decodeHelper.isResourceEncoderAvailable(transformed)){
        encoder=decodeHelper.getResultEncoder(transformed);
        encodeStrategy=encoder.getEncodeStrategy(options);
      }else{
        encoder=null;
        encodeStrategy=EncodeStrategy.NONE;
      }
      Resource<Z> result=transformed;
      boolean isFromAlternateCacheKey=!decodeHelper.isSourceKey(currentSourceKey);
      if(diskCacheStrategy.isResourceCacheable(isFromAlternateCacheKey,dataSource,encodeStrategy)){
        if(encoder==null){
          throw new Registry.NoResultEncoderAvailableException(transformed.get().getClass());
        }
        final Key key;
        switch(encodeStrategy){
          case SOURCE:
            key=new DataCacheKey(currentSourceKey,signature);
            break;
          case TRANSFORMED:
            key=new ResourceCacheKey(decodeHelper.getArrayPool(),currentSourceKey,signature,width,height,appliedTransformation,resourceSubClass,options);
            break;
          default:
            throw new IllegalArgumentException("Unknown strategy:  "+encodeStrategy);
        }
        LockedResource<Z> lockedResult=LockedResource.obtain(transformed);
        deferredEncodeManager.init(key,encoder,lockedResult);
        result=lockedResult;
      }
      return result;
    }
    private final class DecodeCallback<Z> implements DecodePath.DecodeCallback<Z>{
      private final DataSource dataSource;
      DecodeCallback(DataSource dataSource){
        this.dataSource=dataSource;
      }
      @NonNull
      @Override
      public Resource<Z> onResourceDecoded(@NonNull Resource<Z> decoded){
        return DecodeJob.this.onResourceDecoded(dataSource,decoded);
      }
    }
    private static class ReleaseManager{
      private boolean isReleased;
      private boolean isEncodeComplete;
      private boolean isFailed;
      ReleaseManager(){}
      synchronized boolean release(boolean isRemovedFromQueue){
        isRelease=true;
        return isComplelte(isRemovedFromQueue);
      }
      synchronized boolean onEncodeComplete(){
        isEncodeComplete=true;
        return isComplete(false);
      }
      synchronized boolean onFailed(){
        isFailed=true;
        return isComplete(false);
      }
      synchronized void reset(){
        isEncodeComplete=false;
        isReleased=false;
        isFailed=false;
      }
      private boolean isComplete(boolean isRemovedFromQueue){
        return (isFailed||isRemovedFromQueue||isEncodeComplete)&&isReleased;
      }
    }
    private static class DeferredEncodeManager<Z>{
      private Key key;
      private ResourceEncoder<?> encoder;
      private LockedResource<Z> toEncode;
      DeferredEncodeManager(){}
      <X> void init(Key key,ResourceEncoder<X> encoder,LockedResource<X> toEncode){
        this.key=key;
        this.encoder=(ResourceEncoder<Z>)encoder;
        this.toEncode=(LockedResource<Z>)toEncode;
      }
      void encode(DiskCacheProvider diskCacheProvider,Options options){
        GlideTrace.beginSection("DecodeJob.encode");
        try{
          diskCacheProvider.getDiskCache().put(key,new DataCacheWriter<>(encoder,toEncode,options));
        }finally{
          toEncode.unlock();
          GlideTrace.endSection();
        }
      }
      boolean hasResourceToEncode(){
        return toEncdoe!=null;
      }
      void clear(){
        key=null;
        encoder=null;
        toEncode=null;
      }
    }
    interface Callback<R>{
      void onResourceReady(Resource<R> resource,DataSource dataSource);
      void onLoadFailed(GlideException e);
      void reschedule(DecodeJob<?> job);
    }
    interface DiskCacheProvider{
      DiskCache getDiskCache();
    }
    private enum RunReason{
      INITIALIZE,
      SWITCH_TO_SOURCE_SERVICE,
      DECODE_DATA,
    }
    private enum Stage{
      INITIALIZE,
      RESOURCE_CACHE,
      DATA_CACHE,
      SOURCE,
      ENCODE,
      FINISHED,
    }
  }
```
ReferenceQueue:在检测到可达性更改之后，垃圾回收将已注册的引用对象追加到该队列。
```java
  public class ReferenceQueue<T>{
    private static final Reference sQueueNextUnenqueued=new PhantomReference(null,null);
    private Reference<? extends T> head=null;
    private Reference<? extends T> tail=null;
    private final Object lock=new Object();
    
    public ReferenceQueue(){}
    private boolean enqueueLocked(Reference<? extends T> r){
      if(r.queueNext!=null){
        return false;
      }
      if(r instanceof Cleaner){
        Cleaner cl=(sun.misc.Cleaner)r;
        cl.clean();
        r.queueNext=sQueueNextUnenqueued;
        return true;
      }
      if(tail==null){
        head=r;
      }else{
        tail.queueNext=r;
      }
      tail=r;
      tail.queueNext=r;
      return true;
    }
    boolean isEnqueued(Reference<? extends T> reference){
      synchronized(lock){
        return reference.queueNext!=null&&reference.queueNext!=sQueueNextUnqueued;
      }
    }
    boolean enqueue(Reference<? extends T> reference){
      synchronized(lock){
        if(enqueueLocked(reference)){
          lock.notifyAll();
          return true;
        }
        return false;
      }
    }
    private Reference<? extends T> reallyPollLocked(){
      if(head!=null){
        Reference<? extends T> r=head;
        if(head==tail){
          tail=null;
          head=null;
        }else{
          head=head.queueNext;
        }
        r.queueNext=sQueueNextUnenqueued;
        return r;
      }
      return null;
    }
    public Reference<? extends T> poll(){
      synchronized(lock){
        if(head==null){
          return null;
        }
        return reallyPollLocked();
      }
    }
    public Reference<? extends T> remove(long timeout){
      if(timeout<0){
        throw new IllegalArgumentException("Negative timeout value");
      }
      synchronized(lock){
        Reference<? extends T> r=reallyPollLocked();
        if(r!=null){
          return r;
        }
        long start=(timeout==0)?0:System.nanoTime();
        for(;;){
          lock.wait(timeout);
          r=reallyPollLocked();
          if(timeout!=0){
            long end=System.nanoTime();
            timeout-=(end-start)/1000_000;
            if(timeout<=0){
              return null;
            }
            satrt=end;
          }
        }
      }
    }
    public Referenc<? extends T> remove() throws InterruptedException{
      return remove(0);
    }
    public static void enqueuePending(Reference<?> list){
      Reference<?> start=list;
      do{
        ReferenceQueue queue=list.queue;
        if(queue==null){
          Reference<?> next=list.pendingNext;
          list.pendingNext=list;
          list=next;
        }else{
          synchronized(queue.lock){
            do{
              Reference<?> next=list.pendingNext;
              list.pendingNext=list;
              queue.enqueueLocked(list);
              list=next;
            }while(list!=start&&list.queue==queue){
              queue.lock.notifyAll();
            }
          }
        }while(list!=start);
      }
    }
    public static Reference<?> unenqueued=null;
    statice void add(Reference<?> list){
      synchronized(ReferenceQueue.class){
        if(unenqueued==null){
          unenqueued=list;
        }else{
          Reference<?> last=unqueued;
          while(last.pendingNext!=unqueued){
            last=last.pendingNext;
          }
          last.pendingNext=list;
          last=list;
          while(last.pendingNext!=list){
            last=last.pendingNext;
          }
          last.pendingNext=unenqueued;
        }
        ReferenceQueue.class.notifyAll();
      }
    }
  }
```
DecodeHelper
```java
  final class DecodeHelper<Transcode>{
    private final List<LoadData<?>> loadData=new ArrayList<>();
    private final List<Key> cacheKeys=new ArrayList<>();
    private GlideContext gildeContext;
    private Object model;
    private int width;
    private int height;
    private Class<?> resourceClass;
    private DecodeJob.DiskCacheProvider diskCacheProvider;
    private Options options;
    private Map<Class<?>,Transformation<?>> transformations;
    private Class<Transcode> transcodeClass;
    private boolean isLoadDataSet;
    private boolean isCacheKeysSet;
    private Key signature;
    private Priority priority;
    private DiskCacheStrategy diskCacheStrategy;
    private boolean isTransformationRequired;
    private boolean isScaleOnlyOrNoTransform;
    <R> void init(GlideContext glideContext,Object model,Key signature,int width,int height,DiskCacheStrategy diskCacheStrategy,Class<?> resourceClass,Class<R> transcodeClass,Priority priority,Options options,Map<Class<?>,Transformation<?>> transformations,boolean isTransformationRequired,boolean isScaleOnlyOrNoTransform,DiskCacheProvider diskCacheProvider){
      this.glideContext=glideContext;
      this.model=model;
      this.signature=signature;
      this.width=width;
      this.height=height;
      this.diskCacheStrategy=diskCacheStrategy;
      this.resourceClass=resourceClass;
      this.diskCacheProvider=diskCahceProvider;
      this.transcodeClass=(Class<Transcode>) transcodeClass;
      this.priority=priority;
      this.options=options;
      this.transformations=transformations;
      this.isTransformationRequired=isTransformationRequired;
      this.isScaleOnlyOrNoTranform=isScaleOnlyOrNoTransform;
    }
    void clear(){
      glideContext=null;
      model=null;
      signature=null;
      resourceClass=null;
      transcodeClass=null;
      options=null;
      priority=null;
      transformations=null;
      diskCacheStrategy=null;
      loadData.clear();
      isLoadDataSet=false;
      cahceKey.clear();
      isCacheKeysSet=false;
    }
    DiskCache getDiskCache(){
      return diskCacheProvider.getDiskCache();
    }
    DiskCacheStrategy getDiskCacheStrategy(){
      return diskCacheStrategy;
    }
    Priority getPriority(){
      return priority;
    }
    Options getOptions(){
      return options;
    }
    Key getSignature(){
      return signature;
    }
    int getWidth(){
      return width;
    }
    int getHeight(){
      return height;
    }
    ArrayPool getArrayPool(){
      return glideContext.getArrayPool();
    }
    Class<?> getTranscodeClass(){
      return transcodeClass;
    }
    Class<?> getModelClass(){
      return model.getClass();
    }
    List<Class<?>> getRegisteredResourceClasses(){
      return glideContext.getRegistry().getRegisteredResourceClasses(model.getClass(),resourceClass,transcodeClass);
    }
    boolean hasLoadPath(Class<?> dataClass){
      return getLoadPath(dataClass)!=null;
    }
    <Data> LoadPath<Data,?,Transcode> getLoadPath(Class<Data> dataClass){
      return glideContext.getRegistry().getLoadPath(dataClass,resourceClass,transcodeClass);
    }
    boolean isScaleOnlyOrNoTransform(){
      return isScaleOnlyOrNoTransform;
    }
    <Z> Transformation<Z> getTransformation(Class<Z> resourceClass){
      Transformation<Z> result=(Transformation<Z>) transformations.get(resourceClass);
      if(result==null){
        for(Entry<Class<?>,Transformation<?>> entry: transformations.entrySet()){
          if(entry.getKey().isAssignableFrom(resourceClass)){
            result=(Transformation<Z>)entry.getValue();
            break;
          }
        }
      }
      if(reuslt==null){
        if(transformations.isEmpty()&&isTransformationRequired){
          throw new IllegalArgumentException("Missing transformation for"+resourceClass+" If you wish to ignore unknown resource types,use the optional transformation methods.");
        }else{
          return UnitTransformation.get();
        }
      }
      return result;
    }
    boolean isResourceEncoderAvailable(Resource<?> resource){
      return glideContext.getRegistry().isResourceEncoderAvailable(resource);
    }
    <Z> ResourceEncoder<Z> getResultEncoder(Resource<Z> resource){
      return glideContext.getRegistry().getResultEncoder(resource);
    }
    List<ModelLoader<File,?>> getModelLoaders(File file) throws Registry.NoModelLoaderAvailableException{
      return glideContext.getRegistry().getModelLoaders(file);
    }
    boolean isSourceKey(Key key){
      List<LoadData<?>> loadData=getLoadData();
      for(int i=0,size=loadData.size();i<size;i++){
        LoadData<?> current=loadData.get(i);
        if(current.sourceKey.equals(key)){
          return true;
        }
      }
      return false;
    }
    List<LoadData<?>> getLoadData(){
      if(!isLoadDataSet){
        isLoadDataSet=true;
        loadData.clear();
        List<ModelLoader<Object,?>> modelLoaders=glideContext.getRegistry().getModelLoaders(model);
        for(int i=0,size=modelLoaders.size();i<size;i++){
          ModelLoader<Object,?> modelLoader=modelLoaders.get(i);
          LoadData<?> current=modelLoader.buildLoadData(model,width,height,options);
          if(current!=null){
            loadData.add(current);
          }
        }
      }
      return loadData;
    }
    List<Key> getCacheKeys(){
      if(!isCacheKeysSet){
        isCacheKeysSet=true;
        cacheKeys.clear();
        List<LoadData<?>> loadData=getLoadData();
        for(int i=0,size=loadData.size();i<size;i++){
          LoadData<?> data=loadData.get(i);
          if(!cacheKeys.contains(data.sourceKey)){
            cacheKeys.add(data.sourceKey);
          }
          for(int j=0；j<data.alternateKeys.size();j++){
            if(!cacheKeys.contains(data.alternateKeys.get(j))){
              cacheKeys.add(data.alternateKeys.get(j));
            }
          }
        }
      }
      return cacheKeys;
    }
    <X> Encoder<X> getSourceEncoder(X data) throws Registry.NoSourceEncoderAvailableException{
      return glideContext.getRegistry().getSourceEncoder(data);
    }
  }

```
Options:应用于内存和磁盘的一组键
```java
  public final class Options implements Key{
    private final ArrayMap<Option<?>,Object> values=new CachedHashCodeArrayMap<>();
    public void putAll(@NonNull Options other){
      values.putAll((SimpleArrayMap<Options<?>,Object>)other.values);
    }
    @NonNull
    public <T> Options set(@NonNull Option<T> option,@NonNull T value){
      values.put(option,value);
      return this;
    }
    @Nullable
    public <T> T get(@NonNull Option<T> option){
      return values.containKey(option)?(T)values.get(option):option.getDefaultValue();
    }
    @Override
    public boolean equals(Object o){
      if(o instanceof Options){
        Options other=(Options)o;
        return values.equals(other.values);
      }
      return false;
    }
    @Override
    public int hashCode(){
      return values.hashCode();
    }
    @Override
    public void updateDiskCacheKey(@NonNull MessageDigest messageDigest){
      for(int i=0;i<values.size();i++){
        Option<?> key=values.keyAt(i);
        Object value=values.valueAt(i);
        updateDiskCacheKey(key,value,messageDigest);
      }
    }
    @Override
    public String toString(){
      return "Options{values="+values+"}";
    }
    private static <T> void updateDiskCacheKey(@NonNull Option<T> option,@NonNull Object value,@NonNull MessageDigest md){
      option.update((T)value,md);
    }
  }
```
Option:定义具有可选默认值的和能够影响磁盘资源缓存键的可用的组件选项，如：解码器、编码器、模型加载器
```java
  public final class Option<T>{
    private static final CacheKeyUpdater<Object> EMPTY_UPDATE=new CacheKeyUpdater<Object>(){
      @Override
      public void update(@NonNull byte[] keyBytes,@NonNull Object value,@NonNull MessageDigest messageDigest){
        
      }
    };
    private final T defaultValue;
    private final CacheKeyUpdater<T> cacheKeyUpdater;
    private final String key;
    private volatile byte[] keyBytes;
    @NonNull
    public static <T> Option<T> memory(@NonNull String key){
      return new Option<>(key,null,Option.<T> emptyUpdater());
    }
    @NonNull
    public static <T> Option<T> memory(@NonNull String key,@NonNull T defaultValue){
      return new Option<>(key,defaultValue,Option.<T>emptyUpdater());
    }
    @NonNull
    public static <T> Option<T> disk(@NonNull String key,@NonNull CacheKeyUpdater<T> cacheKeyUpdater){
      return new Option<>(key,null,cacheKeyUpdater);
    }
    @NonNull
    public static <T> Option<T> disk(@NonNull String key,@Nullable T defaultValue,@NonNull CacheKeyUpdater<T> cacheKeyUpdater){
      return new Option<>(key,defaultValue,cacheKeyUpater);
    }
    private Option(@NonNull String key,@Nullable T defaultValue,@NonNull CacheKeyUpdater<T> cacheKeyUpdater){
      this.key=Preconditions.checkNotEmpty(key);
      this.defaultValue=defaultValue;
      this.cacheKeyUpdater=Preconditions.checkNotNull(cacheKeyUpdater);
    }
    @Nullable
    public T getDefaultValue(){
      return defaultValue;
    }
    public void update(@NonNull T value,@NonNull MessageDigest messageDigest){
      cacheKeyUpdater.update(getKeyBytes(),value,messageDigest);
    }
    @NonNull
    private byte[] getKeyBytes(){
      if(keyBytes==null){
        keyBytes=key.getBytes(Key.CHARSET);
      }
      return keyBytes;
    }
    @Override
    public boolean equals(Object o){
      if(o instanceof Option){
        Option<?> other=(Option<?>)o;
        return key.equals(other.key);
      }
      return false;
    }
    @Override
    public int hashCode(){
      return key.hashCode();
    }
    @NonNull
    private static <T> CacheKeyUpdater<T> emptyUpdater(){
      return (CacheKeyUpdater<T>)EMPTY_UPDATE;
    }
    @Override
    public String toString(){
      return "Option{key="+key+"}";
    }
    public interface CacheKeyUpdater<T>{
      void update(@NonNull byte[] keyBytes,@NonNull T value,@NonNull MessageDigest messageDigest);
    }
  }
```
ResourceCacheGenerator:从包含采样/转换过的资源数据的缓存文件获取
```java
  class ResourceCacheGenerator implements DataFetchGenerator,DataFetcher.DataCallback<Object>{
    private final FetcherReadyCallback cb;
    private final DecodeHelper<?> helper;
    private int sourceIdIndex;
    private int resourceClassIndex=-1;
    private Key sourceKey;
    private List<ModelLoader<File,?>> modelLoaders;
    private int modelLoaderIndex;
    private volatile LoadData<?> loadData;
    private File cacheFile;
    private ResourceCacheKey currentKey;
    ResourceCacheGenerator(DecodeHelper<?> helper,FetcherReadyCallback cb){
      this.helper=helper;
      this.cb=cb;
    }
    @Override
    public boolean startNext(){
      List<Key> sourceIds=helper.getCacheKeys();
      if(sourceIds.isEmpty()){
        return false;
      }
      List<Class<?>> resourceClasses=helper.getRegisteredResourceClasses();
      if(resourceClasses.isEmpty()){
        if(File.class.equals(helper.getTranscodeClass())){
          return false;
        }
      }
      while(modelLoaders==null||!hasNextModelLoader()){
        resourceClassIndex++;
        if(resourceClassIndex>=resourceClasses.size()){
          sourceIdIndex++;
          if(sourceIdIndex>=sourceIds.size()){
            return false;
          }
          resourceClassIndex=0;
        }
        Key sourceId=sourceIds.get(sourceIdIndex);
        Class<?> resourceClass=resourceClasses.get(resourceClassIndex);
        Transformation<?> transformation=helper.getTransformation(resourceClass);
        currentKey=new ResourceCacheKey(helper.getArrayPool(),sourceId,helper.getSignature(),helper.getWidth(),helper.getHeight(),transformation,resourceClass,helper.getOptions());
        cahceFile=helper.getDiskCache().get(currentKey);
        if(cacheFile!=null){
          sourceKey=sourceId;
          modelLoaders=helper.getModelLoaders(cacheFile);
          modelLoaderIndex=0;
        }
      }
      loadData=null;
      boolean started=false;
      while(!started&&hasNextModelLoader()){
        ModelLoader<File,?> modelLoader=modelLoaders.get(modelLoaderIndex++);
        loadData=modelLoader.buildLoadData(cacheFile,helper.getWidth(),helper.getHeight(),helper.getOptions());
        if(loadData!=null&&helper.hasLoadPath(loadData.fetcher.getDataClass())){
          started=true;
          loadData.fetcher.loadData(helper.getPriority(),this);
        }
      }
      return started;
    }
    private boolean hasNextModelLoader(){
      return modelLoaderIndex<modelLoaders.size();
    }
    @Override
    public void cancel(){
      LoadData<?> local=loadData;
      if(local!=null){
        local.fetcher.cancel();
      }
    }
    @Override
    public void onDataReady(Object data){
      cb.onDataFetcherReady(sourceKey,data,loadData.fetcher,DataSource.RESOURCE_DISK_CACHE,currentKey);
    }
    @Override
    public void onLoadFailed(@NonNull Exception e){
      cb.onDataFetcherFailed(currentKey,e,loadData.fetcher,DataSource.RESOURCE_DISK_CACHE);
    }
  }
```





## 缓存策略
* DiskCacheStrategy.NONE: 表示不缓存任何内容
* DiskCacheStrategy.DATA: 表示只缓存原始图片
* DiskCacheStrategy.RESOURCE: 只缓存转换过后的图片
* DiskCacheStrategy.ALL: 即缓存原始图片，也缓存转换过后的图片
* DiskCacheStrategy.AUTOMATIC: 让glide根据图片资源智能选择使用哪种缓存策略
## hashCode算法
```java
  //String的hashCode公式:s[0]*31^(n-1)+s[1]*31^(n-2)+...+s[n-1],其中s[i]是字符串中的字符，n=字符串的长度
  //有个疑惑，n>=7时，hashCode值不会超出Integer.MAX_VALUE么？
  public int hashCode(){
  int h=hash;
  final int leg=length();
  if(h==0&&len>0){
    for(int i=0;i<len;i++){
      h=31*h+charAt(i);
    }
    hash=h;
  }
  return h;
  }
```




