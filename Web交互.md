## Web交互

基于SpringMVC实现与前端页面的数据交互。为了让业务开发过程更加规范，框架置入多个SpringMVC拦截器，对异常处理，输入参数，输出结果，校验等过程提供了统一封装与调用。

```
框架要求所有Web请求地址统一为REST风格进行。
```

### 通用配置

web.session.timeout : session超时时间\[毫秒\]

### 数据交互

对于声明为controller的Spring服务类代码。包含的方法输入参数可以通过添加特定的注解来控制Spring将特定的数据mapping为特定Java类型。框架基于SpringMVC现有的机制，扩展了输入参数注解，返回值注解，方法注解。

#### 输入参数类型

1. [@CurrentSession](http://localhost:3000/CurrentSession)注解
   `List<UserInfo> listUser(@CurrentSession Session session)`
   获取当前请求的Session，框架会自动将Session对象注入到方法中。
2. [@PathVariable](http://localhost:3000/PathVariable)注解
   `UserInfo getUser(@CurrentSession Session session, @PathVariable("id") String userId)`
   与[@RequestMapping](http://localhost:3000/RequestMapping)注解相互配合，指定方法中某个输入参数为url路径上的一个上下文标识。
   url格式为：[http://XXX/XXX/{userId值](http://xxx/XXX/userid的值)}
3. [@RequestJsonParam](http://localhost:3000/RequestJsonParam)注解：方式一
   `UserInfo bindModelFromPostRequestJsonParam(@RequestJsonParam("userInfo") UserInfo userInfo)`
   要求传入特定标识的参数，并且参数值按照json格式序列化。
   url格式为：[http://XXX/XXX?userInfo=json字符串](http://xxx/XXX?userInfo=json字符串)
4. [@RequestJsonParam](http://localhost:3000/RequestJsonParam)注解：方式二
   `UserInfo bindModelFromGetRequestJsonParamMap(@RequestJsonParam(type = RequestJsonType.DATA_MAP) UserInfo userInfo)`
   要求传入一个K-V集合，集合内元素如果要嵌套类型，可以按照JSON格式序列化
   url格式为：[http://XXX/XXX](http://xxx/XXX)
5. [@RequestBody](http://localhost:3000/RequestBody)注解
   `UserInfo bindModelFromPostRequestBody(@RequestBody UserInfo userInfo)`
   要求传入一个Json字符串，框架会将其反序列化为对应的Java类型对象
   url格式为：[http://XXX/XXX](http://xxx/XXX)

#### 返回值类型

[@ResponseMessage](http://localhost:3000/ResponseMessage)

```java
@ResponseMessage
List<UserInfo> listUser(@CurrentSession Session session)
```

通过该标记，方法的返回值将统一以`com.halo.core.message.model. ResponseMessage`的序列化JSON字符串作为返回，针对每个方法的业务数据返回值，会被封装到`ResponseMessage.data`中。这样在前端JS处理时，可以通过读取`ResponseMessage.data`获取到真实的业务数据。

#### 方法注解

[@RequestMapping](http://localhost:3000/RequestMapping)

```java
@RequestMapping(method = RequestMethod.GET)
public
@ResponseMessage
List<UserInfo> listUser(@CurrentSession Session session)
```

SpringMVC自带的针对方法声明上的资源访问路径描述，其中，method属性用来标识该方法映射为HTTP的METHOD类型（GET，PUT，POST，DELETE，PUT）。Value属性用来描述该方法映射为HTTP的URL路径地址，例如：

```java
@RequestMapping(value = "update/{id}", method = RequestMethod.PUT)
```

### 异常处理

每个Controller类中方法可以抛出运行时异常和com.halo.core.exception. BusinessException。框架通过SpringMVC拦截器将异常捕获后，封装到com.halo.core.message.model. ResponseMessage中，反馈给前台页面。

为了更加规范，在本文档的“异常处理”小节中，会对Web Controller的各个方法中抛出的各类异常做统一的规范和说明。在这里不再赘述。

