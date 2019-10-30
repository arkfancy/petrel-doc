# 代码生成器配置文件模板

校验日期：

---

* generatorConfig.xml模板

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<generatorConfiguration>
    <!-- 数据库连接 -->
    <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                    connectionURL="jdbc:mysql://0.0.0.0:3306/database?useUnicode=true&amp;characterEncoding=utf-8&amp;useSSL=false"
                    userId="username" password="password"/>

    <!-- 实体层 -->
    <javaModelGenerator targetPackage="com.belle.petrel.module.entity" targetProject="petrel-module-core"/>

    <!-- mapper层 -->
    <javaMapperGenerator targetPackage="com.belle.petrel.module.mapper" targetProject="petrel-module-core"/>

    <!-- 服务层Service -->
    <javaServiceGenerator targetPackage="com.belle.petrel.module.service" targetProject="petrel-module-core"/>

    <!-- 服务层ServiceImpl -->
    <javaServiceImplGenerator targetPackage="com.belle.petrel.module.service.impl" targetProject="petrel-module-core"/>

    <!-- xml层 -->
    <javaControllerGenerator targetPackage="" targetProject="petrel-module-core"/>

    <!-- 控制层 -->
    <javaControllerGenerator targetPackage="com.belle.petrel.module.controller" targetProject="petrel-module-api"/>

</generatorConfiguration>

```

> 根据项目修改模板内数据库配置信息及模块名称。

* petrel-gcode-pom.xml模板

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>fast.code.download</groupId>
    <artifactId>fast-code-download</artifactId>
    <version>1.0.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>com.belle.petrel</groupId>
            <artifactId>petrel-gcode</artifactId>
            <version>1.0.0-SNAPSHOT</version>
            <exclusions>
                <exclusion>
                    <groupId>org.freemarker</groupId>
                    <artifactId>freemarker</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>com.google.guava</groupId>
                    <artifactId>guava</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>log4j</groupId>
                    <artifactId>log4j</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>mysql</groupId>
                    <artifactId>mysql-connector-java</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.xerial</groupId>
                    <artifactId>sqlite-jdbc</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>
</project>
```

> 默认情况下无需修改。

* petrel-gcode.bat模板

```
call mvn -f petrel-gcode-pom.xml dependency:copy-dependencies -U
java -jar target/dependency/petrel-gcode-1.0.0-SNAPSHOT.jar
```

> 默认情况下无需修改。

* petrel-gcode.sh模板

```
#!/usr/bin/env bash
mvn -f petrel-gcode-pom.xml dependency:copy-dependencies -U
java -jar target/dependency/petrel-gcode-1.0.0-SNAPSHOT.jar
```

> 默认情况下无需修改。





