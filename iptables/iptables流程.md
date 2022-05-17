iptables流程图
===
下面两张图以第二幅为准
![](../images/20190409154520568.png)
![](../images/Netfilter-packet-flow.svg)

## 说明
1. 数据帧进入数据链路层，首先经过BROURING链的Broute处理，决定是直接路由该数据帧还是让它进入到PREROUTING链；
2. 如果数据帧的目的地址和源地址在同一网段，网桥会屏蔽它；如果数据帧是多播帧或广播帧，则要在同一网段中除了接收端口以外的其他端口发送这个数据帧；
3. 接下来，数据帧到达PREROUTING链后可以改变目的MAC地址（DNAT）；当数据帧通过PREROUTING链后，Ebtables将会根据该数据帧的目的MAC地址决定是否转发该帧，如果这个帧的目的MAC是本机的，就会进入到INPUT链，在这个链中，可以过滤进入本机的数据帧，通过INPUT链后，就到达网络层，数据帧变成数据包；如果数据帧的目的MAC不是本机的，它进入FORWARD链，FORWARD链将过滤  数据帧；然后这个数据帧就会到达POSTROUTING链，在这里可以改变数据帧的源MAC地址（SNAT）。。由本机产生的帧，首先判断是否需要Bridging，如果不需要则进行直接路由；如果需要就会进入到OUTPUT链中，以对数据帧改变目的MAC地址（DNAT）和过滤，接下来这个帧到达POSTROUTING链，这个链可以改变数据帧的源MAC地址（SNAT）；最后，这个帧就到达了NIC。  
4. 桥接方式的处理流程： 当数据帧进入Linux网桥后，先通过Ebtables的BROUTING链和PREROUTING链；接下来，经过iptables的PREROUTING链，这时还是在数据链路层，而不是在iptables通常起作用的网络层，这就是br_netfilter帮助数据帧在数据链路层可以经过iptables链的作用；然后，经过Ebtables的FORWARD链和iptables的FORWARD链；最后经过Ebtables和iptables的POSTROUTING链。