## maven 编译问题

### powershell 执行 maven 命令

```bash
mvn clean install -Prelease “-Dmaven.test.skip=true”
```

普通cmd命令窗口执行

```bash
mvn clean install -Prelease -Dmaven.test.skip=true
```



### 跳过 maven-checkstyle-plugin 检查



```bash
mvn clean install -Dcheckstyle.skip=true -Dmaven.test.skip=true
```

-Dmaven.javadoc.skip=true

如果要从pom禁用checkstyle，则可以将checkstyle插件添加到pom并设置skip标志以阻止检查。

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-checkstyle-plugin</artifactId>
    <configuration>
        <skip>true</skip>
    </configuration>
</plugin>
```


