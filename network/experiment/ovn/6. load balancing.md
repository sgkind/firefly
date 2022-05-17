## 一、目标
### 物理网络


      #-------------------------------------------+ 192.168.209.0/19
            |               |              |            
            |               |              |            
        +-------+       +-------+      +-------+    
        |       |       |       |      |       |     
        | node1 |       | node2 |      | node3 |     
        |       |       |       |      |       | 
        +-------+       +-------+      +-------+ 
ens3 192.168.209.110 192.168.209.111 192.168.209.112
ens9 192.168.209.150

* node1 192.168.209.110/19 作为OVN Central和网关
* node2 192.168.209.111/19 作为OVN Host
* node3 192.168.209.112/19 作为OVN Host

### 逻辑网络
                             --------- 
                            |  client | 192.168.209.115/19 Physical Network
                             ---------
                             ____|____ 
                            |  switch | outside
                             ---------
                                 |
                             ____|____ 
                            |  router | gw1 port 'gw-outside': 192.168.209.120/19
                             ---------      port 'gw-ls2':     172.19.255.1/24
                             ____|____ 
                            |  switch | ls3  172.19.255.0/24
                             ---------  
                             ____|____ 
                            |  router | router1 port 'router1-ls3':  172.19.255.2/24
                             ---------          port 'router1-ls1':  172.18.100.1/24
                                 |              port 'router1-ls2':  172.18.200.1/24
               -----------------------------------
           ____|____                          ____|____ 
          | switch  | ls1 172.18.100.0/24   |  switch | ls2 172.18.200.0/24
           ---------                          ---------  
           /       \                          /       \
     _____/_       _\_______          _______/_       _\_______  
    |  vm1  |     |   vm2   |        |  vm3    |     |   vm4   |
    ---------      ---------          ---------       ---------
  172.18.100.10  172.18.100.20     172.18.200.10    172.18.200.20

  **注：node1因为有两个网卡，在用netplan管理网卡时，需确定ip route只用110的网卡，否则将150的网卡加入到br-eth1后，node1会断网**

## 二、实现
### 1. 安装软件
#### node1作为OVN Central
安装openvswitch-switch、openvswitch-common、ovn-common和ovn-central
```shell
# apt install openvswitch-switch openvswitch-common ovn-common ovn-central
```

#### node2、node3和node4作为OVN Host
安装openvswitch-switch、openvswitch-common、ovn-common和ovn-host
```shell
# apt install openvswitch-switch openvswitch-common ovn-common ovn-host
```

### 2. 在OVN Central(node1)上开启南、北向数据库监听端口
```shell
# ovn-nbctl set-connection ptcp:6641:0.0.0.0
# ovn-sbctl set-connection ptcp:6642:0.0.0.0
```

### 3. 创建集成化网桥
在node2和node3节点上：
```shell
# ovs-vsctl add-br br-int -- set Bridge br-int fail-mode=secure
```

### 4. 将OVN Host连接到OVN Central上
1. 在node2上
```shell
# ovs-vsctl set open . external-ids:ovn-remote=tcp:192.168.209.110:6642
# ovs-vsctl set open . external-ids:ovn-encap-type=geneve
# ovs-vsctl set open . external-ids:ovn-encap-ip=192.168.209.111
```

2. 在node3上
```shell
# ovs-vsctl set open . external-ids:ovn-remote=tcp:192.168.209.110:6642
# ovs-vsctl set open . external-ids:ovn-encap-type=geneve
# ovs-vsctl set open . external-ids:ovn-encap-ip=192.168.209.112
```

3. 在node1上查看已连接的chassis
```shell
# ovn-sbctl show
Chassis "50583630-5cf9-42a6-8f79-fe676049b591"
    hostname: "node3"
    Encap geneve
        ip: "192.168.209.112"
        options: {csum="true"}
Chassis "14e60703-321a-4be7-ad04-846eb1ad114d"
    hostname: "node2"
    Encap geneve
        ip: "192.168.209.111"
        options: {csum="true"}
```

### 5. 创建switch ls1和ls2
```shell
# ovn-nbctl ls-add ls1

# ovn-nbctl lsp-add ls1 ls1-vm1
# ovn-nbctl lsp-set-addresses ls1-vm1 02:ac:10:ff:00:10
# ovn-nbctl lsp-set-port-security ls1-vm1 02:ac:10:ff:00:10

# ovn-nbctl lsp-add ls1 ls1-vm2
# ovn-nbctl lsp-set-addresses ls1-vm2 02:ac:10:ff:00:20
# ovn-nbctl lsp-set-port-security ls1-vm2 02:ac:10:ff:00:20

# ovn-nbctl ls-add ls2

# ovn-nbctl lsp-add ls2 ls2-vm3
# ovn-nbctl lsp-set-addresses ls2-vm3 02:ac:10:ff:10:10
# ovn-nbctl lsp-set-port-security ls2-vm3 02:ac:10:ff:10:10

# ovn-nbctl lsp-add ls2 ls2-vm4
# ovn-nbctl lsp-set-addresses ls2-vm4 02:ac:10:ff:10:20
# ovn-nbctl lsp-set-port-security ls2-vm4 02:ac:10:ff:10:20
```

查看逻辑网络
```shell
# ovn-nbctl show
switch 224bab9f-36f6-42f9-b9b2-8ef0b0e102d5 (ls2)
    port ls2-vm3
        addresses: ["02:ac:10:ff:10:10"]
    port ls2-vm4
        addresses: ["02:ac:10:ff:10:20"]
switch 860460ef-7e75-4687-9f27-9d224fd5a180 (ls1)
    port ls1-vm2
        addresses: ["02:ac:10:ff:00:20"]
    port ls1-vm1
        addresses: ["02:ac:10:ff:00:10"]
```

### 6. 添加虚拟机vm1、vm2、vm3和vm4
在node2上添加虚拟机vm1、vm4
```shell
# ip netns add vm1
# ovs-vsctl add-port br-int vm1 -- set interface vm1 type=internal
# ip link set vm1 netns vm1
# ip netns exec vm1 ip link set vm1 address 02:ac:10:ff:00:10
# ip netns exec vm1 ip addr add 172.18.100.10/24 dev vm1
# ip netns exec vm1 ip link set vm1 up
# ovs-vsctl set Interface vm1 external_ids:iface-id=ls1-vm1

# ip netns add vm4
# ovs-vsctl add-port br-int vm4 -- set interface vm4 type=internal
# ip link set vm4 netns vm4
# ip netns exec vm4 ip link set vm4 address 02:ac:10:ff:10:20
# ip netns exec vm4 ip addr add 172.18.200.20/24 dev vm4
# ip netns exec vm4 ip link set vm4 up
# ovs-vsctl set Interface vm4 external_ids:iface-id=ls2-vm4
```

在node3上添加虚拟机vm2、vm3
```shell
# ip netns add vm2
# ovs-vsctl add-port br-int vm2 -- set interface vm2 type=internal
# ip link set vm2 netns vm2
# ip netns exec vm2 ip link set vm2 address 02:ac:10:ff:00:20
# ip netns exec vm2 ip addr add 172.18.100.20/24 dev vm2
# ip netns exec vm2 ip link set vm2 up
# ovs-vsctl set Interface vm2 external_ids:iface-id=ls1-vm2

# ip netns add vm3
# ovs-vsctl add-port br-int vm3 -- set interface vm3 type=internal
# ip link set vm3 netns vm3
# ip netns exec vm3 ip link set vm3 address 02:ac:10:ff:10:10
# ip netns exec vm3 ip addr add 172.18.200.10/24 dev vm3
# ip netns exec vm3 ip link set vm3 up
# ovs-vsctl set Interface vm3 external_ids:iface-id=ls2-vm3
```

### 7. 创建路由器router1
```shell
# ovn-nbctl lr-add router1

# ovn-nbctl lrp-add router1 router1-ls1 02:ac:10:ff:00:01 172.18.100.1/24

# ovn-nbctl lsp-add ls1 ls1-router1
# ovn-nbctl lsp-set-type ls1-router1 router
# ovn-nbctl lsp-set-addresses ls1-router1 02:ac:10:ff:00:01
# ovn-nbctl lsp-set-options ls1-router1 router-port=router1-ls1

# ovn-nbctl lrp-add router1 router1-ls2 02:ac:10:ff:10:01 172.18.200.1/24

# ovn-nbctl lsp-add ls2 ls2-router1
# ovn-nbctl lsp-set-type ls2-router1 router
# ovn-nbctl lsp-set-addresses ls2-router1 02:ac:10:ff:10:01
# ovn-nbctl lsp-set-options ls2-router1 router-port=router1-ls2
```

查看逻辑网络
```shell
# ovn-nbctl show
switch 224bab9f-36f6-42f9-b9b2-8ef0b0e102d5 (ls2)
    port ls2-vm3
        addresses: ["02:ac:10:ff:10:10"]
    port ls2-vm4
        addresses: ["02:ac:10:ff:10:20"]
    port ls2-router1
        type: router
        addresses: ["02:ac:10:ff:10:01"]
        router-port: router1-ls2
switch 860460ef-7e75-4687-9f27-9d224fd5a180 (ls1)
    port ls1-vm2
        addresses: ["02:ac:10:ff:00:20"]
    port ls1-vm1
        addresses: ["02:ac:10:ff:00:10"]
    port ls1-router1
        type: router
        addresses: ["02:ac:10:ff:00:01"]
        router-port: router1-ls1
router 9eed94cc-3c2a-4c06-a767-5d15e0c8d677 (router1)
    port router1-ls2
        mac: "02:ac:10:ff:10:01"
        networks: ["172.18.200.1/24"]
    port router1-ls1
        mac: "02:ac:10:ff:00:01"
        networks: ["172.18.100.1/24"]
```

### 8. 测试vm1、vm2与vm3和vm4之间的联通性
在vm1中添加路由
```shell
# ip netns exec vm1 ip route add default via 172.18.100.1
```
在vm2中添加路由
```shell
# ip netns exec vm2 ip route add default via 172.18.100.1
```
在vm3中添加路由
```shell
# ip netns exec vm3 ip route add default via 172.18.200.1
```
在vm4中添加路由
```shell
# ip netns exec vm4 ip route add default via 172.18.200.1
```

vm3虚拟机ping vm1
```shell
# ip netns exec vm3 ping 172.18.100.10 -c 3
PING 172.18.100.10 (172.18.100.10) 56(84) bytes of data.
64 bytes from 172.18.100.10: icmp_seq=1 ttl=63 time=1.97 ms
64 bytes from 172.18.100.10: icmp_seq=2 ttl=63 time=0.622 ms
64 bytes from 172.18.100.10: icmp_seq=3 ttl=63 time=0.470 ms

--- 172.18.100.10 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2025ms
rtt min/avg/max/mdev = 0.470/1.022/1.975/0.676 ms
```

vm2虚拟机ping vm4
```shell
# ip netns exec vm2 ping 172.18.200.20 -c 3
PING 172.18.200.20 (172.18.200.20) 56(84) bytes of data.
64 bytes from 172.18.200.20: icmp_seq=2 ttl=63 time=1.80 ms
64 bytes from 172.18.200.20: icmp_seq=3 ttl=63 time=0.449 ms

--- 172.18.200.20 ping statistics ---
3 packets transmitted, 2 received, 33% packet loss, time 2011ms
rtt min/avg/max/mdev = 0.449/1.125/1.801/0.676 ms
```

### 9. 创建gateway router gw1
查看已注册的chassis:
```shell
# ovn-sbctl show
Chassis "50583630-5cf9-42a6-8f79-fe676049b591"
    hostname: "node3"
    Encap geneve
        ip: "192.168.209.112"
        options: {csum="true"}
    Port_Binding "ls2-vm3"
    Port_Binding "ls1-vm2"
Chassis "f0902ac8-b73c-4710-8339-666b0272cdca"
    hostname: "node4"
    Encap geneve
        ip: "192.168.209.113"
        options: {csum="true"}
Chassis "14e60703-321a-4be7-ad04-846eb1ad114d"
    hostname: "node2"
    Encap geneve
        ip: "192.168.209.111"
        options: {csum="true"}
    Port_Binding "ls1-vm1"
    Port_Binding "ls2-vm4"
```

创建router gw1，将其chassis设置为node4，同时创建交换机ls3
```shell
# ovn-nbctl create Logical_Router name=gw1 options:chassis=<chassis of node1>
# ovn-nbctl lr-add gw1

# ovn-nbctl ls-add ls3

# ovn-nbctl lrp-add gw1 gw1-ls3 02:ac:10:ff:ff:01 172.19.255.1/24
# ovn-nbctl lsp-add ls3 ls3-gw1
# ovn-nbctl lsp-set-type ls3-gw1 router
# ovn-nbctl lsp-set-addresses ls3-gw1 02:ac:10:ff:ff:01
# ovn-nbctl lsp-set-options ls3-gw1 router-port=gw1-ls3

# ovn-nbctl lrp-add router1 router1-ls3 02:ac:10:ff:ff:02 172.19.255.2/24
# ovn-nbctl lsp-add ls3 ls3-router1
# ovn-nbctl lsp-set-type ls3-router1 router
# ovn-nbctl lsp-set-addresses ls3-router1 02:ac:10:ff:ff:02
# ovn-nbctl lsp-set-options ls3-router1 router-port=router1-ls3
```

添加静态路由
```shell
# ovn-nbctl lr-route-add gw1 "172.18.0.0/16" 172.19.255.2
# ovn-nbctl lr-route-add router1 "0.0.0.0/0" 172.19.255.1
```

查看逻辑网络:
```shell
# ovn-nbctl show
switch a8f83119-35af-41dc-8ca3-1fac69b93cc2 (ls3)
    port ls3-gw1
        type: router
        addresses: ["02:ac:10:ff:ff:01"]
        router-port: gw1-ls3
    port ls3-router1
        type: router
        addresses: ["02:ac:10:ff:ff:02"]
        router-port: router1-ls3
switch 860460ef-7e75-4687-9f27-9d224fd5a180 (ls1)
    port ls1-vm2
        addresses: ["02:ac:10:ff:00:20"]
    port ls1-vm1
        addresses: ["02:ac:10:ff:00:10"]
    port ls1-router1
        type: router
        addresses: ["02:ac:10:ff:00:01"]
        router-port: router1-ls1
switch 224bab9f-36f6-42f9-b9b2-8ef0b0e102d5 (ls2)
    port ls2-vm3
        addresses: ["02:ac:10:ff:10:10"]
    port ls2-vm4
        addresses: ["02:ac:10:ff:10:20"]
    port ls2-router1
        type: router
        addresses: ["02:ac:10:ff:10:01"]
        router-port: router1-ls2
router 9eed94cc-3c2a-4c06-a767-5d15e0c8d677 (router1)
    port router1-ls3
        mac: "02:ac:10:ff:ff:02"
        networks: ["172.19.255.2/24"]
    port router1-ls2
        mac: "02:ac:10:ff:10:01"
        networks: ["172.18.200.1/24"]
    port router1-ls1
        mac: "02:ac:10:ff:00:01"
        networks: ["172.18.100.1/24"]
router 358a4c35-4c3f-40e4-ad4f-e81c2daeed23 (gw1)
    port gw1-ls3
        mac: "02:ac:10:ff:ff:01"
        networks: ["172.19.255.1/24"]
```

从vm1访问gw1
```shell
# ip netns exec vm1 ping -c 3 172.19.255.1
```

### 10. 连接overlay network和physical network
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

# ovs-vsctl add-port br-eth1 ens9
```

在node1上查看逻辑网络
```shell
# ovn-nbctl show
switch c83d9a90-df55-435b-ae5b-e5b8a832cf88 (outside)
    port outside-gw1
        type: router
        addresses: ["02:0a:7f:18:01:02"]
        router-port: gw1-outside
    port outside-localnet
        type: localnet
        addresses: ["unknown"]
switch a8f83119-35af-41dc-8ca3-1fac69b93cc2 (ls3)
    port ls3-gw1
        type: router
        addresses: ["02:ac:10:ff:ff:01"]
        router-port: gw1-ls3
    port ls3-router1
        type: router
        addresses: ["02:ac:10:ff:ff:02"]
        router-port: router1-ls3
switch 860460ef-7e75-4687-9f27-9d224fd5a180 (ls1)
    port ls1-vm2
        addresses: ["02:ac:10:ff:00:20"]
    port ls1-vm1
        addresses: ["02:ac:10:ff:00:10"]
    port ls1-router1
        type: router
        addresses: ["02:ac:10:ff:00:01"]
        router-port: router1-ls1
switch 224bab9f-36f6-42f9-b9b2-8ef0b0e102d5 (ls2)
    port ls2-vm3
        addresses: ["02:ac:10:ff:10:10"]
    port ls2-vm4
        addresses: ["02:ac:10:ff:10:20"]
    port ls2-router1
        type: router
        addresses: ["02:ac:10:ff:10:01"]
        router-port: router1-ls2
router 9eed94cc-3c2a-4c06-a767-5d15e0c8d677 (router1)
    port router1-ls3
        mac: "02:ac:10:ff:ff:02"
        networks: ["172.19.255.2/24"]
    port router1-ls2
        mac: "02:ac:10:ff:10:01"
        networks: ["172.18.200.1/24"]
    port router1-ls1
        mac: "02:ac:10:ff:00:01"
        networks: ["172.18.100.1/24"]
router 358a4c35-4c3f-40e4-ad4f-e81c2daeed23 (gw1)
    port gw1-outside
        mac: "02:0a:7f:18:01:02"
        networks: ["192.168.209.120/19"]
    port gw1-ls3
        mac: "02:ac:10:ff:ff:01"
        networks: ["172.19.255.1/24"]
```

vm1 访问gw1-outside
```shell
# ip netns exec vm1 ping 192.168.209.120 -c 1
PING 192.168.209.120 (192.168.209.120) 56(84) bytes of data.
64 bytes from 192.168.209.120: icmp_seq=1 ttl=253 time=1.85 ms

--- 192.168.209.120 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.853/1.853/1.853/0.000 ms
```

### 11. 在与node1同一L2网络的节点访问overlay network
以192.168.209.161主机为例：
```shell
# ping 192.168.209.120 -c 1
PING 192.168.209.120 (192.168.209.120) 56(84) bytes of data.
64 bytes from 192.168.209.120: icmp_seq=1 ttl=254 time=1.46 ms

--- 192.168.209.120 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.462/1.462/1.462/0.000 ms
```

添加访问172.18.0.0/16的路由，并测试连通性
```shell
# ip route add 172.18.0.0/16 via 192.168.209.120

# ping 192.168.100.10 -c 1
PING 172.18.100.10 (172.18.100.10) 56(84) bytes of data.
64 bytes from 172.18.100.10: icmp_seq=1 ttl=62 time=3.06 ms

--- 172.18.100.10 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 3.067/3.067/3.067/0.000 ms
```

### 12. overlay访问物理网络(通过NAT)
在node1上
```shell
# ovn-nbctl -- --id=@nat create nat type="snat" logical_ip=172.18.0.0/16 external_ip=192.168.209.120 -- add logical_router gw1 nat @nat
41ba175a-e0e4-45e1-94eb-326f3c931832

# ovn-nbctl lr-nat-list gw1
TYPE             EXTERNAL_IP        LOGICAL_IP            EXTERNAL_MAC         LOGICAL_PORT
snat             192.168.209.120    172.18.0.0/16

# ovn-nbctl list NAT
_uuid               : 41ba175a-e0e4-45e1-94eb-326f3c931832
external_ids        : {}
external_ip         : "192.168.209.120"
external_mac        : []
logical_ip          : "172.18.0.0/16"
logical_port        : []
type                : snat
```

vm1访问192.168.209.161
```shell
# ip netns exec vm1 ping -c 1 192.168.209.161
PING 192.168.209.161 (192.168.209.161) 56(84) bytes of data.
64 bytes from 192.168.209.161: icmp_seq=1 ttl=62 time=3.10 ms

--- 192.168.209.161 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 3.100/3.100/3.100/0.000 ms
```

### 13. 在网关路由器gw1上实现负载均衡
#### 在伪虚拟机上开启http服务器
在node2上
```shell
# mkdir /tmp/www1
# echo "i am vm1" > /tmp/www1/index.html
# cd /tmp/www1
# ip netns exec vm1 python -m SimpleHTTPServer 8000
```

在node3上
```shell
# mkdir /tmp/www3
# echo "i am vm3" > /tmp/www3/index.html
# cd /tmp/www3
# ip netns exec vm3 python -m SimpleHTTPServer 8000
```

#### 创建负载均衡
```shell
# uuid=`ovn-nbctl create load_balancer vips:192.168.209.55="172.18.100.10,172.18.200.10"`
```

将创建的负载均衡设置到路由器gw1上
```shell
# ovn-nbctl set logical_router gw1 load_balancer=$uuid
```

#### 在192.168.209.161主机上访问http服务
```shell
# curl 192.168.209.55:8000
i am vm3
# curl 192.168.209.55:8000
i am vm3
# curl 192.168.209.55:8000
i am vm1
```

#### 清理
```shell
# ovn-nbctl lr-lb-del gw1
# ovn-nbctl lb-del $uuid
```

### 14. 在交换机上实现负载均衡
#### 在伪虚拟机上开启http服务器
在node2上
```shell
# mkdir /tmp/www1
# echo "i am vm1" > /tmp/www1/index.html
# cd /tmp/www1
# ip netns exec vm1 python -m SimpleHTTPServer 8000
```

在node3上
```shell
# mkdir /tmp/www2
# echo "i am vm2" > /tmp/www2/index.html
# cd /tmp/www2
# ip netns exec vm2 python -m SimpleHTTPServer 8000
```

#### 创建负载均衡
```shell
# uuid=`ovn-nbctl create load_balancer vips:192.168.2.2="172.18.100.10,172.18.100.20"`
```

将创建的负载均衡设置到交换机ls2上
```shell
# ovn-nbctl set logical_switch ls2 load_balancer=$uuid
```

#### 在node3上通过vm3访问http服务
```shell
# ip netns exec vm3 curl 172.18.2.2:8000
i am vm1
# ip netns exec vm3 curl 172.18.2.2:8000
i am vm2
# ip netns exec vm3 curl 172.18.2.2:8000
i am vm1
```


OVN中的负载均衡可以应用于逻辑交换机或逻辑路由器。
1. 当应用于逻辑路由器时，需要注意：
* 负载均衡只能应用于“集中式”路由器（即网关路由器）
* 路由器上的负载均衡是非分布式的
2. 应用于逻辑交换机时，需要注意
* 负载均衡是“分布式”的，因为它被应用于潜在的多个OVS主机；
* 近在来自VIF(虚拟接口)的流量入口处评估逻辑交换机上的负载均衡。这意味着它必须应用在“客户端”逻辑交换机上，而不是在“服务器”逻辑交换机上。
* 可以根据需要设计对多个逻辑交换机应用负载均衡