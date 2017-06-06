## 其他功能

除了骨干组件和核心功能之外，框架也为开发者收集并提供了一些方便的工具类组件和功能。这些组件和功能未开发者提供一定的便利。

### 自启动服务

框架提供`com.halo.core.init. InitializingTask`接口，开发者只需在`com.*.*.service.impl`中实现该接口，并声明为Spring事务，则在应用启动时自动执行实现的execute方法。例如：

```java
@Service("testTask")
public class TestInitTask extends InitializingTask {
     /**
     * 作业执行.
     *
     * @param context {@link ApplicationContext}
     * @author maeagle
     * @date 2016-3-2 8 :49:45
     */
     @Override
     public void execute(ApplicationContext context) {
          // 代码实现
     }
}
```

### 定时任务

框架允许在XXX-batch-app应用上实现任务的定时执行，该功能是基于Quartz组件的Spring扩展。  
开发者如果想要创建新的定时任务，需要对编写的类有满足以下条件：

1. 任务类要实现org.quartz.Job接口
2. 将任务类放在com.\*_.\*_.service.impl.\* 包下
3. 用[@Service](http://localhost:3000/Service)将任务类声明为SpringService
4. 在类属性上加入[@BatchJob](http://localhost:3000/BatchJob)注解，并提供corn表达式 具体代码如下：

   ```java
   @BatchJob(cron = "* * * * * ? *")
   @Service("dascCallJob")
   public class DascCallJob implements Job {
       @Override
       public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
           // 代码
       }
   }
   ```

### 属性资源读取

为了管理多个属性文件，框架提供统一的属性信息读取工具类。同时允许在载入配置项时，对不同容器中配置项的合并。面向客户端开发者提供扩展接口。  
参见：`com.halo.core.common.PropertiesUtils`，`com.halo.core.common.api. PropertiesLoader`。

### 通用工具集

框架集成了很多通用工具类组件。以Apache-commons为主的一些通用工具集合。

| 组件名称 | 参考资料 | 使用场景 |
| :--- | :--- | :--- |
| common-lang | [http://commons.apache.org/proper/commons-lang/](http://commons.apache.org/proper/commons-lang/) | 对于一些需要大量if来处理的逻辑，例如字符串默认值，或集合的判空等。适合对代码有洁癖的人使用。 |
| commons-collections | [http://commons.apache.org/proper/commons-collections/](http://commons.apache.org/proper/commons-collections/) | 扩展了JDK默认的集合类型。感兴趣的可以看看。 |
| commons-io | [http://commons.apache.org/proper/commons-io/](http://commons.apache.org/proper/commons-io/) | 对Java中流对象进行操作的工具类组件。很方便。 |
| commons-pool | [http://commons.apache.org/proper/commons-pool/](http://commons.apache.org/proper/commons-pool/) | 简单而强大的对象池。好多组件都内置了它。例如Jedis。 |
| commons-net | [http://commons.apache.org/proper/commons-net/](http://commons.apache.org/proper/commons-net/) | 对FTP等常用资源传输协议的封装 |
| org.springframework.util.Assert |  | Spring的一些常用的断言操作。适用于方法入参判断为空等场景。适合对代码有洁癖的人使用。 |



