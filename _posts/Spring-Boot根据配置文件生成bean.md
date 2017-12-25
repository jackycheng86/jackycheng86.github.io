#Spring-Boot根据配置文件生成Bean
在我们的项目开发过程中经常会遇到将一些固定的配置属性写入配置文件中，在系统运行时自动读取这些配置文件的信息，类似于.NET中web.config文件中定义的各种配置属性。
在java中利用spring-boot可以十分方便的实现类似的功能
>系统环境：spring-boot

我们在项目目录的resources目录添加一个config.properties文件，其中具有以下内容

```
#用于存储系统中可能涉及到的各种配置信息
scuvc.seeyon_key=SEEYONKEY_SSO
scuvc.session_key=scuvcsession
scuvc.sso_dn=xxx.com
scuvc.sso_client_id=123124
scuvc.sso_pwd=framework
scuvc.dn=localhost:8080
#权限类别
scuvc.role_xld=001
scuvc.role_jzg=002
scuvc.role_xs=003
```
定义好了配置文件后新建一个类SystemConfigScuvc
```
/**
 * Created by jacky on 16-9-9.
 */
@Configuration
@ConfigurationProperties(prefix = "scuvc")
public class  SystemConfigScuvc {
    private String seeyon_key;
    private String session_key;
    private String sso_client_id;
    private String sso_pwd;
    private String sso_dn;
    private String dn;
    private String role_xld;
    private String role_jzg;
    private String role_xs;
	//省略get set方法
```
@Configuration 声明一个配置类接受springIOC管理
@ConfigurationProperties(prefix = "scuvc") 声明从配置文件中读取以scuvc开头的属性
从配置文件和类文件能够看出来类的属性是与配置文件中scuvc开头的字符串一一对应的
完成了这些配置后就能在其他类中使用了
```
    @Autowired
    SystemConfigScuvc systemConfig;
```