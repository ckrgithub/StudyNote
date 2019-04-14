# Annotation
Annotation注解是代码里的特殊标记，是java5.0后的一个新特性。
## 作用
注解的主要作用是对类或类的成员进行功能的解释及限制。二是可以用注解替换配置文件，也就是说可以在注解里定义一些配置信息，然后通过反射技术去得到类里的注解，以决定怎么去运行类。
## 怎么用
### 三个基本的注解
1.@Override：限定重写父类方法
```java
  @Override
  public String toString(){
    return "";
  }
```
2.@Deprecated：用于表示某个程序元素(类，方法等)已过时
```java
  @Deprecated
  public List<Object> findAll(){
    return null;
  }
```
3.@SuppressWarnings：抑制编译器警告
```java
  @SuppressWarnings("rawtypes")
  public List findRecords(){
    return null;
  }
```
### 元注解
元注解就是jdk定义的只能修饰注解的注解
#### @Retention
@Retention:只能用于修饰一个Annotation定义，用于指定该Annotation可以保留的域。@Retention包含一个RetentionPolicy类型的成员变量，通过这个变量指定域，指示该注解存在的范围是什么
* RetentionPolicy.CLASS:编译器将把注释记录在class文件中。当运行java程序时，JVM不会保留注释。这是默认值
* RetentionPolicy.RUNTIME: 编译器将把注释记录在class文件中。当运行java程序时，JVM会保留注释。程序可以通过反射获取该注释
* RetentionPolicy.SOURCE: 编译器直接丢弃这种策略的注释

#### @Target
指定注解用于修饰类的哪个成员。@Target包含了一个名为value的成员变量
* @Documented: 用于指定该元Anontation修饰的Annotation类将被javadoc工具提取成文档
* @Inherited: 被它修饰的Annotation将具有继承性。如果某个类使用了被@Inherited修饰的Annotation，则其子类将自动具有该注解

### 自定义注解
注解用@interface关键字。声明注解属性：注解属性只有8种基本数据类型，及String class注解类型，emum枚举类型及以上类型的一维数组
```
@Annotation1(name="bbb",value="vvv")
@Annotation2(name="ccc",value={"ddd","eee","fff"})
```
定义属性的默认值：在()后用default标识
```java
  @Target(ElementType.METHOD)
  @Retention(RetentionPolicy.RUNTIME)
  public @interface DbInfo{
    String url() default "";
    String username() default "ckr";
    String password() default "123456";
  }
```
在类上引用自定义的注解
```java
  public class JdbcUtils{
    @DbInfo(url="https://github.com",username="ckr",passwor="1234")
    public static Connection getConnection(String url,String username,String password){
      System.out.println(url);
      System.out.println(username);
      System.out.println(password);
      return null;
    }
  }
```
通过反射引用注解在方法中运用
```java
  public static void main(String[] args) throws Exception{
    //通过反射获得类的方法对象
    Class<?> clazz=Class.forName("com.ckr.JdbcUtils");
    Method mt=clazz.getMethod("getConnection",String.class,String.class,String.class);
    //通过方法对象获得注解
    DbInfo info=mt.getAnnotation(DbInfo.class);
    String url=info.url();
    String username=info.username();
    String password=info.password();
    //用方法对象运行方法
    mt.invoke(clazz.newInstance(),url,username,password);
  }
```
## junit测试框架
JUnit是由Erich Gamma 和 Kent Beck编写的一个回归测试框架。Junit测试是程序员测试，即所谓白盒测试，因为程序员知道被测试的软件如何完成功能和完成什么样的功能。
### 怎么用
测试一个方法是否正常
```java
  import org.junit.After;
  import org.junit.AfterClass;
  import org.junit.Before;
  import org.junit.BeforeClass;
  import org.junit.Test;
  
  public class Test15{
    @Test
    public void print(){
      char c=(char)65;
      System.out.println(c);
    }
  }
```
2.在每个方法测试前后都运行自定义方法
```java
  char c;
  @Before//方法运行前先执行的方法
  public void before(){
    c=(char)65;
  }
  @Test
  public void print(){
    System.out.println(c);
  }
  @After//方法运行后
  void after(){
    System.out.println("after");
  }
  
```
3.在类加载前后运行，注意：都要是static
```java
static char c;
@BeforeClass//只在类加载前运行一次
public static void before(){
  c=(char)65;
}
@Test
public void print(){
  System.out.println(c);
}
AfterClass//类加载后运行一次
public static void after(){
  System.out.println("after");
}
```
4.断言：判断某个方法的结果是不是你想要的，第一个参数表示你期望的结果
```java
  @Test
  public void test(){
    Print p=new Print();
    Assert.assertEquals("a",p.print());
  }
  
  class Print{
    public String print(){
      return "a";
    }
  }
```
5.测试方法运行的时间
```java
  @Test(timeout=200)
  public void testFindUser(){
    User user=dao.findUser("张三","123");
    assertNotNull(user);
    user=dao.findUser("张三","22");
    assertNull(user);
    user=dao.findUser("22","123");
    assertNull(user);
    user=dao.findUser("22","13");
    assertNull(user);
  }
```
## 枚举
枚举是一个特殊的java类，用关键字enum表示，不用class
### 作用
枚举可以限定外部能使用的类的对象，从而提高程序的安全性
### 怎么用
外部直接用类名打点调获取对象，并调用类成员
```java
  //不带构造方法的枚举
  enum Print{
    A,B,C;
  }
  //带构造方法的枚举
  enum Print{
    A("a"),B("b"),C("c");
    String s;
    private Print(String s){
      this.s=s;
    }
  }

```
#### 枚举常用的特有方法
* String name()返回此枚举常量的名称，在其枚举声明中对其进行声明。如：Print.A.name()=="A";
* int ordinal()返回此枚举常量的索引(它在枚举声明中的位置，其中初始常量序数为零)。public static <T extends Enum<T>> T valueOf(Class<T> enumType,String name)返回带指定名称的指定枚举类型的枚举常量。
#### 特点
* 枚举构造方法必须私有
* 声明枚举，必须在类里的首行，枚举间用逗号隔开，如果还有其它代码则最后一个枚举后必须加分号
* 枚举有类的特性，可以声明属性和方法，也可以实现接口和继承抽象类
* JDK1.5后，switch可以接收枚举类型
* 如果枚举只有一个值时，可以当单例用。

## 反射机制
Java反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制
### 怎么用
获取字节码对象的三种方法：

1.Object类中的getClass()方法：必须明确具体的类，并创建对象
```java
  public static void getClassObject(){
    Person p=new Person();
    Class clazz = p.getClass();
  }
```
2.任何数据类型都具备一个静态的属性.class来获取其对应的Class对象
```java
  Class clazz=Person.class;
```
3.只要通过给定的类的字符串名称就可以获取该类
```java
  String className="com.ckr.bean.Person";
  Class clazz=Class.forName(className);
```
#### 获取构造方法得到对象
当只需要获得无参构造成的对象
```java
  String name="com.ckr.bean.Person";
  //寻找该名称类文件，并加载进内存，并产生Class对象
  Class clazz=Class.forName(name);
  //如何产生该类的对象
  Objcet obj=clazz.newInstance();
```
当想获取有参构造的对象
```java
  String name="com.ckr.bean.Person";
  Class clazz=Class.forName(name);
  //获取到指定的构造函数对象
  Constructor constructor=clazz.getConstructor(String.class,int.class);
  //或者可以用可变参数，也就是用一个数组也可实现
  //Constructor constructor=clazz.getConstructor(new Class[]{String.class,int.class});
  Object obj=constructor.newInstance("ckr",18);
```
#### 获取和改变成员字段的属性
公共(public)字段获取
```java
  Field field=null;
  field=clazz.getField("age");
```
非公共字段获取
```java
  Class clazz=Class.forName("com.ckr.bean.Person");
  field=clazz.getDeclaredField("age");//只获取本类，但包含私有
  //对私有字段的访问取消权限检查
  field.setAccessible(true);
  Object obj=clazz.newInstance();
  field.set(obj,18);
  Oject obj2=field.get(obj);//获取改过字段后的对象
```
#### 获取和运行成员方法
```java
  Class clazz=Class.forName("com.ckr.bean.Person");
  Method method=clazz.getMethod("paramMethod",String.class,int.class);
  Oject obj=clazz.newInstance();
  method.invoke(obj,"ckr",18);
  Method[] methods=clazz.getMethods();//获取的都是公有的方法的数组
  method.invoke(clazz.newInstance(),url,username,password);
```






















