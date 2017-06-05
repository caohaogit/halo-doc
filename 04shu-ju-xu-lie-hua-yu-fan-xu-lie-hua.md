## 数据序列化与反序列化

**所有待序列化对象类代码，都应该标注实现java.io.Serializable接口。  
所有待序列化对象类中所包含的属性类型，也都应该是能够被序列化的。**

### XML

业务的复杂性要求SOAP服务能够传递复杂的Java对象。框架通过“jaxb”实现Java对象的XML结构序列化与反序列化。开发者只需要在普通的JavaBean对象上添加特定的注解，CXF就会调用JAXB将javaBean解析为xml，或是将xml中的数据mapping到一个JavaBean对象中。  
服务中使用的model对象类统一放在“XXX-model”工程的“com.halo.XXX.models”中。

```java
@XmlRootElement(name = "User")
@XmlType(name = "User")
@XmlAccessorType(XmlAccessType.FIELD)
public class UserInfo implements Serializable {
    // 元素属性
}
```

`@XmlRootElement(name = "User")`：xml 文件的根元素的name为User`@XmlAccessorType(XmlAccessType.FIELD)`：表明类内，什么样的成员是可以被xml 转化传输的  
`@XmlType(name = "User")`：类被映射到的 XML 模式类型名称为User

### JSON

框架推荐使用fastjson组件对Java对象进行JSON序列化与反序列化。除了实现一个接口外，Java对象类代码不需要做任何特殊定制。在REST服务发布时，会在XML配置中指定fastjson的messageProvider即可。  
服务中使用的model对象类统一放在“XXX-model”工程的“com.halo.XXX.models”中。

```java
public class UserInfo implements Serializable {
    // 代码
}
```

#### 序列化定制

框架在RPC远程调用中使用FastJSON组件实现序列化数据时，通过使用[@JsonConfig](http://localhost:3000/JsonConfig)注解来对特定的Java对象序列化做配置。样例代码如下：

```java
@POST
@Path("/add")
boolean addUser(@JsonConfig(writeType = true,writeNull = true) UserInfo user);

@GET
@Path("/single")
@JsonConfig(writeType = true,writeNull = true)
UserInfo getUser(@QueryParam("userId") String userId);
```

以上两个例子说明，[@JsonConfig](http://localhost:3000/JsonConfig)需要附加在接口方法声明的参数或返回值上。其中：

1. writeType用于声明序列化该对象数据时，是否将类型信息附加在
   [@type](http://localhost:3000/type)
   属性上。
2. writeNull用于声明序列化过程中，如果遇到null，是否要输出对象的属性。



