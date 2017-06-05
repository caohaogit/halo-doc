## 数据库访问

框架对于数据库的访问封装级别较低，保证了开发者在实现功能代码时，能够充分运用SQL的灵活语法。同时，框架也向开发者提供一套简单的数据库动态路由的功能。在DAO层代码能够实现简单的读写分离和分库分表操作。  
框架采用Druid组件作为数据源托管工具。它提供以下功能：

```
1.DruidDriver     代理Driver，能够提供基于Filter－Chain模式的插件体系。
2.DruidDataSource 高效可管理的数据库连接池。
3.SQLParser
```

对于Druid感兴趣的，可参考：[https://github.com/alibaba/druid/wiki/%E9%A6%96%E9%A1%B5](https://github.com/alibaba/druid/wiki/%E9%A6%96%E9%A1%B5)了解详细的功能使用。

### 数据源配置

框架要求XXX-service-app和XXX-batch-app提供至少一个可连接的关系型数据库数据源信息。数据源信息基于JDBC所需信息配置。代码工程的默认数据源配置在jdbc.properties文件中。

```properties
jdbc.ds.default.name=defaultDataSource
jdbc.ds.default.url=jdbc:mysql://192.168.1.102:3306/test
jdbc.ds.default.user=maeagle
jdbc.ds.default.password=
```

当jdbc.dynamic=true时，可能会需要配置多数据源，这时就需要在工程的applicationContext-service.xml中添加新的数据源配置。举例如下：

```xml
<bean id="readDataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
    <property name="url" value="${jdbc.ds.read.url}"/>
    <property name="username" value="${jdbc.ds.read.user}"/>
    <property name="password" value="${jdbc.ds.read.password}"/>
    <property name="maxActive" value="${jdbc.maxConnectionCount}"/>
    <property name="poolPreparedStatements" value="${jdbc.poolPreparedStatements}"/>
    <property name="maxPoolPreparedStatementPerConnectionSize" value="${jdbc.maxPoolPreparedStatementPerConnectionSize}"/>
    <!--合并多个DruidDataSource的监控数据-->
    <property name="useGlobalDataSourceStat" value="true"/>
    <property name="proxyFilters">
        <list>
            <!--数据库日志记录-->
            <ref bean="druidLogger"/>
            <!--数据库监控-->
            <ref bean="druidStat"/>
            <!--SQL注入防火墙-->
            <ref bean="druidWall"/>
        </list>
    </property>
</bean>
```

如此就新增了一个新的数据源，readDataSource。

### 数据库操作

#### 操作封装工具

通过Spring jdbcTemplate获取对数据库操作的工具封装。开发者在具体的业务代码中直接完成jdbcTemplate对象的注入即可。

```java
@Autowired
JdbcOperations jdbcTemplate;
```

JdbcTemplate的API参考：

[https://docs.spring.io/spring/docs/2.0.x/javadoc-api/org/springframework/jdbc/core/JdbcTemplate.html](https://docs.spring.io/spring/docs/2.0.x/javadoc-api/org/springframework/jdbc/core/JdbcTemplate.html)

#### 操作支持工具

Spring也提供了SimpleJdbcInsert，SimpleJdbcCall两个工具来简化对insert操作和存储过程调用。框架提供DaoUtils类来创建和获取上述工具对象。  
对于查询操作，Spring支持对查询结果集映射，通过Java反射机制填充到JavaBean中。框架提供了一个结果集映射实现：CustomBeanPropertyRowMapper。该映射实现已经充分考虑数据表结构字段的下划线写法与Java的驼峰标记法的转换。

代码参考：

```java
@Override
public CoverOccurBo getCoverOccur(String polCode, String coverageNo) {
    return jdbcTemplate.queryForObject(QUERY_COVER_OCCUR_SQL, new CustomBeanPropertyRowMapper<CoverOccurBo>(CoverOccurBo.class), polCode, coverageNo);
}
```

CustomBeanPropertyRowMapper为开发者提供了可扩展定制结果集映射的接口：PropertyMapping。

### 主键生成

框架推荐使用com.halo.core.common.CommonUtils\#uuid\(\)方法来生成全局唯一的主键。该值是JDK内置的使用当前时间的微秒数（CPU晶振）+网卡的MAC地址作为种子，生成的伪随机数。重复概率几乎为0。  
`com.halo.core.common.CommonUtils#uuid()`生成的ID为32位。

### 数据库路由

在一些情况下，开发者需要访问多个数据源进行数据操作。框架提供两种数据库路由方式，虽然简单，但是有效。

```properties
首先，需要先打开数据库路由开关：jdbc.dynamic=true
其次，维护数据源路由表：jdbc.dynamic.list=
```

在以上两项都维护完成后，就可以按需选择不同的数据源路由策略了。

#### 静态路由配置

1. 需要在DAO类代码中实现读写分离，可以在JdbcTemplate操作上做数据源路由，维护jdbc.properties文件中以下属性（以下属性的每一项对应JdbcTemplate的一类操作，开发者可以通过配置实现执行不同的操作）：
   ```properties
   jdbc.dynamic.interceptor.query=
   jdbc.dynamic.interceptor.update=
   jdbc.dynamic.interceptor.call=
   jdbc.dynamic.interceptor.batch=
   ```
2. 需要在Service类的不同方法上路由多个数据源，可以在Service类方法上做数据库路由。首先，将1中所有对JdbcTemplate的操作路由关掉\(将属性值留为空\)；其次，在需要操作的Service实现类上添加路由信息。
   ```java
   Service实现类上的路由配置：
   @Override
   @DataSource("readDataSource")
   public Date testMethodLevelRouter() {                
       // 实现代码
   }
   ```

#### 动态路由配置

在某些场景下，Service方法中所使用的数据源是由方法传入的参数所确定的。这时就需要用到动态路由配置。框架提供的动态路由配置依然是用Java注解完成的。  
先确定在Service方法中用来路由数据源的入参，之后就可以通过注解完成对Service方法内调用DAO代码中引用的jdbcTemplate示例的路由：

* a）Service实现类上的路由配置：

  ```java
  @DataSource(type = ElementType.PARAMETER, parameterIndex = 0)
  public Date testParameterLevelRouter(String dataSource) {
      // 实现代码
  }
  ```

* b）Service实现类上的路由配置：

  ```java
  @DataSource(type = ElementType.PARAMETER,parameterIndex = 0,propertyName = "id")
  public Date testParameterLevelRouter(RouterInforouteInfo) {
      // 实现代码
  }
  ```

#### 注意事项

1. 在数据库路由功能与数据库事务混合使用时，被包含在同一事务内的数据库路由配置失效。目前框架不支持跨数据源的数据库事务。
2. 不支持父级方法的数据源配置还原。在出现Java方法级联调用时（A方法内调用B方法），如果父级方法（A方法）指定的数据源，子级方法（B方法）也指定了数据源，那么在子级方法调用结束的后续代码中，无法再还原回父级方法的数据源配置，数据源配置会被还原为默认数据源。

### 数据库监控

Druid内置了实时数据库性能监控工具。在应用运行时，可访问：[http://XXXX:XX/XXX/druid/index.html](http://xxxx:XX/XXX/druid/index.html)即可。需要注意的是，对于数据库的访问，框架约束只允许webapp和serviceapp工程访问，因此也只是在这两类工程运行时能够访问数据库监控页面。

### 数据库事务

框架基于Spring提供的事务管理组件，在Service服务层提供了事务托管机制。需要注意的是，目前提供的事务管理，是在同一个数据源操作下的事务支持，对于跨数据源事务，目前框架不支持。**一定要避免在声明了事务的Service服务中调用跨数据源的DAO操作，这会造成未知的错误。**  
以事务性来划分目前框架中的数据库操作有两种：

1. 不需要事务支持的普通SQL操作。例如：单表的CRUD操作。这类操作有些是原子性的，有一些可能属于非原子性数据操作（带有嵌套子查询）。
2. 需要事务支持的操作。这类操作一般是组合多个第一种操作后，以数据库事务来提交数据更改。

   下面对以上两种操作在框架中的编写方法分别做介绍：

   1. 非事务型数据操作：与普通的Service实现类的编写方法类似。**要避免使用以tran开头的方法名称**。

      ```java
      public void addUser() {
          UserInfo user = new UserInfo();
          String id = CommonUtils.uuid();
          user.setUserId(id);
          user.setUserName(CommonUtils.uuid());
          user.setPassword("test1");
          dbTestDao.saveOneRecord();
      }
      ```

   2. 方法名称型事务型数据操作：该类型操作可以组合多个“DAO方法操作”，让其具备事务性操作特征，满足同一事务中的各种特性。  
      业务代码编写方式与Service实现类中普通方法类似。有以下几点注意：

      1. **方法名称要以tran开头**。
      2. **如果项目开启了数据库路由功能，要在方法签名上指定数据源。事务操作的开发者需要确保数据库事务是在正确的数据源上运行的**。
      3. **两个事务型数据操作可以嵌套**。

      其余的事情会由框架通过Spring事务管理和SpringAOP协助完成。

      ```java
      /**
       * 测试数据库事务提交.
       *
       * @author maeagle
       * @date 2016-2-2 13 :43:58
       */
      @DataSource(DynamicDataSource.DEFAULT_DATASOURCE)
      @Override
      public void txTestCommit() {
          dbTestDao.saveOneRecord();
          dbTestDao.saveOneRecord();
      }

      /**
       * 测试数据库事务回滚.
       *
       * @author maeagle
       * @date 2016-2-2 13 :43:58
       */
      @DataSource(DynamicDataSource.DEFAULT_DATASOURCE)
      @Override
      public void txTestRollback() {
          dbTestDao.saveOneRecord();
          dbTestDao.saveOneRecord();
          throw new RuntimeException("testRollback");
      }
      ```

3. 注解型事务型数据操作：该类型操作可以组合多个“DAO方法操作”，让其具备事务性操作特征，满足同一事务中的各种特性。  
   业务代码编写方式与Service实现类中普通方法类似。有以下几点注意：

   1. **方法上要定义注解com.halo.core.dao.annotation.Transaction**。
   2. **如果项目开启了数据库路由功能，要在方法签名上指定数据源。事务操作的开发者需要确保数据库事务是在正确的数据源上运行的**。
   3. **两个事务型数据操作可以嵌套**。

   其余的事情会由框架通过Spring事务管理和SpringAOP协助完成。

   ```java
   /**
    * 测试数据库事务提交.
    *
    * @author maeagle
    * @date 2016-2-2 13 :43:58
    */
   @DataSource(DynamicDataSource.DEFAULT_DATASOURCE)
   @Transaction
   @Override
   public void testCommit() {
       dbTestDao.saveOneRecord();
       dbTestDao.saveOneRecord();
   }

   /**
    * 测试数据库事务回滚.
    *
    * @author maeagle
    * @date 2016-2-2 13 :43:58
    */
   @DataSource(DynamicDataSource.DEFAULT_DATASOURCE)
   @Transaction
   @Override
   public void testRollback() {
       dbTestDao.saveOneRecord();
       dbTestDao.saveOneRecord();
       throw new RuntimeException("testRollback");
   }
   ```



