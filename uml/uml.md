# UML
Unified Modeling Language(**UML**)又称**统一建模语言**或**标准建模语言**，是始于1997年一个OMG标准，它是一个支持模型化和软件系统开发的图形化语言，为软件开发的所有阶段提供模型化和可视化支持，包括由需求分析到规格，到结构和配置 。  
OMG：对象管理组织(Object Management Group)  
UML由3个要素构成：UML的基本构造块、支配这些构造块如何放置在一起的规则和运用于整个语言的公用机制。  
**UML有3种基本的构造块：事物、关系和图。**
* 事物：对模型中最具有代表性的成分的抽象，包括**结构事物**，如：类(class)、接口(interface)、协作(Collaboration)、用例(UseCase)、主动类(ActiveClass)、组件(Component)和节点(Node);**行为事物**，如交互(Interaction)、态机(Statemachine)、分组事物(包，Package)、注释事物(注解，Note).
* 关系：用来把事物结合在一起，包括：依赖、关联、泛化和实现关系
* 图：UML中有九种建模图形，即：**用例图、类图、对象图、顺序图、协作图、状态图、活动图、组件图、配置图**
## UML图
![](https://upload-images.jianshu.io/upload_images/7396903-13357350e07b1b48.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/324/format/webp)

### 用例图(Use Case Diagrams)
描述了作为一个外部的观察者的视角对系统的印象。强调这个系统是什么而不是这个系统怎么工作。用例图与情节紧紧相关的。用例图在三个领域很有作用：
    1.决定特征(需求)。当系统已经分析好并且设计成型时，新的用例产生新的需求；
    2.客户通讯。使用用例图很容易表示开发者与客户之间的联系。
    3.产生测试用例。一个用例的情节可能产生这些情节的一批测试用例。
![](https://upload-images.jianshu.io/upload_images/7396903-cb417b89199a3c93.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/498/format/webp)
### 类图(Class Diagram)
通过显示出系统的类以及这些类之间的关系来表示系统。类图是静态的--它们显示出什么可产生影响但不会告诉你什么时候产生影响。  
UML类的符号是一个被划分成三块的方框：类名、属性、操作。抽象类的名字，是斜体的。类之间的关系是连接线。
![](https://upload-images.jianshu.io/upload_images/7396903-ee1335e8410d3417.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/539/format/webp)  
接口是一种特殊的类。使用一个带有名称的小圆圈来表示: 
![](https://upload-images.jianshu.io/upload_images/7396903-6394bd2131d89181.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/154/format/webp)
* 类图有三种关系：
    1.关联association-表示两种类的实例间的关系。如果一个类的实例必须用另一个类的实例才能完成工作时就要用关联。
    2.聚合aggregation-当一个类属于一个容器。聚合用一个带菱形的连线，菱形指向具体整体性质的类。如：Order是OderDtails的容器。
    3.泛化generalization-一个指向以其他类作为超类的继承连线。泛化关系用一个三角形指向超类。
### 包和对象图
为了简单地表示出复杂的类图，可以把类组合成包packages。一个包是UML上有逻辑关系的元件的集合。
![](https://upload-images.jianshu.io/upload_images/7396903-75aef69969d5b27c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/430/format/webp)
### 顺序图
为交互图，是动态的(它们描述了对象间的交互作用)。顺序图将交互关系表示为一个二维图。纵向是时间轴，时间沿竖线向下延伸。横向代表在协作中各独立对象的类元角色。
![](https://upload-images.jianshu.io/upload_images/7396903-e6bbcdde7bc0238b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/547/format/webp)
### 协作图
也是互动的图表。它们像序列图一样也传递相同的信息，但它们不关心什么时候消息被传递，只关心对象的角色。
![](https://upload-images.jianshu.io/upload_images/7396903-263fb948400a309d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/658/format/webp)
### 状态图
对象拥有行为和状态。对象的状态是由对象当前的行动和条件决定的。状态图显示出对象可能的状态以及由状态改变而导致的转移。
![](https://upload-images.jianshu.io/upload_images/7396903-300d6907aefeef71.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/551/format/webp)  ![](https://upload-images.jianshu.io/upload_images/7396903-b2910c8346ae067d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/630/format/webp)
### 活动图
是一种很特别的流程图。活动图和状态图之间是有关系的。状态图把焦点集中在过程中的对象上，而活动图则集中在一个单独过程动作流程。
![](https://upload-images.jianshu.io/upload_images/7396903-86a8131120c6e419.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/655/format/webp)
### 组件图
组件component是代码模块。组件图是类图的物理实现。
### 配置图
显示软件及硬件的配置。
![](https://upload-images.jianshu.io/upload_images/7396903-a08f01263785bfa8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/643/format/webp)
# 类图
![](https://upload-images.jianshu.io/upload_images/7396903-49663f3f75398068.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/322/format/webp)  
* **泛化(generalization):是一种继承关系，表示一般与特殊的关系，它指定了子类如何特化父类的所有特征和行为**。例如：人是动物的一种，即有人的特性也有动物的共性。**箭头指向：**带三角箭头的实线，箭头指向父类  
![](https://upload-images.jianshu.io/upload_images/1870963-4840160012cd55ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/280/format/webp)  
* **实现(Realization):是一种类与接口的关系，表示类是接口所有特征和行为的实现**。**箭头指向：**带三角箭头的虚线，箭头指向接口  
![](https://upload-images.jianshu.io/upload_images/1870963-bcbcba9d8f78798b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/316/format/webp)  
* **关联(Association):是一种拥有的关系，它使一个类知道另一个类的属性和方法**；如：老师与学生，关联可以是双向的，也可以是单向的。  
双向的关联可以有两个箭头或没有箭头，单向的关联有一个箭头。**代码体现：**成员变量。**箭头及指向：**带普通箭头的实心线，指向被拥有者
![](https://upload-images.jianshu.io/upload_images/1870963-8a875f7f369e5f07.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/924/format/webp)  
**特殊情况：自身关联**
![](https://upload-images.jianshu.io/upload_images/1870963-14ef95a35f2b62c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/608/format/webp)  
* **聚合(Aggregation):聚合是整体与部分的关系，且部分可以离开整体而单独存在**。如：车和轮胎是整体和部分的关系，轮胎离开车任然可以存在。  
聚合关系是关联关系的一种，是强的关联关系；**代码体现：**成员变量。**箭头及指向：**带空心菱形的实心线，菱形指向整体  
![](https://upload-images.jianshu.io/upload_images/1870963-f9e35536e6b46d2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/382/format/webp)  
* **组合(Composition)：是整体与部分的关系，但部分不能离开整体而单独存在**。如：公司和部门，没有公司就不存在部门。**代码体现：**成员变量。**箭头及指向：**带实心菱形的实线，菱形指向整体  
![](https://upload-images.jianshu.io/upload_images/1870963-6a11714f5dbbd22b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/256/format/webp)  
* **依赖(Dependency)：是一种使用的关系，即一个类的实现需要另一个类的协助**。**代码表现：**局部变量、方法的参数或对静态方法的调用。**箭头及指向：**带箭头的虚线，指向被使用者  
![](https://upload-images.jianshu.io/upload_images/1870963-4d1a07e4b0eb70df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/252/format/webp)  
**各种关系的强弱顺序：** 泛化=实现>组合>聚合>关联>依赖
## 类与类之间的关系及表示方式
类之间大体分为5种关系：
![](https://img-blog.csdn.net/20130521213325997)

### 1.依赖关系(Dependency)
单向，表示一个类依赖于另一个类的定义，其中一个类的变化将影响另外一个类，**是一种"use a"关系**。如果A依赖于B，则B表现为A的局部变量、方法参数、静态方法调用等。
```java
  public class Person {
    public void doDriving(){
      Car mCar = new Car();//局部变量
      mCar.drive();
    }
    
    public void doDriving(Car car){//方法参数
      car.drive();
    }
    
    public int getCarPlate(){
      return Card.getId();//静态方法调用
    }
  }
```
### 2.关联关系(Association)
单向或双向(通常我们需要避免使用双向关联关系)，**是一种"has a"关系**。如果A单向关联B，则可以说A has a B,通常表现为全局变量
```java
  public class Person {
    public Phone phone;
    
    public void setPhone(Phone phone){
      this.phone = phone;
    }
    
    public Phone getPhone(){
      return phone;
    }
  }
```
### 3.聚合关系(Aggregation)
单向，关联关系的一种，与关联关系之间的区别是语义上的，关联的两个对象通常是平等的，聚合则一般不平等，有一种整体和局部的感觉。
Class由Student组成，其生命周期不同，**整体不存在了，部分依然存在**；Team解散了，人还在，还可以加入别的组。
```java
  public class Team {
    public Person person;
    
    public Team(Person){
      this.person = person;
    }
  }
```
### 4.组合关系(Composition)
单向，是一种强依赖的特殊聚合关系。Head、Body、Arm和Leg组合成People,其生命周期相同，**如果整体不存在了，部分也将消亡**
```java
  public class Person{
    public Head head;
    public Body body;
    public Arm arm;
    public Leg leg;
    
    public Person(){
      head = new Head();
      body = new Body();
      arm = new Arm();
      leg = new Leg();
    }
  }
```
### 5.继承关系(Inheritance)
类实现接口，类继承抽象类，类继承父类都属于这种关系。
实现(Realization): 类实现接口属于这种关系
泛化(Generalization): 即"is a"关系，类继承抽象类，类继承父类都属于这种关系







# 感谢
[类图及绘制工具：StarUML](https://www.jianshu.com/p/617f6f413452)
[UML与StarUML使用](https://www.jianshu.com/p/abe2df1b96cf)

