Zuul 的主要功能：路由转发、过滤器。Zuul 默认和 Ribbon 结合实现了负载均衡的功能。

其中**路由功能负责将外部请求转发到具体的微服务实例上，是实现外部访问统一入口的基础而过滤器功能则负责对请求的处理过程进行干预，是实现请求校验、服务聚合等功能的基础**。Zuul 和 Eureka进行整合，将 Zuul 自身注册为 Eureka服务治理下的应用，同时从 Eureka 中获得其他微服务的消息，也即以后的访问微服务都是通过Zuul跳转后获得。

注意：Zuul 服务最终还是会注册进 Eureka。

Ribbon 是对多个服务做负载，Zuul 是对外部请求做负载。

# 路由转发

在 Spring Initializer 选择 Cloud Discovery 右侧的 Eureka Discovery；选择 Cloud Rounting 右侧的 Zuul；及Web模块。

1. 启动 Eureka Server、及多个服务提供者实例(8080、8081端口)、通过 Feign、Ribbon 实现的服务消费者；

2. 在主配置类上使用`@EnableZuulProxy`开启 Zuul 功能；

   ```java
   @EnableZuulProxy
   @EnableEurekaClient
   @EnableDiscoveryClient
   @SpringBootApplication
   public class ServiceZuulApplication {
       public static void main(String[] args) {
           SpringApplication.run(ServiceZuulApplication.class, args);
       }
   }
   ```

3. 配置信息：

   ```yaml
   eureka:
     client:
       service-url:
         defaultZone: http://localhost:24946/eureka/
   server:
     port: 8070
   spring:
     application:
       name: Service-Zuul
   zuul:
     routes:
       fms5cmS:
         path: /fms5cmS/**
         serviceId: Service-Ribbon   # 以 /fms5cmS 开头的请求都转发给 Service-Ribbon 服务
       zzk:
         path: /zzk/**
         serviceId: Service-Feign   # 以 /zzk 开头的请求都转发给 Service-Feign 服务
   ```

4. 启动程序，在浏览器中输入`localhost:8070/fms5cmS/hello?name=fS`或`localhost:8070/zzk/hello?name=fS`可以正常访问



# 服务过滤

Zuul 不仅只是路由，并且还能过滤，做一些安全验证。在上面的基础上增加过滤器：

```java
@Slf4j
@Component //添加到容器中
public class MyFilter extends ZuulFilter {
    /**
     * 返回一个字符串代表过滤器的类型，在zuul中定义了四种不同生命周期的过滤器类型：
     * pre：路由之前
	 * routing：路由之时
     * post： 路由之后
     * error：发送错误调用
     */
    @Override
    public String filterType() {
        return "pre";
    }
	/**过滤的顺序*/
    @Override
    public int filterOrder() {
        return 0;
    }
	/**根据业务需求来判断是否要过滤，true表示过滤*/
    @Override
    public boolean shouldFilter() {
        return true;
    }
	/**过滤器的具体逻辑。包括查sql，nosql去判断该请求到底有没有权限访问。*/
    //这里判断有无token
    @Override
    public Object run() throws ZuulException {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        log.info(String.format("%s >>> %s", request.getMethod(), request.getRequestURL().toString()));
        Object accessToken = request.getParameter("token");
        if(accessToken == null) {
            log.warn("token is empty");
            ctx.setSendZuulResponse(false);
            ctx.setResponseStatusCode(401);
            try {
                ctx.getResponse().getWriter().write("token is empty");
            }catch (Exception e){}

            return null;
        }
        log.info("ok");
        return null;
    }
}
```

启动程序，在浏览器中输入`localhost:8070/fms5cmS/hello?name=fS`或`localhost:8070/zzk/hello?name=fS`，浏览器提示“token is empty”，如果在请求参数中增加 token=abc，则可以正常完成访问。

