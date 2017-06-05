# halo技术参考

This file file serves as your book's preface, a great place to describe your book's content and ideas.

## 概述

Halo为企业应用服务平台提供一套“开箱即用”的统一开发与运行框架体系。包含了从Web前端交互，到后端业务处理集群，最终落地到各种资源容器的存储。同时，有效集成了目前业内较为成熟稳定的组件（授权，缓存，异步消息等），让使用者更加关注具体业务的实现。

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

## 文档说明

该文档会依据halo框架版本升级窗口期，以全量形式发布（版本记录只会保留当前版本的内容）。每次发布的全量版本与上一版本相比修订的内容会以修订模式显示。

## 快速入门

待完善

## 体系结构

框架采用分布式系统的代码工程结构。兼容“服务总线”与“服务治理”两种服务调度理念对应的RPC框架。对于资源访问方面，集成了缓存，文件，关系型数据库等资源容器访问组件。

在框架的设计理念中，对于“业务实现与调度”方面的“分治”比较倚重。在后续的框架改造过程中，也会对消息服务于异步服务进行更深的优化。

