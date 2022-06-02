8001 端口的订单服务配置：
```
# 服务端口号
server:
 port: 8001

# 数据库地址
datasource:
 url: localhost:3306/microservice01
# 省略数据库的基本配置

spring:
 application:
   name: microservice-order # 对外暴露的服务名称

# 客户端注册进eureka服务列表里
eureka:
 client:
   service-url:
     defaultZone: http://eureka01:7001/eureka/,http://eureka02:7002/eureka/,http://eureka03:7003/eureka/,
 instance:
   instance-id: 书籍订单服务-8001  # 人性化显示出服务的信息
   prefer-ip-address: true    # 访问路径可显示ip地址
```
8002 端口的订单服务配置：
```
# 服务端口号
server:
 port: 8002

# 数据库地址
datasource:
 url: localhost:3306/microservice02
# 数据库基本配置省略

spring:
 application:
   name: microservice-order # 对外暴露的服务名称

# 客户端注册进eureka服务列表里
eureka:
 client:
   service-url:
     defaultZone: http://eureka01:7001/eureka/,http://eureka02:7002/eureka/,http://eureka03:7003/eureka/,
 instance:
   instance-id: 书籍订单服务-8002  # 人性化显示出服务的信息
   prefer-ip-address: true    # 访问路径可显示ip地址
```
对比后发现，有几个地方需要注意：

对外暴露的服务名称必须要相同，因为都是同一个服务，只不过有多个而已，因为接下来Ribbon是通过服务名来调用服务的；

每个服务连接了不同的数据库，这样用来区分不同的服务，便于测试，实际中也可能是便于维护；

每个服务的个性化名称展示可以区分一下，这样在eureka里可以很好的辨别出来
启动了之后，可以访问下 eureka01:7001，看下三个订单服务是否正常注册到 eureka 集群里。如下图，说明集群和订单服务均正常
2. 如何指定 Ribbon 的负载均衡策略

由上面的结果可知，Ribbon 默认的策略是轮询，那么 Ribbon 除了轮询，还有哪些负载均衡的策略呢？我们如何去设置自己想要的策略呢？

Ribbon 自带的负载均衡策略有如下几个：

- RoundRibbonRule：轮询。人人有份，一个个来！

- RandomRule：随机。拼人品了！

- AvailabilityFilteringRule：先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，以及并发连接数超过阈值的服务，剩下的服务，使用轮询策略。

- WeightedResponseTimeRule：根据平均响应时间计算所有服务的权重，响应越快的服务权重越高，越容易被选中。一开始启动时，统计信息不足的情况下，使用轮询。

- RetryRule：先轮询，如果获取失败则在指定时间内重试，重新轮询可用的服务。

- BestAvailableRule：先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务。

- ZoneAvoidanceRule：复合判断 server 所在区域的性能和 server 的可用性选择服务器

如何指定 Ribbon 自带的负载均衡策略呢？我们需要在配置类中指定一下即可，如下：
```
/**
* 配置RestTemplate
* @author shengwu ni
*/
@Configuration
public class RestTemplateConfig {

   /**
    * '@LoadBalanced'注解表示使用Ribbon实现客户端负载均衡
    * @return RestTemplate
    */
   @Bean
   @LoadBalanced
   public RestTemplate getRestTemplate() {
       return new RestTemplate();
   }

   /**
    * 指定其他负载均衡策略
    * @return IRule
    */
   @Bean
   public IRule myRule() {
       // 指定重试策略：先轮询，若获取失败则在指定时间内重试，重新轮询可用的服务。
       return new RetryRule();
   }
}
```
我们可以 new 出以上对应的策略，来实现对应的负载均衡，读者可以 new RandomRule() 测试一下随机策略，然后重复刷新上面的测试地址，可以发现是随机请求三个服务。其他的策略，读者可以自行尝试一下。

2. Ribbon 的使用

我们在前面文章中，将订单服务注册到 Eureka，然后消费方可以通过 http 请求去获取订单的信息，但是这是最原始的 http 调用，没有任何 Ribbon 的东西在里面，接下来我们要在消费方植入 Ribbon。

2.1 导入 Ribbon 依赖

我们使用的Spring Cloud 版本是 Finchley，该版本需要导入的依赖如下：
```
<!--eureka Client-->
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<!--ribbon负载均衡依赖-->
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```
可以看到，Eureka Client 的依赖也需要导入，因为服务注册到了 Eureka，Ribbon 也需要和 Eureka 整合，所以在消费方也导入了 Eureka 依赖。

2.2 配置 application.yml
```
server:
 port: 9001

eureka:
 client:
   register-with-eureka: false
   service-url:
     defaultZone: http://eureka01:7001/eureka/
```
由前面的内容（Spring Cloud：使用Eureka集群搭建高可用服务注册中心）可知，我们搭建了一个 Eureka 集群，那么就用这个集群，这个消费方我们设置不注册到该集群中。

2.3 向 http 中植入 Ribbon

这是什么意思呢？之前的 消费方是使用 RestTemplate 来发送 http 请求，调用订单服务的，但是没有负载均衡，所以现在我们要让这个 http 调用自带负载均衡。

即修改下 RestTemplate 配置：
```
/**
* 配置RestTemplate
* @author shengwu ni
*/
@Configuration
public class RestTemplateConfig {

   /**
    * '@LoadBalanced'注解表示使用Ribbon实现客户端负载均衡
    * @return RestTemplate
    */
   @Bean
   @LoadBalanced
   public RestTemplate getRestTemplate() {
       return new RestTemplate();
   }
}
```
在方法上添加一个 @LoadBalanced 注解即可开启 Ribbon 负载均衡。这样就可以通过微服务的名字从 Eureka 中找到对应的服务并访问了。

友情提示：别忘了在主启动类上添加 @EnableEurekaClient，因为这个消费方也是一个 Eureka Client，刚刚我们已经导入了 Eureka Client 的依赖了。

2.4 将ip改成服务名称

刚刚提到了，开启 Ribbon 负载均衡后，就可以通过微服务的名字从 Eureka 中找到对应的服务。我们先来看下原来是怎么实现的。
```
/**
* 订单消费服务
* @author shengwu ni
*/
@RestController
@RequestMapping("/consumer/order")
public class OrderConsumerController {

   /**
    * 订单服务提供者模块的 url 前缀
    */
//    private static final String ORDER_PROVIDER_URL_PREFIX = "
   private static final String ORDER_PROVIDER_URL_PREFIX = ";

   @Resource
   private RestTemplate restTemplate;

   @GetMapping("/get/{id}")
   public TOrder getOrder(@PathVariable Long id) {

       return restTemplate.getForObject(ORDER_PROVIDER_URL_PREFIX + "/provider/order/get/" + id, TOrder.class);
   }
}
```
可以看出，注释掉的那部分，是原来的访问方式，订单提供服务是8001端口，现在我们将ip+端口号这种访问方式，改成微服务名称，这个名称就是 Eureka 管理界面显示的注册进去的名称，也即服务提供方的application.yml配置文件中配置的服务名称：
```
spring:
 application:
   name: microservice-order # 对外暴露的服务名称
 ```
在前面文章中已经说了，不再赘述。
