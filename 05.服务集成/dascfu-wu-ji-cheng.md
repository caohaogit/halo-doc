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

