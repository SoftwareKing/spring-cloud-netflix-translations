### Registering with Eureka

When a client registers with Eureka, it provides meta-data about itself such as host and port, health indicator URL, home page etc.  
当客户端向Eureka注册的时候，它需要提供自身的一些元信息，如主机、端口、健康检查URL，主页等。

Eureka receives heartbeat messages from each instance belonging to a service.  
Eureka接收来自服务每个实例的心跳消息。

If the heartbeat fails over a configurable timetable, the instance is normally removed from the registry.  
如果在配置时间内心跳失败，那么这个服务实例将会从Eureka删除。

Example eureka client:
``` java
@Configuration
@ComponentScan
@EnableAutoConfiguration
@EnableEurekaClient
@RestController
public class Application {

    @RequestMapping("/")
    public String home() {
        return "Hello world";
    }

    public static void main(String[] args) {
        new SpringApplicationBuilder(Application.class).web(true).run(args);
    }

}
```
(i.e. utterly normal Spring Boot app).

In this example we use @EnableEurekaClient explicitly, but with only Eureka available you could also use @EnableDiscoveryClient.  
在这个例子中，我们显然使用了@EnableEurekaClient，但是为了让Eureka生效，我们还需要使用@EnableDiscoveryClient注解。

Configuration is required to locate the Eureka server. Example:  
下面配置是为了指定Eureka服务端的地址。例如：  
application.yml
```yml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```
where "defaultZone" is a magic string fallback value that provides the service URL for any client that doesn't express a preference (i.e. it’s a useful default).  
defaultZone是默认配置，如果客户端没有配置的话，那么用这个默认配置。

The default application name (service ID), virtual host and non-secure port, taken from the Environment, are ${spring.application.name}, ${spring.application.name} and ${server.port} respectively.  
默认的应用名称（服务ID），虚拟主机和不安全端口（？），依次从以下位置获取：系统环境变量，spring.application.name，spring.application.name，server.port。

@EnableEurekaClient makes the app into both a Eureka "instance" (i.e. it registers itself) and a "client" (i.e. it can query the registry to locate other services).  
@EnableEurekaClient注解，让app变成Eureka的实例（也就是自注册），同时是一个“客户端”（即：它可以查询注册信息获取其它的服务）。

The instance behaviour is driven by eureka.instance.* configuration keys, but the defaults will be fine if you ensure that your application has a spring.application.name (this is the default for the Eureka service ID, or VIP).  
实例的行为通过eureka.instance.*来配置，但是默认值一般足够用，如果你确定配置了spring.application.name（Eureka服务标识）这个属性。

See EurekaInstanceConfigBean and EurekaClientConfigBean for more details of the configurable options.  
详情请见 EurekaInstanceConfigBean 和 EurekaClientConfigBean 这两个类获取更详细的配置信息。
