github地址： https://github.com/Streamhu/springCloud_demo
### 一、前期准备
##### 1. 环境
```
1） 开发环境：idea + maven + jdk1.8  

2） 主要框架：Spring Boot 
```
##### 2. spring cloud版本
```
<!-- spring cloud版本 -->
<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-dependencies</artifactId>
			<version>Camden.SR6</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>
```


<hr>

### 二、Eureka
```
1. pom中添加依赖	

	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>	
	<!-- Eureka -->
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-eureka-server</artifactId>
	</dependency>	

2. 在启动类中添加@EnableEurekaServer注解	

3. 配置文件

	spring.application.name=spring-cloud-eureka

	server.port=8000
	eureka.client.register-with-eureka=false
	eureka.client.fetch-registry=false

	eureka.client.serviceUrl.defaultZone=http://localhost:${server.port}/eureka/

4. 打开Eureka页面，http://localhost:8000/
```


<hr>

### 三、服务提供者（producer）

```
1. pom中添加依赖		

	<!-- Eureka -->
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-eureka</artifactId>
	</dependency>

2. 启动类中添加@EnableDiscoveryClient注解

3. 配置文件

	spring.application.name=spring-cloud-producer
	server.port=9000
	eureka.client.serviceUrl.defaultZone=http://localhost:8000/eureka/	

4. 建立Controller,提供hello服务

	@RestController
	public class HelloController {
		
	    @RequestMapping("/hello")
	    public String index(@RequestParam String name) {
	        return "hello "+name+"，this is first messge";
	    }
	}

5. 添加@EnableDiscoveryClient注解后，项目就具有了服务注册的功能。启动工程后，就可以在注册中心的页面看到SPRING-CLOUD-PRODUCER服务
```

<hr>

### 四、服务调用者（consumer）

```
1. pom中添加依赖	

	<!-- Eureka -->
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-eureka</artifactId>
	</dependency>	

2. 启动类添加@EnableDiscoveryClient和@EnableFeignClients注解

	1）@EnableDiscoveryClient :启用服务注册与发现

	2）@EnableFeignClients：启用feign进行远程调用		

3. 配置文件

	spring.application.name=spring-cloud-consumer
	server.port=9001
	eureka.client.serviceUrl.defaultZone=http://localhost:8000/eureka/	

4. feign调用实现

	@FeignClient(name= "spring-cloud-producer")
	public interface HelloRemote {
	    @RequestMapping(value = "/hello")
	    public String hello(@RequestParam(value = "name") String name);
	}	

5. web层调用远程服务

	@RestController
	public class ConsumerController {

	    @Autowired
	    HelloRemote HelloRemote;
		
	    @RequestMapping("/hello/{name}")
	    public String index(@PathVariable("name") String name) {
	        return HelloRemote.hello(name);
	    }

	}	

```

<hr>

### 五、熔断器Hystrix（熔断只是作用在服务调用这一端，只需要改动spring-cloud-consumer项目相关代码就可以）

```
1. Feign中已经依赖了Hystrix所以在maven配置上不用做任何改动

2. 配置文件

	feign.hystrix.enabled=true

3. 创建回调类（创建HelloRemoteHystrix类继承HelloRemote实现回调的方法）

	@Component
	public class HelloRemoteHystrix implements HelloRemote{

	    @Override
	    public String hello(@RequestParam(value = "name") String name) {
	        return "hello" +name+", this messge send failed ";
	    }
	}	

4. 添加fallback属性（在HelloRemote类添加指定fallback类，在服务熔断的时候返回fallback类中的内容）

	@FeignClient(name= "spring-cloud-producer",fallback = HelloRemoteHystrix.class)
	public interface HelloRemote {

	    @RequestMapping(value = "/hello")
	    public String hello(@RequestParam(value = "name") String name);

	}

```

<hr>

### 六、Config-server（先建好git仓库）

```
1. pom中添加依赖（他也是服务，所以注册到Eureka） 		

	<!-- Config Server -->
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-config-server</artifactId>
	</dependency>
	<!-- Eureka -->
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-eureka</artifactId>
	</dependency>	

2. 启动类添加@EnableConfigServer,以及@EnableDiscoveryClient	

3. 配置文件

	server:
	  port: 8001
	spring:
	  application:
	    name: spring-cloud-config-server
	  cloud:
	    config:
	      server:
	        git:
	          uri: https://github.com/ityouknow/spring-cloud-starter/     # 配置git仓库的地址
	          search-paths: config-repo                             	  # git仓库地址下的相对地址，可以配置多个，用,分割。
	          username:                                                   # git仓库的账号
	          password:                                                   # git仓库的密码	


4. 测试

	（1）测试server端是否可以读取到github上面的配置信息，直接访问：http://localhost:8001/neo-config/dev	
	
	（2）仓库中的配置文件会被转换成web接口，有几种规则，上面的规则是：/{application}/{profile}[/{label}]      

	（3）以neo-config-dev.properties为例子，它的application是neo-config，profile是dev。client会根据填写的参数来选择读取对应的配置  

```

<hr>

### 七、Config-client

```
1. pom中添加依赖（他也是服务，所以注册到Eureka）

	<!-- Config-client -->
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-config</artifactId>
	</dependency>	
	<!-- Eureka -->
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-eureka</artifactId>
	</dependency>	
	<!-- actuator， 配合@RefreshScope,是一套监控的功能 -->
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-actuator</artifactId>
	</dependency>

2. 启动类添加@EnableConfigServer，以及@EnableDiscoveryClient	

3. 配置文件（需要两个）

	1）application.properties如下：

		spring.application.name=spring-cloud-config-client
		server.port=8002

	2）bootstrap.properties如下：
	
		spring.cloud.config.name=neo-config
		spring.cloud.config.profile=dev
		spring.cloud.config.uri=http://localhost:8001/
		spring.cloud.config.label=master	

4. web测试（@RefreshScope，使用该注解的类，会在接到SpringCloud配置中心配置刷新的时候，自动将新的配置更新到该类对应的字段中）

	@RestController
	@RefreshScope
	class HelloController {
	    @Value("${neo.hello}")
	    private String hello;

	    @RequestMapping("/hello")
	    public String from() {
	        return this.hello;
	    }
	}		

```

<hr>

### 八、服务网关Zuul

```
1. pom中添加依赖（他也是服务，所以注册到Eureka）

	<!-- Zuul -->
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-zuul</artifactId>
	</dependency>
	<!-- Eureka -->
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-eureka</artifactId>
	</dependency>	

2. 启动类添加@EnableZuulProxy，以及@EnableDiscoveryClient	 

3. 配置文件

	spring.application.name=gateway-service-zuul
	server.port=8888

	eureka.client.serviceUrl.defaultZone=http://localhost:8000/eureka/

	#这里的配置表示，访问/producer/** 直接重定向到spring-cloud-producer服务
	zuul.routes.api-a.path=/producer/**
	zuul.routes.api-a.serviceId=spring-cloud-producer

4. 测试，请求 http://localhost:8888/producer/hello?name=hh，会被转发到serviceId对应的微服务
```

<hr>

### 九、分布式链路跟踪Zipkin

```
1. pom中添加依赖（他也是服务，所以注册到Eureka）

	<!-- Eureka -->
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-eureka</artifactId>
	</dependency>
	<!-- Zipkin -->
	<dependency>
		<groupId>io.zipkin.java</groupId>
		<artifactId>zipkin-server</artifactId>
	</dependency>
	<dependency>
		<groupId>io.zipkin.java</groupId>
		<artifactId>zipkin-autoconfigure-ui</artifactId>
	</dependency>	

2. 启动类添加@EnableZipkinServer，以及@EnableDiscoveryClient	 

3. 配置文件

	eureka:
	  client:
	    serviceUrl:
	      defaultZone: http://localhost:8761/eureka/
	server:
	  port: 9000
	spring:
	  application:
	    name: zipkin-server 

4. 访问http://localhost:9000/zipkin/可以看到Zipkin后台页面		    
		
5. 项目添加zipkin支持（比如网关zuul，服务提供者producer,调用者conusmer）	

	1）添加zipkin依赖

		<dependency>
		    <groupId>org.springframework.cloud</groupId>
		    <artifactId>spring-cloud-starter-zipkin</artifactId>
		</dependency>	

	2）配置文件中增加配置
	
		spring:
		  zipkin:
		    base-url: http://localhost:9000
		  sleuth:
		    sampler:
		      percentage: 1.0	

6. 进行验证

	1）通过外部请求访问Zuul网关，Zuul网关去调用spring-cloud-producer对外提供的服务		  

	2）打开zipkin页面，可以看到相关信息（调用顺序，各个微服务调用所消耗的时间等）  
```

<hr>

#### **参考网址**
[Spring Cloud 系列文章](http://www.ityouknow.com/spring-cloud)




<font color=#FF4500 size=3>注：文章是经过参考其他的文章然后自己整理出来的，有可能是小部分参考，也有可能是大部分参考，但绝对不是直接转载，觉得侵权了我会删，我只是把这个用于自己的笔记，顺便整理下知识的同时，能帮到一部分人。
ps : 有错误的还望各位大佬指正,小弟不胜感激</font>