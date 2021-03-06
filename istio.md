# istio
##什么是Service Mesh
**Service Mesh的定义**
```
Service Mesh是一个用来进行请求分发的基础设施层，通常以Sidecar的形式部署，对应用透明的网络代理
```
**Service Mesh的主要功能**
```
1、流量控制
2、路由策略
3、网络安全
4、可观测型
```
**istio的核心功能**
![](media/16456656627047/16456675504675.jpg)

**Service Mesh与API网关的不同点**
```
1、Service Mesh由sidecar接管发送应用的请求，处理完成后在发送到另外的微服务
2、API网关部署在应用边界，无侵入应用内部，主要对内部API进行聚合以及方便外部调用
3、功能有重叠
5、Service Mesh在应用内，API网关在应用之上（边界）
```
## 入门istio
### istio的架构
![](media/16456656627047/16470878838587.jpg)

```
Envoy可为服务网格中的所有服务路由控制所有入站和出站流量。 Envoy代理是与数据平面流量交互的唯一Istio组件Envoy代理被部署为服务的Sidecar
Pilot-agent：
1、生成envoy的配置
2、启动envoy
3、监控并管理envoy的运行状况，比如envoy出错时pilot-agent负责重启envoy，或者envoy配置变更后reload envoy
istio-init or cni：istio-init 和 cni 实现底层原理没什么区别，均为写入iptables，让该 pod 所在的 network namespace 的网络流量转发到 proxy 进程
pilot:将控制流量行为的高级路由规则转换为Envoy特定的配置，并在运行时将其下发到Sidecar。 Pilot提取了特定于平台的服务发现机制，并将它们合成为标准格式，任何符合Envoy API的Sidecar都可以使用
citadel:负责处理系统上不同服务之间的TLS通信。 Citadel充当证书颁发机构(CA)，并生成证书以允许在数据平面中进行安全的mTLS通信
galley:配置管理
```
### istio实现流量控制**
```
1、虚拟服务
2、目标规则
3、网关
4、服务入口
5、sidecar
```
**虚拟服务**
![](media/16456656627047/16456685124589.jpg)

```
1、将流量路由到指定的目标地址
2、请求地址与真实的工作负载解耦
3、包含一组路由规则
4、通常和目标规则成对出现
5、丰富的路由匹配规则
```
**目标规则**
![](media/16456656627047/16456687097429.jpg)

```
1、定义虚拟路由目标地址真实的地址
2、设置负载均衡的方式
3、每个子集配置一个对应目标地址
```
**网关**
![](media/16456656627047/16456687916056.jpg)
```
负责处理进入的流量为ingerss,管理出去的流量的微egress(不是必选项)，内部服务流量可以直接指向外部服务，使用网关可以通过网关管理内部流量一样，管理外部流量，为进出的服务增加负载均衡能力
```
**服务入口**
![](media/16456656627047/16456689456969.jpg)
```
1、将外部的服务注册到网格内
功能：
1、为外部目标转发请求
2、添加超时重试等策略
3、扩展网格
```
**sidecar**
![](media/16456656627047/16456690705380.jpg)
```
1、调整envoy代理接管的端口协议
2、限制envoy代理访问的服务
```
### 服务的可观察型
```
可观察型分为指标、日志、追踪
```
**指标**
```
定义：可以在不同时间点做一些记录，在将这些数据汇总，在通过图表展示运行状态
指标分类：
1、代理级别的指标：用来收集sidecar代理上的一些数据，sidecar接管所有来自服务的请求，可以指定sidecar进行收集，可以将指标数据导入到prometheus
2、服务级别的指标
3、控制平面的指标   
```
### istio的安全架构
![](media/16456656627047/16456700299002.jpg)

```
istio安全分为认证和授权两大部分，基本由istiod控制平面实现
CA负责证书管理，认证策略与授权策略存储在对应模块，由API Server将这些策略变为配置，下发至对应的数据平面
```
**认证**
![](media/16456656627047/16456701860794.jpg)

```
策略会变成具体的配置信息由istiod下发给具体的sidecar代理，支持不加密、加密方式
```
**认证方式**
```
1、对等认证：
用于服务间身份证认证(mTLS)
2、请求认证：
用于终端用户身份认证(JWT)
通过配置yaml文件进行下发策略
```
**授权**
![](media/16456656627047/16456705161684.jpg)
**istio基础资源**
```
1、kiali:微服务应用到展示工具
2、
```
**动态路由**
![](media/16456656627047/16463645059613.jpg)
```
virtualService:
hosts:设置具体目标地址
gateway:匹配的网关进行匹配使用，若在服务网格内则不使用
http:对应具体route对象，用来实现具体的路由匹配规则
exportTo:定义虚拟机服务的可见性，设置其他的namespaces是否可见性
httproute:
match:匹配啥类型的请求，满足啥样的条件，可以被接受到的，不设置默认全部接受
route:通过match匹配到后，路由到目标的地址
DestinationRule:
host:路由到最终的目标地址
subsets:服务限定版本
trafficPolicy:定义策略
TrafficPolicy:
定义一些负载均衡策略
```
**虚拟路由配置文件信息**
![](media/16456656627047/16463650015920.jpg)
**目标规则配置文件信息**
![](media/16456656627047/16463650507237.jpg)
**虚拟路由与目标规则场景**
```
1、按服务版本路由
2、按比例切分路由，例如灰度发布
3、根据匹配规则进行路由
4、定义负载均衡策略
```
**gateway**
![](media/16456656627047/16463776634101.jpg)

![](media/16456656627047/16463773387551.jpg)

```
1、一个运行在网格边缘的负载均衡器
2、接收外部请求、转发网格内的服务
3、配置对外的端口、协议与内部服务的映射关系
4、把内部服务暴露给外部服务访问
```
![](media/16456656627047/16463775492234.jpg)
```
根据定义的url会路由到destination目标路由到details服务
```
**服务入口**
```
1、把外部服务纳入到网格内部访问进行管理
2、管理外部服务的请求
3、扩展Service Mesh,多个集群共享同一个Mesh的时候

```
![](media/16456656627047/16463779850606.jpg)
**服务入口配置信息**
![](media/16456656627047/16463782567350.jpg)

![](media/16456656627047/16463781833138.jpg)
```
hosts：配置定义外部服务的域名
ports:具体的外部服务端口
```
**权重路由**
1. 蓝绿部署：在生产环境中同时有两套完全一致的应用，其中一套正在服务线上环境，当你的应用需要更新时直接在另外一套进行部署新的版本，然后将流量切换至新版本，缺点需要大量的资源

![](media/16456656627047/16463784883269.jpg)

2. 灰度发布(金丝雀发布)，小范围测试、小范围发布，逐渐更新迭代版本，如图中所示将请求流量逐步转移至新版本，若无问题测在将大部分流量请求转移至新版本，缺点管理多个版本需要
**灰度发布配置信息**
![](media/16456656627047/16463791302538.jpg)
![](media/16456656627047/16463786149388.jpg)

3. A/B测试比较版本的优劣，实现方式和灰度发布一致，灰度发布是将流量转移至新版本，A/B测试主要A/B的优劣，那一方的用户数较多就保留那一方
**Ingress基本概念**
```
1、服务的入口。接受外部请求并转发到后段服务
2、针对l4-l6的协议，只定义基础点，所有路由规则交给virtualService处理，因虚拟服务本身可以重复利用进行复用，和其他定义都是解耦的
```
**Ingress配置信息**
![](media/16456656627047/16466344358499.jpg)
![](media/16456656627047/16466345095282.jpg)
**egress基本概念**
```
1、定义网格的出口点、允许你将监控、路由等功能应用离开网格的流量
2、为无法访问公网的内部服务做代理
```
**egress基本信息配置**
![](media/16456656627047/16466350954935.jpg)
![](media/16456656627047/16466353156465.jpg)
```
1、hosts:istio-egressgateway...为egress网关的dns名称
2、istio-egressgateway针对egress网关
3、mesh针对内部网格的
4、由mesh针对内部服务，会将你的请求路由到egress网关对应的dns名称，会将所有的内部请求全部指向网关这个节点
5、istio-egressgateway会将网关的请求指向最终我们外部服务的地址
6、第一跳把内部服务的出流量指向gateway的mesh,然后在将mesh的流量指向外部服务
```
### istio的高可用功能
**高可用能力**
```
1、超时：调用上游服务的时候，如果上游服务没有返回，可以设置一个超时时间，超过换个时间机会返回一个状态码
2、重试：设置一个重试次数，如果超过该次数就直接返回状态码
3、熔断：一个过载保护的手段，目的为了避免级联失败，为了当某个服务出现故障，不影响其他服务，通过熔断将这个服务停止
4、故障注入：人为的注入故障增强系统的稳定性以及问题解决能力
5、流量镜像：实时复制请求至镜像服务中，类似主从机制
```
**超时重试基本配置信息**
![](media/16456656627047/16466364222857.jpg)
**熔断基本配置信息**
![](media/16456656627047/16466371429167.jpg)
```
1、maxConnections tcp最大连接数
2、httpMaxpendingRequest最大等待请求数
3、maxRequsetsPerConnection每个链接最大请求
4、consecutiveErrors失败次数，多少次触发
5、interval：熔断时间间隔时间
6、baseEjectionTime:驱逐时间乘以consecutiveErrors，当你服务失败次数越来越多时熔断时间会越来越长。默认30s
7、maxEjectionPercent最大驱逐比例，指多少个实例可以被熔断驱逐出去
```
![](media/16456656627047/16466382525039.jpg)
**故障注入配置信息**
![](media/16456656627047/16467079774481.jpg)
```
增加fault字段
percentage value 多少流量受到影响
fixedDelay 延迟时间
还可以设置终止故障
```
![](media/16456656627047/16467080762221.jpg)
**流量镜像配置信息**
创建服务
![](media/16456656627047/16467083369359.jpg)
定义路由
![](media/16456656627047/16467083716312.jpg)
在上图中的VirtualService增加mirror字段
![](media/16456656627047/16467084724748.jpg)
![](media/16456656627047/16467085884126.jpg)
### istio工具
**kiali架构**
![](media/16456656627047/16467088270426.jpg)
```
1、kiali fornt-end 前端展示页面
2、kiali back-end 与istio交互获取数据
3、通过prometheus抓取一些指标数据，通过clustrt api抓取具体应用的数据
```
**prometheus**
![](media/16456656627047/16467097304067.jpg)


```
1、1.5之前版本istio通过mixer接口收集监控指标数据
2、1.5后prometheus与envoy和交互，主要采用STAS、METADATA EXCHANAGE来完成之前mixer的工作
```
**Grafana**
```
1、进行指标数据的展示
2、提供告警功能
3、支持多种数据源
```
**Envoy日志**
![](media/16456656627047/16467101999885.jpg)
![](media/16456656627047/16467102292269.jpg)
![](media/16456656627047/16467102505223.jpg)

```

```
**Jaeger**
```
1、端到端的分布式追踪系统
```
**Jaeger架构**
![](media/16456656627047/16467905740212.jpg)

**span**
![](media/16456656627047/16467912452037.jpg)


```
1、span一个逻辑单元，有具体的操作名、执行时间
```
**Trace**
![](media/16456656627047/16467913666769.jpg)

```
1、整个数据的执行路径 
```
```
1、Client lib 客户端依赖的公共库
2、Agent 主要用来和应用抓取到具体span发送给collertor
3、Collertor是一个收集器从应用层面收集到具体的span发送给后续的存储系统
4、Query主要在页面上进行查询
5、Ingester与kafka配套使用相当于Kafka的consumer用来抓取具体的数据然后导入到DB中
```