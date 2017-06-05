## 文件管理

框架提供了一套对文本，文件资源读写访问的公共操作接口。并遵这套接口规范实现了三种针对不同文件资源访问协议的服务。

### 服务配置

公共操作接口所能提供的方法，参见：com.halo.core.filestore.api.FileStoreService的接口文档说明。  
特定的工程中只能选择一种资源访问协议，需要在filestore.properties文件中对fs.protocol进行配置。同时将fs.enabled设置为true。（详细配置项参见“配置清单”）。

### 服务操作

文件管理服务对外提供统一的操作方法结合，开发者代码可以通过注入特定的Spring服务来读写文件资源。

```java
@Resource
FileStoreService fileStoreService;
```

该服务实例隐藏了具体的资源访问细节，面向开发者提供的是对于文本资源，文件资源的读写操作。

#### 写操作的原子性

如果开发者要求在对文件写入时的原子性，则可以在filestore.properties配置fs.backupOnUpdate为true。这样框架会对文件写入操作进行由于的备份。保证文件写入的原子性。

#### 读操作的优化

一些特殊的资源读取API，例如：

```java
File getFile(String resourceId) throws ResourceAccessException
```

在某些特殊容器实现时，其网络特性要求服务在读取时采取一次性读取，缓存至本地后再输出。开发者需要了解这样的API在内部会一次性将文件数据读取至本地，再将文件对象返回给开发者代码的。  
对于本地缓存的临时文件夹，可以在filestore.properties配置fs.tmpdir设置。

### 元数据

所有通过注入ResourceService实例来托管文件资源的操作，所生成的资源在容器（磁盘，MONGO）中，都会产生对应的元数据信息。这部分信息对于开发者代码来说是隐藏的。框架具有隐藏这些实现细节的内部代码，  
文件资源的元数据管理不需要开发者掌握的，不过如果开发者想了解具体的元数据结构和实现细节，可以参考`com.halo.core.filestore.model.StoreMetaInfo`代码结构。

### 文件资源容器类型

框架基于FileStoreService服务接口实现了三种不同的文件资源访问协议的支持。

#### 磁盘文件协议（file://）

参见：`com.halo.core.filestore.impl.DiskStoreServiceImpl`提供了对本地磁盘，NAS的访问协议。

#### FTP协议（ftp://）

参见：`com.halo.core.filestore.impl.FtpStoreServiceImpl`提供了对FTP资源的访问协议。

#### MONGO协议（mongo://）

参见：`com.halo.core.filestore.impl.MongoStoreServiceImpl`提供了对mongodb容器的文件资源访问协议。

