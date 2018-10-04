# jackson
## 一、注解
### 1.声明属性名
@JsonProperty("value")
```java
  public class Person {
    @JsonProperty("name")
    public String cName;
  }
  //json
  {
    "name":"ckr"
  }
```
### 2.忽略某个属性
@JsonIgnore
```java
  public class Person {
    public String name;
    @JsonIgnore
    public int age;
  }
  //to json
  {
    "name:" "ckr"
  }
```
### 3.忽略多个属性
@JsonIgnoreProperties({"value1","value2",...})
@JsonIgnoreProperties(ignoreUnknown=true): 不解析没有声明的属性
```java
  @JsonIgnoreProperties({"sex","age"})
  public class Person{
    public String name;
  }
  或
  @JsonIgnorePropertise(ignoreUnknown=true)
  public class Person{
    public String name;
  }
  //from json
  { "name":"ckr","age":"18","sex":"man"}
```
### 4.@JsonDeserialize()/@JsonSerialize()
```java
  public class Person{
    @JsonDeserialize(as=ValueImpl.class)
    public Value value;
    
    @JsonSerialize(as=BasicType.class)
    public BasicType another
  }
```
### 5.指定构造器解析
@JsonCreator
```java
  public class Person{
    private String name;
    private int age;
    @JsonCreator
    public Person(@JsonProperty("name")String cName,@JsonProperty("age")int cAge){
      name=cName;
      age=cAge;
    }
  }
```
### 6.多类型解析
@JsonTypeInfo():
```java
  @JsonTypeInfo(use=Id.CLASS,include=As.PROPERTY,property="class")
  public abstract class BaseClass{}
  
  public class ImplA extends BaseClass{
    public int x;
  }
  public class ImplB extends BaseClass{
    public String name;
  }
  public class WithTypedObjects{
    public List<BaseClass> items;
  }
  {
    "items":[
      {"class": "ImplA","name":"ckr"},
      {"class": "ImplB","x":18}
    ]
  }
``` 
### 7.自动检测
@JsonAutoDetect
```java
  @JsonAutoDetect(fieldVisibility=JsonAutoDetect.Visibility.ANY)
  public class Person{
    private String name;
  }
  or:
  @JsonAutoDetect(fieldVisibility=JsonAutoDetect.Visibility.NONE)
  public class Person{
    public String name;
  }
```
## 二、jackson databind
### 1.ObjectMapper
```java
  public class Person{
    public String name;
    public int age;
  }
  //也可以使用protected or private修饰符
  public class Person{
    private String name;
    @JsonProperty("name")
    public String getName(){
      return name;
    }
    public void setName(String name){
      this.name=name;
    }
  }
  
  ObjectMapper mapper = new ObjectMapper();//创建一次，重复使用
  //fromJson
  Person person=mapper.readValue(new File("data.json"),Person.class);
  person=mapper.readValue(new URL("http://ckr.com/api/user.json"),Person.class);
  person=mapper.readValue("{\"name\":\"ckr\",\"age\":13}",Person.class);
  //toJson
  mapper.writeValue(new File("user.json"),person);
  byte[] jsonBytes=mapper.writeValueAsBytes(person);
  String jsonString=mapper.writeValueAsString(person);
  //collection
  Map<String,Integer> map = mapper.readValue(jsonString,Map.class);
  List<Stirng> list=mapper.readValue(jsonString,List.class);
  Map<String,ResultValue> results=mapper.readValue(jsonString,new TypeReference<Map<String,ResultValue>>(){});
``
### 2.JsonNode
```java
  ObjectNode root=mapper.readTree("data.json");
  String name = root.get("name").asText();
  int age=root.get("age").asText();
  root.with("other").put("type","student");
  String json=mapper.writeValueAsString(root);
  {
    "name":"ckr",
    "age":18,
    "other":{
      "type":"student"
    }
  }
```
### 3.Streaming parser, generator
```java
  JsonFactory factory = mapper.getFactory();
  File jsonFile=null;
  JsonGenerator generator = factory.createGenerator(jsonFile=new File("test.json"));
  generator.writeStartObject();
  generator.writeStirngField("message","Hello");
  generator.writeEndObject();
  generator.close();
  
  JsonParser parser = factory.createParser(jsonFile);
  JsonToken token=parser.nextToken();//JsonToken.Start_object
  token=parser.nextToken();//jsonToken.field_name
  if(token!=JsonToken.FIELD_NAME || !"message".equals(parser.getCurrentName())){
    //handle error
  }
  token=parser.nextToken();
  if(token!=JsonToken.VALUE_STRING){
    //handle error
  }
  String msg=parser.getText();
  System.out.printf("msg:"+msg);
  parser.close();
```
### 4.convertValue使用
```java
  List<Integer> srcList=Arrays.asList(1,2,3);
  int[] ints=mapper.convertValue(srcList,int[].class);
  Map<String,Object> map=mapper.convertValue(obj,Map.class);
  
  Person person=mapper.convertValue(map,Person.class);
  String base64=" FLKSJDF3LJFSFJS4DLFJSFJ4DKFJSDL4";
  byte[] binary=mapper.convertValue(base64,byte[].class);
```
## 三、jackson core
```java
  JsonFactory factory =new JsonFactory();
  factory.enable(JsonParser.Feature.ALLOW_COMMENTS);
  or:
  JsonFactory factory=mapper.getFactory();
```


















