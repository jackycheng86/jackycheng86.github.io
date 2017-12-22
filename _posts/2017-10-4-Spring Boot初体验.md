---
title: Spring Boot初体验
date: 2017-10-4 23:29:53
tags:
- Spring Boot
- Spring
- Java
categories: Spring
---

我记得在spring boot出来之前做框架整合是一件非常不愉快的事情，各种jar包版本冲突是非常糟糕的体验，同时那时候主要还是通过xml文件来进行配置，各种配置文件十分繁多，hibernate需要，spring需要，如果用struts还需要，总之就是一个及其麻烦的事情。
当我第一次见到spring boot的时候我们被惊艳到了，原来开发可以如此的流畅，真正的约定大于配置，各种方便的注解，包整合十分愉快。
这篇文章就是简单记录一下利用最简单的spring boot快速开发一个web小应用。
>开发环境spring boot + maven

开发环境自己配置（jdk+maven）
##spring boot初始化
首先你可以到spring官网去初始化项目https://start.spring.io/ ，在这里可以针对你自己的应用场景进行配置，同时选择整合你需要的各种包，最后再下载下来用你习惯的开发工具打开即可。
简单看看我自己的pom.xml文件
```
<groupId>com.material</groupId>
	<artifactId>data</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>war</packaging>

	<name>data</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.2.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>com.google.guava</groupId>
			<artifactId>guava</artifactId>
			<version>19.0</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-jasper</artifactId>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>com.oracle</groupId>
            <artifactId>ojdbc6</artifactId>
            <version>11.2.0.4</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.0.25</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
```
因为个人比较习惯jsp所以页面就没有采用其他的模板技术，因此我的application.properties文件配置如下
```
spring.freemarker.checkTemplateLocation: false
spring.mvc.view.prefix: /WEB-INF/jsp/
spring.mvc.view.suffix: .jsp
spring.mvc.locale: zh_CN
spring.http.encoding.charset: UTF-8
spring.http.encoding.enabled: true
spring.http.encoding.force: true
```
同时在src目录下添加一个webapp目录，再在webapp目录下添加 /WEB-INF/jsp/，与各配置文件目录对应

自己添加了applcationContext.xml文件用于提供对一些静态文件的访问
```
<context:property-placeholder location="classpath:application.properties,classpath:config.properties"/>
    <!-- 静态资源访问 -->
    <mvc:resources mapping="/js/**" location="/WEB-INF/js/"/>
    <mvc:resources mapping="/img/**" location="/WEB-INF/img/"/>
    <mvc:resources mapping="/css/**" location="/WEB-INF/css/"/>
    <mvc:resources mapping="/assets/**" location="/WEB-INF/assets/"/>
```

##各类配置
通过spring官网会默认帮你生成两个类
```
@EnableAutoConfiguration
@SpringBootApplication
@ComponentScan
@ServletComponentScan
@EnableCaching
public class MaterialApplication {

	public static void main(String[] args) {
		SpringApplication.run(MaterialApplication.class, args);
	}
}
```
```
@Configuration
@ImportResource({"classpath:applicationContext.xml"})
public class ServletInitializer extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(MaterialApplication.class);
    }

```
具体的注解我也就不详细描述了，百度很多也很详细
通过上面的一系列配置我们就搭建好了一个基本的基于spring boot的web应用

##定义controller和jsp
在完成配置的基础上我们会定义自己的Controller以及jsp页面
controller类
```
@Controller
public class HelloController {

    @RequestMapping("/hello")
    public String greeting(@RequestParam(value="name", required=false, defaultValue="World") String name, Model model) {
        model.addAttribute("name", name);
        return "hello";
    }

}
```
jsp页面
```
<%@ page language="java" contentType="text/html;charset=utf-8"%>
<!DOCTYPE html>

<%@ taglib prefix="spring" uri="http://www.springframework.org/tags"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>

<html>
<head>
    <title>Hello World</title>
</head>
<body>
Hello,${name}
</body>
</html>

```
下一步就是进入命令行，从命令行进入项目目录然后用maven启动项目
>mvn spring-boot:run
