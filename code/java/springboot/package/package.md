简单 `springboot` 项目打包时，因为默认配置了 `spring-boot-maven-plugin` 所以会自动打成 `fat jar`。但是如果是多模块组成的项目，有时候 `spring-boot-maven-plugin` 的配置会丢，所以打包的时候会出现一些错误。

```shell
Source file must be provided
```

这时在命令行执行：`mvn clean package spring-boot:repackage`。

或者在插件中配置：

```xml
<plugin>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-maven-plugin</artifactId>
	<executions>
		<execution>
			<goals>
				<goal>repackage</goal>
			</goals>
		</execution>
	</executions>
</plugin>
```

