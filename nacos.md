## Nacos 服务注册与发现

#### 简单介绍

	1. alibaba开源的微服务架构中服务注册与发现组件
	2. 特性：动态服务发现、服务配置、服务元数据、流量管理
	3. 功能上类似于 Eureka +  Spring Cloud Config
	
##### [下载nacos](https://github.com/alibaba/nacos/releases)

##### 文件目录


 
##### 启动

   - windows : cmd bin目录下， startup.cmd -m standalone

   - 管理界面地址： http://127.0.0.1:8848/nacos/index.html  
	 账号密码：nacos/nacos

### 服务注册

   1. 新建springboot项目
   2. 新增服务  
      2.1 创建新模块module，添加依赖：
      ```xml
      <dependencies>
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-alibaba-nacos-discovery</artifactId>
          </dependency>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-web</artifactId>
          </dependency>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-test</artifactId>
              <scope>test</scope>
          </dependency>
      </dependencies>
       
      <!--  依赖版本定义  -->
      <dependencyManagement>
          <dependencies>
              <dependency>
                  <groupId>org.springframework.cloud</groupId>
                  <artifactId>spring-cloud-aws-dependencies</artifactId>
                  <version>Greenwich.RELEASE</version>
                  <type>pom</type>
                  <scope>import</scope>
              </dependency>
              <dependency>
                  <groupId>org.springframework.cloud</groupId>
                  <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                  <version>0.9.0.RELEASE</version>
                  <type>pom</type>
                  <scope>import</scope>
              </dependency>
          </dependencies>
      </dependencyManagement>
      ```
      注意版本依赖关系：
              
            Spring Cloud Version   | Spring Boot Version | Spring Cloud Alibaba Version
          ------------------------ | ------------------- | ----------------------------
           Spring Cloud Greenwich  |    2.1.X.RELEASE    |       0.9.0.RELEASE
           Spring Cloud Finchley   |    2.0.X.RELEASE    |       0.2.X.RELEASE
           Spring Cloud Edgware    |    1.5.X.RELEASE    |       0.1.X.RELEASE
               
      2.2 文件配置：
      ```yaml
        spring:
          application:
            name: nacos-discovery-consumer # 服务名称
          cloud:
            nacos:
              discovery:
                server-addr: 127.0.0.1:8848 # 服务注册
        
        server:
          port: 8050  #  服务端口号
      ```
      2.3 启动类加注解：`@EnableDiscoveryClient       `  
   3. 调用其他服务  
      [服务消费方] 
      ```java
      @RestController
      public class DemoController {
      
          @Autowired
          private RestTemplate restTemplate;
      
          @Autowired
          private LoadBalancerClient loadBalancerClient;
      
          @GetMapping("/test")
          public String test(String name) {
              //获取服务实例
              ServiceInstance serviceInstance = loadBalancerClient.choose("nacos-discovery-provider");
              //服务uri
              URI uri = serviceInstance.getUri();
              //调用服务方法，返回结果集
              return restTemplate.getForObject(uri + "/demo?name=" + name,String.class);
          }
      }   
      ``` 
      [服务提供方]
      ```java
      @RestController
      public class DemoController {
      
          @GetMapping("/demo")
          public String demo(String name) {
              return "Hello " + name;
          }
      }
      ``` 
   4. 启动服务，管理台服务列表出现服务说明服务注册成功。
   5. 测试服务调用
