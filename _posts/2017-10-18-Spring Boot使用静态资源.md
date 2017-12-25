---
title: Spring Boot使用静态资源
date: 2017-10-18
tags:
- Spring Boot
- Spring
- Java
categories: Spring
---

在我们使用Spring boot进行web开发时都会遇到引入各类js、css、html、image等各种静态资源文件，这时候我们就需要进行相应的配置来允许应用访问这些静态资源。
我们还是基于之前的采用thymeleaf 作为模板的 spring boot来开发
[Spring Boot使用Thymeleaf](http://blog.csdn.net/tyyytcj/article/details/78268310)

首先我们在resource目录下面建3各文件夹分别是：static、js、css，同时在static目录下建一个子目录html，在js下建一个子目录lib，目录结构如下
++----resources
+--------css
+--------js
+----------lib
+--------static
+----------html
在css目录添加了bootstrap.min.css，在js/lib添加了bootstrap.min.js和jquery.mini.js，在static下面添加了test.html，并在static/html下也添加了一个test.html，两个html内容不一致，同时在html页面引用相应的静态资源
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <link rel="stylesheet" href="/bootstrap.min.css">
    <script src="/lib/jquery.min.js"></script>
</head>
<body>
dasfdasfdasgasdg
</body>
</html>
```
这时候如果没有配置静态资源访问，在对静态内容进行访问的时候会出现404错误
这时候我们就需要添加一些配置来完成静态资源的访问，我们在application.properties文件中添加部分配置项
> spring.mvc.static-path-pattern=/**
> spring.resources.static-locations=classpath:/static/,classpath:/js/,classpath:/css/

第一行表示我们将对所有请求的静态资源进行过滤，经过实践证明这段配置可添加可不添加
第二行配置，可以看出来我们是将static、js、css目录标记为静态资源目录，多个目录之间通过逗号隔开如果你有更多的静态资源目录添加相应的路径即可
有了第二行配置，你在页面中引入的js、css就能被正确的识别而不再是报404错误了
当你在Controller中想跳转到静态html界面可以采用forward或者redirect的方式跳转例如：
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
        return "forward:/test.html";
    }
}

```
