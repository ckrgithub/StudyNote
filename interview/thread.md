# 创建线程的三种方式
## 继承Thread类创建线程类
* 重写run()方法
* 创建该类对象，并调用start()方法
```java
  public class CkrThread extends Thread{
    private static final String TAG="CkrThread";
    
    public void run(){
      System.out.println(TAG+"->"+"run")
    }
    public static void main(String[] args){
      for(int i=0;i<10;i++){
        System.out.println(Thread.currentThread().getName()+":"+i);
        if(i==2){
          new CkrThread().start();
          new CkrThread().start();
        }
      }
    }
  }

```
## 实现Runnable接口创建线程类
* 重写run()方法
* 创建Thread对象，并闯入Runnable对象，调用start()方法
```java
  public class CkrRunnable implements Runnable{
    public void run(){
      System.out.println("CkrRunnable->run");
    }
    public static void main(String[] args){
      for(int i=0;i<10;i++){
        System.out.println(Thread.currentThread().getName()+":"+i);
        if(i==2){
          CkrRunnable cr=new CkrRunnable();
          new Thread(cr,"新线程1").start();
          new Thread(cr,"新线程2").start();
        }
      }
    }
  }
```
## 实现Callable创建线程类
* 实现call()方法，该call()方法作为线程执行体，并有返回值
* 使用FutureTask类来包装Callable对象
* 创建Thread对象，传入FutureTask对象并调用start()方法。
* 调用FutureTask对象的get()方法来获得子线程的返回值
```java
  public class CkrCallable implements Callable<Integer>{
    public Integer call() throws Exception{
      System.out.println("CkrCallable->call");
      return 0;
    }
    public static void main(String[] args){
      CkrCallable cc=new CkrCallable();
      FutureTask<Integer> task=new FutureTask<>(cc);
      for(int i=0;i<10;i++){
        System.out.println(Thread.currentThread().getName()+":"+i)
        if(i==2){
          new Thread(task,"有返回值的线程").start();
        }
      }
      try{
        System.out.println("子线程返回值："+task.get());
      }catch(Exception e){
        e.printStackTrace();
      }
    }
  }
```























