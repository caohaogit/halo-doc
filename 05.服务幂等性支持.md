## 服务幂等性支持

Halo框架为所有在Spring容器中托管声明周期的Bean实例提供了幂等性支持。幂等性是基于消息和事件驱动的流程串联的基础。也是在分布式系统下，基于CAP理论保证原子交易数据一致性的必要条件。  
有关服务幂等性概念，可参考如下资料：[http://blog.csdn.net/fbysss/article/details/8024748](http://blog.csdn.net/fbysss/article/details/8024748)[http://coolshell.cn/articles/4787.html](http://coolshell.cn/articles/4787.html)

### 幂等方法的声明

在已经注册为SpringBean的服务方法声明开头，添加[@Idempotent](http://localhost:3000/Idempotent)注解。

```java
@Service
public class IdempTestServiceImpl implements IdempTestService {
    @Idempotent(expression = "#id", withTranCode = false)
    @Override
    public void execute(String id) {
        System.out.println(id);
    }
}
```

如样例代码所示，[@Idempotent](http://localhost:3000/Idempotent)注解包含以下属性：

1. expression：采用spEL表达式语法，基于方法参数列表描述该方法执行的幂等要素构成。
   关于spEL表达式，请参阅资料：
   [http://docs.spring.io/spring/docs/current/spring-framework-reference/html/expressions.html](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/expressions.html)
2. withTranCode：声明该方法执行的幂等性，是否加入线程级的交易码作为幂等要素的一部分。一旦加入则对于不同线程的相同参数列表的请求，会被认为是不同幂等要素的方法调用。该属性默认为false。
3. expire：幂等要素持久化的过期时限。默认为-1，不过期。由于幂等要素的持久化支持数据库和缓存。对于缓存的持久化，可以通过指定过期时间来提高幂等容器的利用率。
4. faildInSilence：当幂等校验失败时，是否以静默方式跳过方法的执行。默认值：false（抛出{[@link](http://localhost:3000/link) com.halo.core.exception.IdempotentException}异常）。如果为true，则只会在后台打印一条日志通知。同样不会执行方法。
5. effectOnException：当方法主体抛出异常时，是否将本次作为有效调用，持久化幂等要素信息。默认值：true（无论方法主体是否报错，都作为有效调用）。

### 幂等方法的校验

已经声明幂等性的服务方法，在被执行时会先经过一个SpringAOP的切面，在切面中，系统会根据已经配置的幂等信息，参数列表和执行记录，判断是否允许再次执行方法。如果校验失败，会根据[@Idempotent](http://localhost:3000/Idempotent)注解上声明的配置抛出异常或静默跳过：`throw new IdempotentException`\("识别到重复调用，未通过幂等性校验！"\);  
对于非静默的模式下，服务方法调用的父级建议捕获该异常信息，并做响应处理。

### 幂等要素的记录存储

为了实现方法的幂等校验，必然涉及到对方法执行的幂等要素记录。框架可以采用数据库或缓存来存储这些幂等要素记录。  
对于数据库存储，该表的表结构PDM放在supports-parent/idemp-support/docs文件夹下。如果要使用该组件和特性，需要将PDM中的表结构导入至oracle中。  
对于缓存的存储，不需要额外的配置。

### 幂等性与数据库事务

服务方法的幂等性与数据库事务的融合是一个较为复杂的议题。为了在前期更为清晰的分割两者对于开发者的边界，目前框架将两者的使用强制割裂开。对于幂等要素的数据库存储上会单独启动事务来处理。

