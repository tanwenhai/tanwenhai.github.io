---
title: Jackson 框架的高阶应用
date: 2018-02-26 17:08:31
tags: java jackson
---

# Jackson 的 注解的使用
Jackson 根据它的默认方式序列化和反序列化 java 对象，若根据实际需要，灵活的调整它的默认方式，可以使用 Jackson 的注解。常用的注解及用法如下。
|注解|用法|
|:---|:-|
|@JsonProperty | 用于属性，把属性的名称序列化时转换为另外一个名称。示例：<br/>@JsonProperty("birth_ d ate")<br/>private Date birthDate;|
|@JsonFormat	| 用于属性或者方法，把属性的格式序列化时转换成指定的格式。示例： <br/>@JsonFormat(timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm") <br/>public Date getBirthDate()|
|@JsonPropertyOrder	|用于类， 指定属性在序列化时 json 中的顺序 ， 示例： <br/>@JsonPropertyOrder({ "birth_Date", "name" }) <br/>public class Person|
|@JsonCreator	| 用于构造方法，和 @JsonProperty 配合使用，适用有参数的构造方法。 示例：<br/>@JsonCreator <br/>public Person(@JsonProperty("name")String name) {…}|
|@JsonAnySetter	| 用于属性或者方法，设置未反序列化的属性名和值作为键值存储到 map 中 <br/>@JsonAnySetter <br/>public void set(String key, Object value) { <br/>map.put(key, value);<br/>}|
|@JsonAnyGetter	| 用于方法 ，获取所有未序列化的属性 <br/>public Map<String, Object> any() { return map; }

# Jackson 的 高阶应用

## 格式处理（含日期格式）
不同类型的日期类型，Jackson 的处理方式也不同。

* 对于日期类型为 java.util.Calendar,java.util.GregorianCalendar,java.sql.Date,java.util.Date,java.sql.Timestamp，若不指定格式， 在 json 文件中将序列化 为 long 类型的数据。显然这种默认格式，可读性差，转换格式是必要的。Jackson 有 很多方式转换日期格式。
* 注解方式，请参照" Jackson 的注解的使用"的@ JsonFormat 的示例。
* ObjectMapper 方式，调用 ObjectMapper 的方法 setDateFormat，将序列化为指定格式的 string 类型的数据。
* 对于日期类型为 java.time.LocalDate，还需要添加代码 mapper.registerModule(new JavaTimeModule())，同时添加相应的依赖 jar 包
    ```xml
    <dependency> 
        <groupId>com.fasterxml.jackson.datatype</groupId> 
        <artifactId>jackson-datatype-jsr310</artifactId> 
        <version>2.9.1</version> 
    </dependency>
    ```

## 泛型反序列化
Jackson 对泛型反序列化也提供很好的支持。
* 对于 List 类型 ，可以调用 constructCollectionType 方法来序列化，也可以构造 TypeReference 来序列化。
```java
CollectionType javaType = mapper.getTypeFactory() 
.constructCollectionType(List.class, Person.class); 
List<Person> personList = mapper.readValue(jsonInString, javaType); 
List<Person> personList = mapper.readValue(jsonInString, new TypeReference<List<Person>>(){});
```

* 对于 map 类型， 与 List 的实现方式相似。
```java
//第二参数是 map 的 key 的类型，第三参数是 map 的 value 的类型 
 MapType javaType = mapper.getTypeFactory().constructMapType(HashMap.class,String.class, Person.class); 
 Map<String, Person> personMap = mapper.readValue(jsonInString, javaType); 
 Map<String, Person> personMap = mapper.readValue(jsonInString, new TypeReference<Map<String, Person>>() {});
```

## 属性可视化
是 java 对象的所有的属性都被序列化和反序列化，换言之，不是所有属性都可视化，默认的属性可视化的规则如下：
* 若该属性修饰符是 public，该属性可序列化和反序列化。
* 若属性的修饰符不是 public，但是它的 getter 方法和 setter 方法是 public，该属性可序列化和反序列化。因为 getter 方法用于序列化， 而 setter 方法用于反序列化。
* 若属性只有 public 的 setter 方法，而无 public 的 getter 方 法，该属性只能用于反序列化。

若想更改默认的属性可视化的规则，需要调用 ObjectMapper 的方法 setVisibility。

下面的示例使修饰符为 protected 的属性 name 也可以序列化和反序列化。
```java
mapper.setVisibility(PropertyAccessor.FIELD, Visibility.ANY); 
public class Person { 
    public int age; 
    protected String name; 
}
// PropertyAccessor 支持的类型有 ALL,CREATOR,FIELD,GETTER,IS_GETTER,NONE,SETTER 
// Visibility 支持的类型有 ANY,DEFAULT,NON_PRIVATE,NONE,PROTECTED_AND_PUBLIC,PUBLIC_ONLY
```

## 属性过滤
在将 Java 对象序列化为 json 时 ，有些属性需要过滤掉，不显示在 json 中 ， Jackson 有多种实现方法。
* 注解方式， 可以用 @JsonIgnore 过滤单个属性或用 @JsonIgnoreProperties 过滤多个属性，示例如下：
```java
@JsonIgnore 
public int getAge() 
@JsonIgnoreProperties(value = { "age","birth_date" }) 
public class Person
```

## 自定义序列化和反序列化
当 Jackson 默认序列化和反序列化的类不能满足实际需要，可以自定义新的序列化和反序列化的类。
* 自定义序列化类。自定义的序列化类需要直接或间接继承 StdSerializer 或 JsonSerializer，同时需要利用 JsonGenerator 生成 json，重写方法 serialize，示例如下：


```java
public class CustomSerializer extends StdSerializer<Person> { 
    @Override 
    public void serialize(Person person, JsonGenerator jgen, SerializerProvider provider) throws IOException { 
        jgen.writeStartObject(); 
        jgen.writeNumberField("age", person.getAge()); 
        jgen.writeStringField("name", person.getName()); 
        jgen.writeEndObject(); 
    } 
}
```
 
JsonGenerator 有多种 write 方法以支持生成复杂的类型的 json，比如 writeArray，writeTree 等 。若想单独创建 JsonGenerator，可以通过 JsonFactory() 的 createGenerator。
* 自定义反序列化类。自定义的反序列化类需要直接或间接继承 StdDeserializer 或 StdDeserializer，同时需要利用 JsonParser 读取 json，重写方法 deserialize，示例如下：
```java
public class CustomDeserializer extends StdDeserializer<Person> { 
    @Override 
    public Person deserialize(JsonParser jp, DeserializationContext ctxt) 
        throws IOException, JsonProcessingException { 
        JsonNode node = jp.getCodec().readTree(jp); 
        Person person = new Person(); 
        int age = (Integer) ((IntNode) node.get("age")).numberValue(); 
        String name = node.get("name").asText(); 
        person.setAge(age); 
        person.setName(name); 
        return person; 
    } 
}
```

JsonParser 提供很多方法来读取 json 信息， 如 isClosed(), nextToken(), getValueAsString()等。若想单独创建 JsonParser，可以通过 JsonFactory() 的 createParser。

* 定义好自定义序列化类和自定义反序列化类，若想在程序中调用它们，还需要注册到 ObjectMapper 的 Module，示例如下：
```java
SimpleModule module = new SimpleModule("myModule"); 
module.addSerializer(new CustomSerializer(Person.class)); 
module.addDeserializer(Person.class, new CustomDeserializer()); 
mapper.registerModule(module); 
// 也可通过注解方式加在 java 对象的属性，方法或类上面来调用它们， 
@JsonSerialize(using = CustomSerializer.class) 
@JsonDeserialize(using = CustomDeserializer.class) 
public class Person
```

### 树模型处理
Jackson 也提供了树模型(tree model)来生成和解析 json。若想修改或访问 json 部分属性，树模型是不错的选择。树模型由 JsonNode 节点组成。程序中常常使用 ObjectNode，ObjectNode 继承于 JsonNode，示例如下：

```java
ObjectMapper mapper = new ObjectMapper(); 
//构建 ObjectNode 
ObjectNode personNode = mapper.createObjectNode(); 
//添加/更改属性 
personNode.put("name","Tom"); 
personNode.put("age",40); 
ObjectNode addressNode = mapper.createObjectNode(); 
addressNode.put("zip","000000"); 
addressNode.put("street","Road NanJing"); 
//设置子节点 
personNode.set("address",addressNode); 
//通过 path 查找节点 
JsonNode searchNode = personNode.path("street")； 
//删除属性 
((ObjectNode) personNode).remove("address"); 
//读取 json 
JsonNode rootNode = mapper.readTree(personNode.toString()); 
//JsonNode 转换成 java 对象 
Person person = mapper.treeToValue(personNode, Person.class); 
//java 对象转换成 JsonNode 
JsonNode node = mapper.valueToTree(person);
```
