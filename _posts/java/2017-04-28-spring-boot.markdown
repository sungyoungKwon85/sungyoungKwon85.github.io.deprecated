---
layout: post
title:  "spring-boot"
date:   2017-04-28 11:10:53 +0900
categories: spring
---

# [스프링부트로 블로그 만들기]  

<br><br><br>

# MySQL + JPA + JDBC 연동  
{% highlight ruby %}
# pom.xml  
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <scope>runtime</scope>
</dependency>

# application.properties
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true // log 출력

spring.datasource.url=jdbc:mysql://localhost:3306/schema_name?autoReconnect=true&useUnicode=true&characterEncoding=utf8
spring.datasource.username=root
spring.datasource.password=root
spring.jpa.database=mysql
{% endhighlight %}

<br><br>

**spring.jpa.hibernate.ddl-auto**  

{% highlight ruby %}
create  : 기존테이블 삭제 후 다시 생성
create-drop: create와 같으나 종료시점에 테이블 DROP
update: 변경분만 반영
validate: 엔티티와 테이블이 정상 매핑되었는지만 확인
none: 사용하지 않음
{% endhighlight %}

<br><br><br>

# [RUN CODE AT SPRING BOOT STARTUP]  
{% highlight ruby %}
@Component
public class ApplicationStartup implements ApplicationListener<ApplicationEvent> {

	@Override
	public void onApplicationEvent(ApplicationEvent event) {
		System.out.println("test");
	}

}
{% endhighlight %}



<br><br><br>



# Logback  
출처 : [http://www.namooz.com/tag/spring-boot-logback/]  


Spring Boot 는 Common Logging API, jcl-over-slf4j 을 기본적으로 가지고 있다.  
web application 의 경우 spring-boot-starter-web 을 설정했을 경우 자동으로 Logback 을 사용할 수 있다.  
(Maven 라이브러리를 살펴보면 이미 존재함.)  
org.springframework.boot.logging.logback 안에 있는 base.xml 을 기본으로 사용 하고 있다.  

{% highlight ruby %}
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
...

private final Logger logger = LoggerFactory.getLogger(this.getClass());
...
	@RequestMapping(value = "/", method = RequestMethod.GET)
	public String helloWorld() {

		logger.debug("logback example - debug level");
		logger.info("logback example - info level");
		logger.warn("logback example - warn level");
		logger.error("logback example - error level");

		return "Hello World!";
	}
{% endhighlight %}

<br><br>
**customized configuration 을 하는 방법?**  
{% highlight ruby %}
# application.properties 이용하기  
# logback configuration
logging.level.org.springframework.web=INFO
logging.level.com.pristinecore.sbtest.rest=DEBUG
logging.file=logs/spring-boot-logging.log

# 프로젝트 루트 폴더에 logs 가 생성된 것을 볼 수 있고 그 안에 spring-boot-logging.log 가 들어 있는 것을 볼 수 있을 것



# logback-spring.xml 이용하기
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

	<!-- Send debug messages to System.out -->
	<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
		<!-- By default, encoders are assigned the type ch.qos.logback.classic.encoder.PatternLayoutEncoder -->
		<encoder>
			<pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{5} - %msg%n</pattern>
		</encoder>
	</appender>

	<!-- Send debug message to file -->
	<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>logs/spring-boot-logging.log</file>

		<encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
			<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
		</encoder>

		<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<fileNamePattern>logs/spring-boot-logging.%d{yyyy-MM-dd}_%i.log</fileNamePattern>

			<!-- each file should be at most 10MB, keep 30 days worth of history -->
			<maxHistory>30</maxHistory>
			<timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
				<maxFileSize>10MB</maxFileSize>
			</timeBasedFileNamingAndTriggeringPolicy>
		</rollingPolicy>
	</appender>

	<root level="INFO">
		<appender-ref ref="STDOUT" />
		<appender-ref ref="FILE" />
	</root>
</configuration>
{% endhighlight %}

**[ELK Stack 등을 이용한 중앙집중 방식]** 다음에 적용 해 보자.  





<br><br><br>



# Profile  
참조  
- [https://dhsim86.github.io/web/2017/03/28/spring_boot_profile-post.html]  

기존 Spring MVC 에서는 다음과 같이 maven 자체 profile 기능을 사용하는 경우가 많다.
{% highlight ruby %}
<profiles>
  <profile>
    <id>local</id>
    <activation>
      <activeByDefault>true</activeByDefault>
    </activation>
    <properties>
      <environment>local</environment>
      <maven.test.skip>true</maven.test.skip>
    </properties>
  </profile>
  <profile>
    <id>dev</id>
    <properties>
      <environment>dev</environment>
      <maven.test.skip>true</maven.test.skip>
    </properties>
  </profile>
  <profile>
    <id>prd</id>
    <properties>
      <environment>prd</environment>
      <maven.test.skip>true</maven.test.skip>
    </properties>
  </profile>
</profiles>

....
<webResource>
<directory>${basedir}/src/main/resources-${environment}</directory>
<targetPath>WEB-INF</targetPath>
</webResource>
{% endhighlight %}

[Spring Boot Reference Guide]에서는 다음과 같은 코드 구조를 추천한다.  
{% highlight ruby %}
com
 +- example
     +- myproject
         +- Application.java
         |
         +- domain
         |   +- Customer.java
         |   +- CustomerRepository.java
         |
         +- service
         |   +- CustomerService.java
         |
         +- web
             +- CustomerController.java
{% endhighlight %}
/src/main/java 및 /src/test/java 디렉토리에는 각각 자바 소스 및 테스트 소스를 놔두고, /src/main/resources 에는 각종 리소스 및 프로퍼티 파일을 위치시킨다.  
Spring MVC 에서는 프로젝트 생성하면 webapp 디렉토리가 생성되고 그 안에 css, img 등 다양한 리소스들을 넣게 되며, WEB-INF 에는 view 파일을 생성했었다.  
Spring Boot 에서는 webapp 이나 WEB-INF 는 아예 존재하지 않고, /src/main/resources 폴더 안에는 보통 다음과 같이 리소스 파일을 구분해서 위치시킴으로써 좀 더 정확하게 관리하고 있다.  

**application.properties 파일을 통하여 profile 설정하고자 할 때는 다음과 같이 application-{profile}.properties 라는 이름으로 /src/main/resource 에 두는 것부터 시작한다.**  
{% highlight ruby %}
# application.properties
spring.profiles.active=local

property.hello=default_hello
property.hi=default_hi
property.hey=default_hey

# MyApplication.java
private static final Logger logger = LoggerFactory.getLogger(MyApplication.class);

    @Value("${property.hello}")
    private String propertyHello;

    @Value("${property.hi}")
    private String propertyHi;

    @Value("${property.hey}")
    private String propertyHey;

	public static void main(String[] args) {
		SpringApplication.run(MyApplication.class, args);
	}

	@Bean
	public CommandLineRunner runner() {
	    return (a) -> {
	        logger.info("CommandLineRunner: " + propertyHello);
	        logger.info("CommandLineRunner: " + propertyHi);
	        logger.info("CommandLineRunner: " + propertyHey);
	    };
	};
{% endhighlight %}

디렉터리별로 관리도 가능하다. 링크를 참조하자.  



<br><br><br>



# Redis  
Spring boot는 기본적으로 [Redis autoConfiguration] 가 되어있기 때문에 따로 설정해줄 필요가 없다.  
(많은 블로그에서 따로 @Bean 설정을 하고 있지만 customizing 할 필요가 없다면 굳이 안해도 됨)  
설정 값을 변경하고 싶다면 application.properties에서 spring.redis.* 속성을 수정하면 된다.  
default로 localhost이기 때문에 local 작업시 따로 써줄 필요가 없다.  


{% highlight ruby %}
# MySQL에 있는 table을 redis에 캐시하고 싶어 아래와 같이 적용해봤다.

# MyApplication.java  
@SpringBootApplication
@EnableCaching

# MyModelRepository.java
public interface MyModelRepository extends JpaRepository<NuguSample, Integer>{
	@Cacheable("myModelList")
	public List<MyModel> findAll();
}


# 수행시간 확인
# test.java
    long start = System.currentTimeMillis();
		List<MyModel> list= myModelRepository.findAll();
		long end = System.currentTimeMillis();
		logger.debug("Cache 수행시간 : "+ Long.toString(end-start));

# terminal > redis-cli 실행 > monitor 실행시 확인해 볼 수 있다!
{% endhighlight %}



<br><br><br>



{% highlight ruby %}
# http://wonwoo.ml/index.php/post/701  
@Autowired
private CacheManager cacheManager;

// caching
Cache cache = cacheManager.getCache("cache.mymodel");
cache.put("mymodel", params);

// get
MyModel hello = cache.get("mymodel", MyModelVO.class);

{% endhighlight %}
<br><br><br>
# [스프링 데이터 레디스 번역]  
나중에 읽어보면 좋을 듯. 레디스 기본적인 내용도 번역되어 있다.  



<br><br><br>



# ResponsBody + Json 관련  
**#1**  
3rd party server로부터 response를 받아오는데 필드명이 Example, Example-Size 등으로 내려온다.  
Java DTO를 작성할 때는 example, exampleSize 등으로 기재를 하고 싶은데, 매핑이 되지 않는다.  
이럴때는 `@JsonProperty("Example"), @JsonProperty("Example-Size")` 등으로 명시해주면 된다.  
참고 : http://ojc.asia/bbs/board.php?bo_table=LecJpa&wr_id=270  




<br><br><br>




# Test  
http://sejong-wiki.appspot.com/assertThat  



<br><br><br>


# [Create a deployable war file], 배포하기  
전통적인 상용 서버, 톰캣, war 배포가 필요한 경우 아래와 같이 진행한다.  
클라우드 서버를 사용한다면 CLI를 이용한 배포가 가능하다.  
{% highlight ruby %}
@SpringBootApplication
public class Application extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application.class);
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Application.class, args);
    }

}

<packaging>war</packaging>

<dependencies>
    <!-- … -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <scope>provided</scope>
    </dependency>
    <!-- … -->
</dependencies>
{% endhighlight %}





[스프링부트로 블로그 만들기]: http://millky.com/@origoni/post/1144
[RUN CODE AT SPRING BOOT STARTUP]: http://blog.netgloo.com/2014/11/13/run-code-at-spring-boot-startup/
[http://www.namooz.com/tag/spring-boot-logback/]: http://www.namooz.com/tag/spring-boot-logback/
[https://dhsim86.github.io/web/2017/03/28/spring_boot_profile-post.html]: https://dhsim86.github.io/web/2017/03/28/spring_boot_profile-post.html
[ELK Stack 등을 이용한 중앙집중 방식]: http://jsonobject.tistory.com/243
[Spring Boot Reference Guide]: http://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-structuring-your-code.html
[Redis autoConfiguration]: https://github.com/spring-projects/spring-boot/blob/v1.5.0.RC1/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/cache/RedisCacheConfiguration.java#L55-L64
[스프링 데이터 레디스 번역]: http://arahansa.github.io/docs_spring/redis.html
[Create a deployable war file]: http://docs.spring.io/spring-boot/docs/current/reference/html/howto-traditional-deployment.html
<br><br><br>

{% highlight ruby %}
{% endhighlight %}
