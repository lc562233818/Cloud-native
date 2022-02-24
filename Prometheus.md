# 深入浅出Prometheus
## 监控
### 基础资源监控
**监控角度来看，可以将基础资源监控分为网络监控、存储监控与服务监控**
### 应用程序监控（APM）
* APM主要针对应用程序监控，包括应用的运行状态、性能监控、日志监控、调用链跟踪等
* 调用链跟踪指的是追踪整个过程
* 通过APM可以通过截获TCP、HTTP网络请求，从而获得执行耗时最长的方法和SQL语句，延迟最大的API等信息
### 监控系统实现
**监控系统主要由以下两大子系统实现**
* 指标采集子系统主要负责信息采集、汇总、存储
* 数据处理子系统主要负责数据分析、展现、预警、告警动作触发、告警等
### 数据采集
* 例如prometheus可以通过各种exporter客户端进行数据的采集，这类统称为客户端数据采集，对于运维人员挑战较大，需要维护多个客户端
* 另外通过类似SNMP协议进行数据的采集，这种统称为协议数据采集，协议采集限制了诸多数据采集数据能力
### 数据传输与过滤
**分为两类实现**
* 可以通过HTTP传输数据通过pull（拉取）/push（推送）数据方式，这类为协议方式
* 使用消息中间件，数据采集器只需从消息中间件获取/推送数据，这类为中间件传输
### 数据存储
**监控数据存储通常借助于时序数据库(TSDB)**
* 每个监控数据都有自己的时间维度（属性）
* 一次写入多次数据，时序数据通常不会修改更新，一旦写入，则后期只会对数据进行查询
* 查询以时间为单位，实际很少会查一周前的数据
### prometheus
* 提供多维度数据模型和灵活的查询方式，提供PromQL查询，提供HTTP查询结果，可以通过Grafana等GUI组件展示数据
* 自带时序数据库，可以对接第三方时序数据库
* 基于HTTP等方式采集数据，支持以PUSH方式向中间网关push推送时序数据
* 支持通过静态文件配置和动态发现机制发现监控对象
## 深入prometheus设计
### prometheus指标定义
**指标定义由指标名称与标签组成**
### prometheus的指标分类
**分为Counter(计数器)、Gauge(仪表盘)、Hisrogram(直方图)、Summary(摘要)**
* Counter特点为只增不减，适用HTTP请求
* Gauge实时监听变化，适用CPU,MEM
* Summary用于凸显数据分布情况
* Hisrogram反映某个区间的样本个数
### prometheus数据样本
**数据样本由三部分组成：指标、样本值、时间戳，时序数据首先保存在内存中，然后批量刷新到磁盘**
### 数据采集
* prometheus默认采用pull，监控中心拉取agent的数据
* 采用push的话，agent主动上报数据到监控中心，prometheus为了兼容push方式提供了pushgateway组件，pushgateway组件接收客户端发过来的数据
* 采用push方式，通常每个agent都需要配置master的地址
* 采用pull方式，通常通过批量配置或自动发现获取采集点
### 服务发现
**静态文件配置**
* 一般用于固定的监控环境、IP地址统一的服务场景
**动态发现**
* prometheus会从这些组件中获取监控对象，并汇总这些组件中获取的数据，从而获取所有监控对象，首次加入监控的状态为unknown，只会对在周期对该数据进行采集
* 若需使用Kubernetes进行动态发现，需配置Kubernetes API地址和认证凭据
### 告警
* prometheus本身不会处理告警，需要借助alertManager，prometheus会配置alertManager对地址
### 集群
* prometheus引入联邦集群概念，联邦节点会定时从下面的prometheus节点获取数据并且汇总，下面的prometheus节点分别处于在不同机房中，这样每个监控对象都会重复采集，数据会被重复保存
### Thanos由四个组件组成querier、sidercar、store、compactor
* 每个prometheus节点都会配置一个sidercar组件，prometheus通过Kubernetes的部署，可以将sidercar容器与prometheus容器集成到衣一个pod，sidercar主要用来代理querier对prometheus本地数据的读取、将prometheus本地监控数据通过存储接口保存到对象存储中
* querier接收http的promql查询，组件负责数据查询汇聚
* 为保证数据持久化，sidercar会监控prometheus的本地存储，若发现监控数据有新的会保存磁盘，则将监控数据保存到对象存储
* compactor一个批处理组件，负责对对象存储进行压缩，从而节省存储占用
* 为了实现节点发现，thanos引入了gossip，gossip底层使用udp,tcp协议，所以为了避免网络中使用http协议，thanos使用基于文件或者dns的方式完成节点注册和发现
### 数据存储
* prometheus提供了两种存储方式，分别为本地存储和远端存储
* prometheus自带的时序数据库数据保存在本地，但是这样会限制其容量，所以prometheus又引入了远端存储
* 为了兼容本地与远端，prometheus提供了fanout接口
#### 本地存储
* prometheus的本地存储称为prometheus TSDB，TSDB设计有两个核心，block,wal
* block
**block大小不固定，按照设定步长倍数递增，默认最小保存2小时监控数据，若步长为3，数据会不断倍增，会将小的合并成大的，例如将3个2h的合并为一个6h，这样可以减少block数量**
**block由四部分组成**
* chunks用于保存压缩后的时序数据，每个大chunk小为512M，若超出将截成多个chunk，将以数字编号命名
* index为了数据监控快速检索设计，主要用来记录chunk中时序的偏移位置
* tombstone用于对数据进行软删除
* meta.json用来记录block的元数据信息，主要包括，起始时间、截止时间、样本数、时序数和数据源等信息，在compaction参数每个块压缩一次level等级就+1
* wal是关系型数据库利用日志来实现事务性持久化的技术，在某个操作前纪律事情下来，以便进行数据回滚
#### 远端存储
**prometheus引入了Adapter适配器，将prometheus的读写转化为第三方远端存储接口，从而远程数据读写**
