#Spring-Boot使用jsp
在最初的j2ee开发时通常情况下我们使用jsp作为我们的页面模板，除了jsp还有很多不同页面模板技术例如：FreeMarker、Thymeleaf等。但是在spring boot jsp并不是官方推荐的模板技术，我们先看看官方怎么说的。

27.3.5 JSP limitations
When running a Spring Boot application that uses an embedded servlet container (and is packaged as an executable archive), there are some limitations in the JSP support.

1.  With Tomcat it should work if you use war packaging, i.e. an executable war will work, and will also be deployable to a standard container (not limited to, but including Tomcat). An executable jar will not work because of a hard coded file pattern in Tomcat.
2.  With Jetty it should work if you use war packaging, i.e. an executable war will work, and will also be deployable to any standard container.
3.  Undertow does not support JSPs.
4.  Creating a custom error.jsp page won’t override the default view for error handling, custom error pages should be used instead.

字面意思就是如果用jsp作为模板页面在打包为应用后会多种限制
1.  打包为war包是可以运行的，但是打包为可执行的jar包是无法运行的
2.  自定义的error.jsp页面将无法替换默认错误处理页面，例如404之类的。

虽然有这么一些限制，但是对于使用惯了jsp的来说可能觉得无所谓啊，/手动滑稽
那么我们怎么在Spring Boot使用jsp呢，下面我们就来简单介绍一下
环境还是基于maven管理
pom.xml文件很简单
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<!-- Your own application should inherit from spring-boot-starter-parent -->
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-samples</artifactId>
		<version>1.5.7.RELEASE</version>
	</parent>
	<artifactId>spring-boot-sample-web-jsp</artifactId>
	<packaging>war</packaging>
	<name>JSP Sample</name>
	<description>JSP Sample</description>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>jstl</artifactId>
		</dependency>
		<!-- Provided -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>org.apache.tomcat.embed</groupId>
			<artifactId>tomcat-embed-jasper</artifactId>
			<scope>provided</scope>
		</dependency>
		<!-- Test -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>
```
主类
```
@Configuration
@SpringBootApplication
@ImportResource({"classpath:applicationContext.xml"})
public class MaterialApplication {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(MaterialApplication.class);
    }

	public static void main(String[] args) {
		SpringApplication.run(MaterialApplication.class, args);
	}
}

```
Controller类
```
@Controller
public class ApplicationController {
    private static Logger logger = LoggerFactory
            .getLogger(ApplicationController.class);

    /**
     * Test页面
     *
     * @return String
     */
    @RequestMapping("/test")
    public String loginTimeOut() {
        return "test/test";
    }
}
```
JSP页面，文件路径为webapp/WEB-INF/jsp/test/test.jsp
```
<%--
  Created by IntelliJ IDEA.
  User: jacky
  Date: 17-10-3
  Time: 下午4:53
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>HttpMethodTest</title>
</head>
<body>
测试页面

</body>
</html>

```
这是我们运行所需要的类和jsp文件，但是有了这些也是没法程序也是没法运行的，需要添加部分配置才行
在主类@ImportResource({"classpath:applicationContext.xml"})用这个注解引入了配置文件，在这个配置文件中有一行
<context:property-placeholder location="classpath:application.properties,classpath:config.properties"/>
这各表示引入了配置文件application.properties，我们看一下这个配置文件中的内容
```
spring.freemarker.checkTemplateLocation: false
spring.mvc.view.prefix: /WEB-INF/jsp/
spring.mvc.view.suffix: .jsp
spring.mvc.locale: zh_CN
spring.http.encoding.charset: UTF-8
spring.http.encoding.enabled: true
spring.http.encoding.force: true
```
有了这些配置我们的应用就能启动起来了，使用mvn命令
>mvn spring-boot:run

就能测试一下了，也可以打包程war在tomcat中运行，命令是
>mvn package