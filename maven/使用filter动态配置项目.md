仔细看springboot的官方文档
https://docs.spring.io/spring-boot/docs/2.0.4.RELEASE/reference/htmlsingle/#using-boot-maven
 Note that, since the `application.properties` and `application.yml` files accept Spring style placeholders (`${…}`), the Maven filtering is changed to use `@..@` placeholders. (You can override that by setting a Maven property called `resource.delimiter`.)

 是说springboot自己使用了{…​}占位符，所以mvn打包的占位符使用@@







```
<profiles>
    <profile>
        <id>development</id>
        <properties>
            <env>development</env>
        </properties>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
    </profile>
    <profile>
        <id>test</id>
        <properties>
            <env>test</env>
        </properties>
    </profile>
    <profile>
        <id>preproduction</id>
        <properties>
            <env>preproduction</env>
        </properties>
    </profile>
    <profile>
        <id>product</id>
        <properties>
            <env>product</env>
        </properties>
    </profile>
</profiles>
<build>
        <filters>
            <filter>src/main/filters/filter-${env}.properties</filter>
        </filters>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>
```

pom元素介绍

| 元素                        | 介绍                                                         |
| --------------------------- | ------------------------------------------------------------ |
| filtering                   | 是否用filter文件中的值替directory下指定类型文件中的el表达式，true：将el表达式替换为相应的内容。 false：不替换el表达式,编译后的文件中还是el表达式 |
| profiles/profile/properties | 定义一个在pom中的变量。 可在当前pom.xml或以当前pom.xml为根的文件中使用 |
|                             |                                                              |
|                             |                                                              |
|                             |                                                              |
|                             |                                                              |
|                             |                                                              |

