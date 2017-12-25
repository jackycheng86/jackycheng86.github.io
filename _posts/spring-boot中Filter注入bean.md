#Spring-boot中Filter注入bean
   在spring中使用Filter的方式不用再多说，但是通常情况下我们在使用filter中都可能会注入部分配置的类或者部分具有特殊功能的业务类。  
   在这种情况下基于@WebFilter的配置方式就不再试用了，这时候需要采用人工配置的方式来进行配置。具体配置方式如下代码所示
   
   	@Configuration
	@ImportResource({ "classpath:applicationContext.xml"})
	public class WebConfig {

		@Bean
		public Filter characterEncodingFilter() {
			CharacterEncodingFilter c = new CharacterEncodingFilter();
			c.setEncoding("UTF-8");
			return c;
		}
    	@Bean
    	public FilterRegistrationBean contextFilterRegistrationBean() {
        	FilterRegistrationBean registrationBean = new FilterRegistrationBean();
        	registrationBean.setFilter(loginFilter());
        	registrationBean.addUrlPatterns("/*");
        	registrationBean.setName("LoginFilter");
        	registrationBean.setOrder(1);
        	return registrationBean;
    	}

    	@Bean
    	public Filter loginFilter() {
        	return new LoginFilter();
    	}
}
该代码用于声明一个手动配置注册的Filter，通过registrationBean配置的各种属性就能设置Filter的各种属性，包括拦截的url，名称，order（filter自定义执行顺序），这样就能在filter中注入bean了
```
public class ScuvcLoginFilter implements Filter {

    @Autowired
    SystemConfigScuvc systemConfig;
    @Autowired
    Mids mids;
    @Autowired
    Security security;
    @Autowired
    LoginService loginService;

```