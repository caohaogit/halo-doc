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

## 技术蓝图

![](http://i.imgur.com/cHBqn3q.png)

## 技术概况

[![](http://i.imgur.com/mjYfLZL.png)](http://i.imgur.com/mjYfLZL.png)

## 服务应用

[![](http://i.imgur.com/ciFrb3D.png)](http://i.imgur.com/ciFrb3D.png)

## 批作业应用

[![](http://i.imgur.com/Nc9nzTR.png)](http://i.imgur.com/Nc9nzTR.png)

## Web交互应用

[![](http://i.imgur.com/sENriqm.png)](http://i.imgur.com/sENriqm.png)

## 工程结构

[![](http://i.imgur.com/Gch9BIr.png)](http://i.imgur.com/Gch9BIr.png)

## 配置清单

### core内置配置

文件名：coreconfig.properties

| 配置项 | 必要性 | 缺省值 | 配置说明 |
| :--- | :--- | :--- | :--- |
| web.message.0000 | 必填 | 处理成功 | 执行成功，message中的消息内容 |
| web.message.9999 | 必填 | 处理失败 | 执行失败，message中的默认消息内容 |
| web.message.0003 | 选填 | 出现空值错误 | 出现NullPointerException时，message中的消息内容 |
| web.message.0004 | 选填 | 数据库操作失败 | 出现SQLException时，message中的消息内容 |
| web.message.0005 | 选填 | 找不到文件 | 出现FileNotFoundException时，message中的消息内容 |
| web.message.0006 | 选填 | 用户认证失败 | 出现TokenInvalidException时，message中的消息内容 |
| web.message.0007 | 选填 | 连接被重置 | 出现ConnectException时，message中的消息内容 |
| web.message.2001 | 选填 | SOAP服务调用失败 | 出现WebServiceException时，message中的消息内容 |
| web.message.2002 | 选填 | RESTful服务调用失败 | 出现ProcessingException时，message中的消息内容 |
| web.message.2003 | 选填 | SOAP服务调用失败 | 出现SOAPFaultException时，message中的消息内容 |
| web.message.2011 | 选填 | 服务执行失败 | 出现Fault时，message中的消息内容 |
| web.message.6666 | 选填 | ESB服务调用异常 | 出现SocketTimeoutException时，message中的消息内容 |
| web.auth.session.userid | 必填 | \_USERID | 使用token机制进行认证时，在sessionMap放置的用户名的Key |
| system.encoding | 必填 | UTF-8 | 应用要求JVM采用的字符集 |

### 数据库访问配置

文件名：jdbc.properties

| 配置项 | 必要性 | 缺省值 | 配置说明 |
| :--- | :--- | :--- | :--- |
| jdbc.dynamic | 必填 | false | 数据库动态路由开关 |
| jdbc.dynamic.list | 选填 |  | 动态路由的数据源列表Bean标识, 多个以逗号分割 |
| jdbc.dynamic.interceptor.query | 选填 |  | 对jdbctemplate中“query\*”的方法拦截。（每个配置项可以填写唯一个路由数据源.多填无效,不填默认走default数据源） |
| jdbc.dynamic.interceptor.update | 选填 |  | 对jdbctemplate中“update\*”的方法拦截。（每个配置项可以填写唯一个路由数据源.多填无效,不填默认走default数据源） |
| jdbc.dynamic.interceptor.call | 选填 |  | 对jdbctemplate中“call\*”的方法拦截。（每个配置项可以填写唯一个路由数据源.多填无效,不填默认走default数据源） |
| jdbc.dynamic.interceptor.batch | 选填 |  | 对jdbctemplate中“batch\*”的方法拦截。（每个配置项可以填写唯一个路由数据源.多填无效,不填默认走default数据源） |
| jdbc.debug.logging | 必填 | true | SQL操作日志开关，需要在logback.xml中打开DEBUG级别才能生效 |
| jdbc.initConnectionCount | 必填 |  | 连接池初始化时的连接数 |
| jdbc.minConnectionCount | 必填 |  | 连接池保持的最小连接数 |
| jdbc.maxConnectionCount | 必填 |  | 最大连接数 |
| jdbc.password.decrypt | 必填 | false | 数据库密码是否加密 |
| jdbc.poolPreparedStatements | 必填 | true | 是否打开PSCache。分库分表较多的数据库，建议配置为false |
| jdbc.maxPoolPreparedStatementPerConnectionSize | 必填 | 20 | 每个连接挂载的最大PS缓存数量 |
| Jdbc.transaction.timeout | 必填 | 60 | 数据库事务超时时长（秒） |
| jdbc.ds.default.name | 必填 | defaultDataSource | 默认数据源的别名。当defaultDataSource的Bean实例为Provider时, 该值为指定真实数据源的Bean名称 |
| jdbc.ds.default.url | 必填 |  | 数据库连接URL |
| jdbc.ds.default.user | 必填 |  | 用户名 |
| jdbc.ds.default.password | 必填 |  | 密码 |
| jdbc.ds.default.password.publickey | 选填 |  | 当“jdbc.password.decrypt”为true时，解密所需的公钥 |

### 日志配置

文件名：logback.xml

| 配置项 | 必要性 | 缺省值 | 配置说明 |
| :--- | :--- | :--- | :--- |
| LOG\_HOME | 必填 |  | 日志文件存储的文件夹根路径 |
| maxQueueSize | 必填 | 512 | 当使用异步日志时，日志队列中存储的最大日志对象个数 |
| discardingThreshold | 必填 | 0 | 当使用异步日志时，日志队列到达20%容量时，是否要丢弃TRACE、DEBUG和INFO级别的event。0为不丢弃 |
| charset | 必填 | UTF-8 | 字符集 |
| contentPattern | 必填 | \[\[\[ %d {yyyy-MM-dd HH:mm:ss.SSS} &\#124; %mdc{tcode} &\#124; %level &\#124; %mdc{path} &\#124; %logger{100} &\#124; %mdc{act} &\#124; %msg \]\]\] %n | 日志的打印形式 |
| fullFileNamePattern | 选填 | full-%d{yyyy-MM-dd}.log | 全量日志文件的时间滚动生成策略 |
| coreFileNamePattern | 选填 | core-%d{yyyy-MM-dd}.log | 核心组件core日志文件的时间滚动生成策略 |
| daoFileNamePattern | 选填 | dao-%d{yyyy-MM-dd}.log | dao日志文件的时间滚动生成策略。XXX-service-app，XXX-batch-app工程必填 |
| interfaceFileNamePattern | 必填 | interface-%d{yyyy-MM-dd}.log | 接口日志文件的时间滚动生成策略 |
| webFileNamePattern | 必填 | web-%d{yyyy-MM-dd}.log | web日志文件的时间滚动生成策略 |
| serviceFileNamePattern | 必填 | service-%d{yyyy-MM-dd}.log | service日志文件的时间滚动生成策略 |
| &lt;pattern&gt;&lt;/pattern&gt; | 必填 | asyncContentPattern | 针对某个日志输出，切换输出格式。syncContentPattern，asyncContentPattern |
| &lt;logger&gt;&lt;/logger&gt; | 必填 |  | 开启或关闭某个包下的日至输出 |

### 文件管理配置

文件名：filestore.properties

| 配置项 | 必要性 | 缺省值 | 配置说明 |
| :--- | :--- | :--- | :--- |
| fs.enabled | 必填 | true | 是否启用资源服务 |
| fs.name | 必填 | 工程对应的名称 | 1.当访问协议为MONGO时，对应的是数据库名。2.当访问协议为DISK时，对应的是文件夹名称 |
| fs.protocol | 必填 | FILE | 资源访问协议：DISK, MONGO |
| fs.tmpdir | 必填 | temp | 本地临时文件夹路径 |
| fs.backupOnUpdate | 必填 | true | 当进行资源更新操作时，进行备份 |
| fs.mongo.address | 选填 |  | 协议为MONGO，访问mongodb的地址 |
| fs.mongo.username | 选填 |  | 协议为MONGO，访问mongodb的用户名 |
| fs.mongo.password | 选填 |  | 协议为MONGO，访问mongodb的密码 |
| fs.mongo.chunk | 选填 | 1024 | 协议为MONGO，每个mongodb文件块大小 |
| fs.mongo.connections | 选填 |  | 协议为MONGO，最大连接数 |
| fs.ftp.address | 选填 |  | 协议为FTP，访问地址 |
| fs.ftp.username | 选填 |  | 协议为FTP，访问用户名 |
| fs.ftp.password | 选填 |  | 协议为FTP，访问密码 |
| fs.file.basedir | 选填 |  | 协议为DISK，访问文件仓库的基础路径 |

### 缓存管理配置

文件名：cache.properties

| 配置项 | 必要性 | 缺省值 | 配置说明 |
| :--- | :--- | :--- | :--- |
| cache.dynamic | 必填 | false | 是否启用动态缓存服务 |
| cache.name | 必填 |  | 作为存入缓存中的Key值前缀 |
| cache.type | 必填 | REDIS | 缓存容器类型：REDIS |
| cache.redis.default.mode | 必填 | SINGLETON | 默认缓存的Redis运行模式: SINGLETON\(单例\)，CLUSTER\(集群\)，SHAREED\(分片\) |
| cache.redis.default.addresses | 必填 |  | 默认缓存的Redis服务地址集合。格式: host1:port1;host2:port2 |
| cache.redis.default.timeout | 选填 | 1000 | 默认缓存的Redis连接超时时长\(毫秒\) |
| cache.redis.default.maxRedirections | 选填 | 6 | 当默认缓存的运行模式为CLUSTER时, 允许重定向的最大次数 |
| cache.redis.default.maxIdle | 必填 | 200 | 默认缓存的Redis连接池的最大空闲数。0为不限制 |
| cache.redis.default.maxActive | 必填 | 1024 | 默认缓存Redis连接池的最大连接数。0为不限制 |
| cache.redis.default.maxWait | 必填 | 1000 | 默认缓存Redis连接池建立连接的最长等待时间。超过此时间将接到异常。-1为无限制。 |
| cache.redis.default.testOnBorrow | 选填 | true | 默认缓存Redis连接池获取连接前是否经过校验 |
| cache.redis.XXX.mode | 选填 |  | 参考cache.redis.default.mode |
| cache.redis.XXX.addresses | 选填 |  | 参考cache.redis.default.addresses |
| cache.redis.XXX.timeout | 选填 |  | 参考cache.redis.default.timeout |
| cache.redis.XXX.maxRedirections | 选填 |  | 参考cache.redis.default.maxRedirections |
| cache.redis.XXX.maxIdle | 选填 |  | 参考cache.redis.default.maxIdle |
| cache.redis.XXX.maxActive | 选填 |  | 参考cache.redis.default.maxActive |
| cache.redis.XXX.maxWait | 选填 |  | 参考cache.redis.default.maxWait |
| cache.redis.XXX.testOnBorrow | 选填 |  | 参考cache.redis.default.testOnBorrow |

### Web交互配置

文件名：webconfig.properties

| 配置项 | 必要性 | 缺省值 | 配置说明 |
| :--- | :--- | :--- | :--- |
| web.message.business.\*\*\* | 选填 |  | 对功能代码中可能抛出的业务异常进行“错误码：错误描述”的映射 |
| web.session.timeout | 必填 | 1200000 | session超时时间\[毫秒\] |
| web.maxUploadSize | 必填 | 5 | Web文件上传的最大容量\(MB\) |
| system.version | 必填 | ${project.version} | 当前工程系统版本号 |
| batch.asynjob.threadLimit | 必填 | 1000 | 执行异步批作业Job的线程池大小 |

### 服务集成配置

文件名：wsconfig.properties

| 配置项 | 必要性 | 缺省值 | 配置说明 |
| :--- | :--- | :--- | :--- |
| ws.common.name | 必填 | 工程名称 | 服务名称。使用场景如下：1.Header.ORISYS，2.Header.FROMSYS，3.dubbo的ApplicationName |
| ws.common.logging | 必填 |  | 开启服务调用日志记录 |
| ws. protocol.rest.path | 必填 | rest | restful服务地址前缀 |
| ws. protocol.soap.path | 必填 | soap | soap服务地址前缀 |
| ws. protocol.http.path | 必填 | http | http服务地址前缀 |
| ws. protocol.soap.header | 必填 | true | 调用soap服务时，是否启用Header |
| ws. protocol.rest.header | 必填 | true | 调用restful服务时，是否启用Header |
| ws.client.timeout | 必填 | 60000 | 客户端服务调用超时\(毫秒\) |
| ws.client.XXX.address | 选填 |  | 服务调用的地址。XXX为系统 |
| ws.service.timeout | 必填 | 60000 | 服务端处理超时\(毫秒\) |
| ws.service.soap.namespace | 选填 | [http://www.e-chinalife.com/soa/](http://www.e-chinalife.com/soa/) | SOAP服务发布时，默认的命名空间 |

文件名：dasc.properties

| 配置项 | 必要性 | 缺省值 | 配置说明 |
| :--- | :--- | :--- | :--- |
| dasc.name | 必填 |  | 作为dasc调用方，向服务方展示的名称 |
| dasc.test.id | 选填 |  | 控制dasc是否处于测试模式下：队列不做持久化，队列名称添加test.id前缀，消息不做持久化，在开发测试场景下，可以隔离不同队列多个消费者产生的数据消费问题 |
| dasc.send.ack | 必填 | true | 要求在发送dasc消息时，要收到ack回执 |
| dasc.send.ack.timeout | 必填 | 10000 | 在dasc.send.ack=true的情况下，等待ack的超时设定（毫秒） |
| dasc.cache.enabled | 必填 | true | 是否在缓存中暂存调用数据 |
| dasc.cache.timeout | 必填 | 600000 | 一次调用涉及到的数据在缓存中存放的时间（毫秒） |
| dasc.cache.limit | 必填 | 1024 | 一次调用涉及到的数据允许放入缓存的大小 \(KB\) |
| dasc.caller.args.address | 必填 | [http://XXXX/XXX/serviceapp/services/rest/dasc/args](http://xxxx/XXX/serviceapp/services/rest/dasc/args) | 调用方提供参数列表服务的服务地址 |
| dasc.caller.args.protocols | 必填 | REST | 调用方提供参数列表服务的服务类型 |
| dasc.caller.args.limit | 必填 | 100 | 调用方的服务参数列表能否放入MQ中的大小（KB） |
| dasc.provider.return.address | 必填 | [http://XXXX/XXX/serviceapp/services/rest/dasc/returnValue](http://xxxx/XXX/serviceapp/services/rest/dasc/returnValue) | 返回值服务的服务地址 |
| dasc.provider.return.protocols | 必填 | REST | 返回值服务的服务类型 |
| dasc.provider.return.limit | 选填 | 100 | 服务的返回值能否放入MQ中的大小（KB） |

### 消息队列配置

文件名：mqconfig.properties

| 配置项 | 必要性 | 缺省值 | 配置说明 |
| :--- | :--- | :--- | :--- |
| queue.list | 选填 |  | 所有注册的broker名称列表。用逗号分割。 |
| queue.addresses | 必填 |  | 默认broker的地址。用分号分割 |
| queue.vhost | 必填 | / | 默认broker上的虚拟主机标识 |
| queue.username | 必填 | guest | 链接默认broker的用户名 |
| queue.password | 必填 | guest | 连接默认broker的密码 |
| queue.recovery.enabled | 必填 | true | 默认broker是否自动恢复连接 |
| queue.recovery.interval | 必填 | 10000 | 默认broker连接自动恢复的执行间隔\(毫秒\) |
| queue.headbeat.interval | 必填 | 60 | 对默认broker的心跳检测间隔\(秒\) |
| XXX.queue.addresses | 选填 |  | 参考queue.addresses |
| XXX.queue.vhost | 选填 |  | 参考queue.vhost |
| XXX.queue.username | 选填 |  | 参考queue.username |
| XXX.queue.password | 选填 |  | 参考queue.password |
| XXX.queue.recovery.enabled | 选填 |  | 参考queue.recovery.enabled，不写默认取queue.recovery.enabled |
| XXX.queue.recovery.interval | 选填 |  | 参考queue.recovery.interval，不写默认取queue.recovery.interval |
| XXX.queue.headbeat.interval | 选填 |  | 参考queue.headbeat.interval，不写默认取queue.headbeat.interval |

### 消息通知与推送通知

文件名：mqconfig.properties

| 配置项 | 必要性 | 缺省值 | 配置说明 |
| :--- | :--- | :--- | :--- |
| queue.list | 必填 | message | 所有注册的broker名称列表。用逗号分割。 |
| message.queue.addresses | 必填 |  | 默认broker的地址。用分号分割。 |
| message .queue.vhost | 必填 | / | 默认broker上的虚拟主机标识。 |
| message .queue.username | 必填 |  | 链接broker的用户名 |
| message .queue.password | 必填 |  | 连接默认broker的密码 |
| message.queue.recovery.enabled | 选填 |  | 是否自动恢复连接。不写默认取queue.recovery.enabled |
| message.queue.recovery.interval | 选填 |  | 连接自动恢复的执行间隔\(毫秒\) 。不写默认取：queue.recovery.interval |
| message.queue.heartbeat.interval | 选填 |  | 心跳检测间隔\(秒\)。不写默认取queue.heartbeat.interval |
| mq.message.prefix | 必填 | message | 对STOMP连接队列的名称 |
| mq.message.qos | 必填 | 2 | STOMP连接队列的消费负载 |
| mq.message.port | 必填 | 15674 | STOMP连接端口 |
| mq.message.test.id | 选填 |  | 控制消息推送组件是否处于测试模式。在开发测试场景下，可以隔离不同队列多个消费者产生的数据消费问题 |

### 幂等组件配置

文件名：可选

| 配置项 | 必要性 | 缺省值 | 配置说明 |
| :--- | :--- | :--- | :--- |
| idemp.persist | 选填 | idempotentDbDao | 使用幂等组件时，对幂等要素存储的容器指定。idempotentDbDao：数据库， idempotentCacheDao：缓存 |

### 协调组件配置

文件名：zkconfig.properties

| 配置项 | 必要性 | 缺省值 | 配置说明 |
| :--- | :--- | :--- | :--- |
| zk.addresses | 必填 |  | zookeeper的连接地址.多个地址用逗号分隔 |
| zk.retry.count | 必填 | 3 | 开始连接zookeeper时，可重试的次数 |
| zk.retry.interval | 必填 | 1000 | 开始连接zookeeper时，每次重试的间隔基数（毫秒） |
| zk.namespace | 必填 | 工程名称 | 工程在zookeeper的命名空间。一般定义为系统标识。 |
| zk.app.type | 必填 | 工程的应用类型 | 工程的应用类型。一般为：serviceapp、webapp、batchapp |
| zk.configcenter.enabled | 必填 | true | zookeeper配置中心开关 |
| zk.configcenter.db.encrypt | 必填 | false | zookeeper配置中心启用后，对数据库密码是否自动启动加密 |
| zk.version.enabled | 必填 | false | zookeeper版本号校验开关 |
| zk.version.db.tablename | 选填 | SCHEMA\_VERSION | 对数据库版本检查的表名 |
| zk.heartbeat.enabled | 必填 | true | zookeeper心跳开关 |



