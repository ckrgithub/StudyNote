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
    
    createInDb =  false, //
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
  
  
  //得到note表，查找所有的note记录
  NoteDao noteDao = session.getNoteDao();
  Query<Note> notesQuery = noteDao.queryBuilder().orderAsc(NoteDao.Properties.Text).build();
  List<Note> notes = notesQuery.list();
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
