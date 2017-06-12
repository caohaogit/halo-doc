### REST+JSON

REST是一种数据访问的规范，而不是一种具体数据协议。我们将一个服务遵循REST标准规范发布，因此这个服务是RESTful的。而这个服务发生的数据交互所使用数据格式与是否使用REST在理论上是无关的。不过，按照目前行业惯例，REST服务所使用的数据交互格式一般为JSON。  
关于REST服务的规范，请自行去网上搜索相关资料。REST服务设计规范参考：[https://zybuluo.com/yanbo-ai/note/17890?utm\_source=tuicool&utm\_medium=referral](https://zybuluo.com/yanbo-ai/note/17890?utm_source=tuicool&utm_medium=referral)  
框架基于CXF和jaxrs提供一套REST+JSON的WebService服务发布规范。

#### 服务接口编写

CXF使用“javax.ws.rs.\*”注解为REST服务接口添加WebService发布信息。服务接口统一放在“XXX-service-api”工程的“com.halo.XXX.service.api”中。

```java
@Path("/restfulService")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public interface RestfulService {
    /**
     * 根据用户ID，获取用户.
     *
     * @param userId the user id
     * @return the user
     */
    @GET
    @Path("/{userId}")
    UserInfo getUser(@PathParam("userId") String userId);
    
    /**
     * 添加用户.
     *
     * @param user the user
     * @return the boolean
     * @author maeagle
     * @修改时间：2016-1-27 9 :58:56
     * @修改内容： *
     */
    @POST
    @Path("/")
    boolean addUser(UserInfo user);
}
```

`@Path("/restfulService")`指定服务暴露的根路径，后续服务方法中也可以添加该注解，不过最终生成的服务路径都是以类注解上的路径为根路径的。  
`@Produces(MediaType.APPLICATION_JSON)`指定服务调用方发送数据的格式。`@Consumes(MediaType.APPLICATION_JSON)`指定服务提供方处理数据的格式。  
`@GET，@POST，@DELETE，@PUT`是HTTP的四种默认传输方式，每一种是对资源的一类操作。`@QueryParam("userId")`从`http://XXXX/?userId=123`类似路径中，解析出123这个值所需的注解。

#### 服务实现编写

服务实现类与普通的Spring服务编写方法相同。服务实现类统一放在“XXX-service-app”工程的“com.halo.XXX.service.impl”中。

```java
@Service("restfulService")
public class RestfulServiceImpl implements RestfulService {
    // 实现代码
}
```

#### 服务发布

基于现有已经写好的业务实现的代码，CXF通过Spring集成提供无缝的服务发布功能。服务发布的XML配置写在“XXX-service-app”工程的“applicationContext-service.xml”中。

```xml
<!-- 发布restfulService -->
<jaxrs:server beanNames="restfulService" address="/${ws.common.restful.path}">
    <jaxrs:providers>
        <bean class="com.halo.core.common.support.FastJsonProvider"/>
    </jaxrs:providers>
</jaxrs:server>
```

`beanNames="restfulService"`指向到具体REST服务实现类的Spring 服务名称。`ws.common.restful.path`是代码工程通用的REST服务发布的根路径，配置在wsconfig.properties下。不推荐修改。`<jaxrs:providers>`为了实现JSON序列化所提供的fastjson provider。每个发布的REST服务都需要指定。

#### 客户端声明

REST服务客户端的声明方式与SOAP服务客户端声明形式类似。都是通过Spring的XML Configuration进行配置的。服务客户端声明的XML配置写在“XXX-service-app”,“XXX-batch-app”或“XXX-web-app”中。

```xml
<!--RESTFul WebService客户端注册 -->
<jaxrs-client:client id="restfulService" serviceClass="com.halo.example.service.api.RestfulService" address="http://${ws.client.example.address}/${ws.protocol.rest.path}">
    <jaxrs-client:providers>
        <bean class="com.halo.core.common.support.FastJsonProvider"/>
    </jaxrs-client:providers>
</jaxrs-client:client>
```

`client id="restfulService"`客户端服务在Spring的唯一bean标识。`serviceClass="com.halo.example.service.api.RestfulService"`中的接口通过以下渠道获得：

* 推荐：将服务端工程的“XXX-service-api”和“xxx-models”两个jar包引入客户端工程即可。
* 目前框架未集成：需要服务端编写者配合，在服务端编写RESTful服务的WADL文件。使用者通过工具从WADL生成Java代码。

`<jaxrs:providers>`为了实现JSON序列化所提供的fastjson provider。每个发布的REST服务的调用者都需要指定。

#### 服务调用

在声明好REST服务客户端后，就可以在工程中以普通Spring注入的方式来调用REST服务。整个过程对于业务代码的视角来看，是察觉不到调用了远程服务的。

```java
/**
 * RestfulService客户端.（自动注入）
 */
@Autowired
RestfulService restfulService;
```

