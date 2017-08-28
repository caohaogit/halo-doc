## 缓存管理

与“文件管理”类似，框架提供一套针对缓存管理的公共服务接口，并且基于这套接口实现了Redis的缓存实现。

```
公共接口：com.halo.core.cache.api.CacheService
```

### 区域划分

框架默认认为多个子系统会共用一个缓存容器，因此，会对REDIS的Key规则做区域划分，不同子系统依然能够访问不同区域的Key和Value，不过主要目的还是为了保证在Redis中的Key唯一性，同时也便于管理。

```
1. 普通资源生成规则：子系统名称（cache.properties中的cache.name）:数据类型（详见com.halo.core.cache.model. CacheDataType）:资源名称（如果还需要继续分子集，则用点号（.）分割）。
2. Session会话生成规则：SYS:SESSION:SessionId的值。
```

### 服务配置

参考“配置清单”即可。  
唯一要注意的是：同一个业务系统的不同工程（如：XXX-service-app和XXX-web-app），其cache.properties中的cache.name推荐保持一致。

### 缓存路由

框架提供对多个Redis缓存容器连接的支持。与数据库路由的操作类似： 首先，打开数据库路由开关：`cache.dynamic=true`其次，维护多个Redis缓存容器的连接。例如：

```properties
cache.redis.default.mode=SINGLETON
cache.redis.default.addresses=localhost:6379
cache.redis.default.timeout=10000
cache.redis.default.maxRedirections=6

cache.redis.session.mode=SINGLETON
cache.redis.session.addresses=localhost:6379
cache.redis.session.timeout=10000
cache.redis.session.maxRedirections=6
```

之后，就可以按需选择不同的数据源路由策略了。具体使用方法与数据源路由类似，通过在serviceBean的实现类上加上`@CacheSource`注解来实现缓存路由指定。样例代码如下：

```java
@Override
@CacheSource("sessionCacheSource")
public void addSession(Session session) {
    cacheService.createSession(session);
}
```

注意，在两个serviceBean出现关联调用时，如果两个serviceBean分别有各自的[@CacheSource](http://localhost:3000/CacheSource)指定，则框架无法再指定第二个[@CacheSource](http://localhost:3000/CacheSource)之后恢复路由信息。因此，避免出现带有[@CacheSource](http://localhost:3000/CacheSource)的关联嵌套调用。

### 服务操作

缓存管理服务对外提供统一的操作方法结合，开发者代码可以通过注入特定的Spring服务来读写缓存资源。

```java
@Autowired
CacheService cacheService;
```

该服务实例隐藏了具体的缓存访问细节，面向开发者提供的是对于缓存资源的读写操作。

### 原子性与事务性

```
- 单个缓存单元的原子性由缓存容器来保证（如REDIS）。
- 多个缓存操作的事务性：由于框架可能面临的是缓存的集群环境，因此框架对于多缓存操作的事务性不做保证。
```

### 元数据

每一个通过框架提供的缓存服务类实例创建的缓存数据单元，框架默认都会创建一个元数据对象（Session数据类型除外，为了更高效的读写和过期，Session对象在缓存中没有元数据）。元数据对象类型为：`com.halo.core.cache.model. CacheMetaInfo`  
元数据对象会对于缓存数据的读写次数，超期时间，资源名称等一系列描述缓存数据单元的信息进行存储。元数据本身的生命周期与缓存单元的生命周期相同。  
元数据另外一个作用是在创建缓存数据，或是在缓存中检索数据时，起到数据描述的作用。元数据对象的创建过程是通过一个工厂类来实现的：`com.halo.core.cache.api. CacheMetaInfoFactory`开发者同样可以通过Spring注入的方式引用这个类实例。

```java
@Autowired
CacheMetaInfoFactory cacheMetaInfoFactory;
```

类中包含创建所有缓存数据类型的元数据对象的方法。

### 缓存数据类型

#### 数据单元计数器（内部使用）

具体操作参见：`com.halo.core.cache.support.redis.holder. CounterHolder`该数据类型是内部使用，用来对缓存数据单元进行计数。

#### 信号量：MUTEX

具体操作参见：`com.halo.core.cache.support.redis.holder.MutexHolder`可缓存数字或字符串。当为数字类型时，提供原子累加功能。

#### 会话：SESSION

具体操作参见：`com.halo.core.cache.support.redis.holder.SessionHolder`

#### 哈希表：MAP

具体操作参见：`com.halo.core.cache.support.redis.holder.MapHolder`Key的类型只允许为String，value类型为一切实现Serializable接口的Java对象，在存储到Redis时，框架采用fastjson对Java对象进行JSON序列化。

#### 有序无重复列表：RANGE

具体操作参见：`com.halo.core.cache.support.redis.holder.RankHolder`score的类型为double，value类型为一切实现Serializable接口的Java对象，在存储到Redis时，框架采用fastjson对Java对象进行JSON序列化。

#### 索引表格：TABLE

具体操作参见：`com.halo.core.cache.support.redis.holder.TableHolder`索引表格能够包含任何二维表结构的集合类型（例如从RDMS中提取的关系型数据），并提供一系列API操作方法来提升对缓存的索引表格数据的检索。

### 缓存数据管理

框架支持多类型应用缓存的集中式管理，对于缓存容器的持续运行有一定要求。因此从技术实现角度对不同应用类型的缓存数据，或同一应用类型，多个应用节点的缓存数据都需要进行生命周期管理。

#### 初始化，手动刷新，手动删除

框架提供对于同一应用类型的应用节点启动时的缓存数据管理服务接口，以方便开发者能够忽略多节点并发启动时可能出现的数据初始化冲突，专注于数据提取与缓存本身。开发者实现`com.halo.core.cache.api.CacheManageService`接口，并通过注解或Xml的形式注册为Spring的Bean。进而实现同构应用中缓存数据的管理需求。举例如下：

```java
@Service
public class DictCacheServiceImpl implements CacheManageService {

    /**
     * 获取或构建待缓存元素的元数据。
     * 如果缓存数据需要通过多个元数据描述，则返回具有唯一性约束的元数据信息即可。
     *
     * @return 元数据信息
     * @author maeagle
     * @date 2016/3/30 8:40
     */
     @Override
     public CacheMetaInfo getCacheMetaInfo() {
        。。。
     }

   /**
     * 新增缓存元素操作
     *
     * @author maeagle
     * @date 2016/3/30 8:40
     */
     @Override
     public void init() {
        。。。
     }

   /**
     * 清除缓存元素操作
     *
     * @author maeagle
     * @date 2016/3/30 8:40
     */
     @Override
     public void clear() {
        。。。
     }

   /**
     * 刷新缓存数据操作。用于通过web端管理缓存数据
     *
     * @author maeagle
     * @date 2016/3/30 8:40
     */
     @Override
     public void refresh() {
        。。。
     }

   /**
     * 声明系统启动时是否自动初始化缓存数据
     *
     * @return 标志位
     * @author maeagle
     * @date 2016/3/30 8:40
     */
     @Override
     public boolean autoInit() {
        return true;
     }

     /**
     * 声明是否允许通过对缓存数据手动刷新和删除
     *   
     * @return 标志位
     * @author maeagle
     * @date 2016/3/30 8:40
     */
     @Override
     public boolean manageEnabled() {
         return true;
     }
}
```

#### 缓存单元的过期与刷新策略

在构建元数据的过程中，框架要求开发者指定所要创建的缓存数据单元的过期策略和刷新策略，并依据不同的策略设置特定的阈值。参考：`com.halo.core.cache.model.ExpireConfig`

| 过期策略 | 阈值 |
| :--- | :--- |
| EXPIRE\_ON\_OWNER\_EXIT | 无 |
| NO\_EXPIRE | 无 |
| EXPIRE\_ON\_GET\_LIMIT | readCountLimit |
| EXPIRE\_ON\_SET\_LIMIT | writeCountLimit |
| EXPIRE\_ON\_TIME\_LIMIT | timeLimit。可设置刷新策略 |

### 面向方法执行的缓存

Spring提供了一个方法执行缓存的组件支持。能够对托管在Spring容器中的服务方法的每次调用结果做缓存，并提供一系列注解来控制缓存数据的声明周期。框架集成了该组件，并基于Redis实现了缓存的容器。  
关于Spring-Cache的详细参考资料：[http://docs.spring.io/spring/docs/current/spring-framework-reference/html/cache.html](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/cache.html)  
框架对Spring-Cache使用的缓存数据类型为MUTEX类型。

