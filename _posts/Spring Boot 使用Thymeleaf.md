#Spring Boot使用Thymeleaf
之前给大家介绍了在spring boot开发中使用jsp模板，但是jsp目前并不是spring boot推荐的做法，同时使用jsp还有一些限制而Thymeleaf则是spring boot官方推荐的模板引擎。
对于Thymeleaf我就不过多介绍网上资料很多，我们直接进入正题，在spring boot中使用Thymeleaf
环境
> spring boot 1.57 + Themeleaf 3.0.8

先上pom.xml<br>
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.hw</groupId>
	<artifactId>myp2c_main</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>war</packaging>

	<name>myp2c_main</name>
	<description>myp2c main page</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.7.RELEASE</version>
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
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
		</dependency>
		
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>

        <!-- https://mvnrepository.com/artifact/org.thymeleaf/thymeleaf -->
        <dependency>
            <groupId>org.thymeleaf</groupId>
            <artifactId>thymeleaf</artifactId>
            <version>3.0.8.RELEASE</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.thymeleaf/thymeleaf-spring4 -->
        <dependency>
            <groupId>org.thymeleaf</groupId>
            <artifactId>thymeleaf-spring4</artifactId>
            <version>3.0.8.RELEASE</version>
        </dependency>
        
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>23.2-jre</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
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
我在这里并没有引入spring boot 官方的thymeleaf包
```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
```
而是引入Thymeleaf的包
```
<!-- https://mvnrepository.com/artifact/org.thymeleaf/thymeleaf -->
        <dependency>
            <groupId>org.thymeleaf</groupId>
            <artifactId>thymeleaf</artifactId>
            <version>3.0.8.RELEASE</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.thymeleaf/thymeleaf-spring4 -->
        <dependency>
            <groupId>org.thymeleaf</groupId>
            <artifactId>thymeleaf-spring4</artifactId>
            <version>3.0.8.RELEASE</version>
        </dependency>
```
因为目前spring boot官方的依然引入的是thymeleaf 2.1.5这个版本的包而不是最新的3.0.8，而Thymeleaf 2 和Thymeleaf 3之间的差别比较大。

引入好包之后我们就可以开始进行配置了，首先是spring boot的主类
```
package com.hw.myp2c;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Myp2cMainApplication {

	public static void main(String[] args) {
		SpringApplication.run(Myp2cMainApplication.class, args);
	}
}


package com.hw.myp2c;

import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.support.SpringBootServletInitializer;

public class ServletInitializer extends SpringBootServletInitializer {

	@Override
	protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
		return application.sources(Myp2cMainApplication.class);
	}

}

```

很简单的控制类
```
package com.hw.myp2c.common.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("")
public class MainController {

    @GetMapping
    public String main(Model model){
        model.addAttribute("w","Welcome Thymeleaf!");
        return "main";
    }
}

```
然后是Thymeleaf的模板html文件，注意目录是在main下建立以下目录webapp/WEB-INF/templates，所有的html模板都放在templates下面供Thymeleaf引擎解析
```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">

<head>
    <title>Good Thymes Virtual Grocery</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>

<body>

<p th:text="'Hello, ' + ${w} + '!'" />

</body>

</html>
```
好了这是我们需要建立的各类文件，下面我们来做一些配置让spring boot识别并整合ThymeleafThymeleaf的配置有两种方法：
> application.properties配置
> javabean配置

两种方法各有优势，如果仅仅是想简单的使用Thymeleaf那就用第一种配置方法，如果想进行一些定制化的配置那就采用第二种方法
首先我们介绍第一种配置方法，很简单就是在application.properties文件中添加如下内容就行了

> spring.thymeleaf.cache=true
> spring.thymeleaf.check-template=true
> spring.thymeleaf.check-template-location=true
> spring.thymeleaf.enabled=true
> spring.thymeleaf.encoding=UTF-8
> spring.thymeleaf.mode=HTML
> spring.thymeleaf.prefix=/WEB-INF/templates/
> spring.thymeleaf.suffix=.html

这样我们就配置好了直接执行mvn spring-boot:run测试运行结果
>Hello, Welcome Thymeleaf!!

下面我们介绍第二种配置方法，基于javabean的配置，首先我们创建一个类SpringWebConfig.class，在这个类里面完成Thymeleaf的配置
```
package com.hw.myp2c;

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.support.ResourceBundleMessageSource;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.thymeleaf.spring4.SpringTemplateEngine;
import org.thymeleaf.spring4.templateresolver.SpringResourceTemplateResolver;
import org.thymeleaf.spring4.view.ThymeleafViewResolver;
import org.thymeleaf.templatemode.TemplateMode;

@Configuration
@EnableWebMvc
@ComponentScan
public class SpringWebConfig extends WebMvcConfigurerAdapter implements ApplicationContextAware{

    private ApplicationContext applicationContext;


    public SpringWebConfig() {
        super();
    }


    public void setApplicationContext(final ApplicationContext applicationContext)
            throws BeansException {
        this.applicationContext = applicationContext;
    }

    /* ******************************************************************* */
    /*  GENERAL CONFIGURATION ARTIFACTS                                    */
    /*  Static Resources, i18n Messages, Formatters (Conversion Service)   */
    /* ******************************************************************* */

    @Override
    public void addResourceHandlers(final ResourceHandlerRegistry registry) {
        super.addResourceHandlers(registry);
        registry.addResourceHandler("/images/**").addResourceLocations("/images/");
        registry.addResourceHandler("/css/**").addResourceLocations("/css/");
        registry.addResourceHandler("/js/**").addResourceLocations("/js/");
    }

    @Bean
    public ResourceBundleMessageSource messageSource() {
        ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
        messageSource.setBasename("Messages");
        return messageSource;
    }



    @Bean
    public SpringResourceTemplateResolver templateResolver(){
        // SpringResourceTemplateResolver automatically integrates with Spring's own
        // resource resolution infrastructure, which is highly recommended.
        SpringResourceTemplateResolver templateResolver = new SpringResourceTemplateResolver();
        templateResolver.setApplicationContext(this.applicationContext);
        templateResolver.setPrefix("/WEB-INF/templates/");
        templateResolver.setSuffix(".html");
        // HTML is the default value, added here for the sake of clarity.
        templateResolver.setTemplateMode(TemplateMode.HTML);
        // Template cache is true by default. Set to false if you want
        // page to be automatically updated when modified.
        templateResolver.setCacheable(true);
        return templateResolver;
    }

    @Bean
    public SpringTemplateEngine templateEngine(){
        // SpringTemplateEngine automatically applies SpringStandardDialect and
        // enables Spring's own MessageSource message resolution mechanisms.
        SpringTemplateEngine templateEngine = new SpringTemplateEngine();
        templateEngine.setTemplateResolver(templateResolver());
        // Enabling the SpringEL compiler with Spring 4.2.4 or newer can
        // speed up execution in most scenarios, but might be incompatible
        // with specific cases when expressions in one template are reused
        // across different data types, so this flag is "false" by default
        // for safer backwards compatibility.
        templateEngine.setEnableSpringELCompiler(false);
        return templateEngine;
    }

    @Bean
    public ThymeleafViewResolver viewResolver(){
        ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();
        viewResolver.setTemplateEngine(templateEngine());
        return viewResolver;
    }

}

```
分别配置了对静态资源（js/css/images）的处理、templateResolver（模板解析器）、templateEngine（模板引擎）、viewResolver（视图解析器）