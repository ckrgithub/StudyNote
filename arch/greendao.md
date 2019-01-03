# greendao
通过注解方式创建实体类
```
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
```
  DaoMaster.DevOpenHelper helper = new DaoMaster.DevOpenHelper(this,"ckr-db");
  Database db = helper.getWritableDb();
  // 加密数据库：需要在build.gradle文件添加SQLCipher依赖
  // DaoMaster.DevOpenHelper helper = new DaoMaster.DevOpenHelper(this,"ckr-db-encrypted");
  // Database db = helper.getEncyptedWritableDb("encryption-key");
  DaoSession session = new DaoMaster(db).newSession();
  
```
* **增**
```
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
```
  //得到note表，查找所有的note记录
  NoteDao noteDao = session.getNoteDao();
  Query<Note> notesQuery = noteDao.queryBuilder().orderAsc(NoteDao.Properties.Text).build();
  List<Note> notes = notesQuery.list();
```
## 基础属性
```
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
```
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
```
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
```
  Customer customer = user.getCustomer();
  //改变customer id
  user.setCustomerId(customerIdB);
  //或者设置一个不同id的customer对象
  user.setCustomer(customerB);
  
  customerB = user.getCustomer();
  assert(customer.getId() != customerB.getId());
```
### 一对多关系
```
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
```
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
```
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







