## 重要扩展插件

> 该部分主要提供的是基于核心扩展插件集合（core-supports工程 下）中包含的扩展插件清单。也会适当将相对重要的业务扩展插件放入其中（例如：数据字典组件，批作业管理组件等）。

### 数据字典组件（dict-support）

平台对数据字典数据提供统一的访问服务。其中包括：

1. 启动自动加载数据字典数据至缓存。
2. 对全部数据字典数据“枚举化”（每个数据字典转换为一个枚举类型供开发人员使用）。
3. 在BO对象中，对属于字典项填充的字段，可以用Dict类声明。框架会自动帮助做数据映射。
4. BO对象中的Dict类型成员变量A，在通过JSON序列化时，会自动拆分为A，和ADesc两个字符串类型数据。
5. 框架对外提供了两个基于URL的Web服务：
   `http://XXX/dict/字典名称`用于查询本应用的某个字典的全部字典项
   `http://XXX/dict/系统标识/字典名称`用于查询指定应用的某个字典的全部字典项

### 文件上传下载组件（fileload-support）

平台对外提供了统一的文件上传与下载服务。其中包括以下几个功能：

1. 单文件下载
   `http://XXX/file/download/文件ID`下载指定文件
2. 单文件上传
   `http://XXX/file/upload`上传单个文件
3. 多文件批量上传
   `http://XXX/files/upload`上传多个文件

### 表格分页组件（datagrid-support）

框架在DAO层提供分页功能，并结合前端Web的DataGrid组件，对接其分页数据展示。

### 报文头定制组件（header-support）

用于CXF报文头格式的定制与生成的组件。

### DASC客户化实现组件（dasc-support）

用于DASC服务实现的组件。

### 缓存管理支持组件（cache-support）

在web层面提供缓存数据的管理urlAPI：  
`http://XXX/web/cache/refresh/{serviceBeanName}`刷新特定缓存数据。  
`http://XXX/web/cache/delete/{serviceBeanName}`删除特定缓存数据。

### 传统用户认证授权组件（auth-classic）

基于页面登录的经典认证与授权模式参考实现。

### oauth2认证服务组件（auth-oauth2-authorizeserver）

基于oauth2的服务端认证模式参考实现。

### oauth2资源授权组件（auth-oauth2-resourceserver）

基于oauth2的服务端资源授权参考实现。

### 批作业触发组件（batch-trigger）

为spring-batch的job执行提供多种触发方式。

### 主机健康检查组件（health-checker）

提供基于url的主机存活和可访问的心跳链路。

### 幂等服务支持组件（idemp-support）

提供方法级别的幂等服务支持功能。

### 批作业监控组件（batch-admin）

提供基于页面的spring-batch的执行监控和日志查看。并集成了“重启”功能。

