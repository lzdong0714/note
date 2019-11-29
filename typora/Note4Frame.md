---

typora-root-url: Pic4Frame
---



Quartz：用来任务调度的框架

Stomp：就像HTTP在TCP套接字之上添加了请求-响应模型层一样，STOMP在WebSocket之上提供了一个基于帧的线路格式（frame-based wire format）层，用来定义消息的语义。

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





``` java
public enum RequestMethod {
	GET, HEAD, POST, PUT, PATCH, DELETE, OPTIONS, TRACE
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

​	spring框架加载

``` java

```

controller 层的参数注解

``` java
@PutMapping(value = "/path")
public ResponseBody controllerMethod(@RequsetParam(value = "value") variable_0,
                                    @RequestHeader variable_1,
                                    @RequestBody variale_2,
                                    @ModelAttribute variable_3)
```

#### Security

客户端，授权服务端，资源端

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
  
  客户端提供
  
  ``` java 
  // 客户端 提供clientId 和clientSecret的token发送验证的Provider
  // 
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
```java
  
  

​``` java
// 授权服务端
@Configuration
@EnableAuthorizationServer
public class OAuth2AuthServerConfig extends AuthorizationServerConfigurerAdapter {
    @Override
	public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
	}

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

	@Override
	public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
	}
}
```

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







#### SCAN

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
@Componemt
public Class SelfDefineClass{}

@Configuration
@Bean


```



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

``` my

```



## MAVEN

``` shell
mvn install:install-file -Dfile=aspose-cells-jdk16-8.5.2.jar -DgroupId=com.aspose.cells -DartifactId=aspose-cells-jdk16 -Dversion=8.5.2 -Dpackaging=jar

mvn install:install-file -Dfile=rules-core-1.1.0.jar -DgroupId=com.hntycloud -DartifactId=rules-core -Dversion=1.1.0 -Dpackaging=jar

mvn install:install-file -Dfile=redis-distributed-lock-starter-1.2.0.jar -DgroupId=com.hntycloud -DartifactId=redis-distributed-lock-starter -Dversion=1.2.0 -Dpackaging=jar

mvn install:install-file -Dfile=hnty-cloud-common-security-0.5.0-SNAPSHOT.jar -DgroupId=com.hnty.cloud -DartifactId=hnty-cloud-common-security -Dversion=0.5.0-SNAPSHOT -Dpackaging=jar

mvn install:install-file -Dfile=hnty-cloud-common-sys-0.5.0-SNAPSHOT.jar -DgroupId=com.hnty.cloud -DartifactId=hnty-cloud-common-sys -Dversion=0.5.0-SNAPSHOT -Dpackaging=jar

mvn install:install-file -Dfile=hnty-cloud-common-security-0.5.0-SNAPSHOT.jar -DgroupId=com.hnty.cloud -DartifactId=hnty-cloud-common-security -Dversion=0.5.0-SNAPSHOT -Dpackaging=jar
```

``` xml
jar包打包
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













