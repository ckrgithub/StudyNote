# greendao
通过注解方式创建实体类
```java
  @Entity(
    schema = "myschema", //多个schema模式时，可以指定实体类属于某个schema
    
    active = true, //具有更新、删除和刷新的方法
    
    nameInDb = "c_user", //表名
    
    indexes = {
      @Index(value = "text, date DESC", unique = true)
    }, //定义跨多个列索引
    
    createInDb =  true, //标记Dao是否应该创建数据库表，如果有多个实体映射到一个表，或表的创建在greendao之外完成，则设置为false
    generateConstructors = true, //是否生成所有属性的构造函数
    generateGettersSetters = true, //生成get、set方法
  )
  public class User {
    @Id
    private Long id;
    private String name;
    @Transient
    private int tempUsageCount;//非持久化
  }
```
* **创建数据库：**
```java
  DaoMaster.DevOpenHelper helper = new DaoMaster.DevOpenHelper(this,"ckr-db");
  Database db = helper.getWritableDb();
  // 加密数据库：需要在build.gradle文件添加SQLCipher依赖
  // DaoMaster.DevOpenHelper helper = new DaoMaster.DevOpenHelper(this,"ckr-db-encrypted");
  // Database db = helper.getEncyptedWritableDb("encryption-key");
  DaoSession session = new DaoMaster(db).newSession();
  
```
* **增**
```java
  Note note = new Note();
  note.setText("ckr");
  note.setComment("ckr comment");
  note.setDate(new Date());
  note.setType(NoteType.TEXT);;
  noteDao.insert(note);
```
* **删**
```
  noteDao.deleteByKey(id);
```
* **改**
```
  noteDao.update(note);
```
* **查**
```java
  //得到note表，查找所有的note记录
  NoteDao noteDao = session.getNoteDao();
  Query<Note> notesQuery = noteDao.queryBuilder().orderAsc(NoteDao.Properties.Text).build();
  List<Note> notes = notesQuery.list();
```
## 基础属性
```java
  @Entity
  public class User {
    @Id(autoincrement = true)
    private Long id;
    @Property(nameInDb = "username")
    private String name;
    @NotNull
    private int repos
    @Transient
    private int tempUsageCount;
  }
```
**@Id(autoincrement = true )**: 选择long/Long型作为实体id
**@Property(nameInDb = "USERNAME")**: 定义列名
**@NotNull**: 数据库中属性为NOT NULL 
**@Transient**: 不持久化该属性
## 主键约束
```java
  @Id
  private Long id;
  //创建唯一的索引
  @Index(unique = true) 
  private String key;
```
**@Index**: 数据库列创建数据索引
**@Unique**: 向数据库列添加唯一约束
## 多个实体关联
### 一对一关系
```java
  @Entity
  public class Order {
    @Id private Long id;
    private long customerId;
    @ToOne(joinProperty = "customerId")
    private Customer customer;
  }
  
  @Entity
  public class Customer{
    @Id private Long id;
  }
```
**@ToOne**: 定义到另一个实体的关系。在内部，greendao需要一个**附加属性来指向目标实体的ID**,该ID由joinProperty参数指定。如果缺少此参数，则会自动创建另一个列来保存键。  
注意： 如果更改外键属性(这里是customerId)，那么对getter(getCustomer())的下一个调用将解析更新ID的实体,如：
```java
  Customer customer = user.getCustomer();
  //改变customer id
  user.setCustomerId(customerIdB);
  //或者设置一个不同id的customer对象
  user.setCustomer(customerB);
  
  customerB = user.getCustomer();
  assert(customer.getId() != customerB.getId());
```
### 一对多关系
```java
  @Entity
  public class Customer {
    @Id private Long id;
    @ToMany(referencedJoinProperty = "customerId")
    @OrderBy("date ASC")
    private List<Order> orders;
  }
  
  @Entity
  public class Order {
    @Id private Long id;
    private Date date;
    private long customerId;
  }
```
**@ToMany**: 定义一组其他实体的关系，应用于目标实体列表的属性
**@referencedJoinProperty**: 指向该实体的id的目标实体中的"外键"属性的名称  
**@joinProperties**: 对于复杂关系，可以指定@JoinProperty注解列表，每个@JoinProperty都需要原始实体中的源属性和目标实体中的引用属性。
```java
  @Entity
  public class Customer{
    @Id private Long id;
    @Unique private String tag;
    @ToMany(joinProperties = {
      @JoinProperty(name = "tag",referencedName = "customerTag")
    })
    @OrderBy("date ASC")
    private List<Site> orders;
  }
  
  @Entity
  public class Order {
    @Id private Long id;
    private Date date;
    @NotNull private String customerTag;
  }
```
**@JoinEntity**: 
```java
  @Entity
  public class Product {
    @Id private Long id;
    @ToMany
    @JoinEntity(
      entity = JoinProductsWithOrders.class,
      sourceProperty = "productId",
      targetProperty = "orderId"
    )
    private List<Order> ordersWithThisProduct;
  }
  
  @Entity
  public class JoinProductsWithOrders{
    @Id private Long id;
    private Long productId;
    private Long orderId;
  }
  
  @Entity
  public class Order {
    @Id private Long id;
  }
```
## 查询
* **queryBuilder**  
以第一个名字为"Joe",最后的名字升序查询所有用户
```java
  List<User> joes = userDao.queryBuilder()
    .where(Properties.FirstName.eq("Joe"))
    .orderAsc(Properties.LastName)
    .list();
```
查询在1970年10月或以后出生的名字为“Joe”的用户：
```java
  QueryBuilder<User> query = userDao.queryBuilder();
  query.where(Properties.FirstName.eq("Joe"),
    query.or(Properties.YearOfBirth.gt(1970),
      query.and(Properties.YearOfBirth.eq(1970),
        Properties.MonthOfBirth.ge(10)
      )
    )
  );
  List<User> joes = query.list();
```
* **order**
```java
  query.orderAsc(Properties.LastName);
  query.orderDesc(Properties.LastName);
  query.orderAsc(Properties.LastName).orderDesc(Properties.YearOfBirth);
```
### 限制、偏移、和分页
limit(int): 限制查询结果的返回数量
offset(int): 结合limit设置查询结果偏移量
### 自定义类型作为参数
通常，greenDao映射查询中使用的类型。如：boolean映射为0或1，Date映射为long型时间戳
* list(): 所有实体加载到内存中
* listLazy(): 实体按需加载到内存中，一旦第一次访问列表中的元素，就会加载并缓存该元素以备将来使用。
* listLazyUncached(): 对list元素的任何访问都会导致从数据库加载其数据
* listIterator(): 让我们通过按需(惰性地)加载数据来迭代结果。没有缓存数据
### 多次查询
```java
  Query<User> query = userDao.queryBuilder().where(
    Properties.FirstName.eq("Joe"),Properties.YearOfBirth.eq(1970)
  ).build();
  List<User> joes = query.list();
  
  query.setParameter(0,"Maria");
  query.setParameter(1,1977);
  List<User> marias = query.list();
```
**多线程查询**：如果在多个线程中使用查询，则必须调用forCurrentThread()来获取**当前线程的查询实例**。查询的对象实例被绑定到它们拥有的构建查询的线程。这使您可以在其他线程无法干预的情况下，安全地在查询对象上设置参数。如果其他线程试图在查询上设置参数或执行绑定到另一个线程的查询，将引发异常。这样，就不需要同步语句。事实上，您应该避免锁定，因为如果并发事务使用相同的查询对象，这可能会导致死锁。  
每次调用forCurrentThread()时，都会在使用构建器构建查询时将**参数设置为初始参数**
### 原始查询
```java
  Query<User> query = userDao.queryBuilder().where(
    new StringCondition(
      "_ID IN" + "(SELECT USER_ID FROM USER_MESSAGE WHERE READ_FLAG = 0)"
    )
  ).build();
```
### 删除查询
批量删除不会删除单个实体，而是删除所有符合某些条件的实体
## 联接
查询通常需要几个实体类型(表)的数据。在SQL世界中，可以通过使用联接条件"联接"两个或多个表来实现这点
```java
  QueryBuilder<User> query = userDao.queryBuilder();
  query.join(Address.class,AddressDao.Properties.userId)
    .where(AddressDao.Properties.Street.eq("步行街"));
  List<User> users = queryBuilder.list();
```
一个用户与地址有一对多关系。想要查询步行街上的用户：必须使用用户ID将地址实体与用户实体联接起来
```java
/**
  * Expands the query to another entity type by using a JOIN.
  * The primary key property of the primary entity for
  * this QueryBuilder is used to match the given destinationProperty.
  */
public <J> Join<T, J> join(Class<J> destinationEntityClass, Property destinationProperty)

/**
 * Expands the query to another entity type by using a JOIN.
 * The given sourceProperty is used to match the primary
 * key property of the given destinationEntity.
 */
public <J> Join<T, J> join(Property sourceProperty, Class<J> destinationEntityClass)

/**
 * Expands the query to another entity type by using a JOIN.
 * The given sourceProperty is used to match the given
 * destinationProperty of the given destinationEntity.
 */
public <J> Join<T, J> join(Property sourceProperty, Class<J> destinationEntityClass,
    Property destinationProperty)
```
### 链式联接
greendao允许跨多个表链接连接。如，使用另一个连接和目标实体定义连接，则第一个连接的目标实体成为第二个连接的开始实体
```java
  QueryBuilder<City> query = cityDao.queryBuilder().where(Properties.Population.ge(1000000));
  Join country = query.join(Properties.CountryId,Country.class);
  Join continent = query.join(country,CountryDao.Properties.ContinentId,Continent.class,ContinentDao.Properties.Id);
  continent.where(ContinentDao.Properties.Name.eq("欧洲"));
  List<City> cities = query.list();
```
查询欧洲所有人口至少100万的城市
```java
/**
 * Expands the query to another entity type by using a JOIN.
 * The given sourceJoin's property is used to match the
 * given destinationProperty of the given destinationEntity.
 * Note that destination entity of the given join is used
 * as the source for the new join to add. In this way,
 * it is possible to compose complex "join of joins" across
 * several entities if required.
 */
public <J> Join<T, J> join(Join<?, T> sourceJoin, Property sourceProperty,
    Class<J> destinationEntityClass, Property destinationProperty)
```
### 自连接
```java
  QueryBuilder<Person> query = personDao.queryBuilder();
  Join father = query.join(Person.class,Properties.FatherId);
  Join grandFather = query.join(father,Properties.FatherId,Person.class,Properties.Id);
  grandFather.where(Properties.Name.eq("林肯"));
  List<Person> persons = query.list();
```
查询所有他们祖父的名字是林肯的人

