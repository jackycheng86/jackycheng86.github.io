Spring-boot使用druid数据库连接池构建数据源
最初做对数据库操作的开发流程都是：打开一个连接，操作数据库，关闭连接，这种传统的方式简单易行但是当遇到并发访问急剧增大的时候这种方式带来的数据库开销就太大了会极大的影响应用系统的效率，这时候数据库连接池就出现了。目前市面上有着多种的数据库连接池而国人也推出了自己的连接池druid。
Druid连接池是由阿里巴巴开源的一套基于监控设计的数据库连接池，具体的介绍以及与其他数据库连接池的对比不再多做赘述可参见项目主网站https://github.com/alibaba/druid/wiki
在spring-boot中引入druid是一个比较简单的事情
>代码环境：maven+spring-boot+spring data jpa
>其中spring data jpa采用的是hibernate的实现方式自动集成了hibernate的库

##引入依赖
在pom.xml文件中引入依赖，当前最新版本为1.1.1
```
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.1</version>
        </dependency>
```
##连接池配置文件
首先修改应用配置文件application.properties，添加druid的数据库配置参数，从属性名字大概也能推断出来意思具体的参数描述可以参见项目主页
```
#XX数据库
spring.datasource.url = jdbc:oracle:thin:@192.168.56.101:1521:orcl
spring.datasource.username = aa
spring.datasource.password = bb
spring.datasource.driverClassName = oracle.jdbc.driver.OracleDriver
spring.datasource.initialSize = 1
spring.datasource.maxActive = 20
spring.datasource.minIdle = 1
spring.datasource.maxWait = 60000
spring.datasource.timeBetweenEvictionRunsMillis = 60000
spring.datasource.minEvictableIdleTimeMillis = 300000
spring.datasource.validationQuery = SELECT 1 FROM dual
spring.datasource.testWhileIdle = true
spring.datasource.testOnBorrow = false
spring.datasource.testOnReturn = false
spring.datasource.poolPreparedStatements = true
spring.datasource.maxPoolPreparedStatementPerConnectionSize = 20
spring.datasource.filters = stat,wall
```
##创建连接池属性类
在项目中创建与数据库连接池配置文件中各项属性一一对应的属性类，get，set方法和构造函数省略
整个配置采用注解的方式
@Configuration 注解为一个实体
@ConfigurationProperties(prefix = "spring.datasource") 加载application配置文件中以spring.datasource开始的各属性项
```
@Configuration
@ConfigurationProperties(prefix = "spring.datasource")
public class DruidDataSourceProperties {
	private String url;
	private String username;
	private String password;
	private String driverClassName;
	private int initialSize;
	private int maxActive;
	private int minIdle;
	private int maxWait;
	private long timeBetweenEvictionRunsMillis;
	private long minEvictableIdleTimeMillis;
	private String validationQuery;
	private boolean testWhileIdle;
	private boolean testOnBorrow;
	private boolean testOnReturn;
	private boolean poolPreparedStatements;
	private int maxPoolPreparedStatementPerConnectionSize;
	private String filters;
```
##数据源配置类
在数据源配置类中配置具体的数据源属性并通过@Configuration生成数据源
```
@Configuration
public class DruidDataSourceConfiguration {
    @Autowired
    private DruidDataSourceProperties properties;

    @Bean(name = "druidDataSource", initMethod = "init", destroyMethod = "close")
    @Qualifier("druidDataSource")
    public DataSource dataSource() throws Exception {
        DruidDataSource druidDataSource = new DruidDataSource();
        druidDataSource.setUrl(properties.getUrl());
        druidDataSource.setUsername(properties.getUsername());
        druidDataSource.setPassword(properties.getPassword());
        druidDataSource.setDriverClassName(properties.getDriverClassName());
        druidDataSource.setInitialSize(properties.getInitialSize());
        druidDataSource.setMaxActive(properties.getMaxActive());
        druidDataSource.setMinIdle(properties.getMinIdle());
        druidDataSource.setMaxWait(properties.getMaxWait());
        druidDataSource.setTimeBetweenEvictionRunsMillis(properties
                .getTimeBetweenEvictionRunsMillis());
        druidDataSource.setMinEvictableIdleTimeMillis(properties
                .getMinEvictableIdleTimeMillis());
        druidDataSource.setValidationQuery(properties.getValidationQuery());
        druidDataSource.setTestWhileIdle(properties.isTestWhileIdle());
        druidDataSource.setTestOnBorrow(properties.isTestOnBorrow());
        druidDataSource.setTestOnReturn(properties.isTestOnReturn());
        druidDataSource.setPoolPreparedStatements(properties
                .isPoolPreparedStatements());
        druidDataSource.setMaxPoolPreparedStatementPerConnectionSize(properties
                .getMaxPoolPreparedStatementPerConnectionSize());
        druidDataSource.setFilters(properties.getFilters());

        try {
            if (null != druidDataSource) {
                druidDataSource.setFilters("wall,stat");
                druidDataSource.setUseGlobalDataSourceStat(true);
                druidDataSource.init();
            }
        } catch (Exception e) {
            throw new RuntimeException(
                    "load datasource error, dbProperties is :", e);
        }
        return druidDataSource;
    }
}
```
##实体/事务/DAO配置
在完成数据库连接池配置后将配置具体的数据库事务、spring扫描Entity所在的包和DAO所在的包
```
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(entityManagerFactoryRef = "entityManagerFactory",
        transactionManagerRef = "transactionManager",
        basePackages = {"com.material.*.dao"})//指定需要扫描的dao所在包
public class RepositoryConfig {

    @Autowired
    private JpaProperties jpaProperties;

    @Autowired
    @Qualifier("druidDataSource")
    private DataSource druidDataSource;

    @Bean(name = "entityManager")
    @Primary
    public EntityManager entityManager(EntityManagerFactoryBuilder builder) {
        return entityManagerFactory(builder).getObject().createEntityManager();
    }

    /**
     * 指定需要扫描的实体包实现与数据库关联
     * @param builder
     * @return
     */
    @Bean(name = "entityManagerFactory")
    @Primary
    public LocalContainerEntityManagerFactoryBean entityManagerFactory(EntityManagerFactoryBuilder builder) {
        return builder
                .dataSource(druidDataSource)
                .properties(getVendorProperties(druidDataSource))
                .packages("com.material.*.entity")
                .persistenceUnit("persistenceUnitMaterial")
                .build();
    }

    /**
     * 通过jpaProperties指定hibernate数据库方言以及在控制台打印sql语句
     * @param dataSource
     * @return
     */
    private Map<String, String> getVendorProperties(DataSource dataSource) {
        Map<String, String> map = jpaProperties.getHibernateProperties(dataSource);
        map.put("hibernate.dialect", "org.hibernate.dialect.Oracle10gDialect");
        map.put("hibernate.show_sql", "true");
        return map;
    }

    /**
     * 创建事务管理
     * @param builder
     * @return
     */
    @Bean(name = "transactionManager")
    @Primary
    PlatformTransactionManager transactionManager(EntityManagerFactoryBuilder builder) {
        return new JpaTransactionManager(entityManagerFactory(builder).getObject());
    }
}

```
至此完成了spring-boot通过druid数据库连接创建数据源（单数据源）