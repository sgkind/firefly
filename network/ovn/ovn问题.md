ovn问题
===

## 一
ovn作为控制ovs的controller，它是通过设置openflow流表来控制的，但是它跟传统的controller的控制方式区别很大，它是怎么控制的？

传统的控制器直接向ovs下发流表，当ovs接收到流表不能处理的包时，将包通过管理端口发送给控制器，由控制器确定处理方案。

ovn比传统控制器更进一步，抽象出了逻辑网络设备，例如逻辑交换机、逻辑路由器、逻辑端口等。ovn中CMS（云管理系统）先从逻辑网络中抽象出逻辑网络设备，然后ovn-northd根据逻辑网络设备抽象出逻辑数据通路流表，将逻辑流表下发到位于各个chassis的ovn-controller中，在ovn-controller中产生openflow流表，控制ovs的行为。

ovn在开发之初就是面向云计算管理平台的，它专注于实现二层和三层网络功能。除了在传输层实现基于L4的ACL外，基本不在L4～L7层实现功能。

传统的controller抽象层次较低，可以通过编程实现更多的功能。

## 二
ovn抽象的这些逻辑概念，基本涵盖了物理交换机、三层网关、路由和防火墙的所有功能，搞清楚它们的含义

* 逻辑交换机/Logical switch：用来做二层转发
* 逻辑路由器/Logical router: 分布式的，用来做三层转发
* L2/L3/L4 ACLs: 二层到四层的ACL，可以根据报文的MAC地址、IP地址和端口号来做访问控制
* Multiple tunnel overlays：支持多种隧道封装技术，有Geneve，STT和VXLAN
* TOR switch or software logical switch gateways: 支持使用硬件TOR switch或者软件逻辑switch当作网关来连接物理网络和虚拟网络

## 三
思考下，它是如何在分布式环境中工作的。

OVN目前具有两种角色：OVN Central和OVN Host

OVN Central：目前只能有一台主机承担这个角色。该主机成为和外部资源（比如云管理平台）集成的API中心节点。中心节点运行着OVN北向数据库和OVN南向数据库。OVN北向数据库，用于描述上层的逻辑网络，比如逻辑交换机/逻辑路由器/逻辑端口。南向数据库，其将北向数据库的逻辑网络数据格式转换为物理网络数据格式并进行存储。

OVN Host：所有提供虚拟机或虚拟机网络的节点。OVN Host运行着"chassis controller"，它上连OVN南向数据库并作为其记录的物理网络信息授权来源。