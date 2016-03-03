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
实例的行为通过eureka.instance.xxx来配置，但是默认值一般足够用，如果你确定配置了spring.application.name（Eureka服务标识）这个属性。

See EurekaInstanceConfigBean and EurekaClientConfigBean for more details of the configurable options.  
详情请见 EurekaInstanceConfigBean 和 EurekaClientConfigBean 这两个类获取更详细的配置信息。

### Status Page and Health Indicator
The status page and health indicators for a Eureka instance default to "/info" and "/health" respectively, which are the default locations of useful endpoints in a Spring Boot Actuator application.  
Eureka实例的状态页面和健康指示页面默认分别为“/info”和“/health”，当然它们也是Spring Boot Actuator应用默认有用的端点。

You need to change these, even for an Actuator application if you use a non-default context path or servlet path (e.g. server.servletPath=/foo) or management endpoint path (e.g. management.contextPath=/admin).  
你需要改变这些，即使是一个Actuator应用，如果使用非默认上下文路径或servlet路径（例如server.servletPath = / foo）或管理端点的路径（例如management.contextPath = / admin）。

Example: application.yml  
```yml
eureka:
  instance:
    statusPageUrlPath: ${management.contextPath}/info
    healthCheckUrlPath: ${management.contextPath}/health
```
These links show up in the metadata that is consumers by clients, and used in some scenarios to decide whether to send requests to your application, so it’s helpful if they are accurate.  
这些链接存储于元数据中，在客户端中会被使用，同时在某些场景中被用来决定是否发送请求到该应用，所以将会很有用假如这些信息是精确的话。

### Eureka Metadata for Instances and Clients
It’s worth spending a bit of time understanding how the Eureka metadata works, so you can use it in a way that makes sense in your platform.  
这值得花一些时间来了解Eureka元数据是如何工作的，以便于在你的平台中使用的更有意义。

There is standard metadata for things like hostname, IP address, port numbers, status page and health check.  
有一些标准的元数据，如主机名、IP地址、端口号、状态页和健康检查页。

These are published in the service registry and used by clients to contact the services in a straightforward way.  
这些信息通过服务注册从而被发布，客户端可以直连到服务端。

Additional metadata can be added to the instance registration in the eureka.instance.metadataMap, and this will be accessible in the remote clients, but in general will not change the behaviour of the client, unless it is made aware of the meaning of the metadata.  
其它的元数据可以通过配置 eureka.instance.metadataMap来发布，同样这些信息可以被远程客户端访问到，但是通常来说这不会改变客户端的行为，除非客户端主动监测这些元数据。

There are a couple of special cases described below where Spring Cloud already assigns meaning to the metadata map.  
下面描述Spring Cloud已经主动处理元数据的特殊案例。

### Using Eureka on Cloudfoundry
Cloudfoundry has a global router so that all instances of the same app have the same hostname (it’s the same in other PaaS solutions with a similar architecture). This isn’t necessarily a barrier to using Eureka, but if you use the router (recommended, or even mandatory depending on the way your platform was set up), you need to explicitly set the hostname and port numbers (secure or non-secure) so that they use the router. You might also want to use instance metadata so you can distinguish between the instances on the client (e.g. in a custom load balancer). For example:  
application.yml
``` yaml
eureka:
  instance:
    hostname: ${vcap.application.uris[0]}
    nonSecurePort: 80
    metadataMap:
      instanceId: ${vcap.application.instance_id:${spring.application.name}:${spring.application.instance_id:${server.port}}}
```
Depending on the way the security rules are set up in your Cloudfoundry instance, you might be able to register and use the IP address of the host VM for direct service-to-service calls. This feature is not (yet) available on Pivotal Web Services (PWS).

### Using Eureka on AWS
If the application is planned to be deployed to an AWS cloud, then the Eureka instance will have to be configured to be Amazon aware and this can be done by customizing the EurekaInstanceConfigBean the following way:
``` java
@Bean
@Profile("!default")
public EurekaInstanceConfigBean eurekaInstanceConfig() {
  EurekaInstanceConfigBean b = new EurekaInstanceConfigBean();
  AmazonInfo info = AmazonInfo.Builder.newBuilder().autoBuild("eureka");
  b.setDataCenterInfo(info);
  return b;
}
```
### Making the Eureka Instance ID Unique
By default a eureka instance is registered with an ID that is equal to its host name (i.e. only one service per host).  
eureka实例注册的时候会带有一个ID，这个ID默认和主机名称相同（也就是说一个主机部署一个应用实例）。

Using Spring Cloud you can override this by providing a unique identifier in eureka.instance.metadataMap.instanceId. For example:  
使用Sping Cloud，你可以自定义一个唯一的标识符，通过设置eureka.instance.metadataMap.instanceId属性，例如：
application.yml
``` yaml
eureka:
  instance:
    metadataMap:
      instanceId: ${spring.application.name}:${spring.application.instance_id:${random.value}}
```      
With this metadata, and multiple service instances deployed on localhost, the random value will kick in there to make the instance unique.  
In Cloudfoundry the spring.application.instance_id will be populated automatically in a Spring Boot Actuator application, so the random value will not be needed.

### Using the DiscoveryClient
Once you have an app that is @EnableEurekaClient you can use it to discover service instances from the Eureka Server.  
一旦你的应用标注@EnableEurekaClient注解，那么你就可以从Eureka服务器来发现其他服务实例。

One way to do that is to use the native com.netflix.discovery.DiscoveryClient (as opposed to the Spring Cloud DiscoveryClient), e.g.  
一种方法是通过使用Netflix原生类：com.netflix.discovery.DiscoveryClient（和Spring Cloud提供的DiscoverClient相比），例如：  
``` java
@Autowired
private DiscoveryClient discoveryClient;

public String serviceUrl() {
    InstanceInfo instance = discoveryClient.getNextServerFromEureka("STORES", false);
    return instance.getHomePageUrl();
}
```
TIPs:  
Don’t use the DiscoveryClient in @PostConstruct method or in a @Scheduled method (or anywhere where the ApplicationContext might not be started yet).  
It is initialized in a SmartLifecycle (with phase=0) so the earliest you can rely on it being available is in another SmartLifecycle with higher phase.

### Alternatives to the native Netflix DiscoveryClient
You don’t have to use the raw Netflix DiscoveryClient and usually it is more convenient to use it behind a wrapper of some sort.  
Spring Cloud has support for Feign (a REST client builder) and also Spring RestTemplate using the logical Eureka service identifiers (VIPs) instead of physical URLs.  
Spring Cloud支持Feign，同时也支持Spring RestTemplate，使用Eureke服务的逻辑标识符来代替物理URL。

To configure Ribbon with a fixed list of physical servers you can simply set <client>.ribbon.listOfServers to a comma-separated list of physical addresses (or hostnames), where <client> is the ID of the client.  
为了给Ribbon配置固定的物理服务列表，通过设置 <client>.ribbon.listOfServers，多个物理地址用逗号隔开，其中<client>是客户端的ID列表。

You can also use the org.springframework.cloud.client.discovery.DiscoveryClient which provides a simple API for discovery clients that is not specific to Netflix, e.g.  
你可以使用 org.springframework.cloud.client.discovery.DiscoveryClient，？？？
``` java
@Autowired
private DiscoveryClient discoveryClient;

public String serviceUrl() {
    List<ServiceInstance> list = client.getInstances("STORES");
    if (list != null && list.size() > 0 ) {
        return list.get(0).getUri();
    }
    return null;
}
```
### Why is it so Slow to Register a Service?
Being an instance also involves a periodic heartbeat to the registry (via the client’s  serviceUrl) with default duration 30 seconds.  
作为一个服务实例，周期性的和注册服务器保持心跳（通过Eureka客户端的serviceUrl），默认30秒。

A service is not available for discovery by clients until the instance, the server and the client all have the same metadata in their local cache (so it could take 3 hearbeats).  
一个服务不能被客户端发现，需要满足：实例、服务和客户端的本地缓存有相同的元数据（也就是说需要3次心跳）。

You can change the period using eureka.instance.leaseRenewalIntervalInSeconds and this will speed up the process of getting clients connected to other services.  
你可以改变默认周期，通过设置eureka.instance.leaseRenewalIntervalInSeconds属性来加快响应速度。

In production it’s probably better to stick with the default because there are some computations internally in the server that make assumptions about the lease renewal period.  
在生产环境中，最好保持默认值，因为有些计算内部的服务器对租赁更新做出假设。
