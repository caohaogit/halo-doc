## 认证与授权

框架基于Apache Shiro组件实现认证与授权机制。能够做到基于java方法级别，url级别的用户认证与鉴权机制。

### 作用域

虽然Apache Shiro支持Java方法级别细粒度的权限控制体系，不过基于目前框架的敏捷性与可能出现的业务复杂性，框架将认证与授权机制局限于Web交互应用层（WebApp），暂时不会将认证与授权机制渗透到服务应用层（ServiceApp）。也就是说，在ServiceApp中暂时不用考虑用户身份和用户权限的问题，不过在WebApp下是需要在SpringMVC的Controller下充分配置的。后续如果实际业务中有明确的要求，可以考虑打开限制。

### 匿名访问

框架允许开发人员在保证系统安全的前提下，为前端页面对某些controller服务提供匿名访问。这些controller在注册url路径时，必须按照约定设置路径URL为“/exp”前缀。

> 需要注意的是：框架对匿名访问请求不会产生会话状态实例。

### 会话注销

框架对客户端浏览器提供了统一的会话注销和登出服务入口：“/logout”。同时，框架允许应用开发者定制的认证授权插件在接入单点登录服务时，在面向SSO服务提供统一的登出服务的开发过程中，改写“/logout”的注销实现（详见“功能集成”）。

### 功能集成

框架基于Shiro组件，为应用开发者提供了一个可支持多协议的认证与授权扩展组件。其围绕着一个HttpRequest请求进入后在应用服务器线程的全生命周期内，以处理总线的理念设计了一套允许开发者扩展的“处理-响应”组件。为了允许开发者控制Http请求的生命周期，框架提供了一套接口供开发者实现。  
com.halo.core.auth.service.api. AuthService:

```
1.标识是否启用该配置规范 
2.判断当前请求是否符合使用该配置要求
3.在判断相对路径时，与其他配置规范的比较顺序号。(小数值优先验证, 只验证第一个符合条件的)。
4.标识规范是否支持testMode模式。
5.构造令牌的管理服务。
6.构造会话的管理服务。
7.构造认证与授权的管理服务。
```

com.halo.core.auth.service.api. SessionConfig:

```
1.标识是否启用Session会话
2.允许Session在创建时，根据已验证通过的令牌实例初始化一些数据。
3.添加了对Session的CRUD操作后的回调方法服务。
```

com.halo.core.auth.service.api. TokenManagerService:

```
1. 基于一次Http请求, 创建响应的token实例
```

com.halo.core.auth.service.api. AuthorizingConfig:

```
1.在认证成功后, 通过request，response控制请求的后续行为。
2.在认证失败后, 通过request，response控制请求的后续行为。
3.构造token的认证与授权逻辑的实例封装。
```

com.halo.core.auth.service.api. AuthRealm:

```
1.验证传入的token，判断是否支持当前token。
2.基于所有的认证凭证集合，生成对应的权限集合。对特定认证信息识别到资源授权内容，进而实现对用户的鉴权。
3.基于令牌实例，生成一个shiro认证凭证。后续由shiro来做凭证比对。具体的用户认证逻辑需要在方法中实现。
```

com.halo.core.auth.service.api. LogoutHandler:

```
1.系统登出操作的处理逻辑封装。
```

### 插件参考实现

为了能够让应用开发者更快的专注于业务功能的实现，框架提供了四种认证与授权组件的参考实现。供应用开发者集成并实现自身逻辑。

```
1.auth-classic：基于页面登录的经典认证与授权模式参考实现。
2.auth-oauth2-authorizeserver：基于oauth2的服务端认证模式参考实现。
3.auth-oauth2-resourceserver：基于oauth2的服务端资源授权参考实现。
```

### 内部配置说明

对于Shiro拦截器的配置，框架已经在web.xml及shiro/applicationContext-shiro.xml中进行了统一的配置。不需要开发者做任何更改。

1. web.xml&lt;filter-mapping&gt;
   ```xml
   <filter-name>shiroFilter</filter-name>
   <url-pattern>/web/*</url-pattern>
   </filter-mapping>
   ```
2. shiro/applicationContext-shiro.xml
   ```xml
   <property name="filterChainDefinitions">
       <value>
           /exp/** = anon
           /logout = urlLogout
           /** = urlAuth
       </value>
   </property>
   ```
3. applicationContext-shiro-ann.xml
   ```xml
   <!-- 打开@RequiresPermissions相关的注解, 如果需要在SpringMVC中使用权限控制，本文件需要引入至springmvc.xml-->
   <bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator" depends-on="lifecycleBeanPostProcessor"/>
   <bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
       <property name="securityManager" ref="securityManager"/>
   </bean>
   ```
4. @Configuration
   ```java
   @ComponentScan(basePackages = {"com.*.*.web"})
   @ImportResource("classpath*:shiro/applicationContext-shiro-ann.xml")
   public class SpringMvcConfig extends WebMvcConfigurationSupport{

   }
   ```

### 会话管理

框架基于ApacheShiro的SessionDAO概念，实现了基于集中式缓存的Session可持久化的管理。提供了对Session对象的“CRUD”。同时为了扩展性需要，对会话的生命周期及相关操作回调功能进行统一规划配置。具体如下：  
会话管理配置项：

1. 会话生命周期：webconfig.properties中的web.session.timeout：会话超时时间（默认为1800000毫秒）
2. 本地会话缓存：为了加速本地服务，框架针对会话在应用程序内部使用了EHCache本地缓存机制。本地会话缓存配置在shiro/localcache.xml中。（**不可更改**）
   ```xml
   <cache name="shiro-activeSessionCache" maxElementsInMemory="10000" overflowToDisk="false" eternal="true" timeToLiveSeconds="300" timeToIdleSeconds="0" diskPersistent="false" diskExpiryThreadIntervalSeconds="600"/>
   ```
3. 会话CRUD回调   ：允许应用开发者对对session的CRUD操作后的回调方法服务。通过实现com.halo.core.auth.service.api.SessionDaoListener接口（或继承com.halo.core.auth.service.api.AbstractSessionDaoListener抽象类），之后向com.halo.core.auth.service.api.SessionConfig中的getSessionDaoListener方法中注册即可。

### 登录入口

框架的认证组件以“总线/消费”模型来处理Http请求，因此，不同的认证协议实现对登录入口的要求也各有不同。  
传统页面登录（auth-classic）：应用系统的登录入口是不需要指定的（也就是说：来自与任意一个url路径的http请求都会被尝试解析其中的令牌信息）。不过为了应用系统的统一性，我们约定登录入口为“/web/login”。  
基于oauth2协议的认证（auth-oauth2-authorizeserver）：oauth2协议对认证路径有统一的要求：获取授权码的路径为：“web/oauth2/authorize”；获取accessToken的路径为：“web/oauth2/token”。

### 用户授权

ApacheSiro提供了基于方法注解的用户权限验证体系。框架默认会打开这一特性。举例如下：

```java
@RequiresPermissions("user:list")
@RequestMapping(method = RequestMethod.GET)
public
@ResponseMessage
List<UserInfo> listUser(@CurrentSession Session session) {
    // 代码实现
}
```

以上代码声明要求能够访问controller中的listUser方法（或者某一url路径）的用户，必须存在“user:list”这个权限表达式字符串的权限。  
至于权限的校验过程，由ApacheShiro将逻辑控制路由到我们实现的Realm子类的doGetAuthorizationInfo\(PrincipalCollection principals\)方法中。  
对于权限表达式的介绍，参见：[http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/Permission.html](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/Permission.html)



