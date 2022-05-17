OVN VTEP网关
===

OVN可以通过VTEP网关把物理网络和逻辑网络连接起来，VTEP网关可以是TOR(Top of Rack) switch，目前很多硬件厂商都支持，比如Arista，Juniper和HP等等；也可以是是软件做的逻辑switch，OVS社区就做了一个简单的VTEP模拟器。VTEP网关需要遵守VTEP OVSDB schema，它里面定义了VTEP网关需要支持的数据表项和内容，VTEP通过OVSDB协议与OVN通信，通信的流程OVN也有相关标准，VTEP上需要一个ovn-controller-vtep来做ovn-conroller所做的事情。VTEP网关和HV之间常用VXLAN封装技术。

![OVN VTEP网关连接物理网络和逻辑网络](../../images/ovn-vtep.png)

如上图所示，PH1和PH2是连在网络网络里面的两个服务器，VM1到VM3是OVN里面的3个虚拟机，通过VTEP网关把物理网络和逻辑网络连接起来，从逻辑上看，PH1、PH2和VM1-VM3就像在一个物理网络里面，彼此之间可以互通。

虽然VTEP OVSDB schema里面定义了三层的表项，但是目前没有硬件厂商支持，VTEP模拟器也不支持，所以VTEP网络只支持二层的功能，也就是说只能连接物理网络的VLAN到逻辑网络的VXLAN，如果VTEP上不同VLAN之间要做路由，需要OVN里面的路由器来做。

![](https://raw.githubusercontent.com/cao19881125/picture_cloud/master/vtep-ovn.png)