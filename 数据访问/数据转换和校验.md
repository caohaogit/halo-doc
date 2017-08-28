## 数据转换和校验

框架的数据转换功能主要是以XML和JSON对Java models的序列化和反序列化。其中，XML转换是通过JAXB组件实现的；JSON转换是通过FastJson组件实现的。  
FastJson参考：[https://github.com/Alibaba/fastjson/wiki/%E9%A6%96%E9%A1%B5](https://github.com/Alibaba/fastjson/wiki/%E9%A6%96%E9%A1%B5)  
JAXB参考：[https://jaxb.java.net/tutorial/](https://jaxb.java.net/tutorial/)  
框架集成了Hibernate-validator组件，并在Web交互层的Controller及Dubbo的服务治理中实现了对业务对象数据的有效性校验。  
关于hibernate-validator的技术资料，参考链接：  
[http://docs.jboss.org/hibernate/validator/5.2/reference/en-US/html\_single/](http://docs.jboss.org/hibernate/validator/5.2/reference/en-US/html_single/)

### Web交互层

首先，在对应的BO，VO对象中添加validate注解。例如：

```java
public class UserInfoimplements Serializable {
    private String userId;
    @NotEmpty
    private String userName;
    @NotEmpty
    private String password;
}
```

对于validate注解的其它可用类型，参见hibernamte-validator的参考资料中描述。  
其次，在一个Spring Controller中声明对该VO对象的校验。

```java
@RequestMapping(value = "/body", method = RequestMethod.POST)
public
@ResponseMessage
@Valid
UserInfo validateRequestBody(@RequestBody @Valid UserInfo userInfo, BindingResult result) {
    result.getAllErrors().stream().forEach(System.out::println);
    return userInfo;
}
```

上面的样例代码中，对方法的入参和返回值都做了validate数据校验，只要其中一个不满足，则会向Web前端输出校验失败的消息内容。

### 编码式校验

如果开发者需要在业务代码中主动对某一业务对象进行数据有效性验证，可以调用hibernate-validate原生api操作：  
框架已经将验证服务注册为SpringService。开发者只需注入即可。

```java
@Autowired
LocalValidatorFactoryBean validatorService;
```

在使用时，参考hibernate-validator的api即可，样例代码实现对于方法返回值的数据校验：

```java
Set<ConstraintViolation<Object>> returnViolations = validator.forExecutables().validateReturnValue(SpringContextHolder.getBean(returnType.getMethod().getDeclaringClass()), returnType.getMethod(), returnValue);
if (CollectionUtils.isNotEmpty(returnViolations))
    throw new ValidateException();
```



