## 消息通知与推送

框架内部实现消息通知与推送机制，底层采用websocket和stomp协议实现，客户浏览器端中包括消息计数，消息列表，消息通知面板和消息列表及详情部分，相关效果如下图标红部分。服务器端message -support提供rest接口，允许系统或者客户推送消息给浏览器端，推送报文中含有推送人标识，如所有人则填\*，是否需要存自身系统数据库（需要根据给定的sql脚本对自身数据库建表）做对应操作。然后消息根据报文简单处理后分别落入本地数据库及推送给浏览器客户端。

### 设计概述

涉及技术包括：html5 websocket及stomp协议，rabbitmq及stomp扩展插件，spring-message，spring-websocket，cxf\(restful\)，参考构件图  
活动图：[![](http://i.imgur.com/VMkEO8a.png)](http://i.imgur.com/VMkEO8a.png)

1. 客户浏览器端初始化消息数据：读取系统数据库，初始化数据未读条数；
2. 客户浏览器端发起让服务器端创建用户队列，指定time to live消息过期时间5分钟，queue length limit数量为11；
3. 客户浏览器端监听通知消息：用户消息\(wsuserid\)；
4. 监听到消息后计算消息数量，消息类型然后消息提示框提示消息信息；
5. 发送消息：系统调用rest接口发送消息，消息对必输字段和合法性做基本校验（不合法返回给调用客户端错误信息），如果需要做对应系统存储，则存储对应系统；
6. 校验合法的信息发送给名字为message的（type=topic）rabbitmq exchange ；
7. message 交换机根据routingkey路由到database\_queue\(\#\)队列和clientmessage（type=topic）\(\#.ws.user.\#\)\) 交换机以及broadcastmsg\(type=fanout\)\(\#.ws.broadcast.\#\)交换机；
8. 监听到database\_queue队列消息后message-support存消息到本地数据库；
9. 消息路由到clientmessage后根据routingkey（ws.\[userid\]）路由到ws\_\[userid\]\_queue
10. 消息路由到broadcastcastmsg路由后，广播到到所有ws\_\[userid\]\_queue

路由详情参看如下图：[![](http://i.imgur.com/0ZSckei.png)](http://i.imgur.com/0ZSckei.png)

构件图如下：[![](http://i.imgur.com/vuWGm2p.png)](http://i.imgur.com/vuWGm2p.png)

### 接口调用

接口包括两部分，服务器端（用于展示消息）和客户端（用于发送消息），参见esb对接接口文档；

服务器端：

1.消息类型汇总计数  
  url:xxx/message/total  
  对应字段为

| **输入** |  |  |  |
| :--- | :--- | :--- | :--- |
| userId | 用户ID | string\(20\) | userId |
| **输出** |  |  |  |
| jsonArray |  | array | jsonArray |
| jsonArray.messageType | 消息类型 | string\(2\) | messageType |
| jsonArray.messageCount | 消息数量 | int\(4\) | messageCount |
| jsonArray.typeName | 消息类型名称 | string\(20\) | typeName |

2.消息列表  
  url:xxx/message/list  
  对应字段为

| **输入** |  |  |  |
| :--- | :--- | :--- | :--- |
| userId | 用户ID | string\(20\) | userId |
| pageStartNo | 当前页首行记录编号（从0开始） | long | pageStartNo |
| pageSize | 每页显示记录数 | int\(4\) | pageSize |
| jsonArray |  | array | jsonArray |
| jsonArray.columnName | 要排序的列名 | string\(20\) | columnName |
| jsonArray.orderType | 升序&降序 | string\(20\) | orderType |
| condition |  |  | condition |
| condition.start | 开始发信时间 | string\(30\) | start |
| condition.end | 发信截止时间 | string\(30\) | end |
| condition.messageType | 消息类型 | string\(2\) | messageType |
| condition.msgReadedStatus | 消息已读标识 | string\(2\) | msgReadedStatus |
| condition.searchKey | 查询关键字 | string\(30\) | searchKey |
| condition.msgReceiver | 消息收件人 | string\(50\) | msgReceiver |
| **输出** |  |  |  |
| totalCount | 总记录数 | long | totalCount |
| jsonArray |  | array | jsonArray |
| jsonArray.messageId | 消息ID | string\(32\) | messageId |
| jsonArray.msgSendSysId | 消息发送方系统ID | string\(30\) | msgSendSysId |
| jsonArray.msgSender | 消息发件人 | string\(50\) | msgSender |
| jsonArray.msgReceiver | 消息收件人 | string\(50\) | msgReceiver |
| jsonArray.title | 标题 | string\(100\) | title |
| jsonArray.msgCont | 消息正文 | string\(800\) | msgCont |
| jsonArray.messageType | 消息类型 | string\(2\) | messageType |
| jsonArray.messageType.key | 代码KEY | string\(8\) | key |
| jsonArray.messageType.description | 描述 | string\(80\) | description |
| jsonArray.msgSendTime | 消息发件时间 | date\(12\) | msgSendTime |
| jsonArray.msgPriority | 消息优先级 | int\(2\) | msgPriority |
| jsonArray.msgDisplayType | 消息展示方式 | string\(2\) | msgDisplayType |
| jsonArray.msgReadedStatus | 消息已读标识 | string\(2\) | msgReadedStatus |
| jsonArray.msgReadedStatus.key | 代码KEY | string\(8\) | key |
| jsonArray.msgReadedStatus.description | 描述 | string\(80\) | description |

3.消息设置已读  
  url:xxx/message/read  
  对应字段为

| **输入** |  |  |  |
| :--- | :--- | :--- | :--- |
| userId | 用户ID | string\(20\) | userId |
| jsonArray |  | array | jsonArray |
| jsonArray.messages | 消息 | string\(128\) | messages |
| **输出** |  |  |  |
| readed | 修改是否成功 | int\(4\) | readed |

4.消息设置未读  
  url:xxx/message/unread  
  对应字段为

| **输入** |  |  |  |
| :--- | :--- | :--- | :--- |
| userId | 用户ID | string\(20\) | userId |
| jsonArray |  | array | jsonArray |
| jsonArray.messages | 消息 | string\(128\) | messages |
| **输出** |  |  |  |
| unreaded | 修改是否成功 | int\(4\) | unreaded |

5.消息删除  
  url:xxx/message/delete  
  对应字段为

| **输入** |  |  |  |
| :--- | :--- | :--- | :--- |
| userId | 用户ID | string\(20\) | userId |
| jsonArray |  | array | jsonArray |
| jsonArray.messages | 消息 | string\(128\) | messages |
| **输出** |  |  |  |
| delete | 删除是否成功 | int\(4\) | delete |

6.获取stomp连接信息。 客户端：

 ① 发送单条消息  
   url:/xxx/message/sendto/{userId}  
   对应字段为

| **输入** |  |  |  |
| :--- | :--- | :--- | :--- |
| messageId | 消息ID | string\(32\) | messageId |
| msgSendSysId | 消息发送方系统ID | string\(30\) | msgSendSysId |
| msgSender | 消息发件人 | string\(50\) | msgSender |
| msgReceiver | 消息收件人 | string\(50\) | msgReceiver |
| title | 标题 | string\(100\) | title |
| msgCont | 消息正文 | string\(800\) | msgCont |
| messageType | 消息类型 | string\(2\) | messageType |
| msgSendTime | 消息发件时间 | date\(12\) | msgSendTime |
| msgPriority | 消息优先级 | int\(2\) | msgPriority |
| msgDisplayType | 消息展示方式 | string\(2\) | msgDisplayType |
| saveLocal | 本地保存 | boolean | saveLocal |
| **输出** |  |  |  |
| success | 发送是否成功 | boolean | success |
| message | 失败原因 | string\(128\) | message |

 ② 发送广播消息 <br>
   url:/xxx/message/broadcast <br>
   对应字段为

| **输入** |  |  |  |
| :--- | :--- | :--- | :--- |
| messageId | 消息ID | string\(32\) | messageId |
| msgSendSysId | 消息发送方系统ID | string\(30\) | msgSendSysId |
| msgSender | 消息发件人 | string\(50\) | msgSender |
| msgReceiver | 消息收件人 | string\(50\) | msgReceiver |
| title | 标题 | string\(100\) | title |
| msgCont | 消息正文 | string\(800\) | msgCont |
| messageType | 消息类型 | string\(2\) | messageType |
| msgSendTime | 消息发件时间 | date\(12\) | msgSendTime |
| msgPriority | 消息优先级 | int\(2\) | msgPriority |
| msgDisplayType | 消息展示方式 | string\(2\) | msgDisplayType |
| saveLocal | 本地保存 | boolean | saveLocal |
| **输出** |  |  |  |
| success | 发送是否成功 | boolean | success |
| message | 失败原因 | string\(128\) | message |

### 使用方法

#### 配置项修改

1. 客户端对接需要引入message-support依赖，并配置mqconfig.properties队列路由信息，增加message路由队列如下。

   ```properties
   # 多个mq时候需要配置queue.list,多个配置以“,”分割，如queue.list=message,XXX
   queue.list=message
   # 消息队列broker的地址.用,分割
   message.queue.addresses=
   # 消息队列上的虚拟主机标识
   message.queue.vhost=
   # 连接时的用户名
   message.queue.username=
   # 连接时的密码
   message.queue.password=
   mq.message.prefix=message
   mq.message.qos=2
   # stomp端口
   mq.message.port=15674
   # 队列前缀标识
   mq.message.test.id=test
   ```

2. 配置 xxx.properties，配置消息队列前缀标识

   ```properties
   maven.mq.message.test.id=test
   ```

#### 服务发布

1. server端发布  
   具体发布代码如下：

   ```xml
   <jaxrs:server beanNames="simpleMessageInfoService" address="/${ws.protocol.rest.path}/simpleMessageInfoService">
       <jaxrs:providers>
           <bean class="com.halo.core.fastjson.support.FastJsonProvider"/>
       </jaxrs:providers>
   </jaxrs:server>
   <!-- database queue服务处理 -->
   <bean id="messageDataDBService" class="com.newcore.supports.service.impl.server.MessageDataDBServiceImpl"></bean>
   <!-- 配置服务器中广播级别消息推送队列和发送对应的exchange -->
   <bean class="com.newcore.supports.service.impl.server.MessageInitFactoryImpl"></bean>
   <bean id="messageCenterService" class="com.newcore.supports.service.impl.server.MessageCenterServiceImpl"></bean>
   <!-- 发布cxf的restfulService-->
   <jaxrs:server beanNames="messageCenterService" address="/${ws.protocol.rest.path}/messageCenterService">
       <jaxrs:providers>
           <bean class="com.halo.core.fastjson.support.FastJsonProvider"/>
       </jaxrs:providers>
   </jaxrs:server>
   ```

2. client端发布  
   具体发布代码如下：

   ```xml
   <!--RESTFul WebService客户端注册 -->
   <jaxrs-client:client id="simpleMessageInfoService" serviceClass="com.newcore.supports.service.api.client.SimpleMessageInfoService" address="http://${ws.client.ipbps.address}/${ws.protocol.rest.path}/simpleMessageInfoService">
       <jaxrs-client:providers>
           <bean class="com.halo.core.fastjson.support.FastJsonProvider"/>
       </jaxrs-client:providers>
   </jaxrs-client:client>

   <!--RESTFul WebService客户端注册 -->
   <jaxrs-client:client id="messageCenterService" serviceClass="com.newcore.supports.service.api.server.MessageCenterService" address="http://${ws.client.ipbps.address}/${ws.protocol.rest.path}/messageCenterService">
       <jaxrs-client:providers>
           <bean class="com.halo.core.fastjson.support.FastJsonProvider"/>
       </jaxrs-client:providers>
   </jaxrs-client:client>
   ```

#### 客户端对接消息

1. 客户端可以直接调用柜面系统的接口，发送消息（不能存本地数据库），消息只能存入柜面数据库（这种方式自身不用任何配置，也不需要引用message-support）；
   示例：
   参见example下/example-web-app/src/main/webapp/example/message/js/message.js
2. 客户端引用message-support包，页面调用对应发送消息的controller；
   示例：
   参见example下/example-web-app/src/main/webapp/example/sendmsg/js/message.js
3. 客户端引用message-support包，service-app中把发送消息作为普通service直接调用接口；
   示例：
   参见support下/message-support/src/test/java/com/newcore/test/service/ TestSendMessage.java
4. 客户端引用message-support包，service-app中把发送消息作为web-service调用接口；



