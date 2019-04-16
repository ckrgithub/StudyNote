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
# junit测试框架
JUnit是由Erich Gamma 和 Kent Beck编写的一个回归测试框架。Junit测试是程序员测试，即所谓白盒测试，因为程序员知道被测试的软件如何完成功能和完成什么样的功能。
## 怎么用
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
# 枚举
枚举是一个特殊的java类，用关键字enum表示，不用class
## 作用
枚举可以限定外部能使用的类的对象，从而提高程序的安全性
## 怎么用
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
### 枚举常用的特有方法
* String name()返回此枚举常量的名称，在其枚举声明中对其进行声明。如：Print.A.name()=="A";
* int ordinal()返回此枚举常量的索引(它在枚举声明中的位置，其中初始常量序数为零)。public static <T extends Enum<T>> T valueOf(Class<T> enumType,String name)返回带指定名称的指定枚举类型的枚举常量。
### 特点
* 枚举构造方法必须私有
* 声明枚举，必须在类里的首行，枚举间用逗号隔开，如果还有其它代码则最后一个枚举后必须加分号
* 枚举有类的特性，可以声明属性和方法，也可以实现接口和继承抽象类
* JDK1.5后，switch可以接收枚举类型
* 如果枚举只有一个值时，可以当单例用。

# 反射机制
Java反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制
## 怎么用
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
### 获取构造方法得到对象
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
### 获取和改变成员字段的属性
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
### 获取和运行成员方法
```java
  Class clazz=Class.forName("com.ckr.bean.Person");
  Method method=clazz.getMethod("paramMethod",String.class,int.class);
  Oject obj=clazz.newInstance();
  method.invoke(obj,"ckr",18);
  Method[] methods=clazz.getMethods();//获取的都是公有的方法的数组
  method.invoke(clazz.newInstance(),url,username,password);
```

# 面向对象的特征
* 1.抽象：抽象是将一类对象的共同特征总结出来构造类的过程，包括数据抽象和行为抽象两方面。抽象只关注对象有哪些属性和行为，并不关注这些行为的细节是什么
* 2.继承：继承是从已有类得到继承信息创建新类的过程。提供继承信息的类被称为父类；得到继承信息的类被称为子类。
* 3.封装：通常认为封装是把数据和操作数据的方法绑定起来，对数据的访问只能通过已定义的接口。
* 4.多态：多态是指允许不同子类型的对象对同一消息作出不同的响应。多态性分为编译时的多态性和运行时的多态性。方法重载(overload)实现的是编译时的多态性(也称为前绑定)，而方法重写(override)实现的是运行时的多态性(也称为后绑定)。

# 访问修饰符public,protected,private,以及不写(默认)时的区别
|作用域|当前类|同包|子类|其他|
|---|---|---|---|---|
|public|y|y|y|y|
|protected|y|y|y|n|
|default|y|y|x|x|
|private|y|x|x|x|

类的成员不写访问修饰时默认为default。默认对于同一个包中的其他类相当于公开(public),对于不是同一个包中的其他类相当于私有(private)。受保护(protected)对子类相当于公开，对于不是同一包中的没有父子关系的类相当于私有

# String是最基本数据类型么
不是。java中基本数据类型只有8个：byte,short,int,long,float,double,boolean,char;除了基本类型和枚举类型，剩下的都是引用类型

# float f=3.4是否正确
不正确。3.4是双精度数，将双精度型赋值给浮点型属于下转型，会造成精度损失，因此需要强制类型转换float f=3.4F;

# short s1=1;s1=s1+1;有错么？short s1=1;s1 +=1;有错么
对于s1=s1+1;由于1是int类型，因此s1+1运算结果也是int类型，需要强制转换类型才能赋值给short类型。而s1 +=1;可以正确编译，因为s1 +=1相当于s1=(short)(s1+1);其中有隐含的强制类型转换.

# int和integer有什么区别
java是一个近乎纯洁的面向对象编程语言，但是为了编程的方便还是引入不是对象的基本数据类型，但是为了能够将这些基本数据类型当成对象操作，java为每个基本数据类型都引入了对应的包装类型(wrapper class)，int包装类就是integer，从jdk1.5开始引入自动装箱/拆箱机制，使得两者可以相互转换。
```java
  public static void main(String[] args){
    Integer a=new Integer(2);
    Integer b=2;
    int c=2;
    System.out.println(a==b);//false，两个引用没有引用同一对象
    System.out.println(a==c);//true，a自动拆箱成int类型，再和c比较
    
    Integer f1=100,f2=100,f3=150,f4=150;
    System.out.println(f1==f2);//true
    System.out.println(f3==f4);//false
  }

```
首先注意f1,f2,f3,f4四个变量都是Integer对象，所以下面的==运算比较的不是值而是引用。装箱的本质是什么呢？当我们给一个Integer对象赋一个int值的时候，会调用Integer类的静态方法valueOf：
```java
  public static Integer valueOf(int i){
    if(i>=IntegerCache.low&&i<=IntegerCache.high){
      return IntegerCache.cache[i+(-IntegerCache.low)]
    }
    return new Integer(i);
  }

```
如果字面量的值在-128到127之间，那么不会new新的Integer对象，而是直接引用常量池中的Integer对象.

# &和&&的区别
&运算符有两种用法：按位与；逻辑与。&&运算符是短路与运算。&&之所以称为短路运算，是因为如果&&左边的表达式的值是false，右边的表达式会被直接短路掉，不会进行运算。

# 解释内存中的栈(stack)、堆(heap)和静态存储区的用法。
通常定义一个基本数据类型的变量，一个对象的引用，还有函数调用的现场保存都使用内存中的栈空间；而通过new 关键字和构造器创建的对象放在堆空间；程序中的字面量,如直接书写的100，"hello"和常量都是放在静态存储区中。栈空间操作最快但也很小，通常大量的对象都是放在堆空间，整个内存包括硬盘上的虚拟内存都可以被当成堆空间来使用。
```java
  String str= new String("hello");
  //str放在栈上，用new创建出来的字符串对象放在堆上，而"hello"这个字面量放在静态存储区。
```

# Math.round(11.5)等于多少？Math.round(-11.5)等于多少？
Math.round(11.5)返回值是12，Math.round(-11.5)返回值是-11.四舍五入原理是在参数上加0.5,然后进行下取整。

# switch是否能作用在byte上，是否能作用在long上，是否能作用在String上？
早起JDk中，switch(expr)中，expr可以是byte,char,short,int。从1.5版本开始，引入枚举类型，expr也可以是枚举，从jdk1.7开始，可以是String。长整型(long)是不可以的

# 用最有效率的方法计算2乘以8
2<<3(左移3相当于乘以2的3次方)。我们为编写类重写hashCode方法时，不太理解为什么要使用这样的乘法运算来产生哈希码，而且为什么这个数是个素数，为什么通常选择31这个数？选择31是因为可以用移位和减法运算来代替乘法，从而得到更好的性能。如：31*num--> (num<<5)-num

# 数组有没有length()方法？String有没有length()方法？
数组没有length()方法，有length属性。String有length()方法。

# 在java中，如何跳出当前的多重嵌套循环？
在最外层循环加一个标记如A,然后用break A;可以跳出多重循环。

# 构造器(constructor)是否可以被重写(override)?
构造器不能被继承，因此不能被重写，但可以被重载。

# 两个对象值相同(x.equals(y)==true),但却可有不同的hash code,这句话对不对
不对，如果两个对象x和y满足x.equals(y)==true，它们的哈希码应当相同。Java对于equals方法和hashCode方法是这样规定的：
* 1.如果两个对象相同(equals方法返回true),那么它们的hashCode值一定要相同；
* 2.如果两个对象的hashCode相同，它们并不一定相同。
首先equals方法必须满足自反性(x.equals(x)必须返回true)、对称性(x.equals(y)返回true时，y.equals(x)也必须返回true)、传递性(x.equals(y)和y.equals(z)返回true时，x.equals(z)也必须返回true)和一致性(当x和y引用的对象信息没有被修改时，多次调用x.equals(y)应该得到同样的返回值)，而且对于任何非null值的引用x，x.equals(null)必须返回false.实现高质量的equals方法的诀窍包括：
  * 使用==操作符检查"参数是否为这个对象的引用"
  * 使用instanceOf操作符检查"参数是否为正确的类型"
  * 对于类中的关键属性，检查参数传入对象的属性是否与之匹配
  * 编写完equals方法后，问自己是否满足对称性、传递性、一致性
  * 重写equals时总是要重写hashCode;
  * 不要将equals方法参数中的Object对象替换为其他类型，在重写时不要忘记@Override注解

# 是否可以继承String类
不可以。因为String类是final类

# 当一个对象被当作参数传递到一个方法后，此方法可改变这个对象的属性，并可返回变化后的结果，那么这里到底是值传递还是引用传递？
是值传递。java编程语言只有值传递参数。当一个对象实例作为一个参数被传递到方法时，参数的值就是对该对象的引用。对象的属性可以在被调用过程中改变，但对象的引用是永远不会改变的。

# String和StringBuilder、StringBuffer的区别
String、StringBuilder、StringBuffer可以储存和操作字符串。其中，String是只读字符串，意味着String引用的字符串内容是不能被改变的。而StringBuffer和StringBuilder类表示的字符串对象可以直接进行修改。StringBuilder是JDK1.5中引入的，它和StringBuffer的方法完全相同，区别在于它是在单线程环境下使用的，因为它的所有方面没有被synchronized修饰，因此它的效率也比StringBuffer略高。补充：如果连接后得到的字符串在静态存储区中是早已存在的，那么用+做字符串连接是优于StringBuffer/StringBuilder的append方法的。
```java
  public static void main(String[] args){
    String a="Programming";
    String b=new String("Programming");
    String c="Program"+"ming";
    System.out.println(a==b);//false
    System.out.println(a==c);//true
    System.out.println(a.equals(b));//true
    System.out.println(a.equals(c));//true
    System.out.println(a.intern()==b.intern());//true
  }
```

# 重载(overload)和重写(override)的区别。重载方法能否根据返回类型进行区分？
方法的重载和重写都是实现多态的方式，区别在于前者实现的是编译时的多态性，而后者实现的是运行时的多态性。重载发生在一个类中，同名的方法如果有不同的参数列表(参数类型不同、参数个数不同或者二者都不同)则视为重载；重写发生在子类和父类之间，重写要求子类被重写方法与父类被重写方法有相同的返回类型，比父类被重写方法更好访问，不能比父类被重写方法声明更多的异常。重载对返回类型没有特殊要求。

# 描述一下jvm加载class文件的原理机制
JVM中类的装载是由类加载器(ClassLoader)和它的子类来实现的，Java中的类加载器是一个重要的java运行时系统组件，他负责在运行时查找和装入类文件中的类。
* 由于java的跨平台性，经过编译的java源程序并不是一个可执行程序，而是一个或多个类文件。当java程序需要使用某个类时，JVM会确保这个类已经被加载、连接(验证、准备和解析)和初始化。类的加载是指把类的.class文件中的数据读入到内存中，通常是创建一个字节数组读入.class文件，然后产生与所加载类对应的Class对象。加载完后，Class对象还不完整，所以此时的类还不可用。当类被加载后就进入连接阶段，这一阶段包括验证、准备(为静态变量分配内存并设置默认的初始值)和解析(将符号引用替换为直接引用)三个步骤。最后JVM对类进行初始化，包括：1.如果类存在直接的父类并且这个类还没有被初始化，那么就先初始化父类；2.如果类中存在初始化语句，就依次执行这些初始化语句。
* 类的加载是由类加载器完成的，类加载器包括：根加载器(BootStrap)、扩展加载器(Extension)、系统加载器(System)和用户定义类加载器(java.lang.ClassLoader的子类)。从jdk1.2开始，类加载过程采取了父亲委托机制(PDM)。PDM更好保证了java平台的安全性，在该机制中，JVM自带的BootStrap是根加载器，其他加载器都有且仅有一个父类加载器。类的加载首先请求父类加载器加载，父类加载器无能为力时，才由其子类加载器自行加载。

# char型变量中能不能存储一个中文汉字？为什么
char类型可以存储一个中文汉字，因为java中使用的编码是Unicode(不选择任何特定编码，直接使用字符在字符集中的编号，这是统一的唯一方法)，一个char类型占2个字节(16bit),所以放一个中文是没问题的。使用Unicode意味着字符在jvm内部和外部有不同的表现形式，在jvm内部都是unicode,当这个字符被从jvm内部转移到外部时，需要进行编码转换。所以java中有字节流和字符流，以及在字符流和字节流之间进行转换的转换流，如：InputStreamReader和OutputStreamReader，这两个类时字节流和字符流之间的适配器类，承担了编码转换的任务；

# 抽象类和接口有什么异同？
* 1.抽象类和接口都不能实例化，但可以定义抽象类和接口类型的引用。一个类如果继承了某个抽象类或实现了某个接口都需要对其中的抽象方法全部进行实现，否则该类任然需要被声明为抽象类。
* 2.接口比抽象类更加抽象，因为抽象类中可以定义构造器，可以有抽象方法和具体方法，而接口中不能定义构造器而且其中的方法全部都是抽象方法。
* 3.抽象类中的成员可以是private、默认、protected、public的，而接口中的成员全都是public的。
* 4.抽象类中可以定义成员变量，而接口中定义的成员变量实际上都是常量。有抽象方法的类必须被声明为抽象类，而抽象类未必要有抽象方法。

# 静态嵌套类和内部类的不同
静态嵌套类是被声明为静态的内部类，它可以不依赖于外部类实例被实例化。而通常内部类需要在外部类实例化后才能实例化。

# java中会存在内存泄漏么
理论上java因为有垃圾回收机制不会存在内存泄漏问题。然而实际开发中，可能会存在无用但可达的对象，这些对象不能被GC回收也会发生内存泄漏。
```java
  public class MyStack<T>{
    private T[] elements;
    private int size=0;
    private static final int INIT_CAPACITY=16;
    
    public MyStack(){
      elements=(T[])new Object[INIT_CAPACITY];
    }
    public void push(T elem){
      ensureCapacity();
      elements[size++]=elem;
    }
    public T pop(){
      if(size==0){
        throw new EmptyStackException();
      }
      return elements[--size];
    }
    private void ensureCapacity(){
      if(elements.length==size){
        elements=Arrays.copyOf(elements,2*size+1);
      }
    }
  }
```
上面实现了一个栈(先进后出)结构，其中pop方法存在内存泄漏的问题，当我们用pop方法弹出栈中的对象时，该对象不会被当作垃圾回收，即使使用栈的程序不再引用这些对象，因为栈内部维护着对这些对象的过期引用。

# 抽象的方法是否可同时是静态的，是否可同时是本地方法，是否可同时被synchronized修饰
都不能。抽象方法需要子类重写，而静态的方法是无法被重写的，因此二者是矛盾的。本地方法是由本地代码(如：c代码)实现的方法，而抽象方法是没有实现的，也是矛盾。synchronized和方法的实现细节有关，抽象方法不涉及实现细节，因此也相互矛盾。

# 静态变量和实例变量的区别
静态变量时被static修饰的变量，也称为类变量，它属于类，不属于类的任何一个对象。实例变量必须依存于某个实例，需要先创建对象然后通过对象才能访问到它。

# 是否可以从一个静态方法内部发出对非静态方法的调用
不可以，静态方法只能访问静态成员，因为非静态方法的调用先创建对象，因此在调用静态方法时可能对象并没有被初始化。

# 如何实现对象clone
* 1.实现Cloneable接口并重写Object类中的clone()方法；
* 2.实现Serializable接口，通过对象的序列化和反序列化实现clone,可以实现真正的深度克隆
```java
  public static <T> T clone(T obj) throws Exception{
    ByteArrayOutputStream baos=new ByteArrayOutputStream();
    OjectOutputStream oos=new ObjectOutputStream(baos);
    oos.writeObject(obj);
    
    ByteArrayInputStream bais=new ByteArrayInputStream(baos.toByteArray());
    ObjectInputStream ois=new ObjectInputStream(bais);
    return (T)ois.readObject();
  }
```
注意：基于序列化和反序列化实现的克隆不仅仅是深度克隆，更重要的是通过泛型限定，可以检查出要克隆的对象是否支持序列化，这项检查是编译器完成的，不是在运行时抛出异常。

# GC是什么？为什么要有GC
GC是垃圾收集的意思，内存处理是编程人员容易出现问题的地方，忘记或错误的内存回收会导致程序或系统的不稳定甚至崩溃，java提供的GC功能可以自动监测对象是否超过作用域从而达到自动回收内存的目的。垃圾回收可以有效的防止内存泄漏，有效的使用可以使用的内存。垃圾回收器通常是作为一个单独的低优先级的线程运行，不可预知的情况下对内存堆中已经死亡的或者长时间没有使用的对象进行清除和回收。垃圾回收机制有很多种，包括：分代复制垃圾回收、标记垃圾回收、增量垃圾回收。java平台对堆内存回收和再利用的基本算法被称为标记和清除，但java对其进行了改进，采用"分代式垃圾收集"。这种方法会跟java对象的生命周期将堆内存划分为不同的区域：
* 伊甸园(Eden):对象最初诞生的区域
* 幸存者乐园(Survivor):从伊甸园幸存下来的对象会被挪到这里
* 终身颐养园(Tenured):这是足够老的幸存对象的归宿。年轻代收集(Minor-GC)过程不会触及这个地方。当年轻代收集不能把对象放进终身颐养园时，就会触发一次完全收集(Major-GC)。

与垃圾回收相关的JVM参数：
* -Xms/-Xmx :堆的初始大小/堆的最大大小
* -Xmn:堆中年轻代的大小
* -XX:-DisableExplicitGC：让System.gc()不产生任何作用
* -XX:+PrintGCDetail: 打印GC的细节
* -XX:+PrintGCDateStamps: 打印GC操作的时间戳

# String s=new String("xyz");创建了几个字符串对象
两个对象，一个是静态存储区的"xyz",一个是用new创建在堆上的对象

# 接口是否可继承接口？抽象类是否可实现接口？抽象类是否可继承具体类？
接口可以继承接口。抽象类可以实现接口，抽象类可继承具体类，但前提是具体类必须有明确的构造函数。

# 一个".java"源文件中是否可以包含多个类(不是内部类)？有什么限制
可以，但一个源文件中最多只能有一个公开类，而且文件名必须和公开类的类名完全保持一致。

# 匿名内部类是否可以继承其它类？是否可以实现接口
可以继承其它类或实现其它接口。

# 内部类可以引用它的外部类的成员么？有没有什么限制
一个内部类对象可以访问创建它的外部类对象的成员，包括私有成员。

# java中的final关键字有哪些用法
* 修饰类：表示该类不能被继承
* 修饰方法：表示方法不能被重写
* 修饰变量：表示变量只能一次赋值以后值不能被修改(常量);

# 指出下面程序运行结果
```java
  class A{
    static{
      System.out.print("1");
    }
    public A(){
      System.out.print("2");
    }
  }
  class B extends A{
    static{
      System.out.print("a");
    }
    public B(){
      System.out.print("b");
    }
  }
  public class Hello{
    public static void main(String[] args){
      A ab=new B();
      ab=new B();
    }
  }
```
执行结果：1a2b2b。创建对象时构造器的调用顺序是：先初始化静态成员，然后调用父类构造器，在初始化非静态成员，最后调用自身构造器。

# 数据类型之间的转换
如何将字符串转换为基本数据类型？如何将基本数据类型转换为字符串？  
调用基本数据类型对应的包装类中的方法parseXXX(String)或valueOf(String)















