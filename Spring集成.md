## Spring集成

### 功能开发

框架内部大量使用了Spring依赖注入与AOP机制。同时也建议开发人员在开发功能代码时，采用Spring的依赖注入对功能实现做单例托管。  
**对于完成功能实现代码类在Spring中托管，Spring提供了两种方式：注解声明，xml配置。**这两种方式框架都提供支持。

1.注解声明

   * a. 声明Spring的Service组件

     ```java
     @Service("soapService")
     public class SoapServiceImpl implements SoapService{
         // 实现代码
     }
     ```

   * b. 声明Spring的DAO组件

     ```java
     @Repository("userInfoDao")
     public class UserInfoDaoImpl implements UserInfoDao{
         // 实现代码
     }
     ```

   * c. 声明Spring的Controller组件

     ```java
     @Controller
     @RequestMapping("users/soap")
     public class SoapServiceTestController {
         // 实现代码
     }
     ```

为了规范代码文件的组织结构，框架有限制的开放特定包路径的注解，以此来规范开发者的功能实现：

| 包路径 | 开放的注解 |
| :---: | :---: |
| ````com.\*.\*.web```` | 全部注解类型 |
| ````com.\*.\*.service```` | @Autowired,@Resource,@Service, halo框架功能注解支持 |
| ````com.\*.\*.business```` | @Autowired，@Resource，@Service |
| ````com.\*.\*.dao```` | @Autowired，@Resource，@Repository |


2.XML配置

   * 配置Spring的Service组件

     ```xml
     <bean id="testService" class="com.newcore.example.service.impl.TestServiceImpl"></bean>
     ```

   * 配置Spring的DAO组件

     ```xml
     <bean id="testDao" class="com.newcore.example.dao.TestDao"></bean>
     ```

     工程启动时，框架会要求Spring读取classpath下的全部\*.xml文件。即便如此，也建议开发者将全部Spring的组件注册放到统一的xml文件中。




