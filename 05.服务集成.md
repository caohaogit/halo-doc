## 服务集成

WebApp与ServiceApp之间的交互都是同服务调用完成的。（WebApp无法连接数据库，但是可以连接缓存和消息队列）。服务集成组件在框架中占据着重要的位置。其中不仅包括服务发布，服务调用。在交互过程中的数据监控，日志管理等也同样重要。  
同样的，在跨系统的serviceapp间做服务调用时，不仅仅需要基于RPC的同步服务调用，实际业务中也对异步服务和数据一致性有诉求。  
基于以上原因，框架提供两种服务集成解决方案：CXF（WebService + REST），DASC（MessageQueue+DB）。

### 通用配置

1. ws.common.name用来标识当前工程在服务调用与服务发布时，标识的唯一服务节点。（如果当前业务服务系统按照集群部署，则该值应该在集群内保证一致。）
2. ws. protocol.\*\*\*.path某种协议的上下文路径。
3. ws.protocol.\*\*\*.header 某种协议是否启用header头（soap用soap header，http用 http header）里面会填充有关这次交易的信息。

### CXF

CXF是较为流行的Apache下WebService实现，它将WebService的服务发布与服务调用过程分别通过SOAP XML与RESTful两种规范分别实现。  
其中SOAP WebService是传统的WebService调用，这也是CXF的强项。RESTful WebService是在CXF 3.X中做了实现，基于jaxrs标准。  
CXF的整体设计理念是：通过“拦截器”和“总线”两套机制，实现对WebService报文从调用方到服务方的数据封装与解封装的过程。  
关于CXF更多的参考，可以访问Apache网站：[http://cxf.apache.org/docs/index.html](http://cxf.apache.org/docs/index.html)

#### 总线配置

为了完成框架框架对CXF默认的总线服务做了定制。定制过程是基于Spring XML Configuration的。代码如下：

```xml
<cxf:bus>
    <cxf:inInterceptors>
        <ref bean="cxfLogInbound"/>
        <ref bean="serviceHeaderInbound"/>
    </cxf:inInterceptors>
    <cxf:outInterceptors>
	<ref bean="cxfEncodingOutbound"/>
        <ref bean="cxfLogOutbound"/>
        <ref bean="serviceHeaderOutbound"/>
    </cxf:outInterceptors>
    <cxf:outFaultInterceptors>
        <ref bean="cxfLogOutbound"/>
        <ref bean="serviceExceptionOutbound"/>
    </cxf:outFaultInterceptors>
    <cxf:inFaultInterceptors>
        <ref bean="cxfLogInbound"/>
    </cxf:inFaultInterceptors>
</cxf:bus>
```

其中，对cxf的接收报文和发出报文的总线路径上，分别做了日志拦截器（cxfLogInbound，cxfLogOutbound），Header拦截器（serviceHeaderInbound，serviceHeaderOutbound），异常拦截器（serviceExceptionOutbound）。

1. 日志拦截器
   控制CXF对接收和发送WebService报文的日志打印。控制开关为“ws.common.logging”
2. Header拦截器
   控制CXF是否对发送的报文进行Header填充，同时也控制CXF是否对接收的报文进行Header验证。
3. 异常拦截器
   当CXF出现异常时，控制对异常信息进一步的加工处理。
4. 编码格式的指定拦截器
   指定当前编码格式“system.encoding=UTF-8”。

#### SOAP+XML

##### 服务接口编写

CXF使用“javax.jws.\*”注解为SOAP服务接口添加WebService发布信息。服务接口统一放在“XXX-service-api”工程的“com.halo.XXX.service.api”中。

```java
@WebService(targetNamespace = "http://www.e-chinalife.com/soa/")
public interface SoapService {
    // 服务方法声明
}
```

##### 服务实现编写

服务实现类与普通的Spring服务编写方法相同。服务实现类统一放在“XXX-service-app”工程的“com.halo.XXX.service.impl”中。

```java
@Service("soapService")
public class SoapServiceImpl implements SoapService {
    // 服务实现
}
```

##### 服务发布

基于现有已经写好的业务实现的代码，CXF通过Spring集成提供无缝的服务发布功能。服务发布的XML配置写在“XXX-service-app”工程的“applicationContext-service.xml”中。

```xml
<!-- 发布soapService -->
<jaxws:endpoint id="SoapService" serviceName="soa:SoapService" implementor="#soapService" address="/${ws.protocol.soap.path}/soapService"/>
```

其中，`serviceName="soa:SoapService"`中的`soa`是在xml文件顶端声明的命名空间：`xmlns:soa="http://www.e-chinalife.com/soa/"`。SoapService是用来声明发布的服务名称的。  
`implementor="#soapService"`用来指引jaxws具体的服务实现（这部分其实是通知jaxws通过spring beans的name去找特定的Spring服务，注入到实现中）。  
`id="SoapService"`是提供该服务实例在Spring的唯一标识。建议这个值与`serviceName="soa:SoapService"`后半部分保持一致。`address="/${ws.common.soap.path}/soapService`的`ws.common.soap.path`是用来区分不同的服务协议调用地址的,不建议修改。

##### WSDL生成与查看

启动serviceapp后，可以通过`http://XXXXXX/services/服务名称?wsdl`来查看服务发布的wsdl文件

##### 客户端声明

SOAP服务客户端的声明方式与服务发布形式类似。都是通过Spring的XML Configuration进行配置的。服务客户端声明的XML配置写在“XXX-service-app”，“XXX-batch-app”或“XXX-web-app”中。

```xml
<!--SOAP WebService客户端注册 -->
<jaxws:client id="soapService" serviceClass="com.halo.example.service.api.SoapService" address="http://${ws.client.example.address}/${ws.protocol.soap.path}/soapService"/>
```

其中，`serviceClass="com.halo.example.service.api.SoapService"`中的接口文件有两种渠道获得：

* 推荐：将服务端工程的“XXX-service-api”和“xxx-models”两个jar包引入客户端工程即可。
* 通过服务端可访问的WSDL文件动态生成服务接口类。
  `address="http://${ws.client.example.address}/${ws.protocol.soap.path}/soapService"`
  建议服务调用地址统一放置在wsconfig.properties中进行统一管理。

##### 服务调用

在声明好SOAP服务客户端后，就可以在工程中以普通Spring注入的方式来调用SOAP服务。整个过程对于业务代码的视角来看，是察觉不到调用了远程服务的。

```java
@Autowired
SoapService soapService;
// 具体代码实现和调用
```

#### REST+JSON

REST是一种数据访问的规范，而不是一种具体数据协议。我们将一个服务遵循REST标准规范发布，因此这个服务是RESTful的。而这个服务发生的数据交互所使用数据格式与是否使用REST在理论上是无关的。不过，按照目前行业惯例，REST服务所使用的数据交互格式一般为JSON。  
关于REST服务的规范，请自行去网上搜索相关资料。REST服务设计规范参考：[https://zybuluo.com/yanbo-ai/note/17890?utm\_source=tuicool&utm\_medium=referral](https://zybuluo.com/yanbo-ai/note/17890?utm_source=tuicool&utm_medium=referral)  
框架基于CXF和jaxrs提供一套REST+JSON的WebService服务发布规范。

##### 服务接口编写

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

##### 服务实现编写

服务实现类与普通的Spring服务编写方法相同。服务实现类统一放在“XXX-service-app”工程的“com.halo.XXX.service.impl”中。

```java
@Service("restfulService")
public class RestfulServiceImpl implements RestfulService {
    // 实现代码
}
```

##### 服务发布

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

##### 客户端声明

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

##### 服务调用

在声明好REST服务客户端后，就可以在工程中以普通Spring注入的方式来调用REST服务。整个过程对于业务代码的视角来看，是察觉不到调用了远程服务的。

```java
/**
 * RestfulService客户端.（自动注入）
 */
@Autowired
RestfulService restfulService;
```

### DASC

DASC是Halo作者基于MQ（消息队列）开发的一套在分布式应用环境下，能够保证数据最终一致性的异步服务调用框架。基于ebay模式，是业内针对数据最终一致性上较为规范成熟的设计理念。

#### 组件依赖关系

DASC是一套多组件支撑的框架体系。对于DASC所需要的halo组件依赖关系如下图：

![](/assets/07.组件依赖关系.png)

#### 使用方法

##### 配置项修改

与DASC服务有关的配置文件为”dasc.properties”, “cache.properties”, “mqconfig.properties”。其中详细的配置项参考【配置清单】小节。需要开发者保证当前工程对消息队列和缓存的连通性。对于”dasc.properties”需要保证两个服务调用地址的IP与当前工程的所在网络IP一致。

##### 服务接口编写

DASC服务CXF服务不同，不需要通过服务接口挂接任何规范的注解。只要能保证符合Java接口定义即可。但是接口方法的入参和返回值要保证实现了Serializable序列化。

```java
/**
 * DASC协议的服务.
 *
 * @author maeagle
 * @date 2016-6-6 9 :58:56
 */
public interface DascService {
    /**
     * 删除用户.
     *
     * @param userId the user
     * @return the boolean
     * @author maeagle
     * @date 2016-1-27 9 :58:56
     */
    void deleteUser(String userId);
     
    /**
     * 添加用户.
     *
     * @param user the user
     * @return the boolean
     * @author maeagle
     * @date 2016-1-27 9 :58:56
     */
    void addUser(UserInfo user);
}

```

##### 服务实现编写

与服务接口开发类似，服务实现类与普通的Spring服务编写方法相同。服务实现类统一放在“XXX-service-app”工程的“com.newcore.XXX.service.impl”中。

```java
/**
 * Dasc测试服务.
 *
 * @author maeagle
 * @date 2016-1-27 9 :58:56
 */
@Service("dasc.user")
public class DascServiceImpl implements DascService {

    @Autowired
    UserInfoDao userInfoDao;
    
    @Autowired
    NotifyService notifyServiceClient;
    
    @Override
    public void deleteUser(String userId) {
        if (!DascUtils.validateIdempotent(userId))
        return;
        userInfoDao.deleteUser(userId);
    }
    
    @Override
    public void addUser(UserInfo user) {
        if (!DascUtils.validateIdempotent(user.getUserId()))
        return;
        userInfoDao.addUser(user);
    }
}
```

##### 服务发布

DASC服务已经通过XSD做了Spring的协议封装。可以通过Spring配置文件来发布DASC服务。具体发布代码如下：

```xml
<beans xmlns="http://www.springframework.org/schema/beans" 
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dasc="http://www.halo.com/schema/dasc"
       xsi:schemaLocation=" http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.halo.com/schema/dasc
       http://www.halo.com/schema/dasc/dasc.xsd">
    <!-- 发布dascService -->
    <dasc:service id="dascService" interface="com.halo.example.service.api.dasc.DascService" impl-ref="dasc.user" qos="2" retry="2"  persistent="false"/>
</beans>
```

其中，有关dasc:service/节点的配置项描述如下：

| 配置项 | 必要性 | 缺省值 | 配置说明 |
| :--- | :--- | :--- | :--- |
| Id | 必填 |  | 要发布的服务在Spring中的唯一ID。不能与”impl-ref”属性的名称相同。 |
| Interface | 必填 |  | 要发布服务的接口类路径 |
| Impl-ref | 必填 |  | 要发布服务的实现类在Spring注册的beanName。 |
| qos | 选填 | -1 | 声明当前应用节点上，处理该服务请求消息的最大处理能力。即同时在当前应用中执行服务逻辑的线程数。-1为无限制。 |
| retry | 必填 |  | 允许服务处理相同请求消息异常的最大重试次数。 |
| persistent | 选填 | true | 服务调用产生的返回值\(returnValue\)， 1. 是否允许尝试附加在消息体中，通过MQ发送（persistent=false），2. 是否需要持久化到数据库（或缓存）中，以供DASC客户端获取（persistent=true） |

##### 客户端声明

与CXF的客户端声明方式类似，由于DASC框架本质上是一个RPC里面的框架，因此在调用DASC服务前，需要在调用方的XML配置文件中声明服务调用的客户端存根。代码如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dasc="http://www.halo.com/schema/dasc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.halo.com/schema/dasc
       http://www.halo.com/schema/dasc/dasc.xsd">
   <dasc:client id="dascServiceClient" interface="com.halo.example.service.api.dasc.DascService" interceptor-ref="dascServiceCallInterceptor" interrupt-allow="true" response-ref="dascServiceResponseHandler" qos="2" retry="2" persistent="false"/>
</beans>
```

其中，有关dasc:client/节点的配置项描述如下：

| 配置项 | 必要性 | 缺省值 | 配置说明 |
| :--- | :--- | :--- | :--- |
| Id | 必填 |  | 服务客户端在Spring中的唯一ID。 |
| Interface | 必填 |  | 要调用服务的接口类路径 |
| Interceptor-ref | 选填 |  | com.halo.core.dasc.client.api.CallerInterceptor接口的实现类在Spring中的beanName，用于在客户端调用时，提供before和after的拦截。 |
| interrupt-allow | 选填 |  | 对于Interceptor-ref声明的拦截器中before，当抛出异常时，是否中断调用过程。 |
| response-ref | 选填 |  | com.halo.core.dasc.client.api.ResponseHandler接口的实现类在spring注册的beanName。处理调用该服务执行后返回消息的Handler |
| qos | 选填 | -1 | 处理该服务执行后返回消息的最大处理能力。即同时在当前应用中处理服务返回消息逻辑的线程数。-1为无限制。 |
| retry | 选填 | -1 | 允许服务处理相同返回值消息异常的最大重试次数。 |
| persistent | 选填 | true | 调用服务时所需的参数列表\(args\)，1. 是否允许尝试附加在消息体中，通过MQ发送（persistent=false），2. 是否需要持久化到数据库（或缓存）中，以供DASC服务端获取（persistent=true） |

##### 服务调用

在声明好DASC服务客户端后，就可以在工程中以普通Spring注入的方式来调用DASC服务。不过与CXF不同的是，需要在使用客户端调用DASC服务的方法体声明上，添加[@AsynCall](http://localhost:3000/AsynCall)注解。整个过程对于业务代码的视角来看，是察觉不到调用了远程服务的。

```java
@Autowired
private DascService dascServiceClient;

@AsynCall
public void dascCall() {
    String userId = CommonUtils.uuid();
    UserInfo userInfo = new UserInfo();
    userInfo.setUserId(userId);
    userInfo.setUserName("test " + i);
    userInfo.setPassword("password");
    dascServiceClient.addUser(userInfo);
}
```

##### 客户端声明与服务调用动态化

在某些特定的情况下，要求DASC客户端方系统能够按照具体的业务场景，动态的调用多个DASC服务。这种需求在上一小节的静态客户端声明显然不能满足。因此，框架提供了一种能够让开发者实现动态注册客户端，并灵活调用DASC服务的方法。

###### 动态注册客户端

开发者首先需要在Spring容器中注册一个实现com.halo.core.dasc.api. DascClientProvider接口的实体类，并在要求实现的接口方法中构造一个包含客户端注册信息的List。![](/assets/bdVm18D.png)  
动态服务调用

由于通过动态方式注册的客户端没有相应的interface存根类，因此需要通过面向消息的代码风格来调用DASC服务。

```java
@Autowired
private DascDynamicCaller dascDynamicCaller;

@AsynCall
@Override
public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
    dascDynamicCaller.execute("com.newcore.example.service.api.dasc.NotifyService", "sendMessage", "hello world!!!");
}
```

需要注意的是：在使用DascDynamicCaller来调用DASC服务的场景中，在同一个[@AsynCall](http://localhost:3000/AsynCall)的方法体内，有且仅能有一次dascDynamicCaller调用代码存在，不同频次的实例调用的服务名，方法名可以动态选择，没有限制。

#### 原理实现简述

DASC框架的实现逻辑有一定复杂性，本节仅对DASC服务实现的重要环节做阐述。实现细节请参考源码。

##### 举例

##### 为了更好的解释DASC执行过程，以下图的调用过程为举例，结合例子描述执行过程：![](/assets/08.DASC执行过程举例.png)**调用时序图**

![](/assets/09.调用时序图.png)

