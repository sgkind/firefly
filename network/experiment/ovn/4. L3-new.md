# ovn创建L3网络

## 一、目标
### 物理网络
4台ubuntu18.04主机
* node110 192.168.209.110   作为OVN Central
* node111 192.168.209.111   作为Host
* node112 192.168.209.112   作为Host
* node113 192.168.209.113 192.168.209.114 作为网关

### 逻辑网络
                                 
                                外网 192.168.200.0/19
                             ____|____ 
                            | node113 | ens3  192.168.209.114/19
                             ---------  ens4  192.168.209.155/19
                             ____|____ 
                            |  switch | gw  172.19.255.0/24
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

## 二、过程

**需要将node113上面与ens4相关的路由项删除**

### router下的网络创建过程见L3.md

### node113安装软件
```shell
# apt install -y openvswitch-switch openvswitch-common ovn-common ovn-host
```

### 将node113连接到OVN Central
```shell
# ovs-vsctl set open . external-ids:ovn-remote=tcp:192.168.209.110:6642
# ovs-vsctl set open . external-ids:ovn-encap-type=geneve
# ovs-vsctl set open . external-ids:ovn-encap-ip=192.168.209.113
```

### 在node110上，将路由器router1的chassis设置为node113
```shell
# ovn-nbctl set Logical_router router1 options:chassis=<chassis of node113>
```

### 在node110上，添加交换机gw
```shell
# ovn-nbctl ls-add gw
# ovn-nbctl lsp-add gw local_port
# ovn-nbctl lsp-set-addresses local_port unknown
# ovn-nbctl lsp-set-type local_port localnet
# ovn-nbctl lsp-set-options local_port network_name=physNet-node114
```

### 在node110上，绑定交换机gw和路由器router1
```shell
# ovn-nbctl lrp-add router1 r1-gw 00:00:00:10:00:00 192.168.209.155/19

# ovn-nbctl lsp-add gw gw-r1
# ovn-nbctl lsp-set-type gw-r1 router
# ovn-nbctl lsp-set-addresses gw-r1 00:00:00:10:00:00
# ovn-nbctl lsp-set-options gw-r1 router-port=r1-gw
```

### 在node113上，配置网桥
```shell
# ovs-vsctl add-br br-ex
# ovs-vsctl add-port br-ex ens4
# ovs-vsctl set Open_vSwitch . external-ids:ovn-bridge-mappings=physNet-node114:br-ex
```

### 在子网192.168.200.0/19内的主机上访问192.168.209.155
```shell
# ping -c 192.168.209.155

```

### 在vm1中访问192.168.209.155
```shell
# ip netns exec vm1 ping -c 3 192.168.209.155

```

### 添加snat，使overlay网络可以访问物理网络
在node110上
```shell
# ovn-nbctl -- --id=@nat create nat type="snat" logical_ip=172.18.0.0/16 external_ip=192.168.209.155 -- add logical_router router1 nat @nat
```

### 添加dnat
在node110上
```shell
# ovn-nbctl -- --id=@nat create nat type="dnat" logical_ip=192.168.209.156 external_ip=172.18.100.10 -- add logical_router router1 nat @nat
```