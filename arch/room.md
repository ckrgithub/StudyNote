# Room
由以下三个主要元素组成：
* DataBase:数据库的持有者和数据库的访问主要入口。使用：1.创建一个类AppDatabase并继承RoomDatabase 2.通过注解把实体Entity与数据库关联
  3.创建一个抽象方法并返回Entity对应的Dao接口
* Entity:数据库中的表
* Dao:包含访问数据库的一些列方法
## 图解
![](https://developer.android.com/images/training/data-storage/room_architecture.png)
## 基本使用
### 创建一个Entity和Dao
```java
  @Entity
  public class User {
    @PrimaryKey
    private int uid;
    
    @ColumnInfo(name = "first_name")//列名
    private String firstName;
    
    @ColumnInfo(name ="last_name")
    private String lastName'
  }
  
  @Dao
  public interface UserDao {
    @Query("SELECT * FROM user")
    List<User> getAll();
    
    @Query("SELECT * FROM user WHERE uid IN (:userIds)")
    List<User> loadAllByIds(int[] userIds);
    
    @Query("SELECT * FROM user WHERE first_name LIKE :first AND last_name LIKE :last LIMIT 1")
    User findByName(String first,String last);
    
    @Insert
    void insertAll(User... users);
    
    @Delete
    void delete(User user);
  }
```
### 创建Database
```java
  @Database(entities = {User.class},version=1)
  public abstract class AppDatabase extends RoomDatabase{
    public abstract UserDao userDao();
  }
  
  AppDatabase db = Room.databaseBuilder(mContext,AppDatabase.class,"database-name").build();
```
### 语法
@ignore和primaryKeys使用
```java
  @Entity(primaryKeys = {"firstName","lastName"})
  public class User {
    public int id;
    public String firstName;
    public String lastName;
    
    @Ignore  //忽略该字段
    Bitmap picture;
  }
```
indices:创建索引
```java
  @Entity(indices={@Index("name"),@Index(value = {"last_name","address"})})
  public classs User {
    @PrimaryKey
    public int id;
    public String firstName;
    public String address;
    
    @ColumnInfo(name="last_name")
    public String lastName;
  }
```
@ForeignKey:创建外键
```java
  @Entity(foreignKeys=@ForeignKey(entity=User.class,parentColumns="id",childColumns="user_id"))
  public class Book{
    @PrimaryKey
    public int bookId;
    public String title;
    
    @ColumnInfo(name="user_id")
    public int userId;
  }
```
@Embedded:嵌入内部类的字段
```java
  @Entity
  public class User {
    @PrimaryKey
    public int id;
    public String firstName;
    @Embedded(prefix="user_address")
    public Address address;
  }
  public class Address {
    public String city;
    @ColumnInfo(name="post_code")
    public int postCode;
  }
```
@Insert:新增记录
```java
  @Dao
  public interface UserDao {
    @Insert(onConflict=OnConflictStrategy.REPLACE)
    public void insertUsers(User... users);
    @Insert
    public void insertBothUsers(User user1,User user2);
    @Insert
    public void insertUsersAndFriends(User user,List<User> friends);
  }
```
@Update: 更新记录
```java
  @Dao
  public interface UserDao {
    @Update
    public void updateUsers(User... users);
  }
```
@Delete: 删除记录
```java
  @Dao
  public interface UserDao{
    @Delete
    public void deleteUsers(User... users);
  }
```
@Query: 查询记录
```java
  @Dao
  public interface UserDao{
    @Query("SELECT * FROM user")
    public User[] laodAllUsers();
    
    @Query("SELECT * FROM user WHERE age > :minAge")
    public User[] loadAllUsersOlderThan(int minAge);
    
    @Query("SELECT * FROM user WHERE age BETWEEN :minAge AND :maxAge")
    public User[] loadAllUsersBetweenAges(int minAge,int maxAge)
  }
  
  public class NameTuple{
    @ColumInfo(name="first_name")
    public String firstName;
    @ColumnInfo(name="last_name")
    public String lastName;
  }
  @Dao
  public interface UserDao{
    @Query("SELECT first_name, last_name FROM user")
    publlic List<NameTuple> laodFullName();
    
    @Query("SELECT first_name, last_name FROM user WHERE region IN (:regions)")
    public List<NameTuple> loadUsersFromRegions(List<String> regions);
  }
```
多张表查询：
```java
@Dao
public interface UserDao{
  @Query("SELECT * FROM book INNER JOIN loan ON loan.book_id = book.id INNER JOIN user ON user.id = loan.user_id 
    WHERE user.name LIKE :userName")
  public List<Book> findBooksBorrowedByNameSync(String userName);
  
  @Query("SELECT user.name AS userName, pet.name AS petName FROM user, pet WHERE user.id=pet.user_id")
  public LiveData<List<UserPet>> loadUserAndPetNames();
}

static class UserPet{
  public String userName;
  public String petName;
}
```
### 数据库升级
```java
  Room.databaseBuilder(mContext,MyDb.class,"database-name").addMigrations(MIGRATION_1_2,MIGRATION_2_3).build();
  static final Migration MIGRATION_1_2 = new Migration(1,2){
    @Override
    public void migrate(SupportSQLiteDatabase database){
      database.execSQL("CREATE TABLE 'Fruit' ('id' INTEGER, 'name' TEXT, PRIMARY KEY('id'))");
    }
  };
  static final Migration MIGRATION_2_3 = new Migration(2,3){
    @Override
    public void migrate(SupportSQLiteDatabase database){
      database.execSQL("ALTER TABLE Book ADD COLUMN pub_year INTEGER");
    }
  }
```
导出数据库的模式信息到json文件
```java
  android {
    defaultConfig {
      javaCompileOptions {
        annotationProcessorOptions {
          arguments = ["room.schemaLocation": "$projectDir/schemas".toString()]
        }
      }
    }
  }
```
### 复杂数据类型
* 使用类型转换(TypeConverter)
* 理解Room为什么不允许对象引用
```java
  public class Converters {
    @TypeConverter
    public static Data fromTimestamp(Long value){
      return value ==null ? null : new Date(value);
    }
    
    @TypeConverter
    public static Long dateToTimestamp(Date date){
      return date == null ? null : date.getTime();
    }
  }
  
  @Database(entities ={User.class},version=1)
  @TypeConverters({Converters.classs})
  public abstract classs AppDatabase extends RoomDatabase{
    public abstract UserDao userDao();
  }
```













## 感谢
[room](https://developer.android.com/training/data-storage/room/)
