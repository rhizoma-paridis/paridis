## 引入依赖

在springboot中使用validate还是很简单。

```XML
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

**SpringBoot 的参数校验有两套机制，执行的时候会同时被两套机制控制。** 两套机制除了控制各自的部份外，有部分是重叠的，这部分又会涉及优先级之类的问题。

springMvc 控制部分只能在 controller 中使用，且无法校验 `RequestParam`这些单个参数，所以尽量使用 aop 控制。

## 非bean对象

```Java
@RestController
@Validated
public class UserController {

    @GetMapping("/user1")
    public String getUser(@NotBlank String username) {
        System.out.println("username: " + username);
        return "getUser";
    }
}
```

1. 在类上添加`@validate`注解
2. 参数前添加相关注解。

这里只能使用 aop的方式。

## bean对象

```Java
@RestController
public class UserController {

    @GetMapping("/user3")
    public String getUser3(@Valid UserInfo userInfo) {
        System.out.println("username: " + userInfo.getUserName());
        return "getUser";
    }
    @GetMapping("/user4")
    public String getUser4(@Validated({Add.class}) UserInfo userInfo) {
        System.out.println("username: " + userInfo.getUserName());
        return "getUser";
    }
}

@Data
public class UserInfo {

    @NotEmpty(groups = {Add.class})
    private String userName;

    @Min(1)
    @Max(100)
    private Long id;

    @NotEmpty
    private String password;

    private String email;

}


```

1. 在参数上添加`valid`或`validated`注解，因为validated 可以指定分组，所以建议使用`validate`
2. 在bean中添加相关校验规则

## 自定义校验规则

```Java
@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER})
@Retention(RUNTIME)
@Documented
@Constraint(validatedBy = {BeautyValidator.class})
public @interface Beauty {

    String message() default "不好看不要";

    Class<?>[] groups() default { };

    Class<? extends Payload>[] payload() default {};
}


public class BeautyValidator implements ConstraintValidator<Beauty, String> {

    @Override
    public void initialize(Beauty constraintAnnotation) {
        ConstraintValidator.super.initialize(constraintAnnotation);
    }

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        if (value != null) {
            return value.equals("beauty");
        }
        return true;
    }
}

```

1. 创建一个约束注解
2. 创建一个 validator、
3. 正常使用

## 嵌套

```Java
    @NotNull(groups = {Save.class, Update.class})
    @Length(min = 6, max = 20, groups = {Save.class, Update.class})
    private String password;

    @NotNull(groups = {Save.class, Update.class})
    @Valid
    private Job job;
```

在嵌套的对象上添加`@valid`。

## 集合校验

如果请求体直接传递了`json`数组给后台，并希望对数组中的每一项都进行参数校验。此时，如果我们直接使用`java.util.Collection`下的`list`或者`set`来接收数据，参数校验并不会生效！我们可以使用自定义`list`集合来接收参数：

```Java
public class ValidationList<E> implements List<E> {

    @Delegate // @Delegate是lombok注解
    @Valid // 一定要加@Valid注解
    public List<E> list = new ArrayList<>();

    // 一定要记得重写toString方法
    @Override
    public String toString() {
        return list.toString();
    }
}

@PostMapping("/saveList")
public Result saveList(@RequestBody @Validated(UserDTO.Save.class) ValidationList<UserDTO> userList) {
    // 校验通过，才会执行业务逻辑处理
    return Result.ok();
}

```

## 手动校验

```Java
@Autowired
private javax.validation.Validator globalValidator;

// 编程式校验
@PostMapping("/saveWithCodingValidate")
public Result saveWithCodingValidate(@RequestBody UserDTO userDTO) {
    Set<ConstraintViolation<UserDTO>> validate = globalValidator.validate(userDTO, UserDTO.Save.class);
    // 如果校验通过，validate为空；否则，validate包含未校验通过项
    if (validate.isEmpty()) {
        // 校验通过，才会执行业务逻辑处理

    } else {
        for (ConstraintViolation<UserDTO> userDTOConstraintViolation : validate) {
            // 校验失败，做其它逻辑
            System.out.println(userDTOConstraintViolation);
        }
    }
    return Result.ok();
}
```

## 快速失败

`Spring Validation`默认会校验完所有字段，然后才抛出异常。可以通过一些简单的配置，开启`Fali Fast`模式，一旦校验失败就立即返回。

```Java
@Bean
public Validator validator() {
    ValidatorFactory validatorFactory = Validation.byProvider(HibernateValidator.class)
            .configure()
            // 快速失败模式
            .failFast(true)
            .buildValidatorFactory();
    return validatorFactory.getValidator();
}
```

## 原理

### `requestBody`参数校验实现原理

在`spring-mvc`中，`RequestResponseBodyMethodProcessor`是用于解析`@RequestBody`标注的参数以及处理`@ResponseBody`标注方法的返回值的。显然，执行参数校验的逻辑肯定就在解析参数的方法`resolveArgument()`中：

```Java
public class RequestResponseBodyMethodProcessor extends AbstractMessageConverterMethodProcessor {
    @Override
    public Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
                                  NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {

        parameter = parameter.nestedIfOptional();
        //将请求数据封装到DTO对象中
        Object arg = readWithMessageConverters(webRequest, parameter, parameter.getNestedGenericParameterType());
        String name = Conventions.getVariableNameForParameter(parameter);

        if (binderFactory != null) {
            WebDataBinder binder = binderFactory.createBinder(webRequest, arg, name);
            if (arg != null) {
                // 执行数据校验
                validateIfApplicable(binder, parameter);
                if (binder.getBindingResult().hasErrors() && isBindExceptionRequired(binder, parameter)) {
                    throw new MethodArgumentNotValidException(parameter, binder.getBindingResult());
                }
            }
            if (mavContainer != null) {
                mavContainer.addAttribute(BindingResult.MODEL_KEY_PREFIX + name, binder.getBindingResult());
            }
        }
        return adaptArgumentIfNecessary(arg, parameter);
    }
}
```

可以看到，`resolveArgument()`调用了`validateIfApplicable()`进行参数校验。

```Java
protected void validateIfApplicable(WebDataBinder binder, MethodParameter parameter) {
    // 获取参数注解，比如@RequestBody、@Valid、@Validated
    Annotation[] annotations = parameter.getParameterAnnotations();
    for (Annotation ann : annotations) {
        // 先尝试获取@Validated注解
        Validated validatedAnn = AnnotationUtils.getAnnotation(ann, Validated.class);
        //如果直接标注了@Validated，那么直接开启校验。
        //如果没有，那么判断参数前是否有Valid起头的注解。
        if (validatedAnn != null || ann.annotationType().getSimpleName().startsWith("Valid")) {
            Object hints = (validatedAnn != null ? validatedAnn.value() : AnnotationUtils.getValue(ann));
            Object[] validationHints = (hints instanceof Object[] ? (Object[]) hints : new Object[] {hints});
            //执行校验
            binder.validate(validationHints);
            break;
        }
    }
}
```

看到这里，大家应该能明白为什么这种场景下`@Validated`、`@Valid`两个注解可以混用。我们接下来继续看`WebDataBinder.validate()`实现。

```Java
@Override
public void validate(Object target, Errors errors, Object... validationHints) {
    if (this.targetValidator != null) {
        processConstraintViolations(
            //此处调用Hibernate Validator执行真正的校验
            this.targetValidator.validate(target, asValidationGroups(validationHints)), errors);
    }
}
```

最终发现底层最终还是调用了`Hibernate Validator`进行真正的校验处理。

### 方法级别的参数校验实现原理

上面提到的将参数一个个平铺到方法参数中，然后在每个参数前面声明`约束注解`的校验方式，就是方法级别的参数校验。实际上，这种方式可用于任何`Spring Bean`的方法上，比如`Controller`/`Service`等。**其底层实现原理就是****`AOP`****，具体来说是通过****`MethodValidationPostProcessor`****动态注册****`AOP`****切面，然后使用****`MethodValidationInterceptor`****对切点方法织入增强**。

```Java
public class MethodValidationPostProcessor extends AbstractBeanFactoryAwareAdvisingPostProcessorimplements InitializingBean {
    @Override
    public void afterPropertiesSet() {
        //为所有`@Validated`标注的Bean创建切面
        Pointcut pointcut = new AnnotationMatchingPointcut(this.validatedAnnotationType, true);
        //创建Advisor进行增强
        this.advisor = new DefaultPointcutAdvisor(pointcut, createMethodValidationAdvice(this.validator));
    }

    //创建Advice，本质就是一个方法拦截器
    protected Advice createMethodValidationAdvice(@Nullable Validator validator) {
        return (validator != null ? new MethodValidationInterceptor(validator) : new MethodValidationInterceptor());
    }
}
```

接着看一下`MethodValidationInterceptor`：

```Java
public class MethodValidationInterceptor implements MethodInterceptor {
    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        //无需增强的方法，直接跳过
        if (isFactoryBeanMetadataMethod(invocation.getMethod())) {
            return invocation.proceed();
        }
        //获取分组信息
        Class<?>[] groups = determineValidationGroups(invocation);
        ExecutableValidator execVal = this.validator.forExecutables();
        Method methodToValidate = invocation.getMethod();
        Set<ConstraintViolation<Object>> result;
        try {
            //方法入参校验，最终还是委托给Hibernate Validator来校验
            result = execVal.validateParameters(
                invocation.getThis(), methodToValidate, invocation.getArguments(), groups);
        }
        catch (IllegalArgumentException ex) {
            ...
        }
        //有异常直接抛出
        if (!result.isEmpty()) {
            throw new ConstraintViolationException(result);
        }
        //真正的方法调用
        Object returnValue = invocation.proceed();
        //对返回值做校验，最终还是委托给Hibernate Validator来校验
        result = execVal.validateReturnValue(invocation.getThis(), methodToValidate, returnValue, groups);
        //有异常直接抛出
        if (!result.isEmpty()) {
            throw new ConstraintViolationException(result);
        }
        return returnValue;
    }
}
```
