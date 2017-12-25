#SpringBoot RequestMapping各方法的使用
在使用SpringBoot开发web应用时实际采用的是spring mvc来实现，现在采用REST风格的开发方式越来越多spring当然也支持这一开发模式。
>代码环境spring boot 页面配置为使用jsp的方式
>

rest模式我就不介绍了，主要是GET、POST、PUT、DELETE方法，spring mvc也提供了对应的实现方式
  @RequestMapping(method = RequestMethod.GET)
  @RequestMapping(method = RequestMethod.POST)
  @RequestMapping(method = RequestMethod.PUT)
  @RequestMapping(method = RequestMethod.DELETE)
当然也可以使用
@GetMapping
@PostMapping
@PutMapping
@DeleteMapping
这与上面的是一样的效果
下面我们来一个一个的实现
##GetMapping
```
    @GetMapping("/httpMethod")
    public String httpMethd(){
        return "test/test";
    }
```
使用这个代码我们定义了一个get方法，通过这个方法spring boot会自动跳转到test目录下的test.jsp页面当然你还可以在@GetMapping下面再加上@ResponseBody，这样返回的就是一个字符串而不是跳转页面了。
##PostMapping
test.jsp页面
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
<form action="/httpMethod" method="post">
    名称
    <input type="text" name="name">
    <br>
    密码
    <input type="text" name="pwd">
    <br>
    <input type="submit" name="提交">
</form>
</body>
</html>
```
controller类
```
    @PostMapping("/httpMethod")
    @ResponseBody
    public String httpMethod(@RequestParam String name,@RequestParam String pwd){
        System.out.println("sent name is "+name);
        System.out.println("sent pwd is "+pwd);
        return "success";
    }
```
这里表单定义了提交方法为post，这时提交表单在后台页面就能打印处理提交的信息，同时会向前台返回success字符串。
##PutMapping DeleteMapping
本质上浏览器端的form表单只支持GET和POST方法并不支持PUT和DELETE方法，但是spring已经解决了这个问题，从spring3.0开始定义了一个filter来支持对PUT和DELETE方法的解析，下面我们就来看看怎么处理。
首先我们要对jsp页面的表单做一个简单的修改
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
<form action="/upload/httpMethod" method="post">
    <input type="hidden" name="_method" value="PUT/DELETE">
    名称
    <input type="text" name="name">
    <br>
    密码
    <input type="text" name="pwd">
    <br>
    <input type="submit" name="提交">
</form>
</body>
</html>
```
对比一下就看到了吧，我们添加了一个隐藏的输入框名字叫_method，值为PUT或者是DELETE
controller代码
```
    @PutMapping("/httpMethod")
    @ResponseBody
    public String httpMethodPut(@RequestParam String name,@RequestParam String pwd){
        System.out.println("put sent name is "+name);
        System.out.println("put sent pwd is "+pwd);
        return "success";
    }

    @DeleteMapping("/httpMethod")
    @ResponseBody
    public String httpMethodDel(@RequestParam String name,@RequestParam String pwd){
        System.out.println("delete sent name is "+name);
        System.out.println("delete sent pwd is "+pwd);
        return "success";
    }
```
下面是重点了，引入filter，直接看代码
```
@Configuration
@ImportResource({"classpath:applicationContext.xml"})
public class ServletInitializer extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(MaterialApplication.class);
    }

    @Bean
    public FilterRegistrationBean httpMethodFilterRegistrationBean() {
        FilterRegistrationBean registrationBean = new FilterRegistrationBean();
        registrationBean.setFilter(httpMethodFilter());
        registrationBean.addUrlPatterns("/*");
        registrationBean.setName("HttpMethodFilter");
        registrationBean.setOrder(1);
        return registrationBean;
    }
    
    @Bean
    public Filter httpMethodFilter(){
        return new HiddenHttpMethodFilter();
    }
```
核心的filter就是HiddenHttpMethodFilter，这是spring自己定义来处理put和delete请求的filter，只需要配置这个filter过滤所有请求就行了，当然你还可以考虑过滤指定的路径请求。
配置好后就可以测试一下结果。