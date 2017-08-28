## 批作业管理

### 批作业运行

框架选用spring-batch作为支持应用系统批作业规划的组件。其优势在于对Spring-Framework的紧密结合，基于数据库表的批作业运行管理，支持批作业任务分片策略，集群策略，并行策略等。这些特性对后续基于其上的的批作业管理与监控功能的扩展性更强。  
  
spring-batch官方文档：  
[http://docs.spring.io/spring-batch/trunk/reference/html/index.html](http://docs.spring.io/spring-batch/trunk/reference/html/index.html)  
对批作业运行态管理，框架没有对spring-batch做任何侵入性修改和外围的封装，仅从配置便捷性上将spring-batch的几个服务类以Bean的形式暴露出来：

```xml
<!--用于在Spring中对已经注册的批作业任务的收集器-->
<bean id="batchJobRegistry" class="org.springframework.batch.core.configuration.support.MapJobRegistry"/>
<!--声明用于浏览批作业执行情况的资源浏览器-->
<bean id="batchJobExplorer" class="org.springframework.batch.core.explore.support.JobExplorerFactoryBean">
    <property name="dataSource" ref="defaultDataSource"/>
</bean>
<!--声明批作业的异步执行服务类-->
<bean id="batchJobAsynLauncher" class="org.springframework.batch.core.launch.support.SimpleJobLauncher">
    <property name="jobRepository" ref="jobRepository"/>
    <property name="taskExecutor">
        <bean class="com.halo.core.batch.support.AsyncJobExecutor"/>
    </property>
</bean>
<!--声明批作业的同步执行服务类-->
<bean id="batchJobSynLauncher" class="org.springframework.batch.core.launch.support.SimpleJobLauncher">
    <property name="jobRepository" ref="jobRepository"/>
    <property name="taskExecutor">
        <bean class="org.springframework.core.task.SyncTaskExecutor"/>
    </property>
</bean>
<!--带有逻辑判断的批作业执行器-->
<bean id="batchJobLauncher" class="com.halo.core.batch.service.impl.batch.BatchJobLauncher"/>
<!--可用于批作业创建重复实例的计数器-->
<bean id="batchJobIncrementer" class="org.springframework.batch.core.launch.support.RunIdIncrementer"/>
<!--默认的批作业管理工具-->
<bean id="batchJobManager" class="org.springframework.batch.core.launch.support.SimpleJobOperator">
    <property name="jobExplorer" ref="batchJobExplorer"/>
    <property name="jobLauncher" ref="batchJobAsynLauncher"/>
    <property name="jobRepository" ref="jobRepository"/>
    <property name="jobRegistry" ref="batchJobRegistry"/>
</bean>
<!--可同步运行的批作业管理工具-->
<bean id="synBatchJobManager" class="org.springframework.batch.core.launch.support.SimpleJobOperator">
    <property name="jobExplorer" ref="batchJobExplorer"/>
    <property name="jobLauncher" ref="batchJobSynLauncher"/>
    <property name="jobRepository" ref="jobRepository"/>
    <property name="jobRegistry" ref="batchJobRegistry"/>
</bean>
<!--可异步运行的批作业管理工具-->
<bean id="asynBatchJobManager" class="org.springframework.batch.core.launch.support.SimpleJobOperator">
    <property name="jobExplorer" ref="batchJobExplorer"/>
    <property name="jobLauncher" ref="batchJobAsynLauncher"/>
    <property name="jobRepository" ref="jobRepository"/>
    <property name="jobRegistry" ref="batchJobRegistry"/>
</bean>
```

### 批作业重启

框架对spring-batch的重启策略，结合应用系统实际场景总结基础上进行了再定义。并将逻辑封装到`com.halo.core.batch.service.impl.batch. BatchJobLauncher`服务中。提供异步触发，Quartz触发两种批作业触发功能。在这两个功能中，已经隐含了批作业重启策略。

先判断当前是否已经有该批作业的运行实例。如果有，则执行下列判断：

* a\) 如果批作业实例状态为`{@link FAILED}`,`{@link ABANDONED}`，并且`restartable=true`，则尝试重启批作业实例。并计数。
* b\) 如果批作业实例状态为`{@link COMPLETED}`,`{@link STOPPED}`，并且声明了incrementer，则用相同的业务参数再创建一个批作业实例并运行。
* c\) 如果批作业实例状态为`{@link COMPLETED}`,`{@link STOPPED}`，但是未声明incrementer，则抛出异常。
* d\) 如果批作业实例状态为`{@link UNKNOWN}`, 则抛出异常。
* e\) 如果批作业实例为其他状态，则打印日志，直接返回。

### 批作业触发

spring-batch组件仅提供基于编译器代码方式的批作业任务触发api，但是在应用系统使用场景中，需要通过不同的渠道来触发一次批作业任务的执行。框架对spring-batch的job任务触发附加了三种方式：

1. 基于Quartz的定时执行策略。在halo-batch组件中实现。以定时任务的形式启动一次批作业的执行。（详见“定时任务”）。批作业的定时触发的注册策略采用Spring配置注册的方式：示例配置如下：

   ```xml
   <batch-trigger:schedule id="simpleBatchJob.trigger" batch-job="simpleBatchJob" cron="* * * * * ? *"/>
   ```

2. 基于DASC的消息触发执行策略。在batch-trigger组件中实现。框架向消息队列上注册一个dasc服务，任何发送到该队列的消息都会异步触发一次批作业任务的执行。

   ```java
   DASC服务接口如下：
       /**
       * 基于消息队列的批作业触发器接口规范.
       *
       * @author maeagle
       * @date 2016-11-14 14 :39:59
       */
       public interface MessageTrigger {
           /**
           * 调用批作业方法
           *
           * @param command 触发批作业一次执行的命令信息
           */
           void execute(BatchJobCommand command);
       }
   ```

3. 基于HTTP请求的出发执行策略。在batch-trigger组件中实现。框架对外暴露一个REST服务，任何调用该服务的请求会触发一次批作业任务的执行。触发机制提供“同步/异步”两种。接口描述如下：

   ```java
   public interface HttpTrigger {
       /**
        * 异步非阻塞调用批作业方法
        *
        * @param command 触发批作业一次执行的命令信息
        * @author maeagle
        * @date 2016-11-14 14 :39:59
        */
       @POST
       @Path("/asyn")
       boolean asynExecute(BatchJobCommand command);
    
       /**
        * 同步阻塞调用批作业方法
        *
        * @param command 触发批作业一次执行的命令信息
        * @author maeagle
        * @date 2016-11-14 14 :39:59
        */
       @POST
       @Path("/syn")
       boolean syncExecute(BatchJobCommand command);
   }
   ```

### 批作业监控

框架通过batch-admin扩展插件来实现批作业监控功能的。

1. 监控数据来源
   框架对spring-batch的执行态监控是基于数据库表来展开的，表数据由两部分构成：
   * a\) 自建表：数据是通过批作业开发者在批作业启停和报错时插入。
   * b\) spring-batch内建表：spring-batch的执行状态数据会持久化到这些表中。内建表结构如下：
     [![](http://i.imgur.com/ScDbVSO.png)](http://i.imgur.com/ScDbVSO.png)
2. 监控维度
   框架对每个批作业任务的监控维度比spring-batch的job粒度要更大一级。监控粒度从大到小分别是：任务（task）-&gt; job实例（job instance）-&gt; job执行（job execution）task与job instance，job execution的关系是通过自建表维护在数据库表中的。
3. 管理功能
   监控组件支持在页面上对某一个task下的job execution进行重启。重启的策略参考“批作业重启”部分。





