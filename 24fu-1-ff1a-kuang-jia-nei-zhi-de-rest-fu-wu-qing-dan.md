## 附1：框架内置的REST服务清单

### 缓存管理服务

#### 缓存刷新操作

```
地址：http://XXXX:XX/services/rest/cache/refresh?cacheManageServiceName=XXXX
HTTP请求方式：GET
ESB服务ID：RefreshCache
服务治理范围：所有系统的service-app节点
请求报文：无。调用url即可。
响应报文：Boolean
```

#### 缓存删除操作

```
地址：http://XXXX:XX/services/rest/cache/clear?cacheManageServiceName=XXXX
HTTP请求方式：GET
ESB服务ID：ClearCache
服务治理范围：所有系统的service-app节点
请求报文：无。调用url即可。
响应报文：Boolean
```

### 数据字典服务

#### 查询特定字典操作

```
地址：http://XXXX:XX/services/rest/dictService/dict?dictName=XXXX
HTTP请求方式：GET
ESB服务ID：DictDictService
服务治理范围：IPBPS系统的service-app节点
请求报文：无。调用url即可。
响应报文：List<DictItem>
```

#### 查询特定字典的键值操作

```
地址：http://XXXX:XX/services/rest/dictService/item?dictName=XXX&key=XXXX
HTTP请求方式：GET
ESB服务ID：ItemDictService
服务治理范围：IPBPS系统的service-app节点
请求报文：无。调用url即可。
响应报文：DictItem
```

#### 查询特定扩展字典操作

```
地址：http://XXXX:XX/services/rest/dictService/extdict?dictName=XXXX
HTTP请求方式：GET
ESB服务ID：ExtdictDictService
服务治理范围：IPBPS系统的service-app节点
请求报文：无。调用url即可。
响应报文：List<ExtDictItem>
```

#### 查询特定扩展字典的键值操作

```
地址：http://XXXX:XX/services/rest/dictService/extitem?dictName=XXX&key=XXXX
HTTP请求方式：GET
ESB服务ID：ExtitemDictService
服务治理范围：IPBPS系统的service-app节点
请求报文：无。调用url即可。
响应报文：ExtDictItem
```

### DASC服务

#### DASC参数列表获取操作

```
地址：http://XXXX:XX/services/rest/dascArgsService/get?messageId=XXX&providerId=XXX&seq=XXX
HTTP请求方式：GET
ESB服务ID：GetDascArgsService
服务治理范围：所有系统的service-app节点，所有系统的batch-app节点
请求报文：无。调用url即可。
响应报文：String
```

#### DASC参数数量查询操作

```
地址：http://XXXX:XX/services/rest/dascArgsService/count?messageId=XXX&providerId=XXX
HTTP请求方式：GET
ESB服务ID：CountDascArgsService
服务治理范围：所有系统的service-app节点，所有系统的batch-app节点
请求报文：无。调用url即可。
响应报文：String
```

#### DASC返回值获取操作

```
地址：http://XXXX:XX/services/rest/dascReturnService/get?messageId=XXX&providerId=XXX&seq=XXX 
HTTP请求方式：GET
ESB服务ID：GetDascReturnService
服务治理范围：所有系统的service-app节点，所有系统的batch-app节点
请求报文：无。调用url即可。
响应报文：String
```

#### DASC服务重做操作-1

```
地址：http://XXXX:XX/services/rest/dascManageService/redo? eventType=XXX& transactionCode=XXX 
HTTP请求方式：GET
ESB服务ID：RedoDascManageService
服务治理范围：所有系统的service-app节点，所有系统的batch-app节点
请求报文：无。调用url即可。
响应报文：String
```

#### DASC服务重做操作-2

```
地址：http://XXXX:XX/services/rest/dascManageService/redo/target?faildId=XXX 
HTTP请求方式：GET
ESB服务ID：RedoTargetDascManageService
服务治理范围：所有系统的service-app节点，所有系统的batch-app节点
请求报文：无。调用url即可。
响应报文：String
```

### 消息通知与推送服务

#### 发送消息给单个用户操作

```
地址：http://XXXX:XX/services/rest/simpleMessageInfoService/sendMessage
HTTP请求方式：POST
ESB服务ID：SendMessageSimpleMessageInfoService
服务治理范围：IPBPS系统的service-app节点（哪个系统调用，服务治理范围则是对应系统的service-app节点）
请求报文：无。调用url即可。
响应报文：JSon
```

#### 发送广播消息操作

```
地址：http:// XXXX:XX/services/rest/simpleMessageInfoService/bdcastMessage
HTTP请求方式：POST
ESB服务ID：BdcastMessageSimpleMessageInfoService
服务治理范围：IPBPS系统的service-app节点（哪个系统调用，服务治理范围则是对应系统的service-app节点）
请求报文：无。调用url即可。
响应报文：JSon
```

#### 进入列表页面操作

```
地址：http:/// XXXX:XX /services/rest/messageCenterService/list
HTTP请求方式：POST
ESB服务ID：ListMessageCenterService
服务治理范围：IPBPS系统的service-app节点
请求报文：无。调用url即可。
响应报文：JSon
```

#### 标记未读操作

```
地址：http:// XXXX:XX /services/rest/messageCenterService/unreaded
HTTP请求方式：POST
ESB服务ID：UnreadedMessageCenterService
服务治理范围：IPBPS系统的service-app节点
请求报文：无。调用url即可。
响应报文：Int
```

#### 标记已读操作

```
地址：http://XXXX:XX/services/rest/messageCenterService/readed
HTTP请求方式：POST
ESB服务ID：ReadedMessageCenterService
服务治理范围：IPBPS系统的service-app节点
请求报文：无。调用url即可。
响应报文：Int
```

#### 删除消息操作

```
地址：http://XXXX:XX/services/rest/messageCenterService/delete
HTTP请求方式：POST
ESB服务ID：DeleteMessageCenterService
服务治理范围：IPBPS系统的service-app节点
请求报文：无。调用url即可。
响应报文：Int
```

#### 进入详情页面操作

```
地址：http://XXXX:XX/services/rest/messageCenterService/detail?userId=XXX&messageId=XXX
HTTP请求方式：GET
ESB服务ID：DetailMessageCenterService
服务治理范围：IPBPS系统的service-app节点
请求报文：无。调用url即可。
响应报文：JSon
```

#### 获取消息总数操作

```
地址：http://XXXX:XX/services/rest/messageCenterService/total?userId=XXX
HTTP请求方式：GET
ESB服务ID：TotalMessageCenterService
服务治理范围：IPBPS系统的service-app节点
请求报文：无。调用url即可。    
响应报文：JSon
```

### 批作业触发服务

#### 异步触发操作

```
地址：http://XXXX:XX/services/rest/batchTrigger/asyn
HTTP请求方式：POST
ESB服务ID：AsynBatchTrigger
服务治理范围：所有系统的batch-app节点
请求报文：com.halo.core.batch.models. BatchJobCommand结构。    
响应报文：boolean 是否触发成功
```

#### 同步触发操作

```
地址：http://XXXX:XX/services/rest/batchTrigger/syn
HTTP请求方式：POST
ESB服务ID：SynBatchTrigger
服务治理范围：所有系统的batch-app节点
请求报文：com.halo.core.batch.models. BatchJobCommand结构。    
响应报文：boolean 是否触发成功
```



