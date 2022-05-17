OVN L3 Gateway
===

>  An OVN Gateway serves as an onramp/offramp between the overlay network and the physical network. They come in two flavors: layer-2 which bridge an OVN logical switch into a VLAN, and layer-3 which provide a routed connection between an OVN router and the physical network. For the purposes of this lab we will focus on creating a layer-3 gateway which will serve as the demarcation point between our physical and logical networks.

## 目标1
### 物理网络

                 +------------------------------------+ 192.168.209.0/19
                     |             |              |
                     |             |              |
                 +-------+     +-------+      +-------+ 
                 |       |     |       |      |       |
                 | node1 |     | node2 |      | node3 |
                 |       |     |       |      |       |
                 +-------+     +-------+      +-------+
        ens3 192.168.209.110 192.168.209.111 192.168.209.112
        ens4 192.168.209.115

* node1 192.168.209.110/19 作为OVN Central和网关节点
* node2 192.168.209.111/19 作为OVN Host
* node3 192.168.209.112/19 作为OVN Host
  
### 逻辑网络
```
         --------- 
        |  client | 192.168.209.115/19 Physical Network
         ---------
         ____|____ 
        |  switch | outside
         ---------
             |
         ____|____ 
        |  router | gw1 port 'gw-outside': 192.168.209.120/19
         ---------      port 'gw-ls2':     192.168.255.1/24
         ____|____ 
        |  switch | ls2  192.168.255.0/24
         ---------  
         ____|____ 
        |  router | router1 port 'router1-ls2':  192.168.255.2/24
         ---------          port 'router1-ls1': 192.168.100.1/24
             |
         ____|____ 
        |  switch | ls1 192.168.100.0/24
         ---------  
         /       \
 _______/_       _\_______  
|  vm1    |     |   vm2   |
 ---------       ---------
192.168.100.10  192.168.100.20
```

## 过程
### 安装软件
#### node1作为OVN Central和Host
安装openvswitch-switch、openvswitch-common、ovn-common、ovn-central
```shell
# apt install openvswitch-switch openvswtch-common ovn-common ovn-central ovn-host
```

#### node2和node3作为OVN Host
安装openvswitch-switch、openvswitch-common、ovn-common、ovn-host
```shell
# apt install openvswitch-switch openvswitch-common ovn-common ovn-host
```

### 在OVN Central(node1)上开启南、北向数据库监听端口
```shell
# ovn-nbctl set-connection ptcp:6641:0.0.0.0
# ovn-sbctl set-connection ptcp:6642:0.0.0.0
```

### 创建集成化网桥
在node2和node3节点上:
```shell
# ovs-vsctl add-br br-int -- set Bridge br-int fail-mode=secure
```

### 将OVN Host连接到OVN Central上
1. 在node2上执行：
```shell
# ovs-vsctl set open . external-ids:ovn-remote=tcp:192.168.209.110:6642
# ovs-vsctl set open . external-ids:ovn-encap-type=geneve
# ovs-vsctl set open . external-ids:ovn-encap-ip=192.168.209.111
```

2. 在node3上执行：
```shell
# ovs-vsctl set open . external-ids:ovn-remote=tcp:192.168.209.110:6642
# ovs-vsctl set open . external-ids:ovn-encap-type=geneve
# ovs-vsctl set open . external-ids:ovn-encap-ip=192.168.209.112
```

3. 在node1上查看已连接的chassis
```shell
# ovn-sbctl show
```

### 创建switch ls1
在node1上
```shell
# ovn-nbctl ls-add ls1

# ovn-nbctl lsp-add ls1 ls1-vm1
# ovn-nbctl lsp-set-addresses ls1-vm1 02:ac:10:ff:00:11
# ovn-nbctl lsp-set-port-security ls1-vm1 02:ac:10:ff:00:11

# ovn-nbctl lsp-add ls1 ls1-vm2
# ovn-nbctl lsp-set-addresses ls1-vm2 02:ac:10:ff:00:22
# ovn-nbctl lsp-set-port-security ls1-vm2 02:ac:10:ff:00:22
```

查看结果：
```shell
# ovn-nbctl show 
switch 2f270b61-0212-466f-9f13-c83a1effc8ab (ls1)
    port ls1-vm1
        addresses: ["02:ac:10:ff:00:11"]
    port ls1-vm2
        addresses: ["02:ac:10:ff:00:22"]
```

### 添加伪虚拟机vm1和vm2
在node2上添加”伪虚拟机“vm1
```shell
# ip netns add vm1
# ovs-vsctl add-port br-int vm1 -- set interface vm1 type=internal
# ip link set vm1 netns vm1
# ip netns exec vm1 ip link set vm1 address 02:ac:10:ff:00:11
# ip netns exec vm1 ip addr add 192.168.100.10/24 dev vm1
# ip netns exec vm1 ip link set vm1 up
# ovs-vsctl set Interface vm1 external_ids:iface-id=ls1-vm1
```

在node3上添加”伪虚拟机“vm2
```shell
# ip netns add vm2
# ovs-vsctl add-port br-int vm2 -- set interface vm2 type=internal
# ip link set vm2 netns vm2
# ip netns exec vm2 ip link set vm2 address 02:ac:10:ff:00:22
# ip netns exec vm2 ip addr add 192.168.100.20/24 dev vm2
# ip netns exec vm2 ip link set vm2 up
# ovs-vsctl set Interface vm2 external_ids:iface-id=ls1-vm2
```

在node1上查看端口绑定状态
```shell
# ovn-sbctl show
Chassis "3e7c3161-1b09-4f4d-8591-3c16021b4f60"
    hostname: "node2"
    Encap geneve
        ip: "192.168.209.111"
        options: {csum="true"}
    Port_Binding "ls1-vm1"
Chassis "78f1fb86-fb9e-4ba3-985a-4463fab94153"
    hostname: "node3"
    Encap geneve
        ip: "192.168.209.112"
        options: {csum="true"}
    Port_Binding "ls1-vm2"
```

### 测试网络连通性
在node2上执行
```shell
# ip netns exec vm1 ping 192.168.100.20 -c 3
```

### 创建router1
```shell
# ovn-nbctl lr-add router1

# ovn-nbctl lrp-add router1 router1-ls1 02:ac:10:ff:00:01 192.168.100.1/24

# ovn-nbctl lsp-add ls1 ls1-router1
# ovn-nbctl lsp-set-type ls1-router1 router
# ovn-nbctl lsp-set-addresses ls1-router1 02:ac:10:ff:00:01
# ovn-nbctl lsp-set-options ls1-router1 router-port=router1-ls1
```

查看逻辑网络
```shell
# ovn-nbctl show
switch 2f270b61-0212-466f-9f13-c83a1effc8ab (ls1)
    port ls1-vm1
        addresses: ["02:ac:10:ff:00:11"]
    port ls1-vm2
        addresses: ["02:ac:10:ff:00:22"]
    port ls1-router1
        type: router
        addresses: ["02:ac:10:ff:00:01"]
        router-port: router1-ls1
router b5ea7920-b397-400b-95cb-5e0b1ba89672 (router1)
    port router1-ls1
        mac: "02:ac:10:ff:00:01"
        networks: ["192.168.100.1/24"]
```

### 创建gateway router gw1
在node1上添加gateway router，首先将node1注册到OVN Central上
```shell
# ovs-vsctl set open . external_ids:ovn-remote=tcp:192.168.209.110:6642
# ovs-vsctl set open . external_ids:ovn-encap-type=geneve
# ovs-vsctl set open . external_ids:ovn-encap-ip=192.168.209.110
```
查看已注册的chassis
```shell
# ovn-nbctl show
Chassis "64ad7556-51ed-4de9-9df4-9dd017b27978"
    hostname: "node1"
    Encap geneve
        ip: "192.168.209.110"
        options: {csum="true"}
Chassis "3e7c3161-1b09-4f4d-8591-3c16021b4f60"
    hostname: "node2"
    Encap geneve
        ip: "192.168.209.111"
        options: {csum="true"}
    Port_Binding "ls1-vm1"
Chassis "78f1fb86-fb9e-4ba3-985a-4463fab94153"
    hostname: "node3"
    Encap geneve
        ip: "192.168.209.112"
        options: {csum="true"}
    Port_Binding "ls1-vm2"
```

创建router gw1和switcher ls2
```shell
# ovn-nbctl create Logical_Router name=gw1 options:chassis=64ad7556-51ed-4de9-9df4-9dd017b27978

# ovn-nbctl ls-add ls2

# ovn-nbctl lrp-add gw1 gw1-ls2 02:ac:10:ff:ff:01 192.168.255.1/24
# ovn-nbctl lsp-add ls2 ls2-gw1
# ovn-nbctl lsp-set-type ls2-gw1 router
# ovn-nbctl lsp-set-addresses ls2-gw1 02:ac:10:ff:ff:01
# ovn-nbctl lsp-set-options ls2-gw1 router-port=gw1-ls2

# ovn-nbctl lrp-add router1 router1-ls2 02:ac:10:ff:ff:02 192.168.255.2/24
# ovn-nbctl lsp-add ls2 ls2-router1
# ovn-nbctl lsp-set-type ls2-router1 router
# ovn-nbctl lsp-set-addresses ls2-router1 02:ac:10:ff:ff:02
# ovn-nbctl lsp-set-options ls2-router1 router-port=router1-ls2
```

添加静态路由
```shell
# ovn-nbctl lr-route-add gw1 "192.168.100.0/24" 192.168.255.2
# ovn-nbctl lr-route-add router1 "0.0.0.0/0" 192.168.255.1
```

查看逻辑网络:
```shell
# ovn-nbctl show
switch 2f270b61-0212-466f-9f13-c83a1effc8ab (ls1)
    port ls1-vm1
        addresses: ["02:ac:10:ff:00:11"]
    port ls1-vm2
        addresses: ["02:ac:10:ff:00:22"]
    port ls1-router1
        type: router
        addresses: ["02:ac:10:ff:00:01"]
        router-port: router1-ls1
switch 1c04624e-765a-44f8-8bc5-b4b33aa79570 (ls2)
    port ls2-router1
        type: router
        addresses: ["02:ac:10:ff:ff:02"]
        router-port: router1-ls2
    port ls2-gw1
        type: router
        addresses: ["02:ac:10:ff:ff:01"]
        router-port: gw1-ls2
router b5ea7920-b397-400b-95cb-5e0b1ba89672 (router1)
    port router1-ls1
        mac: "02:ac:10:ff:00:01"
        networks: ["192.168.100.1/24"]
    port router1-ls2
        mac: "02:ac:10:ff:ff:02"
        networks: ["192.168.255.2/24"]
router dc7088ef-62f3-446d-999e-d372ad85b623 (gw1)
    port gw1-ls2
        mac: "02:ac:10:ff:ff:01"
        networks: ["192.168.255.1/24"]
```

从node2上访问gw1
```shell
# ip netns exec vm1 ip route add default via 192.168.100.1

# ip netns exec vm1 ping 192.168.255.1 -c 3
```

### 连接overlay network和physical network
```shell
# ovn-nbctl lrp-add gw1 gw1-outside 02:0a:7f:18:01:02 192.168.209.120/19

# ovn-nbctl ls-add outside
# ovn-nbctl lsp-add outside outside-gw1
# ovn-nbctl lsp-set-type outside-gw1 router
# ovn-nbctl lsp-set-addresses outside-gw1 02:0a:7f:18:01:02
# ovn-nbctl lsp-set-options outside-gw1 router-port=gw1-outside

# ovs-vsctl add-br br-eth1
# ovs-vsctl set Open_vSwitch . external_ids:ovn-bridge-mappings=phyNet:br-eth1

# ovn-nbctl lsp-add outside outside-localnet
# ovn-nbctl lsp-set-addresses outside-localnet unknown
# ovn-nbctl lsp-set-type outside-localnet localnet
# ovn-nbctl lsp-set-options outside-localnet network_name=phyNet

# ovs-vsctl add-port br-eth1 eth1
```

在node1上查看逻辑网络
```shell
# ovn-nbctl show
switch 2ef65fa4-d56d-4e7c-a393-5e28f2e7abb5 (ls1)
    port ls1-vm2
        addresses: ["02:ac:10:ff:00:22"]
    port ls1-router1
        type: router
        addresses: ["02:ac:10:ff:00:01"]
        router-port: router1-ls1
    port ls1-vm1
        addresses: ["02:ac:10:ff:00:11"]
switch 70c9d700-8318-4095-913e-8e6760eb420f (outside)
    port outside-localnet
        type: localnet
        addresses: ["unknown"]
    port outside-gw1
        type: router
        addresses: ["02:0a:7f:18:01:02"]
        router-port: gw1-outside
switch 9e8f718d-d8d4-429d-9e4b-bbea1aad0e96 (ls2)
    port ls2-gw1
        type: router
        addresses: ["02:ac:10:ff:ff:01"]
        router-port: gw1-ls2
    port ls2-router1
        type: router
        addresses: ["02:ac:10:ff:ff:02"]
        router-port: router1-ls2
router b7571af4-fe69-43d5-9497-74012699b625 (gw1)
    port gw1-ls2
        mac: "02:ac:10:ff:ff:01"
        networks: ["192.168.255.1/24"]
    port gw1-outside
        mac: "02:0a:7f:18:01:02"
        networks: ["192.168.209.120/24"]
router ce66beae-df90-4a61-ba78-c733a31315b1 (router1)
    port router1-ls1
        mac: "02:ac:10:ff:00:01"
        networks: ["192.168.100.1/24"]
    port router1-ls2
        mac: "02:ac:10:ff:ff:02"
        networks: ["192.168.255.2/24"]
```

vm1访问gw1-outside
```shell
# ip netns exec vm1 ping 192.168.209.120 -c 1
PING 192.168.209.120 (192.168.209.120) 56(84) bytes of data.
64 bytes from 192.168.209.120: icmp_seq=1 ttl=253 time=2.61 ms

--- 192.168.209.120 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 2.611/2.611/2.611/0.000 ms
```

### 在与node1同一个L2网络的节点访问overlay network
以192.168.209.10主机为例
```shell
# ping 192.168.209.120 -c 3
PING 192.168.209.120 (192.168.209.120) 56(84) bytes of data.
64 bytes from 192.168.209.120: icmp_seq=1 ttl=254 time=1.51 ms
64 bytes from 192.168.209.120: icmp_seq=2 ttl=254 time=0.827 ms
64 bytes from 192.168.209.120: icmp_seq=3 ttl=254 time=0.846 ms

--- 192.168.209.120 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2027ms
rtt min/avg/max/mdev = 0.827/1.061/1.512/0.320 ms
```

添加访问192.168.100.0/24的路由，并测试连通性
```shell
# ip route add 192.168.100.0/24 via 192.168.209.120

# ping 192.168.100.10 -c 3
PING 192.168.100.10 (192.168.100.10) 56(84) bytes of data.
64 bytes from 192.168.100.10: icmp_seq=1 ttl=62 time=3.50 ms
64 bytes from 192.168.100.10: icmp_seq=2 ttl=62 time=1.17 ms
64 bytes from 192.168.100.10: icmp_seq=3 ttl=62 time=1.02 ms

--- 192.168.100.10 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 1.027/1.901/3.502/1.134 ms
```

现在ping主机vm2也不通，因为vm2没有添加出外网的路由，在node3上
```shell
# ip netns exec vm2 ip route add default via 192.168.100.1
```
然后在192.168.209.10主机上
```shell
# ping 192.168.209.10 -c 3
PING 192.168.100.20 (192.168.100.20) 56(84) bytes of data.
64 bytes from 192.168.100.20: icmp_seq=1 ttl=62 time=3.29 ms
64 bytes from 192.168.100.20: icmp_seq=2 ttl=62 time=1.00 ms
64 bytes from 192.168.100.20: icmp_seq=3 ttl=62 time=1.19 ms

--- 192.168.100.20 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 1.001/1.831/3.296/1.039 ms
```

### overlay访问物理网络(通过NAT)
在node1上
```shell
# ovn-nbctl -- --id=@nat create nat type="snat" logical_ip=192.168.100.0/24 external_ip=192.168.209.120 -- add logical_router gw1 nat @nat
e4b51f21-4c8c-4d0f-9c06-a5db0c7900bd

# ovn-nbctl lr-nat-list gw1
TYPE             EXTERNAL_IP        LOGICAL_IP            EXTERNAL_MAC         LOGICAL_PORT
snat             192.168.209.120    192.168.100.0/24

# ovn-nbctl list NAT
_uuid               : e4b51f21-4c8c-4d0f-9c06-a5db0c7900bd
external_ids        : {}
external_ip         : "192.168.209.120"
external_mac        : []
logical_ip          : "192.168.100.0/24"
logical_port        : []
type                : snat
```

注：（删除nat规则）
```shell
# ovn-nbctl lr-nat-del gw1 snat 192.168.100.0/24
```

vm1访问192.168.209.10，在node2上执行
```shell
# ip netns exec vm1 ping -c 1 192.168.209.10
PING 192.168.209.10 (192.168.209.10) 56(84) bytes of data.
64 bytes from 192.168.209.10: icmp_seq=1 ttl=62 time=2.30 ms

--- 192.168.209.10 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 2.308/2.308/2.308/0.000 ms
```

