# RPC
## 1.概念
  1. RPC（Remote Procedure Call）—远程过程调用，通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。
  2. 为了解决不同服务之间的调用问题，让分布式或者微服务系统中不同服务之间的调用像本地调用一样简单， 一般会包含传输协议和序列化协议两种。
  3. RPC框架可以使用 HTTP协议(应用层协议，基于TCP/IP通信协议)作为传输协议或者直接使用TCP(传输层协议，面向连接的，可靠的)作为传输协议，使用不同的协议一般也是为了适应不同的场景。
  4. 与HTTP协议的区别，就HTTP1.1而言，RPC采用自定义的tcp传输协议能够减少报文头所占用的字节数，极大的精简了传输内容；就HTTP2.0已经优化编码效率问题，因此rpc更多的是封装了服务发现、负载均衡、熔断降级、可视化服务治理等面向服务的高级特性。
## 2.原理
  1. 服务消费端(client)以本地调用的方式调用远程服务；
  2. 客户端(client stub)接收到调用后负责将方法、参数等组装成能够进行网络传输的消息体(序列化)：RpcRequest；
  3. 客户端(client stub)找到远程服务的地址，并将消息发送到服务提供端；
  4. 服务端Stub收到消息将消息反序列化为Java实体类对象：RpcRequest；
  5. 服务端Stub根据RpcRequest中的类、方法、方法参数等信息调用本地的方法；
  6. 服务端Stub得到方法执行结果并将将其序列化:RpcResponse发送至消费方；
  7. 客户端Stub接收到消息并将其反序列化为Java实体类对象：RpcResponse，至此得到最终结果
![image](https://user-images.githubusercontent.com/41152743/140736645-9d59152c-c279-4810-bb69-6478923c87ad.png)

重要点：
    服务注册发布与订阅、远程通信协议与序列化、RPC调用方式(同步 Sync、异步 Future、回调 Callback和单向 Oneway)、线程模型、负载均衡、动态代理
 ## 3.Dubbo
 具体文档见官网：https://dubbo.apache.org/zh/docs/v2.7/
 ### 3.1 解决的问题
    分布式服务架构下，系统被拆分成不同的服务，每个服务独立提供系统的某个核心服务，当服务越来越多，服务调用关系越来越复杂。当应用访问压力越来越大后，需要进行负载均衡以及服务监控。
    
  1. 负载均衡 ： 同一个服务部署在不同的机器时该调用哪一台机器上的服务。
  2. 服务调用链路生成：解决服务之间互相是如何调用的。
  3. 服务访问压力以及时长统计、资源调度和治理：基于访问压力实时管理集群容量，提高集群利用率
  4. 服务降级  
### 3.2 Dubbo的核心角色
  ![image](https://user-images.githubusercontent.com/41152743/140850790-da176bf3-11a4-492f-a5ae-0394279e24e0.png)
  
  注：微内核架构
  微内核架构模式(有时称作插件架构模式)基于产品应用程序的一种自然模式，基于产品的应用程序是已经打包好并且拥有不用版本，可作为三方插件下载的，
  包括核心系统和插件模块，核心系统提供系统所需核心能力，插件模块可以扩展系统的功能
  ![image](https://user-images.githubusercontent.com/41152743/140851199-b2bef217-b277-4bfa-99f4-3a67d0ded0fc.png)

### 3.3 Invoker
    Invoker 就是 Dubbo 对远程调用的抽象，分为服务提供Invoker和服务消费Invoker，调用远程方法时需要动态代理来屏蔽远程调用的细节，这些细节依赖于Invoker的实现，Invoker实现了真正的远程服务调用。
    
![image](https://user-images.githubusercontent.com/41152743/140851891-74b0fc4d-d8e1-4808-bdd0-16b6e18b5cee.png)
### 3.4 URL
  一个标准的URL的格式如下：
  
    protocol://username:password@host:port/path?key=value&key=value
    其中protocol：URL 的协议，常见的就是 HTTP 协议和 HTTPS 协议;
    username/password：用户名/密码；
    host/port：主机/端口，在实践中一般会使用域名，而不是使用具体的 host 和 port；
    path：请求的路径；
    parameters：参数键值对。一般在 GET 请求中会将参数放到 URL 中，POST 请求会将参数放到请求体中。
1. url在服务暴露中的应用(provider)

    org.apache.dubbo.registry.zookeeper.ZookeeperRegistry#doRegister
    
    provider:
    
        dubbo%3A%2F%2F10.144.200.140%3A20881%2Fcom.xxx.service.xxxService%3F
       
        anyhost%3Dtrue%26
       
       application%3Dxxx-provider%26
       
       deprecated%3Dfalse%26
       
       dispatcher%3Dmessage%26
       
       dubbo%3D2.0.2%26
       
       dynamic%3Dtrue%26
       
       generic%3Dfalse%26
       
       interface%3Dcom.xxx.service.xxxService%26
       
       methods%3DgetXxx%2C%2CfindXxx%26
       
       pid%3D38399%26
       
       release%3D2.7.6%26
       
       revision%3D2.0.0%26
       
       side%3Dprovider%26
       
       threads%3D500%26
       
       timeout%3D3000%26
       
       timestamp%3D1635855876799%26
       
       version%3D2.0.0,
    
2. 在订阅服务中的应用(Consumer)

    org.apache.dubbo.registry.zookeeper.ZookeeperRegistry#doSubscribe
    
     consumer:
     
        consumer%3A%2F%2F10.160.4.98%2Fcom.xxx.service.xxxService%3F
       
        application%3Dxxx-consumer%26
       
       category%3Dconsumers%26
       
       check%3Dfalse%26
       
       dubbo%3D2.0.2%26
       
       init%3Dfalse%26
       
       interface%3Dcom.xxx.xxxService%26
       
       methods%3DgetXxx%2C%2CfindXxx%26
       
       pid%3D252427%26
       
       release%3D2.7.6%26
       
       revision%3D2.0.0%26
       
       side%3Dconsumer%26
       
       sticky%3Dfalse%26
       
       timestamp%3D1626745959797%26
       
       version%3D2.0.0
3. dubbo在zk中的存储形式
  详见：https://dubbo.apache.org/zh/docs/v2.7/user/references/registry/zookeeper/

![image](https://user-images.githubusercontent.com/41152743/141222348-31a33809-d6d1-44e8-9314-f374f9acdd5a.png)

    Root层：根目录，通过<dubbo:registry group="dubbo" />的“group”来设置，用于服务注册分组，跨组的服务不会相互影响，也无法相互调用，适用于环境隔离；
    Service层：服务接口的全名；
    Type层：共4类，分别是providers（服务提供者列表）、consumers（服务消费者列表）、routers（路由规则列表）、configurators（配置规则列表）;
    URL层：根据不同的Type目录，有不同的URL;
    流程如下：
      服务提供者启动时: 向 /dubbo/com.foo.BarService/providers 目录下写入自己的 URL 地址
      服务消费者启动时: 订阅 /dubbo/com.foo.BarService/providers 目录下的提供者 URL 地址。并向 /dubbo/com.foo.BarService/consumers 目录下写入自己的 URL 地址
      监控中心启动时: 订阅 /dubbo/com.foo.BarService 目录下的所有提供者和消费者 URL 地址。
### 3.5 dubbo的SPI
#### 1.JDK的spi机制

   SPI（Service Provider Interface）框架开发人员使用的一种技术,将服务接口与服务实现分离以达到解耦、大大提升了程序可扩展性的机制。引入服务提供者就是引入了spi接口的实现者，通过本地的      注册发现获取到具体的实现类，轻松可插拔。
   
     1. 当服务的提供者提供了一种接口的实现之后，需要在 Classpath 下的 META-INF/services/ 目录里创建一个以服务接口命名的文件，此文件记录了该 jar 包提供的服务接口的具体实现类。
     2. JDK SPI 机制通过查找这个 jar 包的 META-INF/services/ 中的配置文件来获得具体的实现类名，进行实现类的加载和实例化，最终使用该实现类完成业务功能
     3. java.util.ServiceLoader#load获取ServiceLoader对象,
        java.util.ServiceLoader#iterator，迭代器延迟加载并实例化ServiceLoader对象的服务提供集合
          java.util.ServiceLoader.LazyClassPathLookupIterator#hasNextService，负责查找META-INF/services 目录下的 SPI 配置文件，并负责实例化读取到的实现类放到缓存中
          java.util.ServiceLoader.LazyClassPathLookupIterator#nextService，负责获取缓存中实例化的实现类  
  JDK SPI 在 JDBC 中的应用
  ![image](https://user-images.githubusercontent.com/41152743/141269856-30ed9d9d-a0c7-4d05-bffe-f44ef5844ba6.png)

  1. 在java.sql.DriverManager#ensureDriversInitialized中实例化com.mysql.cj.jdbc.Driver 对象，并注册到 DriverManager.registeredDrivers 集合中。
  2. java.sql.DriverManager#getConnection从registeredDrivers 集合中获取对应的 Driver 对象创建 Connection。
  
  缺点：
    JDK SPI 在查找扩展实现类的过程中，需要遍历 SPI 配置文件中定义的所有实现类，该过程中会将这些实现类全部实例化，如果配置了多个实现类而只需要使用其中一个实现类时，
    就会生成不必要的对象，导致资源的浪费。
   
#### 2.Dubbo的spi机制
  
  ![image](https://user-images.githubusercontent.com/41152743/141287912-54dbca44-fe13-4b18-b93c-d8fddc313ef6.png)

1. @SPI注解

    Dubbo SPI 不仅解决了JDK SPI资源浪费的问题，还对 SPI 配置文件扩展和修改。
    
        首先：Dubbo 按照 SPI 配置文件的用途，将其分成了三类目录：
          META-INF/services/ 目录：该目录下的 SPI 配置文件用来兼容 JDK SPI 。
          META-INF/dubbo/ 目录：该目录用于存放用户自定义 SPI 配置文件。
          META-INF/dubbo/internal/ 目录：该目录用于存放 Dubbo 内部使用的 SPI 配置文件。
      然后：SPI 配置文件改成了 KV 格式：
          其中key被称为扩展名，不仅可以指定扩展名来选择相应的扩展实现，还可以更容易定位到问题。
      使用：通过注解@SPI 表明该接口是扩展接口,value指定了默认的扩展名称(通过 Dubbo SPI 加载接口实现时，如果没有明确指定扩展名，则默认会将@SPI注解的value 值作为扩展名)
      原理：dubbo-common 模块中的 extension 包中的ExtensionLoader用于解析处理@SPI注解 
  具体原理：
  
      1. org.apache.dubbo.common.extension.ExtensionLoader#getExtensionLoader，从 EXTENSION_LOADERS 缓存中查找相应的 ExtensionLoader 实例；
      
      2. org.apache.dubbo.common.extension.ExtensionLoader#getExtension，在获取到的ExtensionLoader 实例中，根据传入的扩展名称从 cachedInstances 缓存中查找扩展实现的实例，
         如果未找到，则将其实例化返回。调用：org.apache.dubbo.common.extension.ExtensionLoader#createExtension
         该方法完成了 SPI 配置文件的查找以及相应扩展实现类的实例化，同时还实现了自动装配以及自动 Wrapper 包装等功能
 
2. @Adaptive 注解与适配器：
 
      例如，Dubbo中，ExtensionFactory 接口上有 @SPI 注解，AdaptiveExtensionFactory 实现类上有 @Adaptive 注解，该类不实现任何具体的功能，用于适配 ExtensionFactory 的 SpiExtensionFactory 和 SpringExtensionFactory 这两种实现，根据运行时的一些状态选择具体调用哪个实现。
      1. org.apache.dubbo.common.extension.ExtensionLoader#loadClass，识别扩展实现类上的@Adaptive 注解，并将其类型缓存到cachedAdaptiveClass 这个实例字段上；
      2. org.apache.dubbo.common.extension.ExtensionLoader#getAdaptiveExtensionClass，获取适配器实例，并将该实例缓存到cachedAdaptiveInstance 字段（Holder类型）中

3. 自动包装与装配特性

  自动包装：
  
    Dubbo 中的一个扩展接口可能有多个扩展实现类，如果扩展类中包含一些相同的逻辑，在每个实现类中都写一遍，重复代码难以维护，提供自动包装特性，将多个扩展实现类的公共逻辑，抽象到Wrapper类中，Wrapper 类与普通的扩展实现类一样，也实现了扩展接口，在获取真正的扩展实现对象时，在其外面包装一层 Wrapper 对象。
    
    1. org.apache.dubbo.common.extension.ExtensionLoader#loadClass，判断该扩展实现类是否包含拷贝构造函数（即构造函数只有一个参数且为扩展接口类型），如果包含，则为 Wrapper 类。
    
    2. 将该类缓存到cachedWrapperClasses（Set<Class<?>>类型）这个实例字段中，然后使用时遍历全部 Wrapper 类并一层层包装到真正的扩展实例对象外层。
   
 自动装配：org.apache.dubbo.common.extension.ExtensionLoader#injectExtension

        Dubbo SPI 在拿到扩展实现类的对象（以及 Wrapper 类的对象）后调用该方法，扫描其全部setter方法，
        根据setter方法的名称及参数的类型加载相应的扩展实现，
        然后调用相应的 setter 方法填充属性。
        即自动装配属性指的是在加载一个扩展接口时，将其依赖的扩展接口一并加载，并进行装配。

4. @Activate注解与自动激活特性

    以Dubbo中的Filter为例子，在某些场景中可能需要几个Filter扩展实现类协调工作，某些场景可能需要另外几个Filter扩展实现类协同工作，此时需要指定当前场景中哪些filter实现是可用的，即需要用到@Active注解。
    
            @Activate 注解标注在扩展实现类上，有 group、value 以及 order 三个属性：
              group 属性：修饰的实现类是在 Provider 端被激活还是在 Consumer 端被激活。
              value 属性：修饰的实现类只在 URL 参数中出现指定的 key 时才会被激活。
              order 属性：用来确定扩展实现类的排序
   1. org.apache.dubbo.common.extension.ExtensionLoader#loadClass，将包含@Activate注解的实现类缓存到cachedActivates集合中；
   2. ExtensionLoader#getActivateExtension，使用cachedActivates集合，首先获取激活的扩展集合，需要满足以下条件：
        1. 在 cachedActivates 集合中存在；
        2. @Activate 注解指定的 group 属性与当前 group 匹配；
        3. 扩展名没有出现在 values 中（即未在配置中明确指定，也未在配置中明确指定删除）；
        4. URL 中出现了 @Activate 注解中指定的 Key。
      然后，按照注解中的order属性对激活的扩展集合进行排序，最后，按序添加自定义扩展实现类的对象，自定义的扩展添加到默认扩展集合后面

#### 3.Dubbo的时间轮
  1. 时间轮：一种高效的、批量管理定时任务的调度模型。

    环形结构，类似于时钟，分为很多槽，一个槽代表一个时间间隔，每个槽使用双向链表存储定时任务。
    指针周期性的跳动，跳动到一个槽位，就执行该槽位的定时任务。
    单层时间轮的容量和精度都是有限的，对于精度要求特别高、时间跨度特别大或是海量定时任务需要调度的场景，通常会使用多级时间轮以及持久化存储与时间轮结合

  2. Dubbo的时间轮：dubbo-common 模块的org.apache.dubbo.common.timer

  核心接口：

      TimerTask 接口：所有的定时任务需要实现该接口，定义了一个run()方法，入参是TimeOut对象；
      Timeout接口：通过Timeout对象，可以查看定时任务的状态、操作定时任务，与TimeTask对象一一对应；
      Timer接口：定义了定时器的基本行为，核心方法是newTimeout()，提条一个定时任务(TimeeTask)并返回关联的Timeout对象。
  具体实现：

    HashedWheelTimeout:Timeout 接口的唯一实现，是 HashedWheelTimer 的内部类,主要用于：

      1. 时间轮中双向链表的节点，即定时任务 TimerTask 在 HashedWheelTimer 中的容器。
      2. 定时任务 TimerTask 提交到 HashedWheelTimer 之后返回的句柄（Handle），用于在时间轮外部查看和控制定时任务。

    HashedWheelBucket：HashedWheelTimer 的内部类，是时间轮中的一个槽，主要用于：

      1. 用于缓存和管理双向链表的容器，双向链表中的每一个节点就是一个 HashedWheelTimeout 对象，也就关联了一个 TimerTask 定时任务
    
   HashedWheelTimer：Timer 接口的实现，它通过时间轮算法实现了一个定时器。
      
      1. 根据当前时间轮指针选定对应的槽（HashedWheelBucket）；
      2. 从双向链表的头部开始迭代，对每个定时任务（HashedWheelTimeout）进行计算，属于当前时钟周期则取出运行，不属于则将其剩余的时钟周期数减一操作。
  
    原理：
      1. org.apache.dubbo.common.timer.HashedWheelTimer#newTimeout：用于提交定时任务，
          首先确定时间轮的 startTime 字段；
          然后启动 workerThread 线程，开始执行 worker 任务；
          最后根据startTime 计算该定时任务的 deadline 字段，并将定时任务封装成 HashedWheelTimeout 并添加到 timeouts 队列。
      2. org.apache.dubbo.common.timer.HashedWheelTimer.Worker#run：执行定时任务
          1. 时间轮指针转动，时间轮周期开始。
          2. 清理用户主动取消的定时任务，该任务记录到 cancelledTimeouts 队列中。在每次指针转动的时候，时间轮都会清理该队列。
          3. 将缓存在 timeouts 队列中的定时任务转移到时间轮中对应的槽中。
          4. 根据当前指针定位对应槽，处理该槽位的双向链表中的定时任务。
          5. 检测时间轮的状态。如果时间轮处于运行状态，则循环执行上述步骤，不断执行定时任务。如果时间轮处于停止状态，则执行下面的步骤获取到未被执行的定时任务并加入            unprocessedTimeouts 队列：遍历时间轮中每个槽位，并调用 clearTimeouts() 方法；对 timeouts 队列中未被加入槽中循环调用 poll()。
          6. 最后再次清理 cancelledTimeouts 队列中用户主动取消的定时任务。
    应用：
    在 Dubbo 中，时间轮并不直接用于周期性操作，而是只向时间轮提交执行单次的定时任务。
    在上一次任务执行完成的时候，调用 newTimeout() 方法再次提交当前任务，这样就会在下个周期执行该任务。
    即使在任务执行过程中出现了GC、IO阻塞等情况导致任务延迟或卡住，也不会导致任务堆积。
      1. 失败重试：例如：Provider 向注册中心进行注册失败时的重试操作，
      2. 周期性定时任务：例如：定期发送心跳请求，请求超时的处理

#### 4.Dubbo的Registry
1. 相关接口

    Node接口：表示 Provider 和 Consumer 节点，以及注册中心节点；
    RegistryService 接口：抽象注册了服务的基本行为
![image](https://user-images.githubusercontent.com/41152743/144981690-5f25ba08-b259-4888-bc1a-dcda3c51610b.png)
    Registry 接口：表示的就是一个拥有注册中心能力的节点，继承了 RegistryService 接口和 Node 接口；
    RegistryFactory 接口： Registry 的工厂接口，负责创建 Registry 对象；
    RegistryFactoryWrapper：RegistryFactory 接口的 Wrapper 类，会将register()、subscribe() 等事件通知到 RegistryServiceListener 监听器。
2. AbstractRegistry-本地缓存
![image](https://user-images.githubusercontent.com/41152743/144987755-04bc5054-659a-45cd-9e72-1a9e1f0cc946.png)

    1.org.apache.dubbo.registry.support.AbstractRegistry#notify(org.apache.dubbo.common.URL, org.apache.dubbo.registry.NotifyListener, java.util.List<org.apache.dubbo.common.URL>) 
    
        第一个URL参数表示的是Consumer，第二个NotifyListener是第一个参数对应的监听器，第三个参数是Provider端暴露的URL的全量数据
        当 Provider 端暴露的 URL 发生变化时，ZooKeeper 等服务发现组件会通知 Consumer 端的 Registry 组件，Registry 组件会调用 notify() 方法，
        被通知的 Consumer 能匹配到所有 Provider 的 URL 列表并写入 properties 集合中
    2. org.apache.dubbo.registry.support.AbstractRegistry#getCacheUrls
    
        在网络抖动等原因而导致订阅失败时，Consumer 端的 Registry 就可以调用 getCacheUrls() 方法获取本地缓存，从而得到最近注册的 Provider URL；
        AbstractRegistry 通过本地缓存提供了一种容错机制，保证了服务的可靠性。
    3. 注册/订阅、恢复/销毁，操作当前节点要注册的 URL 缓存到的registered 集合    
 2. org.apache.dubbo.registry.support.FailbackRegistry-重试机制
 
    1. org.apache.dubbo.registry.support.FailbackRegistry#register
      
        1. 根据 registryUrl 中 accepts 参数指定的匹配模式，决定是否接受当前要注册的 Provider URL；
        2. 调用父类 AbstractRegistry 的 register() 方法，将 Provider URL 写入 registered 集合中；
        3. 调用 removeFailedRegistered() 方法和 removeFailedUnregistered() 方法，将该 Provider URL 从 failedRegistered 集合和 failedUnregistered 集合中删除，并停止相关的重试任务；
        4. 调用 doRegister() 方法，由子类实现，每个子类只负责接入一个特定的服务发现组件。
        5. 在 doRegister() 方法出现异常的时候，判断是否直接抛出异常，否的话，创建重试任务并添加到 failedRegistered 集合中，然后提交到时间轮中。
 3.  ZooKeeper 注册中心实现-官方推荐
    
    1. ZookeeperRegistryFactory：
![image](https://user-images.githubusercontent.com/41152743/145003821-0d20a4ac-8962-4660-9885-400a39ef8154.png)
    
    2. ZookeeperTransporter---org.apache.dubbo.remoting.zookeeper.ZookeeperTransporter
    
      只负责一件事情，那就是创建 ZookeeperClient 对象，其中AbstractZookeeperTransporter 的核心功能：
          缓存 ZookeeperClient 实例；
          在某个 Zookeeper 节点无法连接时，切换到备用 Zookeeper 地址。
    3. ZookeeperClient： Dubbo 封装的 Zookeeper 客户端，该接口定义了大量的方法，都是用来与 Zookeeper 进行交互的。
          其中,AbstractZookeeperClient 作为 ZookeeperClient 接口的抽象实现:
           1. 缓存当前 ZookeeperClient 实例创建的持久 ZNode 节点；
           2. 管理当前 ZookeeperClient 实例添加的各类监听器,包括三种类型的监听器：
                StateListener(负责监听Dubbo 与 Zookeeper 集群的连接状态)、
                DataListener(主要监听某个节点存储的数据变化) 、
                ChildListener (主要监听某个 ZNode 节点下的子节点变化。)；
           3. 管理当前 ZookeeperClient 的运行状态。
    4. ZookeeperRegistry：org.apache.dubbo.registry.zookeeper.ZookeeperRegistry#ZookeeperRegistry
           1. 通过 ZookeeperTransporter 创建 ZookeeperClient 实例并连接到 Zookeeper 集群，同时还会添加一个连接状态的监听器；
           2. doRegister() 方法和 doUnregister() 方法的实现都是通过 ZookeeperClient 找到合适的路径，然后创建（或删除）相应的 ZNode 节点，并根据dynamic参数决定是临时还是持久节点
    ![image](https://user-images.githubusercontent.com/41152743/145134281-dfd44b8a-78d0-4720-a5d3-0fb2354608d7.png)
           3. doSubscribe()方法：
              通过 ZookeeperClient 在指定的 path 上添加 ChildListener 监听器，当订阅的节点发生变化时，会通过 ChildListener 监听器触发 notify() 方法，在 notify() 方法中会触发传入的 NotifyListener 监听器。

#### 5.Dubbo的Remoting：Exchange、Transport和Serialize (dubbo-remoting-api 模块)
1.相关接口
    
     1. buffer包：定义了缓冲区相关的接口、抽象类以及实现类；
     2. exchange包：抽象了 Request 和 Response 两个概念，并为其添加了很多特性；
     3. transport 包：对网络传输层的抽象，但它只负责抽象单向消息的传输；
     4. 其他接口：Endpoint、Channel、Transporter、Dispatcher 等顶层接口，该模块的核心接口
2. 传输层核心接口

     1. Endpoint：
        通过一个 ip 和 port 唯一确定一个端点，两个端点之间会创建 TCP 连接，可以双向传输数据；
        dubbo将Endpoint 之间的 TCP 连接抽象为通道（Channel），发起请求的 Endpoint 抽象为客户端（Client），将接收请求的 Endpoint 抽象为服务端。
     2. channel接口：继承了 Endpoint 接口，也具备开关状态以及发送数据的能力；另一个是可以在 Channel 上附加 KV 属性。
     3. ChannelHandler接口：注册在 Channel 上的消息处理器，可以处理 Channel 的连接建立以及连接断开事件，还可以处理读取到的数据、发送的数据以及捕获到的异常
     4. Client 和 RemotingServer 两个接口：分别抽象了客户端和服务端，两者都继承了 Channel、Resetable 等接口，也就是说两者都具备了读写数据能力。
     5. Transporter 接口： 在 Client 和 Server 之上又封装了一层Transporter 接口，从而确定使用何种nio库，例如：Netty、Mina、Grizzly ；
     6. Transporters类：门面类，其中封装了 Transporter 对象的创建（通过 Dubbo SPI）以及 ChannelHandler 的处理
3. Transport层

    1. AbstractPeer：同时实现了 Endpoint 接口和 ChannelHandler 接口，也是 AbstractChannel、AbstractEndpoint 抽象类的父类

![image](https://user-images.githubusercontent.com/41152743/145322602-d4dc3abb-8940-4215-b349-b6dcd1d7fec1.png)
          AbstractChannel、AbstractServer、AbstractClient 都是要关联一个 ChannelHandler 对象的。
          
    2. AbstractServer 是对服务端的抽象，实现了服务端的公共逻辑。
 ![image](https://user-images.githubusercontent.com/41152743/145171443-98b585fd-c1b8-420b-bb7b-42a90f302c8d.png)
 
            1. org.apache.dubbo.remoting.transport.AbstractServer#AbstractServer，构造方法中根据url初始化参数，调用子类的doOpen()方法完成server的启动。
            2. 例如：org.apache.dubbo.remoting.transport.netty4.NettyServer#doOpen NettyServer的启动，主要是添加四个核心channelHandler：
                1. InternalDecoder、InternalEncoder；
                2. IdleStateHandler：Netty 提供的一个工具型 ChannelHandler，用于定时心跳请求的功能或是自动关闭长时间空闲连接的功能
                3. NettyServerHandler：继承了 ChannelDuplexHandler，Netty 提供的一个同时处理 Inbound 数据和 Outbound 数据的 ChannelHandler
                
    3. AbstractClient是对客户端的封装   
![image](https://user-images.githubusercontent.com/41152743/145320464-e2774f2d-d235-4428-98d9-141d174f3aaa.png)

          1. org.apache.dubbo.remoting.transport.AbstractClient#AbstractClient，构造方法中解析url中的参数初始化executor，调用子类的doOpen()方法完成client端的启动
          2. 例如：org.apache.dubbo.remoting.transport.netty4.NettyClient#doOpen，添加四个核心的channelHandler
                 1. InternalDecoder、InternalEncoder；
                 2. IdleStateHandler：Netty 提供的一个工具型 ChannelHandler，用于定时心跳请求的功能或是自动关闭长时间空闲连接的功能
                 3. NettyClientHandler：继承了 ChannelDuplexHandler，Netty 提供的一个同时处理 Inbound 数据和 Outbound 数据的 ChannelHandler
          3.创建底层连接，调用子类的doConnect()方法
          
    4. channel继承路线
      
          1. org.apache.dubbo.remoting.transport.netty4.NettyChannel
![image](https://user-images.githubusercontent.com/41152743/145323269-4a8854f2-e7b6-4669-9925-dbe4d6c8587b.png)
            org.apache.dubbo.remoting.transport.netty4.NettyChannel#send：通过底层关联的 Netty 框架 Channel，将数据发送到对端。
            
    5. channelhandler继承路线
        
          1. ChannelHandlerDispatcher类：负责将多个 ChannelHandler 对象聚合成一个 ChannelHandler 对象。
          2. ChannelHandlerAdapter类： ChannelHandler 的一个空实现
          3. ChannelHandlerDelegate接口： 继承ChannelHandler接口，对ChannelHandler 对象的封装，它的两个实现类 AbstractChannelHandlerDelegate 和 WrappedChannelHandler 
          4. AbstractChannelHandlerDelegate有三个实现类：
            1. MultiMessageHandler：专门处理 MultiMessage 的 ChannelHandler 实现，
            2. DecodeHandler：专门处理 Decodeable 的 ChannelHandler 实现
            3. HeartbeatHandler：专门处理心跳消息的 ChannelHandler 实现，received() 方法接收心跳请求的时候，会生成相应的心跳响应并返回；在收到心跳响应的时候，会打印相应的日志；在收到其他类型的消息时，会传递给底层的 ChannelHandler 对象进行处理
          5. WrappedChannelHandler ：决定了 Dubbo 以何种线程模型处理收到的事件和消息，即消息派发机制
             1. org.apache.dubbo.remoting.Dispatcher，每个 WrappedChannelHandler 实现类的对象都由一个相应的 Dispatcher 实现类创建；默认是AllDispatcher
             2. 具体派发策略如下：
                all 所有消息都派发到线程池，包括请求，响应，连接事件，断开事件，心跳等。
                direct 所有消息都不派发到线程池，全部在 IO 线程上直接执行。
                message 只有请求响应消息派发到线程池，其它连接断开事件，心跳等消息，直接在 IO 线程上执行。
                execution 只有请求消息派发到线程池，不含响应，响应和其它连接断开事件，心跳等消息，直接在 IO 线程上执行。
                connection 在 IO 线程上，将连接断开事件放入队列，有序逐个执行，其它消息派发到线程池；
           6.org.apache.dubbo.remoting.transport.netty4.NettyServer#NettyServer，在其构造函数中对传入的 ChannelHandler 对象进行修饰：
                MultiMessageHandler->HeartbeatHandler->AllChannelHandler->DecodeHandler
    6. ExecutorRepository：负责创建并管理 Dubbo 中的线程池，默认实现DefaultExecutorRepository 
      
           1. 维护了一个 ConcurrentMap<String, ConcurrentMap<Integer, ExecutorService>> 集合（data 字段）缓存已有的线程池，
                第一层 Key 值表示线程池属于 Provider 端还是 Consumer 端，第二层 Key 值表示线程池关联服务的端口。
           2. org.apache.dubbo.common.threadpool.manager.DefaultExecutorRepository#createExecutor
           
                  ThreadPool 接口@SPI 注解修饰，默认使用 FixedThreadPool 实现，
                  其中getExecutor() 方法被 @Adaptive 注解修饰，动态生成的适配器类会优先根据 URL 中的 threadpool 参数选择 ThreadPool 的扩展实现。dubbo支持的线程池：
                  fixed ：固定大小线程池，启动时建立线程，不关闭，一直持有。
                  cached： 缓存线程池，空闲一分钟自动删除，需要时重建。
                  limited： 可伸缩线程池，但池中的线程数只会增长不会收缩。
                  eager： 优先创建Worker线程池，在任务数量大于corePoolSize但是小于maximumPoolSize时，优先创建Worker来处理任务。当任务数量大于maximumPoolSize时，将任务放入阻塞队列中。阻塞队列充满时抛出RejectedExecutionException。
    7. ThreadlessExecutor：消费端线程池优化，内部不管理任何线程
    
        1. 简介：org.apache.dubbo.common.threadpool.ThreadlessExecutor
          调用 ThreadlessExecutor 的execute() 方法，将任务提交给这个线程池，但是提交的任务不会被任何调度到任何线程执行，而是存储到阻塞队列中，
          只有当其他线程调用ThreadlessExecutor.waitAndDrain() 方法时才会真正执行(任务执行与调用waitAndDrain()是同一个线程)
        2. 引入原由：
          1. dubbo 2.7.5之前的版本，WrappedChannelHandler 中会为每个连接启动一个线程池。消费端的线程池模型如下：
  ![image](https://user-images.githubusercontent.com/41152743/145351165-388d5048-a567-4cf6-adcd-b4956ac07742.png)
  
              1. 业务线程发出请求，拿到一个 Future 实例；
              2. 业务线程紧接着调用 Future.get() 阻塞等待请求结果返回；
              3. 当业务数据返回后，交由独立的 Consumer 端线程池进行反序列化等解析处理，
              4. 待处理完成之后，将业务结果通过 Future.set() 方法返回给业务线程。
          弊端：
              Consumer 端会维护一个线程池，而且线程池是按照连接隔离的，即每个连接独享一个线程池，当面临需要消费大量服务且并发数比较大的场景时，会出现消费端线程数分配过多的问题，导致线程调度消耗过多 CPU ，也可能因为线程创建过多而导致 OOM。
              
         2. Dubbo 在 2.7.5 版本之后，引入了 ThreadlessExecutor，修改线程模型如下：
![image](https://user-images.githubusercontent.com/41152743/145351765-4662132b-1ece-4a37-9742-21a3ab815bb2.png)

              1. 业务线程发出请求之后，拿到一个 Future 对象。
              2. 业务线程会调用 ThreadlessExecutor.waitAndDrain() 方法，waitAndDrain() 方法会在阻塞队列上等待；
              3. 当收到响应时，IO 线程会生成一个任务，填充到 ThreadlessExecutor 队列中；
              4. 业务线程将 Task 取出并在本线程中执行：反序列化业务数据并set 到 Future，此时 waitAndDrain() 方法返回；
              5. 业务线程从 Future 中拿到结果值。
4. Exchange层

    Dubbo Remoting 层中的最顶层，将信息交换行为抽象成Exchange层，并封装了请求-响应的语义，关注一问一答的交互模式，实现同步转异步。
    
    1. Request 和 Response

        Request的核心字段如下：
 ![image](https://user-images.githubusercontent.com/41152743/145355135-7423a691-97a5-429d-be4a-0fc013bb1b88.png)
        Resopnse的核心字段如下：
![image](https://user-images.githubusercontent.com/41152743/145355528-146a7340-c19e-4bcc-934e-f6040691351e.png)

     2. ExchangeChannel接口： Channel 接口之上抽象了 Exchange 层的网络连接。 
![image](https://user-images.githubusercontent.com/41152743/145355798-0ada9d89-58de-45c4-8fb1-5c8666459bb1.png)

        1. HeaderExchangeChannel类：是ExchangeChannel 的实现，其send() 方法和 request() 方法的实现都是依赖底层修饰的这个 Channel 对象实现的：
        
            1. HeaderExchangeChannel.request() 方法中完成 DefaultFuture 对象的创建，包括创建请求相应超时的定时任务，加入时间轮中；
            2. 将请求通过底层的 Dubbo Channel 发送出去，发送过程中会触发沿途 ChannelHandler 的 sent() 方法，
            3. 其中的 HeaderExchangeHandler 会调用 DefaultFuture.sent() 方法更新 sent 字段，记录请求发送的时间戳，如果响应超时，则会将该发送时间戳添加到提示信息中；
            4. 一段时间后，Consumer 会收到对端返回的响应，在读取到完整响应之后，会触发 Dubbo Channel 中各个 ChannelHandler 的 received() 方法；
                例如：AllChannelHandler 子类会将后续 ChannelHandler.received() 方法的调用封装成任务提交到线程池中，响应会提交到 DefaultFuture 关联的线程池中。
            5. 当响应传递到 HeaderExchangeHandler 的时候，会通过调用 handleResponse() 方法进行处理，其中调用了 DefaultFuture.received() 方法，将 DefaultFuture 设置为完成状态。
            
       2. HeaderExchangeHandler类： 
             ExchangeHandler 的装饰器，其中维护了一个 ExchangeHandler 对象。
             ExchangeHandler 接口是 Exchange 层与上层交互的接口之一，上层调用方可以实现该接口完成自身的功能；
             然后再由 HeaderExchangeHandler 修饰，具备 Exchange 层处理 Request-Response 的能力。
       
                1.received()方法
                    1. 只读请求会由handlerEvent() 方法进行处理，它会在 Channel 上设置 channel.readonly 标志；
                    2. 双向请求由handleRequest() 方法进行处理，会先对解码失败的请求进行处理，返回异常响应；然后将正常解码的请求交给上层实现的 ExchangeHandler 进行处理，并添加回调。
                    3. 单向请求直接委托给上层 ExchangeHandler 实现的 received() 方法进行处理，不关注结果；
                    4. Response 的处理，通过handleResponse() 方法将关联的 DefaultFuture 设置为完成状态（或是异常完成状态）；
                    5. 于 String 类型的消息，根据是否支持telnet，进行相应的处理
             2. connected()、disconnected()、sent()、received()、caught() 方法最终都会转发给上层提供的 ExchangeHandler 进行处理
       3. HeaderExchangeClient： Client 装饰器，主要是添加这两个功能：
       
            1. 维持与 Server 的长连状态，这是通过定时发送心跳消息实现的；
            2. 在因故障掉线之后，进行重连，这是通过定时检查连接状态实现的。
       4. HeaderExchangeServer：RemotingServer 的装饰器
            
            1. 定期关闭长时间空闲的连接
       5. HeaderExchanger：是 Exchangers 这个门面类，用于获取具体的ExchangeClient或者ExchangeServer
       
            1. 在Transport 层的 Client 和 Server 实现基础之上，添加 HeaderExchangeClient 和 HeaderExchangeServer 装饰器。
            2. 为上层实现的 ExchangeHandler 实例添加了 HeaderExchangeHandler 以及 DecodeHandler 两个修饰
 #### 6.Dubbo的RPC
 1. 包结构
        
        1. filter 包：在进行服务引用时会进行一系列的过滤，其中包括了很多过滤器；
        2. listener 包：在服务发布和服务引用的过程中，添加一些 Listener 来监听相应的事件；
        3. protocol 包：为 Protocol 接口的具体实现以及 Invoker 接口的具体实现提供一些公共逻辑。
        4. proxy 包：提供了创建代理的能力；
        5. support 包：包括了 RpcUtils 工具类、Mock 相关的 Protocol 实现以及 Invoker 实现
 2. 核心接口
 
        1. Invoker 接口
 ![image](https://user-images.githubusercontent.com/41152743/145740202-36b83222-ef08-4994-8c8a-6bf9b93c188e.png)
                
  ![image](https://user-images.githubusercontent.com/41152743/145740282-0f99baf8-dbb3-4d9b-a23e-5ec25b9e4dec.png)
               Provider逻辑调用：服务实现类被封装成为一个 AbstractProxyInvoker 实例，并新生成对应的 Exporter 实例，当 Dubbo Protocol 层收到一个请求之后，会找到这个 Exporter 实例，并调用其对应的 AbstractProxyInvoker 实例，从而完成 Provider 逻辑的调用
               Invocation 接口：invoke() 方法的参数，抽象了一次 RPC 调用的目标服务和方法信息、相关参数信息、具体的参数值以及一些附加信息；
               Result 接口：invoke() 方法的返回值，抽象了一次调用的返回值，包含了被调用方返回值（或是异常）以及附加信息，可以添加回调方法，在 RPC 调用方法结束时会触发这些回调。
                    
                    
      

