---
title: 向filter过滤器中的request对象添加额外的参数
date: 2017-11-05
tags:
- Java
categories: Java
---


有时候我们会遇到这么一些需求，在filter中获取一些参数进行处理，同时将处理好的参数重新添加到request对象中，这时候我们在filter中直接使用request.setAttribute()是无效的。我们怎么来解决这个问题呢，j2ee已经给我们提供了解决的办法，使用HttpServletRequestWrapper类来解决向request添加额外参数的功能。
>环境：spring boot

##新建一个类
新建一个类，这个类继承自HttpServletRequestWrapper
```
package com.hw.myp2c.common.filter;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;
import java.util.HashMap;
import java.util.Map;

/**
 * com.hw.myp2c.common.filter
 * Administrator
 * 2017/10/18
 **/
public class RequestParameterWrapper extends HttpServletRequestWrapper {

    private Map<String, String[]> params = new HashMap<String, String[]>();

    public RequestParameterWrapper(HttpServletRequest request) {
        super(request);
        //将现有parameter传递给params
        this.params.putAll(request.getParameterMap());
    }

    //重载构造函数
    public RequestParameterWrapper(HttpServletRequest request, Map<String, Object> extraParams) {
        super(request);
        addParameters(extraParams);
    }

	//添加额外参数
    public void addParameters(Map<String, Object> extraParams) {
        for (Map.Entry<String, Object> entry : extraParams.entrySet()) {
            addParameter(entry.getKey(), entry.getValue());
        }
    }

    public void addParameter(String name, Object value) {
        if (value != null) {
            if (value instanceof String[]) {
                params.put(name, (String[]) value);
            } else if (value instanceof String) {
                params.put(name, new String[]{(String) value});
            } else {
                params.put(name, new String[]{String.valueOf(value)});
            }
        }
    }
}
```
通过建立这个类我们就能完成向request对象添加自己想要的参数了。
##添加自定义参数
我们实现一个filter，这个filter实现一个功能，获取用户浏览器所访问的域名，并且将用户访问的域名构建为一个新的参数添加到request对象中去。
```
package com.hw.myp2c.common.filter;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

/**
 * com.hw.myp2c.common.filter
 * Administrator
 * 2017/10/18
 **/
public class RequestFilter implements Filter {

    private final Logger logger = LoggerFactory.getLogger(this.getClass());

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        // dn                                   ：域名
        //
        String dn = request.getServerName();
        Map<String, Object> extraParams = new HashMap<String, Object>();
        extraParams.put("dn", dn);
        //利用原始的request对象创建自己扩展的request对象并添加自定义参数
        RequestParameterWrapper requestParameterWrapper = new RequestParameterWrapper(request);
        requestParameterWrapper.addParameters(extraParams);
        filterChain.doFilter(requestParameterWrapper, servletResponse);
    }

    @Override
    public void destroy() {

    }
}

```
##spring boot配置filter
我们在完成了添加自定义参数之后就需要在spring boot配置我们定义filter了，具体代码如下
```
package com.hw.myp2c;

import com.hw.myp2c.common.filter.RequestFilter;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.boot.web.servlet.ServletComponentScan;
import org.springframework.boot.web.support.SpringBootServletInitializer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.servlet.Filter;

@Configuration
@ServletComponentScan
public class ServletInitializer extends SpringBootServletInitializer {

	@Override
	protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
		return application.sources(Myp2cMainApplication.class);
	}

    @Bean
    public FilterRegistrationBean contextFilterRegistrationBean() {
        FilterRegistrationBean registrationBean = new FilterRegistrationBean();
        registrationBean.setFilter(requestFilter());
        registrationBean.addUrlPatterns("/");
        registrationBean.setName("requestFilter");
        registrationBean.setOrder(1);
        return registrationBean;
    }

    @Bean
    public Filter requestFilter() {
        return new RequestFilter();
    }
}
```
这样我们就实现了向request对象中添加我们自定义好了的参数了。