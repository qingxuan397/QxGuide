SpringBoot默认达成jar包，使用SpringBoot构想web应用，默认使用内置的Tomcat。但考虑到项目需要集群部署或者进行优化时，就需要打成war包部署到外部的Tomcat服务器中。

    本文所使用SpringBoot版本为：2.0.3.RELEASE

一、修改pom.xml文件将默认的jar方式改为war：

```
<groupId>com.example</groupId>
<artifactId>application</artifactId>
<version>0.0.1-SNAPSHOT</version>
<!--默认为jar方式-->
<!--<packaging>jar</packaging>-->
<!--改为war方式-->
<packaging>war</packaging>
```


二、排除内置的Tomcat容器（两种方式都可）：

1.排除spring-boot-starter-web中的Tomcat

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```


2.添加依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <!--打包的时候可以不用包进去，别的设施会提供。事实上该依赖理论上可以参与编译，测试，运行等周期。
        相当于compile，但是打包阶段做了exclude操作-->
    <scope>provided</scope>
</dependency>
```


三、继承org.springframework.boot.web.servlet.support.SpringBootServletInitializer，实现configure方法：

为什么继承该类，SpringBootServletInitializer源码注释：

Note that a WebApplicationInitializer is only needed if you are building a war file and deploying it. 

If you prefer to run an embedded web server then you won't need this at all.

注意，如果您正在构建WAR文件并部署它，则需要WebApplicationInitializer。

如果你喜欢运行一个嵌入式Web服务器，那么你根本不需要这个。

启动类代码：

```
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

1.方式一，启动类继承SpringBootServletInitializer实现configure：

```
@SpringBootApplication
public class Application extends SpringBootServletInitializer {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
    
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(Application.class);
    }

}
```

2.方式二，新增加一个类继承SpringBootServletInitializer实现configure：

```
public class ServletInitializer extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        //此处的Application.class为带有@SpringBootApplication注解的启动类
        return builder.sources(Application.class);
    }

}
```

注意事项：

使用外部Tomcat部署访问的时候，application.properties(或者application.yml)中配置的

```
server.port=

server.servlet.context-path=
```

将失效，请使用tomcat的端口，tomcat，webapps下项目名进行访问。

为了防止应用上下文所导致的项目访问资源加载不到的问题，

建议pom.xml文件中<build></build>标签下添加<finalName></finalName>标签：

```
<build>
    <!-- 应与application.properties(或application.yml)中context-path保持一致 -->
    <finalName>war包名称</finalName>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

使用mvn命令行打包，运行：

clean是清除之前的包，-Dmaven.test.skip=true是忽略测试代码

jar 方式打包，使用内置Tomcat：mvn clean install -Dmaven.test.skip=true

运行：java -jar 包名.jar

war方式打包，使用外置Tomcat：mvn clean package -Dmaven.test.skip=true

运行：${Tomcat_home}/bin/目录下执行startup.bat(windows)或者startup.sh(linux)
————————————————
版权声明：本文为CSDN博主「Geek-Rs」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_33512843/article/details/80951741