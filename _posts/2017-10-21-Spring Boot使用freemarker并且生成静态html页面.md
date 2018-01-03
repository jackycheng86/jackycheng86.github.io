---
title: Spring Boot使用freemarker并且生成静态html页面
date: 2017-10-21
tags:
- Spring Boot
- Spring
- Java
categories: Spring
---
之前我介绍了在spring boot中使用thymeleaf模板，这次我会给大家介绍在spring boot中使用freemarker模板技术，同时利用freemarker生成静态html页面。生成静态html页面就能实现网站的静态化进而提高网站的访问速度以及提高SEO能力。
首先在pom.xml中添加依赖
##添加依赖
```
<dependency>
            <groupId>org.freemarker</groupId>
            <artifactId>freemarker</artifactId>
            <version>2.3.23</version>
        </dependency>
```
##application配置
在application.properties中添加freemarker的配置参数

```
##freemarker
spring.freemarker.cache=false
spring.freemarker.charset=UTF-8
spring.freemarker.check-template-location=true
spring.freemarker.content-type=text/html
spring.freemarker.enabled=true
spring.freemarker.suffix=.ftl
spring.freemarker.template-loader-path=classpath:/templates
```
##Controller和ftl模板
下一步我们就建一个基础Controller类和配套的ftl模板
Controller类
```
package com.hw.myp2c.common.controller;

import freemarker.template.Configuration;
import freemarker.template.Template;
import freemarker.template.TemplateException;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.annotation.Resource;
import java.io.*;
import java.net.URISyntaxException;
import java.util.HashMap;
import java.util.Map;

@Controller
@RequestMapping("")
public class MainController {

    @GetMapping
    public String main(Model model){
        String w="Welcome FreeMarker!";
        Map root = new HashMap();
        root.put("w",w);
        model.addAttribute("w","Welcome FreeMarker!");
        return "test";
    }
}

```
可以看到很简单，跟之前的thymelefa和jsp的没有区别。
freemarker模板
```
<html>
<head>
    <title>Welcome!</title>
    <link rel="stylesheet" href="/bootstrap.min.css">
    <script src="/lib/jquery.min.js"></script>
</head>
<body>
<h1>Hello ${w}!</h1>
</body>
</html>
```
这样之后我们就能完成了基础freemarker的使用，更多的使用参见freemarker官方网站，这里不做过多的描述。
这里我们已经完成了标准的freemarker集成，下面我们将介绍如何利用freemarker生成静态html页面，直接上代码，作为演示我们还是在Controller中完成，在实际应用中我们可以按照自己的实际需要进行封装。
```
package com.hw.myp2c.common.controller;

import freemarker.template.Configuration;
import freemarker.template.Template;
import freemarker.template.TemplateException;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.annotation.Resource;
import java.io.*;
import java.net.URISyntaxException;
import java.util.HashMap;
import java.util.Map;

@Controller
@RequestMapping("")
public class MainController {

    @Resource
    Configuration cfg;

    @GetMapping
    public String main(Model model){
        String w="Welcome FreeMarker!";
        Map root = new HashMap();
        root.put("w",w);
        freeMarkerContent(root);
        model.addAttribute("w","Welcome FreeMarker!");
        return "test";
    }

    private void freeMarkerContent(Map<String,Object> root){
        try {
            Template temp = cfg.getTemplate("test.ftl");
            //以classpath下面的static目录作为静态页面的存储目录，同时命名生成的静态html文件名称
            String path=this.getClass().getResource("/").toURI().getPath()+"static/test.html";
            Writer file = new FileWriter(new File(path.substring(path.indexOf("/"))));
            temp.process(root, file);
            file.flush();
            file.close();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (TemplateException e) {
            e.printStackTrace();
        } catch (URISyntaxException e) {
            e.printStackTrace();
        }
    }
}

```
利用freemarker生成静态页面我理解的流程是这样的
1.利用Configuration读取想生成静态页面的模板，这里是test.ftl
2.解析模板文件，并将模板中的${}!包含的参数替换成真实的数据
3.最终将读取了真实数据的模板生成相应的html文件，并写入指定目录
这样我们就完成了spring boot中使用freemarker模板，并且利用freemarker生成静态html文件