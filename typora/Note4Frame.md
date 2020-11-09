---
SET PASSWORD FOR 'username'@'host' = PASSWORD('newpassword');




typora-root-url: Pic4Frame
---



Quartz：用来任务调度的框架

Stomp：就像HTTP在TCP套接字之上添加了请求-响应模型层一样，STOMP在WebSocket之上提供了一个基于帧的线路格式（frame-based wire format）层，用来定义消息的语义。

## WEB





## Sping

#### 配置规则

​	application文件读取顺序规则

``` tex
		1、config/application.properties（项目根目录中config目录下）

        2、config/application.yml

        3、application.properties（项目根目录下）

        4、application.yml

        5、resources/config/application.properties（项目resources目录中config目录下）

        6、resources/config/application.yml

        7、resources/application.properties（项目的resources目录下）

        8、resources/application.yml
        
        
# 加载多个配置文件,系统加载了application.properties application-database.properties  application-dev.properties 三个配置文件
        spring.profiles.active = dev,database
```

``` yaml
# Sequence of Scalars  简单数据列表
- Mark McGwire
- Sammy Sosa
- Ken Griffey

# Mapping Scalars to Scalars 简单的额数据键值对以及注释
hr: 65
avg: 0.278
rbi: 147

# Mapping Scalars to Sequences 简单数据列表键值对
american:
  - Boston Red Sox
  - Detroit Tigers
  - New York Yankees
national:
  - New York Mets
  - Chicago Cubs
  - Atlanta Braves
# Sequence of Mappings 键值对列表 
-
  name: Mark McGwire
  hr:   65
  avg:  0.278
-
  name: Sammy Sosa
  hr:   63
  avg:  0.288

# YAML 还支持流类型，用中括号括起来表示列表，用逗号分隔元素；用大括号括起来表示键值对，用逗号分隔元素。
# Sequence of Sequences 列表的列表 
- [name        , hr, avg  ]
- [Mark McGwire, 65, 0.278]
- [Sammy Sosa  , 63, 0.288]

#  Mapping of Mappings  键值对的键值对
Mark McGwire: {hr: 65, avg: 0.278}
Sammy Sosa: {
    hr: 63,
    avg: 0.288
  }
```



``` properties
Map类型的数据
	property.name.map.key_1 = value_1
	property.name.map.key_2 = value_2	
	对应的Map类型数据 Map<String, String> map ,等价 {"key_1":"value_1", "key_2":"value_2"}
List类型数据
	property.name.list[0] = value_1
	property.name.list[1] = value_2
所以List<Map<String, String>> list的数据类型书写：
	property.name.list[0].key_1 = value_1
	property.name.list[0].key_2 = value_2
    property.name.list[1].key_1 = value_1
	property.name.list[1].key_2 = value_2

```

``` java
//@PropertySource(value = "classpath:thread.yaml") // 从自定义的配置文件中读取数据
@ConfigurationProperties(prefix = "thread.pool") // 读取属性对象
public class ThreadPoolParamConfig {
    private int corePoolSize;

    private int maxPoolSize;
    
    private int keepAliveSeconds;
   
}

@Componement
public class Service{
    @Value("${drug.ip:47.92.186.194}") // 读取属性值，并给出默认值
    private String ip; 
}
```







``` java
public enum RequestMethod {
	GET, HEAD, POST, PUT, PATCH, DELETE, OPTIONS, TRACE
}
```

``` java
@Configuration
public class OAuth2ResourceServerConfig extends ResourceServerConfigurerAdapter{
//	web应用启动的顺序是：listener->filter->servlet，先初始化listener，然后再来就filter的初始化，
//	再接着才到我们的dispathServlet的初始化，
//	因此，当我们需要在filter里注入一个注解的bean时，就会注入失败，因为filter初始化时，注解的bean还没初始化，没法注入。
//	所以需要创建filter的时候 需要@Bean初始化一下

	@Bean
	public DoubleLoginFilter doubleLoginFilter(){
		return new DoubleLoginFilter(jwtTokenStore());
	}

	/**
	 * 对OAuth2资源服务器中的资源URL进行访问权限配置
	 */
	@Override
	public void configure(HttpSecurity http) throws Exception {
		// 增加操作日志记录的Filter。放在异常处理Filter和权限验证Filter之间
        http.antMatcher("/**").addFilterBefore(new OperationLoggingFilter(operationLogService),
        		FilterSecurityInterceptor.class);
        // 添加登录检测Fliter。放在异常处理之前，抛出在线检测异常，然后捕获显示。
		http.antMatcher("/**").addFilterBefore(doubleLoginFilter(),
				OperationLoggingFilter.class);
	}
}
```



### MVC

到HttpServlet还是原生的javax ，其他的都是Spring中的。MVC的核心DispatcherServlet地位如图。

![](/企业微信截图_15872941021799.png)

统计全网站访问量，用自定义Servlet，检验用户登录，过滤权限用Filter。对Handler的增强用Interceptor。用netty自定义一个服务

![](/企业微信截图_15872974382275.png)

filter  servlet intercept的构造与注册

``` java

@WebListener
public class IndexServletContextListener implements ServletContextListener {
    private static final Logger LOGGER = LoggerFactory.getLogger(IndexServletContextListener.class);
    public static final String INITIAL_CONTENT = "Content created in servlet Context";

    @Override
    public void contextInitialized(ServletContextEvent sce) {
        LOGGER.info("Start to initialize servlet context");
        ServletContext servletContext = sce.getServletContext();
        servletContext.setAttribute("content", INITIAL_CONTENT);
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        LOGGER.info("Destroy servlet context");
    }
}

public class IndexHttpServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws IOException {
        resp.getWriter().println(format("Created by %s", getInitParameter("createdBy")));
        resp.getWriter().println(format("Created on %s", getInitParameter("createdOn")));
        resp.getWriter().println(format("Servlet context param: %s",
                req.getServletContext().getAttribute("content")));
    }
}

public class FirstIndexFilter implements Filter {
    private static final Logger LOGGER = LoggerFactory.getLogger(FirstIndexFilter.class);

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        LOGGER.info("Register a new filter {}", filterConfig.getFilterName());
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        LOGGER.info("FirstIndexFilter pre filter the request");
        String filter = request.getParameter("filter1");
        if (isEmpty(filter)) {
            response.getWriter().println("Filtered by firstIndexFilter, " +
                    "please set request parameter \"filter1\"");
            return;
        }
        chain.doFilter(request, response);
        LOGGER.info("FirstIndexFilter post filter the response");
    }

    @Override
    public void destroy() {
        LOGGER.info("Destroy filter {}", getClass().getName());
    }
}

public class FirstIndexInterceptor implements HandlerInterceptor {
    private static final Logger LOGGER = LoggerFactory.getLogger(FirstIndexInterceptor.class);

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        LOGGER.info("FirstIndexInterceptor pre intercepted the request");
        String interceptor = request.getParameter("interceptor1");
        if (isEmpty(interceptor)) {
            response.getWriter().println("Filtered by FirstIndexFilter, " +
                    "please set request parameter \"interceptor1\"");
            return false;
        }
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        LOGGER.info("FirstIndexInterceptor post intercepted the response");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        LOGGER.info("FirstIndexInterceptor do something after request completed");
    }
}




@Configuration
public class WebServiceConfig implements WebMvcConfigurer {
    
     @Bean // 注册listener
	public ServletListenerRegistrationBean<MyListener> myServletListener() {
		ServletListenerRegistrationBean<MyListener> myListener = new 	ServletListenerRegistrationBean<MyListener>();
		myListener.setListener(new MyListener());
		return myListener;
    }

    @Bean // 注册servlet
    public ServletRegistrationBean dispatcherServlet(){
        return new ServletRegistrationBean(new CXFServlet(),"/service/*");//发布服务名称
    }
    
    
@Bean // 注册filter
	public FilterRegistrationBean myFilter() {
		FilterRegistrationBean myFilter = new FilterRegistrationBean();
		myFilter.addUrlPatterns("/*");
		myFilter.setFilter(new MyFilter());
		return myFilter;
	}
    
 @Override // 注册intercepter
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new FirstIndexInterceptor()).addPathPatterns("/index/**");
        registry.addInterceptor(new SecondIndexInterceptor()).addPathPatterns("/index/**");
        LOGGER.info("Register FirstIndexInterceptor and SecondIndexInterceptor onto InterceptorRegistry");
    }

}


```



### AOP

​	解决“业务”与“功能”的耦合，“功能”比如：登录验证，异常提示，有效期验证。将功能添加到“业务”中，

用AOP的方式，用的是切点的方式。可以定义调用的业务类，方法（excutaion），可以用注解，

``` java
//切点表达式
@Pointcut()
execution：用于匹配方法执行的连接点；
within：用于匹配指定类型内的方法执行；
this：用于匹配当前AOP代理对象类型的执行方法；注意是AOP代理对象的类型匹配，这样就可能包括引入接口也类型匹配； 

```



​	aop是使用代理模式，将指定被pointcut的类，用jdk动态代理或值cglib字节代理，将类代理增强

##### 代理模式简介

``` java
public interface Star{
    public void act();
}

public Actor implements Star{
    public void act(){
        System.out,println("actor is acting a drama");
    }
}

public Agent implements Star{
    private Star star;

	public Agent(Star star){
        this.star = star;
    }
    
    publc void setStar(Star star){
        this.star = star;
    }
    
    public void act(){
        before();
        star.act();
        after();
    }
    
    public Object before(){ 
        //do sth 
    }
    
    pubilc Object after(){
        // do sth
    }
}

// 实现代理人，代理明星的经纪工作
// 以上代理方式还有一个缺陷，就是必须实现统一个接口
//
```

​	

``` java
// JDK 中有方法实现类似的代理功能，但是还是没有摆脱，继承同样的接口形式
// 例子

public class JdkProxyHandle{
    private Object star;
    
    public JdkProxyHandle(Star star){
        super();
        this.star = star;
    }
    
    public Object getProxyInstance(){
        return Proxy.newProxyInstance(star.getClass().getClassLoader(),
                              star,getClass().getInterface(),
                              (proxy,methode,args)->{
                                  before();
                                  star.act();
                                  after();
                                  return object
                              })
    }
    
    Proxy.newProxyInstance
    
    
}
```





``` java
    @Pointcut("args()")
    private void cutArgs(){}

    @Pointcut("@annotation(com.example.demo.annotated.Constraints)")
    private void cutAnnotation(){}

    //匹配注解有AdminOnly注解的方法
    @Pointcut("@annotation(com.sample.security.AdminOnly)")
    public void demo1(){}

    //匹配标注有admin的类中的方法，要求RetentionPolicy级别为CLASS
    @Pointcut("@within(com.sample.annotation.admin)")
    public void demo2(){}

    //匹配传入参数的类标注有Repository注解的方法
    @Pointcut("args(org.springframework.stereotype.Repository)")
    public void demo3(){}

    //注解标注有Repository的类中的方法，要求RetentionPolicy级别为RUNTIME
    @Pointcut("target(org.springframework.stereotype.Repository)")
    public void demo4(){}



    @Before(value = "@annotation(com.example.demo.annotated.Constraints)",argNames = "")
    public void before(JoinPoint joinPoint){}

    @Around(value = "cutMethod()",argNames = "args")
    public Object doAround(ProceedingJoinPoint pjp,String[] args){
        return null;
    }

    @After(value = "",argNames = "")
    public void after(JoinPoint joinPoint){}

    @AfterReturning(pointcut = "execution()",returning = "",value = "",argNames = "")
    public void afterReturning(JoinPoint joinPoint){}

    @AfterThrowing(value = "",pointcut = "",throwing = "",argNames = "")
    public void afterThrowing(JoinPoint joinPoint){}
```



 ### IOC

​	spring框架加载上下文基本接口

![](/WebApplicationContext.jpg)



![](/企业微信截图_15764622843291.png)





controller 层的参数注解

同一个方法中@RequestBody是不能有两个的

``` java
@PutMapping(value = "/path")
public ResponseBody controllerMethod(@RequsetParam(value = "value") variable_0,
                                    @RequestHeader variable_1,
                                    @RequestBody variale_2,
                                    @ModelAttribute variable_3)
```

#### Security

客户端，授权服务端，资源端

authentication: 身份验证

authorization:授权

- 配置资源服务器

- 配置认证服务器

- 配置spring security

  
  
  ![](/企业微信截图_15750097997253.png)
  
  ``` xml
  <!-- 注意是starter,自动配置 -->
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-security</artifactId>
  </dependency>
  <!-- 不是starter,手动配置 -->
  <dependency>
      <groupId>org.springframework.security.oauth</groupId>
      <artifactId>spring-security-oauth2</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
</dependency>
  ```
  
  
  
##### 客户端



  ``` java 
  // 客户端 提供clientId 和clientSecret的token发送验证的Provider
  // 这里与WebSecurityConfigurerAdapter 使用没有什么关系
  @Configuration
  public class OAuth2ClientConfig extends WebSecurityConfigurerAdapter{
  	@Bean
      public ClientCredentialsAccessTokenProvider ClientCredentialsAccessTokenProvider() {
          ClientCredentialsAccessTokenProvider details = new ClientCredentialsAccessTokenProvider();
          return details;
      }
  
      @Bean
      public ResourceOwnerPasswordAccessTokenProvider ResourceOwnerPasswordAccessTokenProvider() {
          ResourceOwnerPasswordAccessTokenProvider passwordAccessTokenProvider = new ResourceOwnerPasswordAccessTokenProvider();
          return passwordAccessTokenProvider;
      }
      
      @Autowired
      RestTemplateBuilder builder;
  
      @Bean
      public RestTemplate restTemplate(){
          return builder.build();
      }
      
      /**
       * 所有接口不进行安全保护
       */
      @Override
      public void configure(WebSecurity web) {
          web.ignoring().antMatchers("/**");
      }
  }
  ```

##### 验证授权服务器端

``` java
// 授权服务端，都是对 ***Configurer的设定与自定义
@Configuration
@EnableAuthorizationServer
public class OAuth2AuthServerConfig extends AuthorizationServerConfigurerAdapter {
        /**
     * 配置OAuth2授权服务器的访问安全策略，所有人都可以方法，但是要求检查授权，授权模式访问设定，。
     */
    @Override
	public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        
	}

     /**
     * 配置OAuth2授权服务器的ClientDetailsService。
     * 
     * 在身份认证时，从指定的ClientDetailsService中获取与请求Client对应的详细信息。
     * OAuth2中有两个基本的ClientDetailsService实现类，分别是：
     * {@link InMemoryClientDetailsService}，内存中存取ClientDetails；
     * {@link JdbcClientDetailsService}，使用OAuth2内置表结构通过JDBC存取ClientDetails。
     * 
     * 可以指定自定义{@link ClientDetailsService}的实现类，以自定义方式存取ClientDetails。
     */
	@Override
	public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        // 对客户端的访问的处理，对访问信息自定义处理：
        clients.withClientDetails(customClientDetailsService());
        //其中的
        public class CustomClientDetailsService implements ClientDetailsService{
            @Override
			public ClientDetails loadClientByClientId(String clientId)
			throws ClientRegistrationException {
                // TODO 自己的处理方式。
            }
        }
	}

    // 设置端点的信息的配置
    /**
     * 配置OAuth2授权服务器端点信息。
     * 
     * OAuth2授权服务器内置以下几个端点：
     * /oauth/authorize : 授权码模式获取code，隐式模式获取token，参见{@link AuthorizationEndpoint}
     * /oauth/token : 授权码、用户密码、客户端三种模式获取token，参见{@link TokenEndpoint}
     * /oauth/check_token : 用于资源服务器调用此端口校验和解码token，http basic认证保护，参见{@link CheckTokenEndpoint} 
     * /oauth/token_key : 使用JWT时，通过此端点访问JWT签名的加密key，参见{@link TokenKeyEndpoint}
     * /oauth/confirm_access : 用于对访问进行确认，需要认证保护，否则报500。参见{@link WhitelabelApprovalEndpoint}
     * /oauth/error : 错误处理端点，默认实现类参见{@link WhitelabelErrorEndpoint}
     * 
     * 可通过调用endpoints的pathMapping方法，改变默认的端点URL。
     * 
     */
	@Override
	public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        // 其中的
        public final class AuthorizationServerEndpointsConfigurer{
            // 全是一些对象属性。对属性用set 方法添加定义
        }
	}
}

//WebSecurityConfigurerAdapter 有点类是总体管理的一个类，管理登录访问的形式与配置
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
     @Override
    public void configure(WebSecurity web) {
        web.ignoring().antMatchers("/oauth/check_token");
    }
    
    /**
     * 在此处配置认证管理器AuthenticationManager
     */
     @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {   
    	auth.authenticationProvider(customAuthenticationProvider());
    }
    
     /**
     * 在此处配置URI匹配认证规则，配置过滤器，配置跨域、csrf、登入/登出、成功/失败等
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
    }
}



```





##### 资源服务端

``` java
// 资源服务器
@Configuration
@EnableResourceServer
public class OAuth2ResourceServerConfig extends ResourceServerConfigurerAdapter{
    @Override
	public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
	}

	@Override
	public void configure(HttpSecurity http) throws Exception {
		http.authorizeRequests().anyRequest().authenticated();
	}
}
```







#### Spring注解说明

大家熟悉一下三个扫描装配bean的Spring 注解

```css
@ServletComponentScan(basePackages="com.shine")
```

Servlet、Filter、Listener 可以直接通过 @WebServlet、@WebFilter、@WebListener 注解自动注册，无需其他代码,如上,com.shine包层级一下的的都会被扫描装配

```css
@MapperScan(basePackages="com.shine")
```

该注解来扫描注册mybatis数据库接口类，其中basePackages属性表明接口类所在的包,**扫描的包的接口都会被装配到Spring beansFactory中**

```css
@ComponentScan(basePackages="com.shine")
```

该注解主要就是定义扫描的路径从中找出标识了需要装配的类自动装配到spring的bean容器中,意思就是@Component的封装注解也是会被扫描装配,例如@Repository @Service @Controller

``` css
@SpringBootApplication(scanBasePackages = {"com.hnty.security", "com.hnty.cloud.sys", "com.hnty.cloud.utilization"})
```

其实调用的是@ComponentScan

``` java
@Componemt(name = "selfDefineName")
public Class SelfDefineClass{}

@Configuration
class {
	@Bean
    public SystemDefineClass methondName(){
        // 用来注解方法，调用的时候和方法名相同，
        // 通常用来构建依赖包中的类，交给spring管理
    }
}

// ------------------调用-------------------------------
@AutoWire(默认是用类名称)
@Qualify(设定的名称) // default by type 


@Resource(name = "设定的名称") // default by name


```

Model中常用注解

``` java
@Data // 简化get，set
@Access(chain = "true") // 链式调用
@Bulider // 简化建造模式
@ApiModel(description = "swagger 描述")
class ModelDataItem{
    @ApiProperty("")
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss"， timezone = "GMT+8")// 格式化时间输出
    @JsonProperty(value = "time") // 传递给前端的参数改名
    LocalDateTime localDateTime;
}
```

Controller中常用注解

``` java

```

 ###  Spring的工具类





## Mybaits

#### 与spring的配置说明

``` java
@MapperScan(value = {"com.hnty.cloud.utilization"})
//给mybaits加载报下的组件类，不同于其他的加载方式，在interface的继承类中，当仅有一个是，自动注入唯一的一个实现类。@MapperScan无法区分interface 和其实现类 interfaceImp，所以定义需要限定其范围为，避免都被扫描到。
当有多个类实现接口时，用@Qualified定义名字区别
扫描接口用的

```

``` yaml
mybatis-plus:
  config-location: classpath:mybatis-config.xml
  # sql返回值的限定范围，与MapperScan区别应该是不会加载到spring的bean仓库中
  type-aliases-package: [com.hnty.modules,com.hnty.cloud.utilization,com.hnty.cloud.sys.service]
  # xml文件位置
  mapper-locations: [classpath:mappings/**/*.xml,classpath:datan.model/*.xml,classpath*:/com/hnty/cloud/sys/mapper/**/*.xml]
  global-config:
```





#### 操作返回值

``` xml
insert：插入n条记录，返回影响行数n。（n>=1，n为0时实际为插入失败）

update：更新n条记录，返回影响行数n。（n>=0）

delete： 删除n条记录，返回影响行数n。（n>=0）
```

### 插入返回主键

``` xml
//注意mapper这里如果添加了@Param,则无法返回主键
int insertAnalyzer(@Param("analyzer")Analyzer analyzer);
<insert id="insertAnalyzer" parameterType="Analyzer" useGeneratedKeys="true" keyProperty="id" keyColumn="id" >
    <!--<selectKey resultType="java.lang.Integer" order="AFTER" keyProperty="id">-->
        <!--SELECT LAST_INSERT_ID() AS id-->
    <!--</selectKey>-->
        INSERT INTO
          qrtz_alart_analyzer_test(
            analyzer_id,
            analyzer_type,
            analyzer_expression
          )
        VALUES (
          #{analyzerId},
          #{analyzerType},
          #{analyzerExpression}
        )
    </insert>
```

#### foreach 遍历

``` xml
<insert id ="INSERT-CODE-BATCH" parameterType="java.util.List" >
      insert into redeem_code
      (bach_id, code, type, facevalue,create_user,create_time)
      values
       <foreach collection ="list" item="reddemCode" index= "index" separator =",">
           (
           #{reddemCode.batchId}, #{reddemCode.code},
           #{reddemCode.type},
           #{reddemCode.facevalue},
           #{reddemCode.createUser}, #{reddemCode.createTime}
           )
       </foreach >
</insert >

<update id="WHOLESALE-UPDATE-REBATE-CONFIG-BY-PRODUCTCODE" >
	UPDATE 
		rebate_config
	SET
		rebate_type = 'BY_RATE',rebate_config_value = #rebateConfigValue#
	WHERE
		<iterate  property="list" conjunction="or" open="(" close=")">
		  product_code =#list[]#
		</iterate>
</update>

```

 

删除

``` xml
// 批量删除用 where in + <foreach>
    
<delete id="batchDelete" parameterType="java.util.List">
        DELETE from 
        list
        where id in
        <foreach collection="idList" item="id" open="(" separator="," close=")">
            #{id}
        </foreach>
    </delete>

    
  // 关联删除用 LEFT JOIN 
     <delete id="deleteSite">
        DELETE MS,MSR
        FROM monitoring_site MS
          LEFT JOIN monitoring_site_range MSR ON MS.site_id = MSR.site_id
        WHERE
          MS.site_id = #{siteId}
    </delete>
```



#### mybits-plus使用：





#### 不同数据源的连接

``` xml
// 相同数据服务下，不同的数据库联表查询时，带上非配置数据库的库名
spring.datasource.url = jdbc:mysql://ip:port/database_name_1
同样的ip：port下有database_name_2,database_name_3,
那么 xml中

select  
	a.meta_a,
	b.meta_b,
	c.meta_c
FROM 
	table_A a 
		LEFT JOIN database_name_2.table_B b ON ...
		LEFT JOIN database_name_3.table_C c ON ...


/// 不同的数据连接那么
spring.datasource.source-a.url = jdbc:mysql://ip_a:port_a/database
spring.datasource.source-b.url = jdbc:mysql://ip_b:port_b/database
spring.datasource.source-c.url = jdbc:sqlserver://ip_c:port_c/database

那么要配置不同的@confingure
并且设置扫描的xml和java功能包。所以在工程结构目录上就要区分不同数据源的功能结构。

```

#### 一个标签多条语句一起执行

``` properties
dbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=utf-8&allowMultiQueries=true
```

``` xml
 <delete id="deleteByPrimaryKey" parameterType="java.lang.Integer" >
2     delete from music_favorite where id = #{id,jdbcType=INTEGER};
3     delete from music_favorite_song where f_id = #{id,jdbcType=INTEGER};
4   </delete>
```



## MySQL

``` bash
# mysql配置文件
my.ini
# 会创建一个data文件夹，里面.err文件包含cmd登陆第一次密码
mysqld --initialize
# 启动一个默认名称为mysql的数据服务
mysqld install 
# 启动服务进程
net start mysql
# 关闭服务进程
net stop mysql

```







#### 数据库的时间字段更新与自动更新

``` mysql
// 数据库的时间字段更新与自动更新
DROP TABLE IF EXISTS `mytesttable`;  
CREATE TABLE `mytesttable` (  
  `id` int(11) NOT NULL,  
  `name` varchar(255) DEFAULT NULL,  
  `createtime` datetime DEFAULT CURRENT_TIMESTAMP,  
  `updatetime` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,  
  PRIMARY KEY (`id`)  
) ENGINE=InnoDB DEFAULT CHARSET=gbk; 
```

#### 数据库删除，自增重新开始

``` sql
用 alter table table_name AUTO_INCREMENT = n 命令来重设自增的起始值
```



 #### 去重插入

``` sql
nsert tbl_warning(
        coluqOrganizationID,
        coluqSchemeID,
        coluqSchemeStationPlanID,
        colWarningItemID,
        colWarningTime,
        colWarningContent
        ) SELECT
        #{warningEntity.coluqOrganizationID},
        #{warningEntity.coluqSchemeID},
        #{warningEntity.coluqSchemeStationPlanID},
        #{warningEntity.colWarningItemID},
        #{warningEntity.colWarningTime},
        #{warningEntity.colWarningContent}
        FROM DUAL
        WHERE
        not exists(
            SELECT 1
            FROM tbl_warning
            WHERE
            (
            coluqOrganizationID = #{warningEntity.coluqOrganizationID} or coluqOrganizationID is null
            )
            AND (
            coluqSchemeID = #{warningEntity.coluqSchemeID} or coluqSchemeID is null
            )
            AND (
            coluqSchemeStationPlanID = #{warningEntity.coluqSchemeStationPlanID} or coluqSchemeStationPlanID is null
            )
            AND (
            colWarningItemID = #{warningEntity.colWarningItemID} or colWarningItemID is null
            )
            AND (
            colWarningContent = #{warningEntity.colWarningContent} or colWarningContent is null
            )
        )
```



#### 函数变量

``` mysql
set @item.reportTimeListStr = "2019-01-20,2019-04-10";
     UPDATE monitor_site.monitoring_site_instrument msi
      SET msi.report_time_list =(
          case
          when msi.report_time_list IS NULL
            then  @item.reportTimeListStr
          when msi.report_time_list IS NOT NULL
            then  CONCAT(msi.report_time_list,",",@item.reportTimeListStr)
          end)
      WHERE
        msi.bind_id = 116;
```

``` mysql
mysql连接空字符串会造成空返回；
CONCAT(IFNULL(express_1, express_2),column_1,column_2)
```



#### MYSQL函数与存储过程

``` sql
<!--  -->





```

### MYSQL 递归查询

``` sql

WITH RECURSIVE cte_name(columns * ) AS (
    initial_query  -- anchor member
    UNION ALL
    recursive_query -- recursive member that references to the CTE name
)SELECT * FROM cte_name;
-- 递归CTE由三个主要部分组成：

-- 形成CTE结构的基本结果集的初始查询(initial_query)，初始查询部分被称为锚成员。
递归查询部分是引用CTE名称的查询，因此称为递归成员。递归成员由一个UNION ALL或UNION DISTINCT运算符与锚成员相连。
终止条件是当递归成员没有返回任何行时，确保递归停止。
递归CTE的执行顺序如下：

首先，将成员分为两个：锚点和递归成员。
接下来，执行锚成员形成基本结果集(R0)，并使用该基本结果集进行下一次迭代。
然后，将Ri结果集作为输入执行递归成员，并将Ri+1作为输出。
之后，重复第三步，直到递归成员返回一个空结果集，换句话说，满足终止条件。
最后，使用UNION ALL运算符将结果集从R0到Rn组合。
注意：递归成员只能在其子句中引用CTE名称，而不是引用任何子查询



create table item_tree(
  sn_id int auto_increment comment '行号' primary key,
  moniker varchar(20) null comment '名称',
	context varchar(20) NULL COMMENT '内容',
	item_id int(11) null comment '归属id',
  parent_id int(11) null comment '父id',
	item_level int(11) NULL COMMENT '等级'
)comment '层级递归';
-- # 插入测试数据
INSERT INTO item_tree (item_id, parent_id,item_level) VALUES (1001, 101 ,4);
INSERT INTO item_tree (item_id, parent_id,item_level) VALUES (1002, 101 ,4);
INSERT INTO item_tree (item_id, parent_id,item_level) VALUES (101, 11,3);
INSERT INTO item_tree (item_id, parent_id,item_level) VALUES (11, 2 ,2);
INSERT INTO item_tree (item_id, parent_id,item_level) VALUES (2, null ,1);


-- 自下往上搜索
WITH RECURSIVE
child(item_id, parent_id, item_level) AS(
  SELECT item_id, parent_id, item_level
  FROM item_tree WHERE item_id = 2 -- 顶层节点终止条件
  UNION ALL
  SELECT A.item_id, A.parent_id,  A.item_level 
  FROM item_tree A JOIN child B ON A.parent_id = B.item_id
)
SELECT * FROM child;


--- 自上向下递归 给出收索路径的节点值
WITH RECURSIVE
parent(item_id, parent_id, item_level) AS(
  SELECT item_id, parent_id,item_level
  FROM item_tree WHERE item_id = 1002  -- 底层节点终止条件
  UNION ALL
  SELECT A.item_id, A.parent_id, A.item_level 
  FROM item_tree A JOIN parent B ON A.item_id = B.parent_id
)SELECT  * FROM parent
```





#### 选择表达式

![](/企业微信截图_15750097997253.png

``` mysql
IF(expr1, expr2, expr3) 等价 expr1?expr2:expr3

IFNULL(expr1,expr2) 等价 expr1!=null?expr1:expr2

SELECT
	CASE column_name 
		WHEN 1 THEN '男'  
		WHEN 2 THEN '女'
		ELSE '???'
     END
     
     
IF search_condition THEN 
    statement_list  
[ELSEIF search_condition THEN]  
    statement_list ...  
[ELSE 
    statement_list]  
END IF

create procedure dbname.proc_getGrade  
(stu_no varchar(20),cour_no varchar(10))  
BEGIN 
declare stu_grade float;  
select grade into stu_grade from grade where student_no=stu_no and course_no=cour_no;  
if stu_grade>=90 then 
    select stu_grade,'A';  
elseif stu_grade<90 and stu_grade>=80 then 
    select stu_grade,'B';  
elseif stu_grade<80 and stu_grade>=70 then 
    select stu_grade,'C';  
elseif stu_grade70 and stu_grade>=60 then 
    select stu_grade,'D';  
else 
    select stu_grade,'E';  
end if;  
END
```

#### 聚合分组





### 系统默认库与表

库 ： information_schema ，performance_schema

管理整个服务的表与库

``` mysql
<!-- 获取 服务下所有的表 -->
select table_name ,create_time , engine, table_collation, table_comment from information_schema.tables 
where table_schema = (select database())  
order by create_time DESC;


information_schema.COLUMNS

select column_name, is_nullable, data_type, column_comment, column_key, extra from information_schema.columns 
where table_name = ${tableName} and table_schema = (select database()) order by ordinal_position 
```

### 用户管理

``` sql

CREATE USER 'username'@'host' IDENTIFIED BY 'password';
GRANT privileges ON databasename.tablename TO 'username'@'host'
DROP USER username;
SET PASSWORD FOR 'username'@'host' = PASSWORD('newpassword');
```

​	版本 8.0.21 单节点 ： mysql-8.0.21-winx64.zip

配置文件my.ini内容 

```ini
[mysqld]
# 设置3306端口
port=3306
# 设置mysql的安装目录
basedir=E:\Program Files\MySQL
# 设置mysql数据库的数据的存放目录
datadir=E:\Program Files\MySQL\data
# 允许最大连接数
max_connections=200
# 允许连接失败的次数。
max_connect_errors=10
# 服务端使用的字符集默认为utf8mb4
character-set-server=utf8mb4
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
# 默认使用“mysql_native_password”插件认证
#mysql_native_password
default_authentication_plugin=mysql_native_password
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8mb4
[client]
# 设置mysql客户端连接服务端时默认使用的端口
port=3306
default-character-set=utf8mb4
```



```ini
# mysql 启动命令：
 # 会创建一个data文件夹，里面.err文件包含cmd登陆第一次密码
mysqld --initialize
# 启动一个默认名称为mysql的数据服务
mysqld install 
 # 将数据服务日志输出前台显示
mysqld --consolmy
一般启动服务： net start mysql
# 修改默认密码（myslq 8 以上）
alter user 'root'@'localhost' identified by '${newPwd}'
#重启
Windows

　　1.点击“开始”->“运行”(快捷键Win+R)。

　　2.启动：输入 net stop mysql

　　3.停止：输入 net start mysql
# 删除服务，重启服务
sc query mysql
sc delete mysql
mysqld -install
任务管理器中->"服务项"->启动MySQL
```

```sql
授权一个新用户用于开发访问: 
GRANT ALL ON ${databaseName}.${tables} TO '${userName}'@'${host}';
flush privileges;

CREATE USER 'username'@'host' IDENTIFIED BY 'password';
GRANT privileges ON databasename.tablename TO 'username'@'host'

create user 'pacs_united'@'%' IDENTIFIED BY 'psw_pacs_united';
GRANT SELECT ON yhh_bss.t_pacs_united TO 'pacs_united'@'%';
flush privileges;



```

##### 数据备份与恢复

逻辑备份 和 物理备份：

逻辑备份：输出为MySQL可以解析的格式中，XX.sql XX.csv ，就是navicat的导出功能

物理备份：直接复制原始文件 原始的  .idb 文件， binlog 文件

```sh
# 虚拟shell版的备份脚本 ，然后改装为 .bat 使用的windows脚本
mysqldump 全量备份（热备份，不用停mysql）会自动的flush binlog（全量之后的增量备份）
但是binlog不知道什么时候刷出来，所以定时flush binlog 刷出来，保存binlog文本备份
# 保存数据
mysqldump  > file.sql 
mysql flush logs;

# 还原数据
	#全量还原
	> mysql -uroot -phntyhnty < file.sql
	# 增量还原 如果是shell直接 $ mysqlbinlog mysql-bin.0000xx | mysql -u${usr} -p${pwd} ${db_name}
	> mysqlbinlog XXX-bin.0000XX > XXX-bin.sql
	> mysql -uroot -phntyhnty < XXX-bin.sql 
	> mysqlbinlog -v XXX-bin.0000XX > XXX.txt
		#从 XXX.txt 文件中找到误操作，以及对应的开始,结束 编号 
			mysqlbinlog XXX-bin.0000XX --stop-position 开始编号 >  XXX-bin-start.sql
			mysqlbinlog XXX-bin.0000XX --stop-position 结束编号 >  XXX-bin-end.sql
		# 然后依次执行两个SQL，一定要按顺序执行
		# 也可以用sql语句直接查看binlog的文件，然后找到误操作事件的开始与结束
		
```

win10下的脚本

全量备份bat脚本

```bat
rem 按时间分开保存

rem @echo off
set THISDATE=%DATE:~0,4%%DATE:~5,2%%DATE:~8,2%
set DB=yhh_bss
set fileName=%DB%%THISDATE%weekly_backup
mysqldump -uroot -phntyhnty -B --databases yhh_bss  -F -R --master-data=2 --events --single-transaction > ./%fileName%.sql

rem 覆盖保存

rem @echo off
rem set DB=yhh_bss
rem set fileName=%DB%weekly_backup
rem mysqldump -uroot -phntyhnty -B --databases yhh_bss  -F -R --master-data=2 --events --single-transaction > ./%fileName%.sql
rem pause


```



增量备份脚本 一个可以批量执行SQL或SQL文本的脚本样例（文本名处理太麻烦，不搞了）

```bat
@echo off
for %%i in (./daily_backup.sql) do (
    echo execute %%i
    mysql -uroot -phntyhnty < %%i
)
rem 应当在做一个追加复制脚本，把flush出来的日志文件保存到本地或者其他服务器，并记录时间到文件名中
echo successd
```



daily_backup.sql 内容

```sql
flush logs;
```

应当在做一个追加复制脚本，

剩下的就是用win定时任务设定执行脚本时间



```ini
log-bin = /xxx/xxx/mysql_bin #binlog日志文件,以mysql_bin开头，六个数字结尾的文件：mysql_bin.000001，并且会将文件存储在相应的xxx/xxx路径下，如果只配置mysql_bin的话默认在C:\ProgramData\MySQL\MySQL Server 5.7\Data下；
binlog_format = ROW #binlog日志格式，默认为STATEMENT：每一条SQL语句都会被记录；ROW：仅记录哪条数据被修改并且修改成什么样子，是binlog开启并且能恢复数据的关键；
expire_logs_days= 7 #binlog过期清理时间；
max_binlog_size = 100m #binlog每个日志文件大小；
binlog_cache_size = 4m #binlog缓存大小；
max_binlog_cache_size = 512m #最大binlog缓存大小
```

##### mysql主从配置

``` ini
# 主节点 XXX.21.XX.184
# 从节点 XXX.21.XX.186

# 会相互授权访问权限与地位 ， 所以要建立好关键的 'user'@'host'
# 那么主节点的 JavaApp使用用户是 'dev'@'%'
# 从节点的用户是 'dev'@'%'fl
# 那么尝试：主节点
>>$mysql > create user 'replcate'@'172.21.44.186' identified by 'hntyhnty';
>>$mysql > alter user 'replcate'@'172.21.44.186' identified with mysql_native_password by 'hntyhnty';
>>$mysql> grant replcation slave on yhh_bss.* to 'replcate'@'172.21.44.186';
>>$mysql> flush privileges;
# 以上尝试不好，带IP的host 换为 %
>>$mysql > create user 'replcate'@'%' identified by 'XXXX';
>>$mysql > alter user 'replcate'@'%' identified with mysql_native_password by 'XXXX';
# 这里是grant replcation slave on 是全局命令，不能指定特定的数据库
>>$mysql> grant replcation slave on *.* to 'replcate'@'XXX.21.XX.186'; 


# 通过执行sql 
>>$mysql > show master status;
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000003 |     1173 | localtest    |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
可以指找到执行MASTER_LOG_FILE 和 MASTER_LOG_POS的值
##### ==============主节点配置===================
# config for setting master-slave doc in delay db test
#主数据库端ID号
server_id=1           
#开启二进制日志                  
log-bin = mysql-bin    
#需要复制的数据库名，如果复制多个数据库，重复设置这个选项即可                  
binlog-do-db=XXX
binlog_format=ROW      
#将从服务器从主服务器收到的更新记入到从服务器自己的二进制日志文件中                 
log-slave-updates=1                        
#控制binlog的写入频率。每执行多少次事务写入一次(这个参数性能消耗很大，但可减小MySQL崩溃造成的损失) 
sync_binlog=100                    
#这个参数一般用在主主同步中，用来错开自增值, 防止键值冲突
#auto_increment_offset=1           
#这个参数一般用在主主同步中，用来错开自增值, 防止键值冲突
#auto_increment_increment=1            
#二进制日志自动删除的天数，默认值为0,表示“没有自动删除”，启动时和二进制日志循环时可能删除  
# expire_logs_days=7                    
#将函数复制到slave  
log_bin_trust_function_creators=1

##### ==============从节点配置===================
[mysqld]
server_id = 2
log-bin = mysql-bin
log-slave-updates=1
sync_binlog = 0
#log buffer将每秒一次地写入log file中，并且log file的flush(刷到磁盘)操作同时进行。该模式下在事务提交的时候，不会主动触发写入磁盘的操作
innodb_flush_log_at_trx_commit = 0        
#指定slave要复制哪个库
replicate-do-db = XXX         
#MySQL主从复制的时候，当Master和Slave之间的网络中断，但是Master和Slave无法察觉的情况下（比如防火墙或者路由问题）。Slave会等待slave_net_timeout设置的秒数后，才能认为网络出现故障，然后才会重连并且追赶这段时间主库的数据
slave-net-timeout = 60                    
log_bin_trust_function_creators = 1
read-only=1

# 修改配置文件重启mysql

# 执行mysql
从节点配置文件
>>$mysql> CHAGE MASTER TO 
	> MASTER_HOST='XXX.21.XXX.184',
	> MASTER_USER='XXX',
	> MASTER_LOG_FILE='mysql-bin.000000X',
	> MASTER_LOG_POS=525;
>>$mysql > SALVE START;
>>$mysql > show slave state\G;

## 以下俩个字段都是yes那么是成功的，其他的No, connecting 都是失败
SLAVE_IO_Running:yes 
SLAVE_SQL_Running:yes


## 剩下的命令就是将 从库改为延时：

>>$mysql > stop slave;
# 按秒计算4小时，可以试试直接执行这一句
>>$mysql > change master to master_delay= 144000
>>$mysql> start slave;
# 查看结果
mysql> show slave status/G;
# SQL_DELAY:144000
# SQL_Remaining_Delay:120023

# 停止延时
mysql> stop slave;
mysql> CHANGE MASTER TO MASTER_DELAY = 0;
mysql> start slave;
```

##### mysql主从恢复



## 查看状态

## MAVEN

[](https://www.runoob.com/maven/maven-tutorial.html) // meaven 的基本配置说明

``` shell
mvn install:install-file -Dfile=hnty-cloud-common-data-0.5.0-SNAPSHOT.jar -DgroupId=com.hnty.cloud -DartifactId=hnty-cloud-common-data -Dversion=0.5.0-SNAPSHOT -Dpackaging=jar

mvn install:install-file -Dfile=rules-core-1.1.0.jar -DgroupId=com.hntycloud -DartifactId=rules-core -Dversion=1.1.0 -Dpackaging=jar

mvn install:install-file -Dfile=redis-distributed-lock-starter-1.2.0.jar -DgroupId=com.hntycloud -DartifactId=redis-distributed-lock-starter -Dversion=1.2.0 -Dpackaging=jar

mvn install:install-file -Dfile=hnty-cloud-common-security-0.5.0-SNAPSHOT.jar -DgroupId=com.hnty.cloud -DartifactId=hnty-cloud-common-security -Dversion=0.5.0-SNAPSHOT -Dpackaging=jar

mvn install:install-file -Dfile=hnty-cloud-common-sys-0.5.0-SNAPSHOT.jar -DgroupId=com.hnty.cloud -DartifactId=hnty-cloud-common-sys -Dversion=0.5.0-SNAPSHOT -Dpackaging=jar

mvn install:install-file -Dfile=hnty-cloud-common-security-0.5.0-SNAPSHOT.jar -DgroupId=com.hnty.cloud -DartifactId=hnty-cloud-common-security -Dversion=0.5.0-SNAPSHOT -Dpackaging=jar

mvn install:install-file -Dfile=eladmin-common-2.5.jar -DgroupId=me.zhengjie -DartifactId=eladmin -Dversion=2.5 -Dpackaging=jar

```

``` xml
jar包打包


maven 主要使用的插件有maven-jar-plugin，maven-assembly-plugin，maven-dependency-plugin
 <!-- 不含依赖包，将依赖包设定位/lib目录下的读取,一般打出来的文件比较小 -->
	<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-jar-plugin</artifactId>
				<configuration>
					<archive>
						<manifest>
							<addClasspath>true</addClasspath>
							<classpathPrefix>lib/</classpathPrefix>
								    	       <mainClass>com.hnty.cloud.sso.SecurityServerApplication</mainClass>
						</manifest>
					</archive>
				</configuration>
			</plugin>

<!--复制依赖包到/lib文件目录下-->
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-dependency-plugin</artifactId>
				<executions>
					<execution>
						<id>copy-dependencies</id>
						<phase>package</phase>
						<goals>
							<goal>copy-dependencies</goal>
						</goals>
						<configuration>
							<outputDirectory>target/lib</outputDirectory>
							<excludeTransitive>false</excludeTransitive>
							<stripVersion>false</stripVersion>
							<includeScope>runtime</includeScope>
						</configuration>
					</execution>
				</executions>
			</plugin>

<!--打包不含main，包含依赖的jar，这样与后面的在依赖的 mybatis 避免干扰-->
			<plugin>
				<artifactId>maven-assembly-plugin</artifactId>
				<configuration>
					<appendAssemblyId>false</appendAssemblyId>
					<descriptorRefs>
						<descriptorRef>jar-with-dependencies</descriptorRef>
					</descriptorRefs>
					<archive>
						<manifest>
							<!-- 此处指定main方法入口的class -->
							<mainClass></mainClass>
						</manifest>
					</archive>
				</configuration>
				<executions>
					<execution>
						<id>make-assembly</id>
						<phase>package</phase>
						<goals>
							<goal>single</goal>
						</goals>
					</execution>
				</executions>
			</plugin>

```



``` xml
将springboot类型的微服务打包			
<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass></mainClass>
                    <!--layout 的属性 ====> JAR:可执行jar； WAR：可执行war，需要servlet容器； ZIP:类是JAR  -->
                    <!--layout 的属性 ====> MODULE:出scope为provide外的所有；NONE： 所有依赖库打包 -->
                    <layout>NONE</layout>
                </configuration>
			</plugin>



spring-boot:repackage，默认goal。在mvn package之后，再次打包可执行的jar/war，同时保留mvn package生成的jar/war为.origin
spring-boot:run，运行Spring Boot应用
spring-boot:start，在mvn integration-test阶段，进行Spring Boot应用生命周期的管理
spring-boot:stop，在mvn integration-test阶段，进行Spring Boot应用生命周期的管理
spring-boot:build-info，生成Actuator使用的构建信息文件build-info.properties

maven命令： mvn package springboot springboot:repackage 
```





### VMare 和 Ubantu

``` sh
服务器：node000 输入都是node000
服务器：node001 输入都是node001
```

## Java8-SE

#### Oject

``` java
private static final long serialVersionUID = 1L; // 序列化的标记

// 对象相等
equal：比较字符串内容相等
== ：比较对象的地址相等
hashcode :将对象放入HashSet， HashMap等Hash相关的集合时，如果两个对象的内容相同，即equal 为true，但是构造生成的hashcode不同，因而还是会放入两个相同的元素。所以要@Override hashcode 方法，使对象某些相同字段相同时计算相同的得到相同的hashcode值。覆写equals方法之后，一定要覆写hashCode
```



## Git

``` sh
git init
git clone ${url} #g
git add
git log #查看日志,head历史
git reset HEAD ${head}
git reset --hard HEAD
git diff #查看缓存的改动
git status
git rm  ${file}#删除文件
git rm --cache  ${fileName}#从git同步清单中去除
git 本地分支相互merge， 
```





### GitLab

#### GitLab-Runner

``` sh
root@iZ8vbgiaul3ujqlibwm521Z:~# curl -sSL https://get.daocloud.io/docker | sh
# Executing docker install script, commit: 1b02882d63b9cfc484ad6b0180171c679cfe0f3a
+ sh -c apt-get update -qq >/dev/null
+ sh -c DEBIAN_FRONTEND=noninteractive apt-get install -y -qq apt-transport-https ca-certificates curl >/dev/null
+ sh -c curl -fsSL "https://download.docker.com/linux/ubuntu/gpg" | apt-key add -qq - >/dev/null
Warning: apt-key output should not be parsed (stdout is not a terminal)
+ sh -c echo "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable" > /etc/apt/sources.list.d/docker.list
+ sh -c apt-get update -qq >/dev/null
+ [ -n  ]
+ sh -c apt-get install -y -qq --no-install-recommends docker-ce >/dev/null

+ sh -c docker version
Client: Docker Engine - Community
 Version:           19.03.8
 API version:       1.40
 Go version:        go1.12.17
 Git commit:        afacb8b7f0
 Built:             Wed Mar 11 01:25:46 2020
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.8
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.17
  Git commit:       afacb8b7f0
  Built:            Wed Mar 11 01:24:19 2020
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.2.13
  GitCommit:        7ad184331fa3e55e52b890ea95e65ba581ae3429
 runc:
  Version:          1.0.0-rc10
  GitCommit:        dc9208a3303feef5b3839f4323d9beb36df0a9dd
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
If you would like to use Docker as a non-root user, you should now consider
adding your user to the "docker" group with something like:

  sudo usermod -aG docker your-user

Remember that you will have to log out and back in for this to take effect!

WARNING: Adding a user to the "docker" group will grant the ability to run
         containers which can be used to obtain root privileges on the
         docker host.
         Refer to https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface
         for more information.

```

## RabbitMQ



``` ini
# sbin 目录下执行 , 用户名不能为中文不然erl环境不能支持
# 安装延时队列插件
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
# 启动插件管理，启动后会看到服务中有延时队列的插件存在
rabbitmq-plugins enable rabbitmq_management
# 启动server
点击 rabbitserver.bat或者开始菜单启动
```



### Linux 

``` bash
# 新建user的指定文件路径
useradd -d /home/user  -m user 

# 用root查询已安装的软件路径
# 根据安装的方式的不同，
# 可以通过 yum list installed |grep ${software}
#  		  rpm -qa
# 可以通过 which ${software}
#		 whereis ${software}
#		find / -name *.rpm
#        find /home/user -name '*.txt'
[root@iZbp19tbls4i7mqa3ibjxtZ ~]# whereis docker
#		执行路径		配置路径		安装路径			默认站点目录
docker: /usr/bin/docker /etc/docker /usr/libexec/docker /usr/share/man/man1/docker.1.gz
# 但是docker是按照socket嵌套字去绑定，而非TCP端口，所以用sudo命令可以执行 
# 否则执行，将所有的变为可读（-R 递归的所有文件可读） chmod -R 755 ${安装路径} 
# 执行,授权给用户可执行 -R 是可执行 chmon -R ${usr} ${安装路径}  
# 但是CentOS的sudo 需要修改sudoer文件配置
# 修改方式
>>$cd /etc/
>>$chmod u+w sudoers
>>vi sudoers
#编辑sudoer文件， :q! 不保存退出  :wq 保存退出
# 在 root ALL = (ALL)       ALL 下
# 加 ${usr} ALL = (ALL)      ALL
#保存退出

# 回到只读状态
>>$chmod u-w sudoers
# 然后 
[lzdong]>>$sudo docker container list
# 会要求输入密码执行命令，不方便，所以将 lzdong 赋予与root一样的权限不用每次都输入密码
# 于是用户组权限添加
>>$cat /etc/group | grep docker
# 发现存在dockerroot 用户组
# 添加到用户组
>>$usermod -aG dockerroot lzdong
# 最后还是socket异常
# 用添加权限方式，访问scoket，但是重起了就是要重新监听这个文件
>>$ chmod a+rw /var/run/docker.sock
```

软件下载与安装

``` bash
# wget 通过网络连接下载软件 wget -O wordpress.zip http://www.centos.zb/download.php?id=1080
# 下载一个软件并且保存为wordpress.zip的文件名
wget -b ${url}  # 后台下载
tail -f wget-log # 查看下载日志


# curl 可以通过用户密码的输入下载

# rpm是 rethat 的软件安装统一管理
# yum是基于 rpm的安装统一管理


```



## Docker

虚拟容器，与vmare相似，可以用来装虚拟机，用来管理各个组件与app。

1 每个组件的启动，有自己的配置文件，以及自己的插件，如何正确的启动各个组件与服务？？？

2 如何执行镜像中的组件的 命令与功能？？？ （在linux和window下通过 ，配置环境变量 shell 或者cmd执行命令，在dockers下如何进入到各个容器中执行）

3 docker自身的管理与监控，docker的ip映射等问题。

``` sh
# docker 基本命令
# 启动
>>$service docker start
>>$sudo systemctl start docker
>>$sudo systemctl restart docker
# 关闭
>>$

# 镜像列表
>>$docker images 
>>$docker search ${program}
# 运行的容器列表
>>$docker container list

# 所有容器列表
>>$docker container list -a

# 进入到容器中的环境，以bash的形式
docker exec -it b0c3db2eff77 /bin/bash
# 这样就可以修改配置文件
```

阿里云的docker加速

```bash

sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://btqmwvi3.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker

```

docker执行

``` bash
# 搜索3000star 以上的
>>$ docker search ${programName} --filter=STAR=3000 
>>$ docker image pull ${imageName}
>>$ docker image pull ${imageName}:${tagVersion}
>>$ docker container run ${imageName}
# 删除容器，多个用 空格隔开
>>$ docker image rm ${imageName}  
# 删除所有镜像
>>$ docker rmi f $(docker images -aq)

# 运行
>> $ docker run ${param} image
param:
	--name="name"   容器名字 tomcat01 tomcat02
	-d              后台运行方式,
	-it 		   使用交互是运行，进入容器内部查看
		/bin/bash 进入以bash的方式查看，要退出用exit，但是会停止容器
		Ctrl + p + q 容器退出不停止执行
	-p			指定容器端口 -p 8080:8080
		-p ip:主机端口：容器端口
		-p 主机端口：容器端口（常用）
		-p 容器端口
		
>>$ docker run -it mysql 
	>>$mysql
    
# 停止
>> docker container ls #当前运行的容器
	-a #所有运行过的容器
## 停止的容器不会消失，会有containerID保留，要删除停止的容器文件
	
>> docker ps -a
>> docker stop ${containerID}
>> docker start ${containerID}
>> docker restart ${containerID}
>> docker kill ${containerID}
>> docker container rm ${containerID} #删除
>> docker rm -f $(docker ps -aq) #删除所有
# 也可以使用docker container run命令的--rm参数，在容器终止运行后自动删除容器文件。
docker container run --rm -p 8000:3000 -it mysql
```

docker 下载不同的版本，存在分层下载，可用不同的镜像共享使用文件

一些问题

后台启动

``` bash
docker run -d centos

# docker ps 发现centos 停止

#常见的坑： docker 容器使用后台运行，没有前台执行服务，那么就会自行关掉，所以nigx，java无法运行，除非执行一个服务程序

docker run -d --name tomcat01 -p 8880:8080 tomcat
# 访问是 404因为没有加载 webapp
# 从tomcat镜像中复制 webapp.dist 进入 webapp
docker exec -it tomcat01 /bin/bash
cp -r webapps.dist/* webapps
那么需要将 外部的webapps的文件加载到容器中，那么将可以制作一个新的镜像
```

日志问题

``` bash
docker logs -f -t -tail ${containerID} #没有容器日志

-tf #显示容器
docker logs -tf --tail 10 ${containerID} #查看容器执行的后面100条日志

docker ps ${containerID} 
docker top ${containerID} # 查看docker中的进程信息
docker inspect ${containerID} #查看docker中的数据元信息
```

进入正在运行的容器中修改配置

``` bash
>>$ docker [command] --help # 命令帮助

# 进入容器后开启一个新的终端，开启新线程
>>$ docker exec -it ${containerID} /bin/bash
 
 # 进入正在执行的终端
>>$ docker attach  ${containerID} 

```

从容器内拷贝文件到主机上

``` sh
# 拷贝，不用管容器是否运行
>>$ docker cp ${containerID}:/dir/file  /dir
>>$ docker cp /dir ${containerID}:/dir/file
# 复制运行中额tomcat到当前目录下（不需要 -r 递归参数）
docker cp  ce5349044d98:/usr/local/tomcat/webapps.dist ./webapp

实际是从docker镜像文件中cp过来
/var/lib/docker/overlay2/9a6e4aad05b8e2008c0f52ab345814d14a89d2c0f882327bb2a7fe10e3b2cfba/merged/usr/local/tomcat/webapps.dist/

# 以后使用 -v （volume 技术）同步容器与主机的文件

>>$ docker stats #查看docker的CUP， 内存暂用
>>$ docker -e  # 添加环境配置
>>$ dokcer run -d --name elasticsearch -p 9200:9200 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx521m"
```

 docker获取：

​	远程下载

​	复制拷贝

​	制作一个docker

dockers有一个联合文件系统UnionFS，镜像进行叠加，

bootfs(boot file system)主要包含bootload和kernel

docker 的镜像都是只读的，启动容器时，一个新的可写成添加到容器顶部，这一层就是我们说的容器。

将我们容器一起打包变为一个镜像，成为我提交制作的docker

``` sh


>>$ docker commit -a='lzdong' -m='add webapps' ce5349044d98  tomcat_my:1.0
将containerID 为ce5349044d98的容器制作为镜像 tomcat_my:1.0

```

#### dockers数据卷

将dockers 容器中运行中的程序数据挂载到linux中，比如mysql的容器，需要将数据路径挂载到linux文件目录下。当docker停止，或者容器停止运行，那么运行的数据就丢失了，这是在数据库是不可用的，所以必然要挂载出来

容器间也可以数据共享。

``` shell
# 方式1 使用 -v 主机目录：容器目录 命令来挂载
# 将 docker 中centos的 /home 目录挂载到 服务器下的 /home/test 目录下
>>$ docker run -it -v /home/test home/centos
#实际是将 容器文件 同步挂载到了 设定的文件

# 匿名挂载，只指定了容器文件 ，没有指定服务器文件
>>$ docker run -it -v /ect/nginx nigx

# 用volunme查看卷
>>$ docker volume ls
>> local xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# 具名挂载
>> $ docker run -d --name nginx01 -v ${volumeName}:/ect/nginx nginx
>> $ docker volumn ls
>> local ${volumeName}


# 目录的文件都在服务器的
/var/lib/docker/volume
## 只读 可读可写
docker run -it -v /ect/nginx:ro nigx
docker run -it -v /ect/nginx:rw nigx


```



#### dockerfile

``` dockerfile
# 生成镜像的通过脚本 mydockfile
# dockerfile 的例子
# 基于某一个镜像${imagesID}
FROM centos
# 镜像中自定义的匿名挂载数据卷 ，有对应的外部同步目录，
VOLUME ["volume01", "volume02"]
# 构建后执行命令
CMD "------end--------"
CMD /bin/bash

```



docker 命令build执行dockerfile，生成镜像

``` shell
docker build 
>>$ docker
# 末尾加 . 表示加载在当前文件夹下
docker build -f ${dockerFileName} -t ${imageName:target} .
docker build -f  mydockfile -t centos/myOS:1.0 . 
```





数据卷挂载出来，可以进行服务见的数据同步

``` shell
# mysql 同步数据
# 运行一个新容器${docker03}，挂载到一个已有的容器${docker01} 基于 镜像 ${image:tag}
docker run -it --name docker02 --volume-from docker01 centos/myOS:1.0

docker run -it --name docker03 --volume-from docker01 centos/myOS:1.0
 
# 那么docker01 docker02 docker03 的文件是复制共享的，即docker02， docker03 复制docker01的volume01,volume02到docker02，docker03下，并且文件之间是同步的
```



docker 共享

1  编写一个dockerfile

2 docker build 构建一个镜像

3 docker run 运行一个镜像

4 docker push (dockerhub, 阿里dockers镜像库，或者自己的库？？)

``` shell
 FROM #基础镜像,
 MAINTAINER  #镜像维护人 ${createrName}<${email}>
 RUN  # 执行
 ADD  # 步骤，添加其他依赖镜像，
 WORKDIR # 工作目录 ，默认工作目录
 VOLUME  # 挂载卷
 EXPOSE  # 指定暴露端口，仔细编排，避免端口冲突
 CMD     # 指定容器启动时候的运行命令，只有最后一条CMD命令执行
 ENTRYPOINT # 追加的方式添加命令
 ONBULID  # 当构建一个被继承的 dockerfile，这个时候运行
 COPY    # 类似 add
 ENV     # 构建的时候设定环境变量

```



``` dockerfile
FROM centos
MAINTAINER lzdong<282139155@qq.com>

ENV MYPATH /usr/local # 配置变量
WORKDIR $MYPATH 	  # 指定运行环境

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "------end-------"
CMD /bin/bash
```



使用官方命名 文件为 "Dockerfile" ，那么docker builder 不用使用 -f 指定文件名称 

``` dockerfile
FROM centos
MAINTIANER lzdong<282139155@qq.com>
COPY readme.txt /usr/local/readme.txt
ADD jdk-8ull-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-9.0.22.tar.gz /usr/local/
RUN yum -y install vim
ENV MYPATH /usr/local
WORKDIR $MYPATH

ENV JAVA_HOME /usr/local/jdk1.8.0_11
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tool.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.22
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.22
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib/:$CATALINA_HOME/bin

EXPOSE 8080
CMD /usr/local/apache-tomcat-9.0.22/bin/startup.sh && tali -F /usr/local/apache-tomcat-9.0.22/bin/logs/catatlina.out

```



Docker 网络

使用 evth-pair技术，桥接模式，使用协议相连接。使网卡成对出现。

``` shell
[root@iZbp19tbls4i7mqa3ibjxtZ ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:16:3e:10:59:b4 brd ff:ff:ff:ff:ff:ff
    inet 172.26.205.208/20 brd 172.26.207.255 scope global dynamic eth0
       valid_lft 314416761sec preferred_lft 314416761sec
    inet6 fe80::216:3eff:fe10:59b4/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ca:13:94:f4 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:caff:fe13:94f4/64 scope link 
       valid_lft forever preferred_lft forever
11: veth583d913@if10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether b2:89:d9:dc:cb:08 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::b089:d9ff:fedc:cb08/64 scope link 
       valid_lft forever preferred_lft forever
19: veth5f6dc35@if18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether a6:fa:03:53:6d:7a brd ff:ff:ff:ff:ff:ff link-netnsid 2
    inet6 fe80::a4fa:3ff:fe53:6d7a/64 scope link 
```



``` sh
[root@iZbp19tbls4i7mqa3ibjxtZ ~]# docker exec -it myjenkins ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
18: eth0@if19: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.3/16 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:acff:fe11:3/64 scope link 
       valid_lft forever preferred_lft forever

```





``` shell
# docker 网络查询
docker network ls

[root@iZbp19tbls4i7mqa3ibjxtZ ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
c187e6614d1f        bridge              bridge              local
ff577852f263        host                host                local
aca6b11d6cea        none                null                local

# 状态呢模式
bridge: 桥接模式，
none: 不适用模式
host： 宿主机

# 创建一个docker网络
docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet

#将容器启动到自己的网络中
docker run --name tomcat-net-01 --net mynet ${tomcatImagesName}
```

这里就可以编排集群在同意docker中，路由6个端口，启动6个redis容器，配置docker内的集群。

// 之后就是运维工作了，不要浪费主要经历，最多编写Dockerfile,部署到docker中

### Docker Compose

单独的容器没有意义，只有集群的服务编排才会有docker有用.

dockerCompose是一个官方的开源项目，需要安装，可以运行包含多个服务service（数据服务，微服务）组成的项目(project),但是还是单个docker启动多个服务，所以要跨服务器的编排服务。

``` shell
# 官方下载地址
sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# 国内加速下载镜像地址
sudo curl -L "https://get.daocloud.io/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

>>$ cd /usr/local/bin/
>>$ chmod +x docker-compose


```

官方启动用例https://docs.docker.com/compose/gettingstarted/

启动之后有内部访问

``` shell
[root@iZbp19tbls4i7mqa3ibjxtZ ~]# curl localhost:5000
Hello World! I have been seen 1 times.
[root@iZbp19tbls4i7mqa3ibjxtZ ~]# curl localhost:5000
Hello World! I have been seen 2 times.
[root@iZbp19tbls4i7mqa3ibjxtZ ~]# curl localhost:5000
Hello World! I have been seen 3 times.

# 查看服务
[root@iZbp19tbls4i7mqa3ibjxtZ ~]# docker network ls
NETWORK ID          NAME                  DRIVER              SCOPE
c187e6614d1f        bridge                bridge              local
d34253cf09ad        composetest_default   bridge              local
ff577852f263        host                  host                local
aca6b11d6cea        none                  null                local

```





启动docker-compose会给启动的service默认建立一个docker的网络。

``` yml
## ----docker-compose的编写 3层-------------
version: ${版本}
services:
	
```



### DockerSwarm



Jenkeins CI/CD



#### SpringC





 







