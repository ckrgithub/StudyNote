# Lifecycle+Retrofit+Room
## 一、六大设计原则
* 单一职责原则-Single Responsibility Principle(SRP):类的职责单一，引起类变化的原因单一
* 开闭原则-Open-Closed Principle(OCP):对外扩展开放，对修改关闭
* 里氏替换原则-Liskov Substitution Principle(LSP):子类可以在任何地方替换它的父类
* 依赖倒置原则-Dependency Inversion Principle(DIP):高层模块不依赖于底层模块，二者都依赖于抽象；抽象不依赖于具体，具体依赖于抽象
* 接口分隔原则-Interface Segregation Principle(ISP):尽量使用职能单一的接口，而不使用职能复杂、全面的接口
* 迪米特法则-Law of Demeter(LOD):一个对象应当对其他对象有尽可能少的了解，不和陌生人说话
## 二、结构图
![](http://www.canking.win/2017/12/09/mvvm/arch1.png)
## 三、组件用法
### 1.Lifecycle
一个android组件生命周期感知回调的控件，可以感知activity或fragment的生命周期变化，并回调相应接口。
```java
  public abstract class Lifecycle{
    @MainThread
    public abstract void addObserver(@NonNull LifecycleObserver observer);
    
    @MainThread
    public abstract void removeObserver(@NonNull LifecycleObserver observer);
    
    @MainThread
    @NonNull
    public abstract State getCurrentState();
    
    public enum Event{
      ON_CREATE,
      ON_START,
      ON_RESUME,
      ON_PAUSE,
      ON_STOP,
      ON_DESTROY,
      ON_ANY
    }
    
    public enum State{
      DESTROYED,
      INITIALIZED,
      CREATED,
      STARTED,
      RESUMED
    }
    
    public boolean isAtLeast(@NonNull State state){
      return compareTo(state) >=0;
    }
  }
```
在最新support v4包中，activity和fragment都实现了LifecycleOwner接口
```java
  public interface LifecycleOwner{
    @NonNull
    Lifecycle getLifecycle();
  }
```
### 2.LiveData
LiveData是一个持有泛型类型的数据组件，将App生命组件与数据关联到一起。
```java
  public abstract class LiveData<T>{
    private final Objcet mDataLock = new Object();
    static final int START_VERSION=-1;
    private static final Object NOT_SET=new Object();
    
    private SafeIterableMap<Observer<T>,ObserverWrapper> mObserver=new SafeIterableMap<>();
    
    private int mActiveCount=0;
    private volatile Object mData=NOT_SET;
    private volatile Object mPendingData=NOT_SET;
    private int mVersion=START_VERSION;
    
    private boolean mDispatchingValue;
    private boolean mDispatchInvalidated;
    private final Runnable mPostValueRunnable =new Runnable(){
      @Override
      public void run(){
        Object newValue;
        synchronized(mDataLock){
          newValue=mPendingData;
          mPendingData=NOT_SET;
        }
        setValue((T) newValue);
      }
    };
    
    private void considerNotify(ObserverWrapper observer){
      if(!observer.mActive){
        return;
      }
      if(!observer.shouldBeActive()){
        observer.activeStateChanged(false);
        return;
      }
      if(observer.mLastVersion>=version){
        return;
      }
      observer.mLastVersion = version;
      observer.mObserver.onChanged((T) mData);
    }
    
    private void dispatchingValue(@Nullable ObserverWrapper initiator) {
      if (mDispatchingValue) {
          mDispatchInvalidated = true;
          return;
      }
      mDispatchingValue = true;
      do {
          mDispatchInvalidated = false;
          if (initiator != null) {
              considerNotify(initiator);
              initiator = null;
          } else {
              for (Iterator<Map.Entry<Observer<T>, ObserverWrapper>> iterator =
                      mObservers.iteratorWithAdditions(); iterator.hasNext(); ) {
                  considerNotify(iterator.next().getValue());
                  if (mDispatchInvalidated) {
                      break;
                  }
              }
          }
      } while (mDispatchInvalidated);
        mDispatchingValue = false;
    }
    
    @MainThread
    public void observe(@NonNull LifecycleOwner owner, @NonNull Observer<T> observer) {
        if (owner.getLifecycle().getCurrentState() == DESTROYED) {
            return;
        }
        LifecycleBoundObserver wrapper = new LifecycleBoundObserver(owner, observer);
        ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);
        if (existing != null && !existing.isAttachedTo(owner)) {
            throw new IllegalArgumentException("Cannot add the same observer"
                    + " with different lifecycles");
        }
        if (existing != null) {
            return;
        }
        owner.getLifecycle().addObserver(wrapper);
    }
    
    @MainThread
    public void observeForever(@NonNull Observer<T> observer) {
        AlwaysActiveObserver wrapper = new AlwaysActiveObserver(observer);
        ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);
        if (existing != null && existing instanceof LiveData.LifecycleBoundObserver) {
            throw new IllegalArgumentException("Cannot add the same observer"
                    + " with different lifecycles");
        }
        if (existing != null) {
            return;
        }
        wrapper.activeStateChanged(true);
    }
    
    @MainThread
    public void removeObserver(@NonNull final Observer<T> observer) {
        assertMainThread("removeObserver");
        ObserverWrapper removed = mObservers.remove(observer);
        if (removed == null) {
            return;
        }
        removed.detachObserver();
        removed.activeStateChanged(false);
    }
    
    @SuppressWarnings("WeakerAccess")
    @MainThread
    public void removeObservers(@NonNull final LifecycleOwner owner) {
        assertMainThread("removeObservers");
        for (Map.Entry<Observer<T>, ObserverWrapper> entry : mObservers) {
            if (entry.getValue().isAttachedTo(owner)) {
                removeObserver(entry.getKey());
            }
        }
    }
    
    protected void postValue(T value) {
        boolean postTask;
        synchronized (mDataLock) {
            postTask = mPendingData == NOT_SET;
            mPendingData = value;
        }
        if (!postTask) {
            return;
        }
        ArchTaskExecutor.getInstance().postToMainThread(mPostValueRunnable);
    }
    
    @MainThread
    protected void setValue(T value) {
        assertMainThread("setValue");
        mVersion++;
        mData = value;
        dispatchingValue(null);
    }
    
    @Nullable
    public T getValue() {
        Object data = mData;
        if (data != NOT_SET) {
            //noinspection unchecked
            return (T) data;
        }
        return null;
    }
    
    @SuppressWarnings("WeakerAccess")
    public boolean hasObservers() {
        return mObservers.size() > 0;
    }
    
    @SuppressWarnings("WeakerAccess")
    public boolean hasActiveObservers() {
        return mActiveCount > 0;
    }
    
    class LifecycleBoundObserver extends ObserverWrapper implements GenericLifecycleObserver {
        @NonNull final LifecycleOwner mOwner;

        LifecycleBoundObserver(@NonNull LifecycleOwner owner, Observer<T> observer) {
            super(observer);
            mOwner = owner;
        }

        @Override
        boolean shouldBeActive() {
            return mOwner.getLifecycle().getCurrentState().isAtLeast(STARTED);
        }

        @Override
        public void onStateChanged(LifecycleOwner source, Lifecycle.Event event) {
            if (mOwner.getLifecycle().getCurrentState() == DESTROYED) {
                removeObserver(mObserver);
                return;
            }
            activeStateChanged(shouldBeActive());
        }

        @Override
        boolean isAttachedTo(LifecycleOwner owner) {
            return mOwner == owner;
        }

        @Override
        void detachObserver() {
            mOwner.getLifecycle().removeObserver(this);
        }
    }

    private abstract class ObserverWrapper {
        final Observer<T> mObserver;
        boolean mActive;
        int mLastVersion = START_VERSION;

        ObserverWrapper(Observer<T> observer) {
            mObserver = observer;
        }

        abstract boolean shouldBeActive();

        boolean isAttachedTo(LifecycleOwner owner) {
            return false;
        }

        void detachObserver() {
        }

        void activeStateChanged(boolean newActive) {
            if (newActive == mActive) {
                return;
            }
            mActive = newActive;
            boolean wasInactive = LiveData.this.mActiveCount == 0;
            LiveData.this.mActiveCount += mActive ? 1 : -1;
            if (wasInactive && mActive) {
                onActive();
            }
            if (LiveData.this.mActiveCount == 0 && !mActive) {
                onInactive();
            }
            if (mActive) {
                dispatchingValue(this);
            }
        }
    }

    private class AlwaysActiveObserver extends ObserverWrapper {

        AlwaysActiveObserver(Observer<T> observer) {
            super(observer);
        }

        @Override
        boolean shouldBeActive() {
            return true;
        }
    }

    private static void assertMainThread(String methodName) {
        if (!ArchTaskExecutor.getInstance().isMainThread()) {
            throw new IllegalStateException("Cannot invoke " + methodName + " on a background"
                    + " thread");
        }
    }
  }
```
LiveData有两个实现类:MutableLiveData和MediatorLiveData。其中，MutableLiveData值暴露两个方法：postData()和setData().
MediatorLiveData有个addSource()方法,可以监听另一个或多个LiveData数据源变化
### 3.ViewModel
ViewModel相当于一层数据隔离层，将UI层的数据逻辑全部抽离干净，管理底层数据的获取方式和逻辑
```java
  ViewModel viewModel = ViewModelProviders.of(this).get(CkrModel.class);
```
### 4.Room
Room是一种ORM(对象关系映射)模式数据库框架,对android SQLite的抽象封装。Room用法：
* 继承RoomDatabase的抽象类，暴露抽象方法getCkrDao()
```java
  @Database(entities={BannerEntity.class,UserInfo.class},version=1)
  @TypeConverters(DataConverter.class)
  public abstract class AppDB extends RoomDatabase{
    private static AppDB instance;
    @VisibleForTesting
    public static final String TABLE_NAME="ckr.db";
    public abstract LoginDao loginDao();
  }
```
* 获取db实例
```java
  AppDB db=Room.databaseBuilder(mContext,AppDB.class,TABLE_NAME).build();
```
* 实现dao层逻辑
```java
  @Dao
  public interface UserInfoDao{
    @Query("SELECT * FROM user_info WHERE id = :id")
    Flowable<UserInfo> loadUserInfo(String id);
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    void insertUserInfo(UserInfo userInfo);
  }
```
* 添加一张表结构
```java
  @Entity(tableName="user_info")
  public class UserInfo{
    @PrimaryKey
    private int uid;
    
    @ColumnInfo(name="first_name")
    private String fisrtName;
    
    public String date;//默认columinfo为date
  }
```
## 感谢
[Lifecycle+Retrofit+Room](http://www.canking.win/2017/12/09/mvvm/)



















