## SOAP服务集成

### 服务接口编写

CXF使用“javax.jws.\*”注解为SOAP服务接口添加WebService发布信息。服务接口统一放在“XXX-service-api”工程的“com.halo.XXX.service.api”中。

```java
@WebService(targetNamespace = "http://www.e-chinalife.com/soa/")
public interface SoapService {
    // 服务方法声明
}
```

### 服务实现编写

服务实现类与普通的Spring服务编写方法相同。服务实现类统一放在“XXX-service-app”工程的“com.halo.XXX.service.impl”中。

```java
@Service("soapService")
public class SoapServiceImpl implements SoapService {
    // 服务实现
}
```

### 服务发布

基于现有已经写好的业务实现的代码，CXF通过Spring集成提供无缝的服务发布功能。服务发布的XML配置写在“XXX-service-app”工程的“applicationContext-service.xml”中。

```xml
<!-- 发布soapService -->
<jaxws:endpoint id="SoapService" serviceName="soa:SoapService" implementor="#soapService" address="/${ws.protocol.soap.path}/soapService"/>
```

其中，`serviceName="soa:SoapService"`中的`soa`是在xml文件顶端声明的命名空间：`xmlns:soa="http://www.e-chinalife.com/soa/"`。SoapService是用来声明发布的服务名称的。  
`implementor="#soapService"`用来指引jaxws具体的服务实现（这部分其实是通知jaxws通过spring beans的name去找特定的Spring服务，注入到实现中）。  
`id="SoapService"`是提供该服务实例在Spring的唯一标识。建议这个值与`serviceName="soa:SoapService"`后半部分保持一致。`address="/${ws.common.soap.path}/soapService`的`ws.common.soap.path`是用来区分不同的服务协议调用地址的,不建议修改。

### WSDL生成与查看

启动serviceapp后，可以通过`http://XXXXXX/services/服务名称?wsdl`来查看服务发布的wsdl文件

### 客户端声明

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

### 服务调用

在声明好SOAP服务客户端后，就可以在工程中以普通Spring注入的方式来调用SOAP服务。整个过程对于业务代码的视角来看，是察觉不到调用了远程服务的。

```java
@Autowired
SoapService soapService;
// 具体代码实现和调用
```

### JAX-WS规范扩展
`jaxws`是一套成熟的基于Java语言的WebService注解声明规范，覆盖了SOAP1.1/1.2规范的所有xml控制方法。下面以两个例子来说明对JAXWS的使用。

1. `<![CDATA[]]>`标签注入
> 在某些场景下，XML标签节点的value域内需要包含XML语法关键字符。这时就需要通过JAXWS组件向CXF声明包含CDATA标签的XML字段内容。框架已经对该问题提供了组装好的拦截器来实现。具体代码样例如下：

```java
@WebService(targetNamespace = "http://www.e-chinalife.com/soa/")
@OutInterceptors(classes = {ServiceCDATAOutboundImpl.class})
public interface SoapService {
    boolean addUser(UserInfo user);
}
```

如上所示，在添加了`@OutInterceptors(classes = {ServiceCDATAOutboundImpl.class})
`后，框架会通过cxf拦截器，对发送报文的每个字段进行关键字符匹配，如果监测到字段中包含关键字符（`>`, `<`, `'`, `"`, `&`）,就会自动在当前XML节点的value域添加`<![CDATA[]]>`标签。

2. `WRAPPED`与`BARED`
SOAP1.2规范中允许请求/响应报文的业务数据内容是否被构造的根结点包裹。默认情况下，CXF使用的是`WRAPPED`模式，请求和响应报文会根据Java的interface声明情况，自动将请求和返回报文的业务数据部分通过一个XML根节点包裹。如果想变更这种模式，可以参考一下代码：

```java
@WebService(targetNamespace = "http://www.e-chinalife.com/soa/")
@SOAPBinding(parameterStyle = SOAPBinding.ParameterStyle.BARE)
public interface SoapService {
    boolean addUser(UserInfo user);
}
```









