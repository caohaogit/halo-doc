## 版本记录

### coldnoodle.2.2.0-RELEASE

* 代码重构

  1. 重新规划批作业监控组件，原有batch-support改为batch-trigger。
  2. 将supports组件分类，分为：core-supports，business-supports，develop-supports。
  3. 在message-support组件中，添加对消息队列的容错机制。
  4. 在header-support组件中，为每个REST/SOAP请求添加默认的报文头信息。
  5. 在halo-dao中，添加了初始化连接数和最小连接数的配置。

* BUG修复

* 新特性  
  **1. 改造核心组件：clic-auth**

  * a）重新规划整个应用认证与授权体系流程。支持认证协议的可插拔式。面向supports提供一套对接接口。并提供标准登陆实现，oauth2服务端实现。
  * b）支持会话缓存与业务缓存分离。

  **2. 改造核心组件：clic-dasc**

  * a）对存在的providerId的场景下，判断providerId是否与当前方法签名相同，进而决定是否发送消息。

  **3. 新增核心组件：clic-tune**

  * a）配置项集中管理。
  * b）基于zookeeper的心跳服务。
  * c）基于flyway，maven，zookeeper，检查数据库与应用节点的程序版本一致性。

  **4. 新增应用代码工程：xxx-db**

  * a）使用flyway管理数据库脚本版本基线。

  **5. 新增扩展组件：batch-trigger**

  * a）基于http，MQ的批作业触发器组件封装。

  **6. 新增扩展组件：batch-admin**

  * a）围绕spring-batch做的批作业监控与管理组件。

  **7. 新增扩展组件：batch-admin**

  * a）围绕spring-batch做的批作业监控与管理组件。

  **8. 新增扩展组件：dbversion-checker**

  * a）读取数据库基线的版本号

  **9. 新增扩展组件：session-router**

  * a）用于对会话缓存的路由分离。

  **10. 新增扩展组件：auth-classic**

  * a）halo-auth扩展，基于oauth2的授权认证组件。

  **11. 新增扩展组件：auth-oauth2-authorizeserver**

  * a）halo-auth扩展，基于oauth2的授权认证组件。

  **12. 新增扩展组件：auth-oauth2-resourceserver**

  * a）halo-auth扩展，基于oauth2的资源访问组件。

  **13. 新增扩展组件：health-checker**

  * a）用于外部代理服务器或负载均衡服务的健康检查服务。

