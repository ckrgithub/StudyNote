# RxJava 2.x
## 一、概念
Rx是ReactiveX的缩写，而ReactiveX是Reactive Extensions的缩写。Rxjava顾名思义即是Java上的异步和基于事件响应式编程库。
RxJava基于观察者模式，主要有四个部分：观察者、被观察者、订阅、事件。
```java
  FlowableSubscriber<String> subscriber = new FlowableSubscriber<String>(){
    @Override
    public void onSubscribe(Subscription s){
      //订阅时候的操作
      s.request(Long.MAX_VALUE);//请求多少事件，这里表示不限制
    }
    @Override
    public void onNext(String s){
      Log.i("tag",s);
    }
    @Override
    public void onError(Throwable t){
      
    }
    @Override
    public void onComplete(){
    
    }
  }
  由此可见，观察者身上有onNext、onError、onComplete事件
  Flowable<String> flowable = Flowable.create(new FlowableOnSubscribe<String>(){
    @Override
    public void subscribe(FlowableEmitter<String> e) throw Exception {
      e.onNext("test1");
      e.onNext("test2");
      e.onComplete();
    }
  },BackpressureStrategy.BUFFER);//背压设置
  flowable.subsribe(subscriber);
  输出： test1->test2
```
### 1.Actions
#### Consumer
从上面可以看到FlowableSubscriber中我们只关心onNext方法，那么可以用Consumer来作观察者：
```java
  flowable.subscribe(new Consumer<String>(){
    @Override
    public void accept(String s){
      //相当于onNext事件处理
      Log.i("tag",s);
    }
  });
  或
  flowable.subscribe(new Consumer<String>(){
    @Override
    public void accept(String s) throws Exception{//onNext
    }
  },new Consumer<Throwable>(){//相当于onError
    @Override
    public void accept(Throwable throwable) throws Exception{
    }
  },new Action(){//相当于onComplete，注意这里是Action
    @Override
    public void run() throws Exception{
    }
  },new Consumer<Subscription>(){//相当于onSubscribe
    @Override
    public void accept(Subscription subscription) throws Exception{
    }
    }
  }
  其中，Action和Consumer都属于Actions,不过Action属于无参数类型，Consumer属于单一参数类型。多参数类型使用：  
  BigConsumer<T1,T2>: 双参数类型  
  Consumer<Object[]>: 多参数类型  
```
### 2.Observable和Observer
```java
  Observable<String> observale = Observable.create(new ObservableOnSubscribe<String>(){
    @Override
    public void subscribe(ObservableEmitter<Stirng> e) throws Exception{
      e.onNext("text1);
      e.onNext("text2");
      e.onComplete();
    }
  });
  由于Observable不支持订阅Subscriber观察者，需要使用Observer作为观察者
  Observer<String> observer = new Oberser<String>(){
    @Override
    public void onSubscribe(Disposable d){
      //订阅时操作，无需request
    }
    @Override
    public void onNext(String s){
      Log.i("observer",s);
    }
    @Override
    publid void onError(Throwable e){
    }
    @Override
    public void onComplete(){
    }   
  };
  observable.subscribe(observer);
```
### 3.Observable和Flowable区别
以上发现，Observable和Flowable,前者不需要背压参数和请求资源操作。
使用Observable: 不超过1000个元素、随着时间流逝基本不会oom;不支持Java Steam(Java 8新特性）；开销比Flowable低
使用Flowable: 超过10K个元素;读写硬盘操作；通过JDBC读取数据库；网络IO操作
### 4.BackPressure
背压就是生产者的生产速度大于消费者消费速度从而导致的问题
### 5.Single和SingleObserver
单一事件流，即只有一个onNext事件，接着就触发onComplete或者onError
Single只包含两个事件，一个是正常处理成功的onSuccess，另一个是处理失败的onError,它只发送一次消息
```java
  Single<String> single = Single.create(new SingleOnSubscriber<String>(){
    @Override
    public void subscribe(SingleEmitter<String> e) throws Exception{
      e.onSuccess("text1");
      e.onSuccess("test3");//错误写法，重复调用不会处理
    }
  });
  single.subscribe(new SingleObserver<String>(){
    @Override
    publid void onSubscribe(Disposable d){
    }
    @Override
    public void onSuccess(String s){
      //相当于onNext和onComplete
    }
    @Override
    publid void onError(Throwable e){}
  });
  可用Actions简化Obverser：
  single.subscribe(new BigConsumer<String,Throwable>(){
    @Override
    public void accept(String s,Throwable throwable) throws Exception{
      //onSuccess和onError操作都在这里
    }
  });
```
### 6.Completable和CompletableObserver
如果观察者连onNext事件都不关心，可以使用Completable,它只有onComplete和onError两个事件：
```java
  Completable.create(enw CompletableOnSubscribe(){
    @Override
    public void subscribe(CompletableEmitter e) throws Exception {
      e.complete();//单一onComplete或者onError
    }
  }).subscribe(new CompletableObserver(){
    @Override
    public void onSubscribe(Disposable d){}
    @Override
    public void onComplete(){}
    @Override
    public void onError(Throwable e){}
  });
```
### Maybe和MaybeObserver
如果你有一个需求是可能发送一个数据或者不会发送任何数据，这时候你就需要Maybe,它类似于Single和Completable的混合体
Maybe可能会调用以下其中一种情况：
onSuccess或者onError
onComplete或者onError
```java
  Maybe<String> maybe = Maybe.create(new MaybeOnSubscribe<String>(){
    @Override
    public void subscribe(MaybeEmitter<String> e) throws Exception{
      e.onSuccess("test");//发送一个数据的情况，或者onError
      //e.onComplete();//不需要发送数据的情况，或者onError
    }
  }).subscribe(new MaybeObserver<String>(){
    @Override
    public void onSubscribe(Disposable d){}
    @Override
    public void onSuccess(String s){
      Log.i("tag",s);
    }
    @Override
    public void onComplete(){
      Log.i("tag","onComplete");
    }
    @Override
    public void onError(Thrwoble e){}
  });
``` 
## 二、操作符
### 1.just
```java
  Flowable.just("test","test2")
          .subscribe(str -> Log.i("tag",str));
  相当于顺序调用onNext("test")和onNext("test2"),让后调用onComplete()方法;另外，还可以使用
  Observable/Single/Maybe来调用这个操作符，但Completable不能使用(因为没有onNext事件)。对于Flowable和Observable最多能接受10个参数，而Single和Maybe只能接受1个参数(只能发送一次onNext事件)。
```
### 2.fromArray
fromArray可以接受任意长度的数据数组
```java
  Flowable.fromArray(1,2,3,4,5)
          .subscribe(num -> Log.i("tag",String.valueOf(num)));
  fromArray可以直接传入一个数组，如：fromArray(new int[]{1,2,3}),但不要直接传递list集合
```
### 3.empty
不会发送任何数据，而是直接发送onComplete事件
```java
  Flowable.empty().subscribe(
    obj -> Log.i("tag","onNext:"+obj.toString()),
    e -> Log.i("tag","onError:"+e),
    () -> Log.i("tag","onComplete")
  );
  只会输出onComplete,其他回调不会触发
```
### 4.error
```java
  Flowable.error(new RuntimeException("test")).subscribe(
    obj -> Log.i(TAG,"onNext:"+obj.toString()),
    e -> Log.i(TAG,"onError:"+e),
    () -> Log.i(TAG,"onComplete")
  );
  只会输出onError,其他回调不会触发
```
### 5.never
什么都不会发送的操作符never,也不会触发观察者任何的回调：
```java
  Flowable.never().subscribe(
    obj -> Log.i(TAG,"onNext:"+obj.toString()),
    e -> Log.i(TAG,"onError"),
    () -> Log.i(TAG,"onComplete")
  );
  不会输出任何log
```
### 6.fromIterable
可以遍历可迭代数据集合：
```java
  List<String> list = new ArrayList<>();
  list.add("a");
  list.add("b");
  list.add("c");
  Flowabel.fromIterable(list).subscribe(
    s -> Log.i(TAG,"s:"+s)
   );
   顺序输出：a -> b -> c
```
### 7.timer
时间间隔操作符，可以指定一段时间发送数据(固定值0L):
```java
  Flowable.timer(1,TimeUnit.SECONDS)
          //这里s是0,1,2,3....
          .subscribe(s -> Log.i(TAG,String.valueOf(s)));
  延迟1秒后调用onNext(0)，然后调用onComplete()事件
```
### 8.interval
不断地发送数据：
```java
  Flowable.interval(3,1,TimeUnit.SECONDS)
          .subscribe(s -> Log.i(TAG,String.valueOf(s)));
  第一个参数是第一次延迟，第二个参数是间隔
```
### 9.intervalRange
指定发送范围：
```java
  Flowable.intervalRange(1,10,3,1,TimeUnit.SECONDS)
          //x从1到10，初始间隔2秒，之后间隔1秒发送一次
          .subscribe(s -> Log.i(TAG,String.valueOf(s)));
  注意，当x从1开始发送到10后(参数10是发送10个数量的意思，类似于request(10)操作)调用onComplete方法,从最后一个元素发出dao9onComplete之间并不会有period长度的延迟
```




















## 官谢
[Rxjava 2.x使用详解](https://maxwell-nc.github.io/android/rxjava2-1.html)
