# dependency

`<optional>true</optional>` 与 `<scope>provided</scope>` 的区别。

最终效果是一样，都是不传递相关依赖，但是语义不一样。

`optional` 是表示包是可选的，即项目中有多个功能相同的依赖，这些包是相互替代关系，最终想用哪个，由最终项目决定。如，项目中包含 mysql, PostgreSQL,Oracle等多种数据库。但项目中实际会选用其中一部分。

`provided` 是表示自己不提供，使用方提供。即使用自己的项目自己提供。如，开发时依赖 tomcat 相关包，运行时环境在tomcat中。