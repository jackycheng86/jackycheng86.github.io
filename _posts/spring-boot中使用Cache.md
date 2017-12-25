#Spring-boot中使用Cache
随着应用系统的访问数量增加数据规模也越来越大，如何提高数据的检索响应尤其是对经常性访问的数据成为大家探索的方式。在这种前提下缓存技术就成为了大家的首要选择。
下面简单介绍在spring-boot使用cache的方法（简单使用）
在sping-boot中启用缓存具体如下，首先需要在pom.xml配置文件中启用cache配置

	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-cache</artifactId>
	</dependency>	
随后在应用启动类上启用注解

	@EnableCaching
随后在Repository注解类中启用相应的注解配置

	@Cacheable(value = "xwYrdwxxs", key = "#pageable.pageNumber")
    Page<XwYrdwxxEntity> findAll(Pageable pageable);
一个分页方法，在这个方法上注解缓存的名称，同时制定缓存的key，通过key来标注不同的缓存，该方法通过分页的页号作为缓存key，同时spring还提供了很多不同的key生成方式，详细资料参见

	http://docs.spring.io/spring/docs/4.3.7.RELEASE/spring-framework-reference/htmlsingle/#cache