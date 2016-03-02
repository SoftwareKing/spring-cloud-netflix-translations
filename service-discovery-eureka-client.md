Service Discovery is one of the key tenets of a microservice based architecture.  
服务发现是微服务架构最重要的原则之一。

Trying to hand configure each client or some form of convention can be very difficult to do and can be very brittle.  
尝试配置每个客户端或者某种形式的约定一半很难去做并且是不可靠的（易碎的）。

Eureka is the Netflix Service Discovery Server and Client.  
Eureka是Netflix服务发现的服务端和客户端。

The server can be configured and deployed to be highly available, with each server replicating state about the registered services to the others.  
服务端是可配置的，而且可以部署为高可用的，通过在不同服务器间复制已注册服务的状态。
