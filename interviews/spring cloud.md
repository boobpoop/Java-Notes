
马丁弗勒--提出微服务的概念，提出熔断概念。

**Eureka**

spring cloud底层使用http协议通信。rpc基于自定义协议。都是应用曾协议。

传统服务调用：A直接调用B，它的坏处是当B宕机了，A不知道B是否宕机，还会去想B通信，这样会一直重传。

微服务调用：微服务启动时，发送一条数据给Eureka注册中心。注册中心维护一个Map，key表示微服务的名称，value是提供该微服务的具体ip相关信息。
其他微服务B需要调用该服务A时，从Eureka中获取A的服务地址ip，B就向A服务的一个ip发送请求。
通过注册中心Eureka，调用者就知道是否现在有机器可以进行访问。而不是直接向服务B访问。

Eruka实现：
- 在注册中心的启动类上加上@EnableEurekaServer。
- 在客户端的启动类上加入@EnableEurekaServer，将该微服务作为Eureka服务器的客户端（不加这个注解也行，Spring cloud自动加）。
- 在客户端的启动类上加入@EnableDiscoveryClient，那么该微服务可以作为任何注册中心的客户端。

@EnableErukaServer源码：
在项目中，@EnableEurekaServer注解被@Import(EurekaServerMarkerConfig.class)修饰，@Import是Spring注解，@Import用于引入配置类，将将配置类EurekaServerMarkerConfig中的@Bean方法的返回值Marker作为Bean进行注册到Spring容器中。

EurekaServerMarkerConfig的具体的类是定义在Eruka-server的jar包中，spring boot项目中，可以在项目的META-INF/spring.factories文件中添加 接口全路径名 = 实现类全路径名，那么当前的spring boot项目的spring容器会自动装配/注册这个bean。

但是在Eureka-server.jar中的EurekaServerMarkerConfig定义中，使用@ConditionalOnBean(EurekaServerMarkerConfig.Marker.class)的注解，即只有当spring容器中存在该bean时，才能装配EurekaServerMarkerConfig这个bean。

因此@EnableEukekaServer就是一个开关，加这个注解后，容器就可以得到Marker的bean，才能注入EurekaServer相关的配置类信息。不加这个注解，就会被@ConditionalOnBean拦截，不能装配Eureka的bean。

心跳连接/服务剔除/服务下架/服务注册/集群通信/自我保护机制都使用HTTP进行通信。底层使用Jersey代替SpringMVC。使用Filter代替DispatcherServlet。使用XxxResource代替XxxController。

服务注册流程：
外部的微服务在Eureka中的存储结构是：ConcurrentHashMap<String, Map<String, Lease<InstanceInfo>>>。其中第一个String代表的是微服务名，第二个String代表实例名，Lease表示实例的详细信息。

微服务名和实例名的区别：为了缓解单个微服务的并发压力，可以配置多个微服务A，B，C对外服务。A,B,C三个服务的配置中的微服务名spring.application.name字段相同，这个微服务名就是Eureka的存储结构的第一个String字段。第二个string是微服务集群中具体提供服务的机器，将每个机器对应一个实例Id，将Id作为key，机器的详细信息作为Lease信息存储。

当注册信息冲突时，将最后最活跃的微服务机器信息覆盖原来的信息，以防新结点异常无法访问。

心跳：默认心跳时间是30s，如果90s没收到某个微服务的心跳，就会把该服务剔除。

服务剔除：当前时间>最后更新时间+过期时间(90s)+Eureka集群同步时间时，认为该节点不可用，从map中剔除该节点。在剔除前，查看自我保护机制是否打开，如果15min，85%结点挂掉，Eureka就会认为是自己的网络出了问题，接收不到微服务的心跳，就开启自我保护机制，不剔除，当网络正常通讯时，就关闭自我保护机制。

集群信息同步：同步与注册都是一个方法：addInstance，但是同步时，方法的isPeplication为true，表示该请求为同步请求，对于同步请求，后续不会再将再发送同步信息，例如：A注册了节点信息，同步给B，B收到同步信息后执行同步，不会再将同步请求发给A了，不然就会发生死循环。


**CAP理论**

1.Consistency：集群中的所有数据必须一致。当一个主机的数据发生改变，那么其他主机必须等待完全同步好了才能向外提供服务。

- 一致性分类：

- 场景：集群中A，B两个节点中value都是1，修改A节点的value为2。此时A会向B发送同步信息，将B节点的value值会变成2.

- 1.强一致性：必须等到B结点的value从1同步到2时，B结点才能向外提供服务。Mysql就是强一致性
- 2. 最终一致性：访问B结点时value没有更新到2也可以，但必须保证最后B结点的value为2.
- 3.弱一致性：B最终时1或者2都没有关系。

2.Availability:任何时刻集群中的节点都可以对外提供服务。

3.Partition tolenrance：集群中的任意节点故障，集群仍能够向外提供服务。


**zookeeper**

zookeeper的一致性：在配置好zookeeper集群后，当一个微服务节点进行增删改查，会通过zab协议件数据同步到集群中的其他机器中。

过半机制：超过一半机器给同一台机器投票时，才会将该机器选举成Leader。

Leader选举机制(leader读写，follower读)

前提知识：
- zxid：zk的日志id，每个zkserver在接收到一条命令后，会在日志中增加一条记录，每条记录的zxid都是递增的，将日志持久化到磁盘中，再更新内存。是选举的一个依据。
- myid：zk服务器的配置信息，myid不能重复，代表每个服务器的重要成都，是选举的一个依据。

***选举过程：***
机器A，B，C，它们的zxid都是100，代表它们最初的状态相同，A，B，C的myid分别是1，2，3。最开始A启动，将投票给自己，后来B启动将投票投给自己，然后B向A发送自己的(myid,zxid)=(2,100)，机器A收到数据后，首先比较zxid，相同，后来比较myid，由于B的myid=2大于A的myid=1，因此A认为B机器更适合做Leader，因此给B投一票：向B发送(2,100)，B收到后，就得知A给B投票了，此时B的票数是2。由于此时2超过了3的一半，满足过半原则，因此此时选择B作为Leader。此时A成为Follower，B是Leader。C启动时，想A，B发自己的投票信息，A，B此时高速C，B是Leader，此时C就作为Follower。

特殊情况：如果A，B，C中B是Leader，并且接收了新请求，zxid变成101，在zxid同步前集群全挂，此时令A，C先启动，那么C就是Leader，B最后启动，B是follower，此时B的101号记录会回滚到101，也就是最开始的修改请求无效了，被丢弃了。如果在集群挂掉之前，已经将A的zxid同步到101，此时A，C先启动，A成为Leader，C是follower，B最后启动，B也是follower，请求没有被丢弃。最终的数据都以Leader结点为准。

zk服务器的连接建立过程：
A和B发送数据的时候会创建Socket连接，如果A和B同时启动时，可能同时建立两个连接，即A和B连接，B和A连接，这没有必要。zookeeper规定大的myid的主机建立连接成功，小的myid发起连接后，连接无效。

leader挂掉/follower挂掉一半也会进行重新选举，为何？2PC

***zab协议：2pc（信息同步过程）：***
- 1.Leader接收到修改操作，生成zxid++，并持久化。
- 2.然后Leader向Followers发送预提交日志，Followers会接收日志，并持久化。leader阻塞等待followers持久化完成，当一半以上的followers持久化完成。这是leader向下执行。
- 3.commit，leader更新内存，向其他followers节点发起commit请求，让followers更新内存。

由2pc的同步过程知道，follower必须超过一般节点修改zxid后，leader才会发送commit请求，如果挂了一般的节点，不能发送commit请求，此时内存中的zxid不能更新，必须重新选举。

为何zookeeper是最终一致性？
- 在2pc的第三步commit，Leader会将commit请求放到队列中，马上给客户端发送成功的消息。其他主机会从队列里面拿commit请求，再更新内存，因此follower内存更新是异步的。无法保持强一致性。

心跳：
Leader向followers节点发送自己最大的zxid。follower比较zxid，如果比leader中的小，就进行同步。

顺序读写：对于每个连接的去写请求，有一个sessionId对应，每次根据sessionId内的请求有序处理。

***为什么zookeeper的节点必须是奇数？***

解决脑裂问题，脑裂问题是指两个区域网络断开，如果没有过半原则。那么每个区域中会选举leader，每个区域都选取一个leader，当网络联通是，此时集群有两个leader，发生脑裂。

采用过半原则时就不会有脑裂问题。比如每个区域都是3个主机。这样每个区域都选不出leader，因为要选leader，必须有4个节点。因此zookeeper不能向外提供服务了。

采用过半原则时，如果是偶数的话，浪费主机，因为5和6台机器都是最多只能有2台主机断开，还能选举leader，对外提供服务。

zookeeper如何保持写的顺序性：



**Eureka与zookeeper比较**

- Eureka追求可用性，最终的数据是一致的，但是在一定的时间内Eruka集群还没同步好时，Eureka依然可以对外提供服务。因此Eruka性能较好。例如抢票时，显示有5张票，实际上已经没有票。AP。数据默认存在内存中。没有主从概念。
- Zookeeper是CP。是最终一致性。数据默认存到磁盘中。由于zab协议，zookeeper会保证一半以上的follower节点的内存中信息最新，而Eureka不保证其他节点是否最新。虽然都是最终一致性，但是很明显zookeeper比Eureka而言更可靠。


**Ribbon**

Ribbon是客户端负载均衡器。

Ribbon客户端根据指定的微服务获取到它的实例ip信息的list集合。



**Hystrix**

服务雪崩：

- 场景一：在微服务架构中，一个请求需要调用多个服务是非常常见的。如客户端访问 A 服务，而 A 服务需要调用 B 服务，B 服务需要调用 C 服务，由于网络原因或者自身的原因，如果 B 服务或者 C 服务不能及时响应，A 服务将处于阻塞状态，直到 B 服务 C 服务响应。此时若有大量的请求涌入，容器的线程资源会被消耗完毕，导致服务瘫痪。服务与服务之间的依赖性，故障会传播，造成连锁反应，会对整个微服务系统造成灾难性的严重后果，场景：

- 场景二：当用户调用微服务A，微服务A调用微服务B，此时如果B对A响应超时，那么A会一直在运行，等待B的返回结果，将返回结果进行处理返回给客户端。从客户端来看，调用着A服务，A服务却响应很慢，会认为A服务出现了异常。其实此时A服务没有异常，只是B服务发生来异常，导致A服务响应慢。令微服务提供的服务需要运行3s，当2w个请求同时到一台微服务的服务器上，其他的请求过来，该请求的响应很慢，服务会卡死。

现象：在微服务的相互调用中，一旦一个服务出错，那么调用该服务的链路上的服务可能都会产生影响。

解决方法：

- 超时不再等待、宕机/出错要有兜底：设置微服务B自身调用超时时间的峰值。峰值内可正常响应；超过峰值时，使用fallback降级。

-- 实现方法：在被调用的请求方法上加@HystrixCommand(fallbackMethod="自定义方法名", 超时时间maxTime)。当超过maxTime时，或者B服务发生异常时，都会调用自定义方法，将出错信息返回给用户，这样用户不用等待太久，就可以得到响应。

Hystrix如何才会降级？
- 1.程序运行发生异常。
- 2.程序响应超时。
- 3.服务发生熔断。
- 4.线程池/信号量打满。

配置Hystrix：

一般降级都是在客户端的。在yml或者application的配置文件汇总配置feign.hystrix.enable = true。在启动类上加上@EnableHystrix。

上述服务降级的实现方法有两个重大缺点：
- 1.每个提供服务的方法都要配置兜底的服务降级fallbackMethod，这样代码会产生冗余情况。如果提供的服务有100个，那么要在controller中定义100个降级方法，太冗余。
- 2.controller中的业务代码和降级方法放在一个类中，使得代码耦合度过高，可读性差。

优化方法：
- 1.使用@DefaultProperties(defaultFallback="global自定义方法")修饰Controller类。这样controller类中每个方法只要加了@HystrixCommand，就默认使用defaultFallback内的降级方法。这种方法还是需要在controller类中配置全局的服务降级方法+特定的服务降级方法方法，使业务方法和降级方法耦合在一起。
- 2.在@FeignClient注解中加入fallback参数，构造一个Feign接口的实现类，实现类的方法就是服务的降级方法，将实现类配置到fallback参数中。这样当调用服务B时，一旦服务B异常，就会调用接口实现类的降级方法。这解决类代码冗余，代码耦合的问题。


**短路器/熔断器**

在@HystrixCommand中使用HystrixProperty属性，设置如果10个请求的失败率达到30%，就将服务进行熔断，以后每次调用该微服务，直接调用降级方法，而不是业务方法，直到过了5s后允许少量请求访问，尝试半开放，看此时微服务是否可用，如果请求成功，从半开到闭合。

配置方法：

在启动类上加入@EnableCircuitBreaker注解开启熔断器。



**RabbitMQ**

接口幂等问题：
- 场景：微服务A访问微服务B，微服务B收到A的请求会修改数据库。
- 问题：A和B使用HTTP连接，而HTTP连接的传输层使用TCP，TCP有超时重传机制。当微服务A向微服务B发起请求时，B需要话费大量时间处理该请求，因此微服务B没有在A的指定范围内返回响应，此时A认为响应已经超时，会重传HTTP请求。由于TCP的重传机制，会导致一个修改请求多次发送，这样数据库会多次修改，这就是接口幂等问题，这就是TCP的超时重传的弊端。
- 解决办法：在请求中添加两个字段：全局唯一的userId和自增的version。同时在数据库中添加这两个字段。
-- userId作用：对于接口是用于插入操作的，当多个同样的插入操作到数据库时，查询是否已经存在该userId，如果userId已存在，说明该请求是无用的重复请求。
-- ABA问题：对于接口是用于修改操作的，当请求A1请求修改数据库时，此时超时了，在重传之前，请求A2请求修改同一条数据，修改成功，此时重传请求过来，又将数据改为来了，这就是ABA的问题。(删除其实就是修改is_active字段)。
-- version作用：为了解决ABA问题，使用自增的version区分修改的次序，例如A1的原请求和重传请求的version都是1，新请求A2的version会自增变成2，数据库的version字段会变成2，每次修改时，如果数据库的version大于当前请求的version，该请求被认为无效请求。



MQ与多线程的取舍？服务雪崩？：
- 场景：对于高并发场景，即同时有1w+请求到达服务B，每个请求都会创建一个线程，而64位的服务器线程数最多位7000左右。
- 问题：B处理每个请求需要耗费大量时间，当前7000个请求还没响应，后面的请求由于申请不到线程资源而阻塞，请求会一直等待。这就是服务雪崩问题。
- 底层分析：由于创建的线程过多，这些线程会竞争CPU资源，CPU会频繁切换线程，每个线程在切换时要保存线程，需要将线程数据存到使用CPU的寄存器中，这样线程多时，寄存器资源也不够用，服务会很卡。
- 解决方法：使用消息中间件，将A的请求放到RabbitMQ中，B从MQ中取请求，这样A和B的服务就进行解耦了。同时高并发也不会打到B中，而是到达消息中间件中，由于消息中间件只负责存储请求，因此会很快就响应A，服务B从MQ中拿消息，虽然每次处理都很慢，但是由于每次处理完一个请求，才会取下一个请求，因此不会产生高并发问题。


消息队列满了，如何解决？
- 1.队列满了，说明消费能力很低，此时可以将消费者做成集群。每个消费者通过offset取到唯一的请求。
- 2.消息持久化到硬盘中。

MQ的幂等问题：
- 消费者启动时，会从MQ中拉取一个请求，启动后，都是MQ想消费者推送消息的。消费者从MQ中取出一个请求，如果在MQ指定的时间内消费完成并响应给MQ，MQ就会从队列中删除该请求。如果消费者没有在指定时间内消费成功，就会重新向消费者发送该请求，产生幂等问题。
- 解决方法：对每个消息加上messageId，如果已经存在该messageId，就认为是该消息是重传的，作为无效化处理。

MQ消息顺序问性：
- 场景：三个消息分别是create,update,delete操作，如果是消费者是集群，那么可能三台机器分别拿到了这三条信息。这样update操作可能在create操作之前发生发生，因此破坏了原有逻辑的顺序性。
- 解决方法：对具有顺序性的消息分区，将分区的消息放到一个消费者上。






