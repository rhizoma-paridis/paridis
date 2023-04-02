#aop #dynamicProxy

## aop 基本概念

- Aspect（切面）： Aspect 声明类似于 Java 中的类声明，在 Aspect 中会包含着一些 Pointcut 以及相应的 Advice。
- Join point（连接点）：表示在程序中明确定义的点，典型的包括方法调用，对类成员的访问以及异常处理程序块的执行等等，它自身还可以嵌套其它 joint point。
- Pointcut（切点）：表示一组 joint point，这些 joint point 或是通过逻辑关系组合起来，或是通过通配、正则表达式等方式集中起来，它定义了相应的 Advice 将要发生的地方。
- Advice（增强）：Advice 定义了在 Pointcut 里面定义的程序点具体要做的操作，它通过 before、after 和 around 来区别是在每个 joint point 之前、之后还是代替执行的代码。
- Target（目标对象）：织入 Advice 的目标对象。
- Weaving（织入）：将 Aspect 和其他对象连接起来, 并创建 Adviced object 的过程

![](attachments/Pasted%20image%2020230224160928.png)

```xml
<aop:config>
	<aop:aspect id="time" ref="timeHandler">
		<aop:pointcut id="addAllMethod" expression="execution(* com.xrq.aop.HelloWorld.*(..))" />
		<aop:before method="printTime" pointcut-ref="addAllMethod" />
		<aop:after method="printTime" pointcut-ref="addAllMethod" />
	</aop:aspect>
</aop:config>
```

pointCut 是对 joinPoint 的抽象，joinPoint 描述的是一个方法，PointCut 是指一类方法，如 PointCut 表达式 `execution(* com.xrq.aop.HelloWorld.*(..))` 以正则的方式指明函数签名。

剩下的就是 advice。这个容易理解，就是要做的增强方法，最后所有这些组装起来的对象叫 aspect。

## 动态代理

代理一般是为了增强函数的默认行为。所以要有原函数，增强后的函数。

### JDK 动态代理

```java
public interface Worker {
    void work();
}

public class Programmer implements Worker {
    @Override
    public void work() {
        System.out.println("coding...");
    }
}
```

增强后的函数如下。

```java
public class AHandler implements InvocationHandler {
    private Object target;
    
    AHandler(Object target){
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if (method.getName().equals("work")) {
            System.out.println("A...");
            System.out.println("before work...");
            Object result = method.invoke(target, args);
            System.out.println("after work...");
            return result;
        }
        return method.invoke(target, args);
    }
}

public class BHandler implements InvocationHandler {
    private Object target;
    
    BHandler(Object target){
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if (method.getName().equals("work")) {
            System.out.println("B...");
            System.out.println("before work...");
            Object result = method.invoke(target, args);
            System.out.println("after work...");
            return result;
        }
        return method.invoke(target, args);
    }
}
```

在 handler 中已经对方法做了拼装。

生成代理对象。

```java
public static void main(String[] args) {
    Programmer programmer = new Programmer();
    Worker worker = (Worker) Proxy.newProxyInstance(
            programmer.getClass().getClassLoader(),
            programmer.getClass().getInterfaces(),
            new AHandler(programmer));
    worker.work();
}
```

> InvocationHandler 即增强方法跟原对象有关联。

代理对象反编译

```java
public final class $Proxy0 extends Proxy implements Worker{
    public $Proxy0(InvocationHandler invocationhandler){
        super(invocationhandler);
    }

    public final void work(){
        try{
            super.h.invoke(this, m3, null);
            return;
        }catch(Error _ex) { }
        catch(Throwable throwable){
            throw new UndeclaredThrowableException(throwable);
        }
    }

    private static Method m3;
    static {
        try{           
            m3 = Class.forName("com.hydra.test.Worker").getMethod("work", new Class[0]);   
            //省略其他Method
        }//省略catch
    }
}
```

代理类继承了 `Proxy` 类，这也是为什么 JDK 动态代理只能代理接口实现类。

### CGLIB 动态代理

引入相关的包。

```xml
<dependency>  
  <groupId>cglib</groupId>  
  <artifactId>cglib</artifactId>  
  <version>3.2.5</version>  
</dependency>
```

原函数。

```java
public class UserService {  
   public void saveUser(){  
       System.out.println("---- 保存用户 ----");  
   }  
}
```

增强后的函数。

```java
public class AMethodInterceptor implements MethodInterceptor {

    public Object intercept(Object obj, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        System.out.println("---- A 方法拦截 ----");
        Object object = methodProxy.invokeSuper(obj, args);
        return object;
    }
}

public class BMethodInterceptor implements MethodInterceptor {

    public Object intercept(Object obj, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        System.out.println("---- B 方法拦截 ----");
        Object object = methodProxy.invokeSuper(obj, args);
        return object;
    }
}
```

参数含义。

-   Object obj: obj 是 CGLIB 动态生成代理类实例
-   Method method: Method 为实体类所调用的被代理的方法引用
-   Objectp[] args: 这个就是方法的参数列表
-   MethodProxy methodProxy : 这个就是生成的代理类对方法的引用。

可以看出这里是对原函数增强。

> 增强函数与原函数没有关联，与 jdk 动态代理不同。

拼装代理对象。

```java
public static void main(String[] args) {

  Enhancer enhancer = new Enhancer();
  enhancer.setSuperclass(UserService.class);
  enhancer.setCallback(new AMethodInterceptor());

  UserService userService = (UserService)enhancer.create();

  userService.saveUser();
}
```

过程都一样，都是给原函数，增强后的函数，然后创建代理对象。