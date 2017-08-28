## 批作业监控

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





