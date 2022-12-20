#hikari #spring #java

任务执行 sql，每过一段时间后就会报一个错误。

```text
HikariPool-1 - Connection is not available, request timed out after 30002ms.
```

## 方法一

1. 启用 `com.zaxxer.hikari` debug 日志。

```xml
<logger name="com.zaxxer.hikari" level="debug" additivity="false">  
    <appender-ref ref="CONSOLE"/>
</logger>
```

程序会打印相关日志

```text
2022-12-20 16:39:21.264 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : mysql_ds - configuration:
2022-12-20 16:39:21.265 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : allowPoolSuspension................................false
2022-12-20 16:39:21.266 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : autoCommit................................true
2022-12-20 16:39:21.266 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : catalog................................none
2022-12-20 16:39:21.266 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : connectionInitSql................................none
2022-12-20 16:39:21.266 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : connectionTestQuery................................"SELECT 1"
2022-12-20 16:39:21.266 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : connectionTimeout................................30000
2022-12-20 16:39:21.266 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : dataSource................................none
2022-12-20 16:39:21.266 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : dataSourceClassName................................none
2022-12-20 16:39:21.266 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : dataSourceJNDI................................none
2022-12-20 16:39:21.266 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : dataSourceProperties................................{password=<masked>}
2022-12-20 16:39:21.266 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : driverClassName................................"com.mysql.cj.jdbc.Driver"
2022-12-20 16:39:21.266 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : exceptionOverrideClassName................................none
2022-12-20 16:39:21.266 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : healthCheckProperties................................{}
2022-12-20 16:39:21.266 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : healthCheckRegistry................................none
2022-12-20 16:39:21.266 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : idleTimeout................................60000
2022-12-20 16:39:21.266 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : initializationFailTimeout................................1
2022-12-20 16:39:21.266 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : isolateInternalQueries................................false
2022-12-20 16:39:21.267 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : jdbcUrl................................jdbc:mysql://localhost:3306/metric
2022-12-20 16:39:21.267 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : keepaliveTime................................0
2022-12-20 16:39:21.267 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : leakDetectionThreshold................................0
2022-12-20 16:39:21.267 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : maxLifetime................................1800000
2022-12-20 16:39:21.267 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : maximumPoolSize................................10
2022-12-20 16:39:21.267 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : metricRegistry................................none
2022-12-20 16:39:21.267 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : metricsTrackerFactory................................com.zaxxer.hikari.metrics.micrometer.MicrometerMetricsTrackerFactory@37f7401c
2022-12-20 16:39:21.267 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : minimumIdle................................5
2022-12-20 16:39:21.267 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : password................................<masked>
2022-12-20 16:39:21.267 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : poolName................................"mysql_ds"
2022-12-20 16:39:21.267 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : readOnly................................false
2022-12-20 16:39:21.267 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : registerMbeans................................false
2022-12-20 16:39:21.267 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : scheduledExecutor................................none
2022-12-20 16:39:21.267 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : schema................................none
2022-12-20 16:39:21.267 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : threadFactory................................internal
2022-12-20 16:39:21.267 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : transactionIsolation................................default
2022-12-20 16:39:21.267 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : username................................"root"
2022-12-20 16:39:21.268 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariConfig           : validationTimeout................................5000
2022-12-20 16:39:21.268  INFO 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariDataSource       : mysql_ds - Starting...
2022-12-20 16:39:21.305 DEBUG 21612 --- [-192.160.30.125] com.zaxxer.hikari.pool.HikariPool        : mysql_ds - Added connection com.mysql.cj.jdbc.ConnectionImpl@5fbf938f
2022-12-20 16:39:21.306  INFO 21612 --- [-192.160.30.125] com.zaxxer.hikari.HikariDataSource       : mysql_ds - Start completed.
```

2. 观察日志

等程序报错 

```text
HikariPool-1 - Connection is not available, request timed out after 30001ms.
```

这时可以看到连接池内的连接数量。
```text
HikariPool-1 - Timeout failure stats (total=10, active=10, idle=0, waiting=0)
```

这里并不是说 30 秒就会出错，而是连接池满了后 30 秒出错。如上面信息没有空闲的连接了。这说明这些连接都没有被关闭，造成连接泄漏。

所以记得在程序中把连接关掉就可以了。

像下面代码因为没有关闭 connection，所以就会有问题。

```java
// 错误代码
@Override
public void update(int postId, String metaValue) throws SQLException {

    try (PreparedStatement ps = 
            dataSource.getConnection().prepareStatement(SQL_UPDATE_POST_META)) {

        ps.setString(1, metaValue);
        ps.setInt(2, postId);
        ps.setString(3, GlobalUtils.META_KEY);

        ps.executeUpdate();

    } catch (SQLException e) {
        logger.error("[UPDATE] [POST_ID] : {}, [META_VALUE] : {}", postId, metaValue);
        throw e;
    }

}
```

修复后如下。

```java
@Override
public void update(int postId, String metaValue) throws SQLException {

    try (Connection connection = dataSource.getConnection();
         PreparedStatement ps = connection.prepareStatement(SQL_UPDATE_POST_META)) {
         
        ps.setString(1, metaValue);
        ps.setInt(2, postId);
        ps.setString(3, GlobalUtils.META_KEY);

        ps.executeUpdate();

    } catch (SQLException e) {
        logger.error("[UPDATE] [POST_ID] : {}, [META_VALUE] : {}", postId, metaValue);
        throw e;
    }

}
```

## 方法二

连接池配置中启用 `leak-detection-threshold` 。这个配置是控制在记录消息之前连接可能离开池的时间量，表明可能存在连接泄漏。值为0意味着泄漏检测被禁用。启用泄漏检测的最低可接受值为2000（2秒）。 默认值：0

简单说是会定时检测可能存在连接泄漏的点。严格来说这并不算是个常规的方法，只是说有这么个途径事先知道程序里是否有连接泄漏的点。所以还是建议在开发，测试环境里开启该配置项。

```yaml
hikari:
  leak-detection-threshold: 3000
```

启动项目后会看到如下日志。

```log
2022-12-20 17:16:03.342  WARN 23124 --- [_ds housekeeper] com.zaxxer.hikari.pool.ProxyLeakTask     : Connection leak detection triggered for com.mysql.cj.jdbc.ConnectionImpl@29a1c8f8 on thread TimerWheel-Executor-5, stack trace follows

java.lang.Exception: Apparent connection leak detected
	at com.zaxxer.hikari.HikariDataSource.getConnection(HikariDataSource.java:128)
	at com.camel.km.dag.handler.AbstractJdbcMetricHandler.handle(AbstractJdbcMetricHandler.java:27)
	at com.camel.km.dag.task.MetricTask.doTask(MetricTask.java:29)
```

这里就已经看到了出问题的地方，所以直接就修复代码就可以了。

## 其他

网上有很多说是去修改参数的。大多说的都是没头没尾。而且没有思路。