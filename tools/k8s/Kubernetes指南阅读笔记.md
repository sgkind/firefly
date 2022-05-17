## kubernetes简介
1.kubernetes的主要功能为：
* 基于容器的应用部署、维护和滚动升级
* 负载均衡和服务发现
* 跨机器和跨地区的集群调度
* 自动伸缩
* 无状态服务和有状态服务
* 广泛的 Volume 支持
* 插件机制保证扩展性

2.kubernetes主要由以下几个核心组件组成：
* etcd保存了整个集群的状态；
* apiserver提供了资源操作的唯一入口，并提供了认证、授权、访问控制、API注册和发现等机制；
* controller manager负责维护集群的状态，比如故障检测、自动扩展、滚动更新等；
* scheduler负责资源的调度，按照预定的调度策略将Pod调度到相应的机器上；
* kubelet负责维护容器的声明周期，同时也负责Volume(CVI)和网络(CNI)的管理；
* Container runtime负责镜像管理以及pod和容器的真正运行(CRI);
* kube-proxy负责为Service提供cluster内部的服务发现和负载均衡

除了核心组件，还有一些推荐的Add-ons:
* kube-dns负责为整个集群提供DNS服务
* Ingress Controller为服务提供外网入口
* Heapster提供资源监控
* Dashboard提供GUI
* Federation提供跨可用区的集群
* Fluentd-elasticsearch提供集群日志采集、存储与查询

## kubernetes基本概念
1.Container

Container(容器)是一种便携式、轻量级的操作系统级虚拟化技术。它使用namespace隔离不通的软件运行环境，并通过镜像自包含软件的运行环境，从而使的容器可以很方便的在任何地方运行。

2.Pod

kubernetes使用Pod来管理容器，每个Pod可以包含一个或多个紧密关联的容器。

Pod是一组紧密关联的容器集合，它们共享PID、IPC、Network和UTS namespace，是Kubernetes调度的基本单位。Pod内的多个容器共享网络和文件系统，可以通过进程间通信和文件共享这种简单高效的方式组合完成服务。

3.Node

Node是Pod真正运行的主机，可以是物理机，也可以是虚拟机。为了管理Pod，每个Node节点上至少要运行container runtime(比如docker或者rkt)、kubelet和kube-proxy服务。

4.Namespace

Namespace是对一组资源和对象的抽象集合，比如可以用来将系统内部的对象划分为不同的项目组或用户组。常见的pods、services、replication controller和deployments等都是属于某一个namespace的（默认是default），而node、persisentVolumes等则不属于任何namespace。

5.Service

Service是应用服务的抽象，通过labels为应用提供负载均衡和服务发现。匹配labels的Pod IP和都南口列表组成endpoints，由kube-proxy负责将服务IP负载均衡到这些endpoints上。

每个Service都会自动分配一个cluster IP（仅在集群内部可访问的虚拟地址）和DNS名，其他容器可以通过该地址或DNS来访问服务，而不需要了解后端容器的运行。

6.Label

Label是识别kubernetes对象的标签，以key/value的方式附加到对象上（key最长不超过63字节，value可以为空，也可以是不超过253字节的字符串）。

Label不提供唯一性，并且实际上经常是很多对象（如Pods）都使用相同的label来标志具体的应用。

Label定义好后其他对象可以使用Label Selector来选择一组相同label的对象（比如ReplicaSet和Service用label来选择一组Pod）。Label Selector支持以下几种方式：
* 等式，如`app=nginx`和`env!=production`
* 集合，如 `env in (production, qa)`
* 多个label（它们之间是AND关系），如`app=nginx,env=test`

7.Annotations

Annotations是key/value形式附加于对象的注解。不同于Labels用于标志和选择对象，Annotations则是用来记录一些附加信息，用来辅助应用部署、安全策略以及调度策略等。比如deployment使用annotations来记录rolling update的状态。

## kubernetes集群

一个Kubernetes集群由分布式存储etcd、控制节点controller以及服务节点Node组成。
* 控制节点主要负责整个集群的管理，比如容器的调度、维护资源的状态、自动扩展以及滚动更新等
* 服务节点是真正运行容器的主机，负责管理镜像和容器以及cluster内的服务发现和负载均衡
* etcd集群保存了整个集群的状

## kubernetes设计理念

1.API对象

API对象是K8s集群中的管理操作单元。K8s集群系统每支持一项新功能，引入一项新技术，一定会新引入对应的API对象，支持对该功能的管理操作。

每个API对象都有3大类属性：元数据metadata、规范spec和状态status。元数据是用来标识API对象的，每个对象都至少有3个元数据: namespace, name和uuid；除此以外还有各种各样的标签labels用来标识和匹配不同的对象，例如用户可以用标签env来标识区分不同的服务部署环境，分别用env=dev、env=testing、env=production来标识开发、测试、生产的不同服务。规范描述了用户期望k8s集群中的分布式系统达到的理想状态(Desired State)。

2.Pod

k8s中的业务主要可以分为长期伺服型(long-running)、批处理型(batch)、节点后台支撑型(node-daemon)和有状态应用型(stateful application)，分别对应的控制器为Deployment、Job、DaemonSet和StatefulSet。