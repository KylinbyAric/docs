Dubbo

1、易扩展 
多加节点就是了
2、易维护
3、高可用
4、可管理
服务治理、链路追踪、负载均衡



zkServer start-foreground

//🐂🍺
telnet ip port 
Ls  列出当前服务
dubbo>invoke 服务.方法（参数）直接调用服务。


通信协议：
dubbo:单一长链接 NIO异步 hessian序列化。   传输小(100kb以内)，并发高，适用消费者远大于提供者的情况
hessian：底层是http  hessian序列化 多个短链接 适用提供者大于消费者的情况。

均衡策略：
RandomLoadBalance：随机调用
RoundRobinLoadBalance：均衡轮循
LeastActiveLoadBalance：最小活跃优先
每个提供者都有一个active，每次调用进来+1，处理完-1。active数越小说明处理效率越高。
ConsistentHashLoadBalance：相同参数的请求一定分发在同一个provider上，provider挂掉之后会基于虚拟节点均分剩余的流量。

一致性hash：
所有的hash值组成一个环，服务节点分配在这些环上。如果请求是打到请求hash值的下一个(逆时针上的下一个)服务节点。
解决了什么问题？
解决了因为普通hash因为扩/缩容问题，导致同一个请求会请求到不同节点的问题。比如请求hash是100，总的服务节点是3，那么普通hash时请求的就是100%3=1号节点。但如果这时扩容，那么会请求到100%4=0号节点上。而一致性hash则只会影响到逆时针上下一个节点。
这是又会有一个新的问题，比如hash值是在[100,正无穷]都是请求到了第一台节点上，这会导致第一个节点请求过重，这叫数据倾斜。怎么解决？
虚拟节点。每一台实际的节点，都有若干个虚拟节点号均匀分配在环上，那么只要分配地够均匀，理论上就破除数据倾斜。



集群容错策略：
Failover Cluster 模式：自动重试其他机器，默认是该模式
<dubbo:service retries="2" />
或
<dubbo:reference retries="2" />
或
<dubbo:reference>
    <dubbo:method name="findFoo" retries="2" />
</dubbo:reference>

Failfast Cluster 模式：一次调用失败就立即失败，常见于非幂等性的写操作，比如新增一条记录（调用失败就立即失败）

Failsafe Cluster 模式：出现异常时忽略
<dubbo:service cluster="failsafe" />
或者
<dubbo:reference cluster="failsafe" />

Failback Cluster 模式：失败了后台自动记录请求，然后定时重发，比较适合于写消息队列这种。

Forking Cluster 模式：并行调用多个provider，哪个成功用哪个，适合实时性很高的操作。可通过 forks="2" 来设置最大并行数。

Broadcast Cluster 模式：逐个调用，任何一个出错则出错。通常用于通知所有提供者更新缓存或日志等本地资源信息。常用于通知所有提供者更新缓存或日志。

动态代理：
默认javaassist动态字节码，可以通过spi来配置自己动态代理策略。

spi:
spi就是根据配置——通常是一个文件，内容是key=类全限定路径名去找到对应的实现类，然后加载进来。
比较有名的就是jdbc就是用spi思想，只定义了接口类，没定义实现类。

dubbo中也是如此：
Protocol protocol = ExtensionLoader.getExtensionLoader(Protocol.class).getAdaptiveExtension();
在META-INFO/dubbo/intenal路径下就有很多spi配置文件，估计是会被一并加载，然后放到map里，以供查看

@SPI("dubbo")
public interface Protocol {

    int getDefaultPort();

    @Adaptive
    <T> Exporter<T> export(Invoker<T> invoker) throws RpcException;

    @Adaptive
    <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException;

    void destroy();

}
@SPI("dubbo")
意思就是在spi文件中找到key为simple的类然后注入使用
@Adaptive
这个意思就是接口会被代理实现
如果我们要自定义自己的protocol，需要怎么做呢？
自己写一个jar包，src/main/resources/META-INF/services(不是dubbo/internal????)下放一个com.alibaba.dubbo.rpc.Protocol文件，里面设置myProtocol=XXXX。
然后在provider工程中配置<dubbo:protocol name=”myProtocol” port=”20000” />即可。
其他的配置也可以通过spi方式扩展开来。


服务治理：
1、调用链路自动生成
要能自动将各个服务之间的依赖关系和调用链路生成出来
2、服务访问压力和时长统计
	1、一个是接口级，TP50/TP90/TP99
	TP50/TP90/TP99 
假设你统计了一段时间内访问系统的100次请求数据，他们的响应耗时分别是：1ms、2ms、3ms...100ms。
基于本次统计结果，你系统的 TP90，请求耗时从小到大排列，取第 100*0.9次请求耗时 ，为90ms（代表 90% 请求响应不超过90ms），类似地 TP50=50ms、TP99=99ms。

	2、完整调用级别，完整的调用需要几个服务？每天全链路走了多少？请求的TP50、TP90、TP99是多少。

3、其他
	服务分层：避免循环依赖
	失败监控和告警
	服务鉴权？？
	可用行？99%？几个9？

服务降级：比如retry失败之后，是不是需要mock一个数据回去？或者写一下自己的mock逻辑？


幂等性：
如何不重复扣款呢？或者说重复请求没什么副作用？
1、唯一索引
2、先查询后判断
3、token机制：每次请求前获取token，下次请求也加上token，后台验证，验证通过则删除token，下次请求再次判断token

####优雅下线
原理：是通过JVM的shutdownHook调用一些方法实现优雅下线
注：对于kill -9 pid 命令不生效。kill pid会生效
下线前做那些事：
提供方：
1、标记下线，不再接受请求，让客户端请求其他机器。关闭zk链接。
2、等待线程池的线程执行完，超时则强制关闭
调用方：
1、不再发送调用请求，在客户端就报错。
2、等待旧响应返回，超时强制关闭
总结：
1、服务从注册中心下线
2、停止接受新请求，旧请求可以正常返回数据。调用方直接报错
3、等待配置时间，然后强制关机。


网站博文：
1、https://cloud.tencent.com/developer/article/1854370























