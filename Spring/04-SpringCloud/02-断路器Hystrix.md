​Hystrix是一个用于**处理分布式系统的延迟和容错**的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。

“断路器”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），**向调用方返回一个符合预期的、可处理的备选响应（FallBack），而不是长时间的等待或者抛出调用方无法处理的异常**，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。



# 服务熔断

熔断机制是应对雪崩效应的一种微服务链路保护机制。

**当扇出链路的某个微服务不可用或者响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回"错误"的响应信息**。当检测到该节点微服务调用响应正常后恢复调用链路。在SpringCloud框架里熔断机制通过Hystrix实现。**Hystrix会监控微服务间调用的状况，当失败的调用到一定阈值，缺省是5秒内20次调用失败就会启动熔断机制**。熔断机制的注解是`@HystrixCommand`。



## 在Ribbo中使用

在 Spring Initializer 选择 Cloud Discovery 右侧的 Eureka Discovery；选择 Cloud Rounting 右侧的 Ribbon；选择 Cloud Circuit Breaker 右侧的 Hystrix；及Web模块。

下面的代码是在上一节 Ribbon 的代码中修改的。

1. 启动 Eureka 注册中心、及多个服务提供者实例(8080、8081端口)

2. 配置服务消费者（见上一节）

3. 在主配置类上添加`@EnableHystrix`来启用 Hystrix

   ```java
   @EnableHystrix
   @EnableEurekaClient
   @EnableDiscoveryClient
   @SpringBootApplication
   public class ServiceRibbonApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(ServiceRibbonApplication.class, args);
       }
   
       @Bean
       @LoadBalanced
       public RestTemplate restTemplate(){
           return new RestTemplate();
       }
   }
   ```

4. 在 Service 方法上添加`@HystrixCommond(fallbackMethod="回调方法名")`，该注解对该方法创建了熔断器的功能，并指定了fallbackMethod熔断方法

   ```java
   @Service
   public class HelloService {
       @Autowired
       RestTemplate restTemplate;
   
       @HystrixCommand(fallbackMethod = "sorry")
       public String hello(String name) {
           return restTemplate.getForObject("http://EUREKA-CLIENT/hi?name="+name,String.class);
       }
   
       public String sorry(String name){ //参数要和标注了@HystrixCommond的方法一致
           return "sorry " + name + ",there is an error";
       }
   }
   ```

5. 启动工程，访问`localhost:8090//hello?name=fS`，发现访问正常，关闭服务提供者工程后，再次访问，浏览器显示“sorry fS,there is an error”



## 在Fegin中使用

在上一节 Feign 的代码上进行修改。

Feign是自带断路器的，需要在配置文件中配置打开它：

```yaml
feign:
  hystrix:
    enabled: true
```

1. 编写一个实现了 Service 接口的类，并注入容器

   ```java
   @Component
   public class HystrixService implements SchedualServiceHi {
       @Override
       public String sayHello(String name) {
           return "sorry " + name + " there is an error";
       }
   }
   ```

2. 在 Service 接口上的`@FeignClient`中指定回调的类，当原本的服务调用失败后，回调类中的对应同名方法

   ```java
   @FeignClient(value = "EUREKA-CLIENT",fallback = HystrixService.class)
   public interface SchedualServiceHi {
   
       @GetMapping("/hi")
       String sayHello(@RequestParam(value = "name") String name);
   }
   ```

3. 启动工程，访问`localhost:8091//hello?name=fS`，发现访问正常，关闭服务提供者工程后，再次访问，浏览器显示“sorry fS,there is an error”





# 服务降级

Hystrix服务降级，其实就是线程池中单个线程障处理，防止单个线程请求时间太长，导致资源长期被占有而得不到释放，从而导致线程池被快速占用完，导致服务崩溃。

Hystrix能解决如下问题：

1. 请求超时降级，线程资源不足降级，降级之后可以返回自定义数据
2. 线程池隔离降级，分布式服务可以针对不同的服务使用不同的线程池，从而互不影响
3. 自动触发降级与恢复
4. 实现请求缓存和请求合并



