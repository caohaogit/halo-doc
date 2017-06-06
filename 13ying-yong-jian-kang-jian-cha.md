## 应用健康检查

框架内置了主机的心跳服务和健康检查机制。心跳服务为运维人员在应用集群场景下，快速定位存在问题的应用提供帮助；健康检查通过url，为前端负载均衡提供主机存活状态。

### 基于zookeeper的心跳服务

在应用启动后，框架通过系统运行时自动在zookeeper上创建基于Connection实例的多个临时节点：

```
1.“ /应用服务类型/heartbeat/主机名: listenIP”：在线的主机对应的IP清单。
2.“ /应用服务类型/heartbeat/主机名: startTime”：主机上线时间。
3.“ /应用服务类型/heartbeat/主机名: accessable”：主机是否可访问。
```

### 基于url的健康检查

框架对“Web代理转发/负载均衡服务”提供了一个基于URL的主机健康检查服务。该服务会实时监控zookeeper中的“/应用服务类型/heartbeat/主机名: accessable”的节点值。一旦该值发生变更，健康检查服务会实施感知到开关值的变化。进而能够满足运维人员实时控制应用节点的优雅卸载。

1. 监控的zookeeper节点：“/应用服务类型/heartbeat/主机名: accessable”。
2. 暴露的url地址：“[http://XXXX/healthchecker?type=accessable”。](http://xxxx/healthchecker?type=accessable%E2%80%9D%E3%80%82)
3. HTTP状态码为200，并且报文内容为“true”，则认为主机可访问；其他情况都视为主机不可访问。
4. 每个主机应用程序中需要添加health-checker插件包。同时要在web.xml中添加一个servlet。具体代码如下：

   ```xml
   <servlet>
       <servlet-name>healthChecker</servlet-name>
       <servlet-class>com.halo.supports.web.CheckHealthDispather</servlet-class>
   </servlet>
   <servlet-mapping>
       <servlet-name>healthChecker</servlet-name>
       <url-pattern>/healthchecker</url-pattern>
   </servlet-mapping>
   ```



