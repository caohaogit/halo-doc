## 统一配置中心

框架基于zookeeper为应用系统提供便捷的统一配置中心。对应用系统中所需的配置项内容做统一的管理。应用系统在启动时会自动连接zookeeper来获取所有的配置。

### 配置项的继承与隔离

1. 对同一个配置项，框架允许按照“应用系统名称”和“主机名称”两个两个维度做拆分设置。

2. 框架对同一个配置项的读取顺序如下，后面的读取会覆盖前面读取的值。

   本地properties文件读取 -&gt; zookeeper中按“应用系统名称”读取配置 -&gt; zookeeper中按“主机名称”读取配置项。

### 配置项的动态变更

虽然zookeeper支持节点值变更的通知，但目前框架尚未实现配置项的动态修改。

### 数据库密码加密策略

应用系统的数据库密码加密开关由配置项“jdbc.password.decrypt”控制。不过框架提供开关切换当使用统一配置中心时，采用数据库密码自动加密策略，由配置项“zk.configcenter.db.encrypt”控制。

### 配置项批量导入

为了方便应用系统开发者能够批量向zookeeper中导入配置信息，在这推荐两种方式执行导入操作：

1. Windows环境：在dev-tool组建包中提供了ZkScriptExecutor的java方法来读取特定语法的脚本，批量导入配置项。脚本语法样例如下：

   ```bash
   # 创建系统节点
   create /example
   # 创建系统的数据库节点
   create /example/db
   create /example/db/version
   create /example/db/version/global1.1.0-SNAPSHOT
   ```

2. Linux环境：直接通过shell脚本调用zkcli来批量执行导入。脚本样例如下：

   ```bash
   #!/usr/bin/env bash
   . /Library/Java_runtime/zookeeper-3.4.7/bin/zkCli.sh <<!!!
   create /example ''
   create /example/db ''
   close
   !!!
   ```



