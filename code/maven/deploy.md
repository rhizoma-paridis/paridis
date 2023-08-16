#maven

如果想把 jar 包上传到私服，则可以执行命令 `mvn clean deploy` 。但是执行之前需要配置好目标仓库。

## 私服配置

### 密码

#### 明文密码

```xml
<server>
    <id>release</id>
    <username>foo</username>
    <password>test</password>
</server>
<server>
    <id>snapshot</id>
    <username>foo</username>
    <password>test</password>
</server>
```

完事。

#### 加密密码

加密密码有点复杂，需要先设定个 password A。A 用来生成加密密钥的，所以可以随便生成一个，不需要记忆。用 A 生成的密钥来加密实际的密码。详细说明可以参考[官方文档](https://maven.apache.org/guides/mini/guide-encryption.html)。

```shell
mvn --encrypt-master-password <password>
```

这个命令用来生成加密密钥。当然新版本的 maven 不允许把密码写在命令行，也就是这个命令不允许带参数，执行后提示输入密码，效果是一样的。命令输出如下。

```shell
{jSMOWnoPFgsHVpMvz5VrIt5kRbzGpI8u+9EF1iFQyJQ=}
```

手动创建文件 `${user.home}/.m2/settings-security.xml`。内容如下。

```xml
<settingsSecurity>
    <master>{jSMOWnoPFgsHVpMvz5VrIt5kRbzGpI8u+9EF1iFQyJQ=}</master>
</settingsSecurity>
```

master 的值就是上面生成的密钥。这样我们就有了对密码加密的密钥。

然后再执行命令如下。

```shell
mvn --encrypt-password <password>
```

输出如下。

```shell
{COQLCE6DU6GtcS5P=}
```

这样我们就拿到了最终密码。在 setting.xml 中设置如下。

```xml
<server>
    <id>release</id>
    <username>foo</username>
    <password>{COQLCE6DU6GtcS5P=}</password>
</server>
<server>
    <id>snapshot</id>
    <username>foo</username>
    <password>foo reset this password on 2022-03-11, expires on 2023-04-11{COQLCE6DU6GtcS5P=}</password>
</server>
```

这里在 snapshot 里有点不一样。其实对于 `{}` 前的字符，maven 会当成注释。所以这样也是正确的。

### distributionManagement

私服配置分两种，一种是直接在项目的 Pom 文件中配置，一种是在 setting.xml 中。

#### pom.xml

```xml
<profile>
<distributionManagement>
    <repository>
	    <!--id要和setting文件中server节点中配置的一样-->
        <id>release</id>
        <url>http://localhost/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>snapshot</id>
        <url>http://localhost/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
</profile>
```

#### setting.xml

```xml
<profiles>
    <profile>
        <id>nexus</id>
        <properties>
            <altSnapshotDeploymentRepository>snapshot::default::http://nexus.in.camelgames-data.com/repository/maven-snapshots</altSnapshotDeploymentRepository>
            <altReleaseDeploymentRepository>release::default::http://nexus.in.camelgames-data.com/repository/maven-releases</altReleaseDeploymentRepository>
        </properties>
    </profile>
</profiles>

<activeProfiles>
    <activeProfile>nexus</activeProfile>
</activeProfiles>
```

其中 `altSnapshotDeploymentRepository` 的格式是：`id::layout::url`，id 就是 server 中配置的 id ，layout 这里不多介绍，maven3 之后使用 default 就可以了，具体可以但相关资料，url 就是仓库地址。

但是这个配置需要 deploy 插件版本在 2.8 以上。所以如果你的 deploy 插件版本太低需要配置下。

```xml
<plugin>
    <artifactId>maven-deploy-plugin</artifactId>
    <version>2.8</version>
</plugin>
```

这两种方式在公司中的话其实我个人觉得第二种会更好点。因为这种信息没有必要保存到代码中。开发者会不应去关心上传到仓库的事情，都交给 CI 来做，所以只用 CI 的打包机有这些配置就可以了。

## 打包配置

默认情况下，maven 只打包一个 jar 包，不会打包 source，javadoc，当然如果你想往官方仓库发包还需要使用 gpg 签名。

```xml
<!-- Package source codes -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-source-plugin</artifactId>
    <version>${maven-source-plugin.version}</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>jar-no-fork</goal>
            </goals>
        </execution>
    </executions>
</plugin>
<!-- Java Doc -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-javadoc-plugin</artifactId>
    <version>${maven-javadoc-plugin.version}</version>
    <configuration>
        <!-- Prevent JavaDoc error from affecting building project. -->
        <failOnError>false</failOnError>
        <!-- Non-strict mode -->
        <additionalJOption>-Xdoclint:none</additionalJOption>
    </configuration>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>jar</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

这里只写的 source 跟 javadoc 插件，配置了这两个插件就可以打包出相应的内容。至于 gpg 这里不再说明。
