# linux

```shell
mvn archetype:generate \
    -DgroupId=com.alibaba.cola.demo.web \
    -DartifactId=demo-web \
    -Dversion=1.0.0-SNAPSHOT \
    -Dpackage=com.alibaba.demo \
    -DarchetypeArtifactId=cola-framework-archetype-web \
    -DarchetypeGroupId=com.alibaba.cola \
    -DarchetypeVersion=4.3.2
```

# win

win 中的命令行是真恶心，得加上 `"`。

```powershell
mvn archetype:generate `
    "-DgroupId=com.x.demo.web" `
    "-DartifactId=demo-web" `
    "-Dversion=1.0.0-SNAPSHOT" `
    "-Dpackage=com.x.demo" `
    "-DarchetypeArtifactId=cola-framework-archetype-web" `
    "-DarchetypeGroupId=com.alibaba.cola" `
    "-DarchetypeVersion=4.3.2"
```