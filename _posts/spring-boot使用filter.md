#spring-boot使用Filter
Filter是J2EE提供的过滤器，在spring-boot如何使用呢？
需要用spring注解标签@WebFilter对filter类进行注解，具体代码如下
	
	@WebFilter(filterName = "ScuvcLoginFilter", urlPatterns = "/scuvc/*")
public class ScuvcLoginFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
    	 //do something...
    	 // 通过判断如果没有问题就继续执行，如果有问题可以通过response.sendRedirect()的方式进行跳转
    	 filterChain.doFilter(servletRequest, servletResponse);
    }

    @Override
    public void destroy() {
    	}
    }

	