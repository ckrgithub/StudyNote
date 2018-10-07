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
    
    @ColumnInfo(name = "first_name")
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


























## 感谢
[room](https://developer.android.com/training/data-storage/room/)
