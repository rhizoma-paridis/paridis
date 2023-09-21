## 使用方式

### 1. 启用 async

```Java
@Configuration
@EnableAsync
public class SpringAsyncConfig { ... }
```

### 2. 添加 @Async

1. 需要异步执行的方法的所在类由Spring管理
2. 在类或方法上添加 @Async

   在类上添加后，所有类中方法都是异步执行。

   在方法上加的话，相应方法异步执行。

```Java
@Async
public void async_demo() throws InterruptedException {
    TimeUnit.SECONDS.sleep(5);
    System.out.println("hello async 1");
}
```

### 3. 最佳实践

Spring应用默认的线程池，指在@Async注解在使用时，不指定线程池的名称。@Async的默认线程池为**SimpleAsyncTaskExecutor**。这个线程池有点问题，所以要自定义一个线程池。

设置 spring 线程池有很多种方法，像下面这种往容器中添加一个`TaskExecutor` 就可以。

```Java
@Bean("myPool")
public Executor getTimingPool() {
    ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
    executor.setCorePoolSize(Runtime.getRuntime().availableProcessors());
    executor.setMaxPoolSize(Runtime.getRuntime().availableProcessors() * 4);
    // use SynchronousQueue
    executor.setQueueCapacity(0);
    executor.setKeepAliveSeconds(60);
    executor.setThreadNamePrefix("myPool-");
    executor.setRejectedExecutionHandler(RejectedExecutionHandlerFactory.newThreadRun("PowerJobTiming"));
    return executor;
}
```

但是不建议这样做，为了处理异步方法的异常，要添加其他配置。继承 ` AsyncConfigurerSupport  ` 或 `AsyncConfigurer` 实现 `getAsyncUncaughtExceptionHandler` 方法

```Java
public class CustomAsyncExceptionHandler
  implements AsyncUncaughtExceptionHandler {

    @Override
    public void handleUncaughtException(
      Throwable throwable, Method method, Object... obj) {
 
        System.out.println("Exception message - " + throwable.getMessage());
        System.out.println("Method name - " + method.getName());
        for (Object param : obj) {
            System.out.println("Parameter value - " + param);
        }
    }
    
}

```

```Java
@Override
public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
    return new CustomAsyncExceptionHandler();;
} 
```

## 原理

TODO

[Spring使用@Async注解 - 无涯Ⅱ - 博客园 (cnblogs.com)](https://www.cnblogs.com/wlandwl/p/async.html)

[Spring中异步注解@Async的使用、原理及使用时可能导致的问题_做一个认真的程序员-CSDN博客](https://daimingzhi.blog.csdn.net/article/details/107500036)
