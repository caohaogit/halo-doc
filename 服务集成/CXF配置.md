## CXF配置

1. ws.common.name用来标识当前工程在服务调用与服务发布时，标识的唯一服务节点。（如果当前业务服务系统按照集群部署，则该值应该在集群内保证一致。）
2. ws. protocol.\*\*\*.path某种协议的上下文路径。
3. ws.protocol.\*\*\*.header 某种协议是否启用header头（soap用soap header，http用 http header）里面会填充有关这次交易的信息。

#### CXF介绍

CXF是较为流行的Apache下WebService实现，它将WebService的服务发布与服务调用过程分别通过SOAP XML与RESTful两种规范分别实现。  
其中SOAP WebService是传统的WebService调用，这也是CXF的强项。RESTful WebService是在CXF 3.X中做了实现，基于jaxrs标准。  
CXF的整体设计理念是：通过“拦截器”和“总线”两套机制，实现对WebService报文从调用方到服务方的数据封装与解封装的过程。  
关于CXF更多的参考，可以访问Apache网站：[http://cxf.apache.org/docs/index.html](http://cxf.apache.org/docs/index.html)

#### 总线配置

为了实现一系列的控制和旁路功能，框架对CXF默认的总线服务做了定制。定制过程是基于Spring XML Configuration的。代码如下：

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



