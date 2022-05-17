# ovn实现dhcp

## 目标
### 物理网络
2台ubuntu18.04主机
* ovn150 192.168.209.150
* ovn151 192.168.209.151

### 逻辑网络
创建如下图所示的网络

                +--------+
                |        |
                | router |
                |        |
                +--------+
                     |
                     |
                +--------+
                |        |
                |switcher|
                |        |
                +--------+
                     |
                     |
                    vm1
             02:ac:10:ff:01:30
              ipv4地址自动获取

## 过程
### 安装软件
#### ovn150作为central
安装openvswitch-switch、openvswitch-common、ovn-common、ovn-central
```shell
# apt install openvswitch-switch openvswitch-common ovn-common ovn-central ovn-host
```

#### ovn151作为host
安装openvswitch-switch、openvswitch-common、ovn-common和ovn-host
```shell
# apt install openvswitch-switch openvswitch-common ovn-common ovn-host
```

### central上启动OVN南、北向数据库监听端口
```shell
# ovn-nbctl set-connection ptcp:6641:0.0.0.0
# ovn-sbctl set-connection ptcp:6642:0.0.0.0
```

查看结果
```shell
# netstat -lntp | grep 664[1-2]
tcp        0      0 0.0.0.0:6641            0.0.0.0:*               LISTEN      1467/ovsdb-server   
tcp        0      0 0.0.0.0:6642            0.0.0.0:*               LISTEN      1478/ovsdb-server
```

### 创建集成化网桥
OVS的网桥要连接到一个集成化网桥后才能够被OVN管理。通常给这个集成化网桥命名为"br-int".

在ovn151上执行：
```shell
# ovs-vsctl add-br br-int -- set Bridge br-int fail-mode=secure
```

### 将Chassis控制器连接到中央控制器上
在ovn151上执行
```shell
# ovs-vsctl set open . external_ids:ovn-remote=tcp:192.168.209.150:6642
# ovs-vsctl set open . external_ids:ovn-encap-type=geneve
# ovs-vsctl set open . external_ids:ovn-encap-ip=192.168.209.151
```

查看是否连接到ovn central
```shell
# netstat -antp | grep 192.168.209.150
tcp        0      0 192.168.209.151:51980   192.168.209.150:6642    ESTABLISHED 1567/ovn-controller
```

### 在ovn150中查看已连接的chassis
```shell
# ovn-sbctl show
Chassis "2ed3bb26-234a-4cfa-9302-7d90fe337051"
    hostname: "ovn151"
    Encap geneve
        ip: "192.168.209.151"
        options: {csum="true"}
```

### 创建逻辑网络
在central(ovn150)上

创建路由器和交换机
```shell
# ovn-nbctl lr-add tenant1
# ovn-nbctl ls-add dmz 

# ovn-nbctl lrp-add tenant1 tenant1-dmz 02:ac:10:ff:02:01 172.16.209.1/24
# ovn-nbctl lsp-add dmz dmz-tenant1
# ovn-nbctl lsp-set-type dmz-tenant1 router
# ovn-nbctl lsp-set-addresses dmz-tenant1 02:ac:10:ff:02:01
# ovn-nbctl lsp-set-options dmz-tenant1 router-port=tenant1-dmz

# ovn-nbctl lsp-add dmz dmz-vm1
# ovn-nbctl lsp-set-addresses dmz-vm1 "02:ac:10:ff:02:02 172.16.209.2"
# ovn-nbctl lsp-set-port-security dmz-vm1 "02:ac:10:ff:02:02 172.16.209.2"
```

创建dhcp
```shell
# dhcp=`ovn-nbctl create DHCP_Options cidr=172.16.209.0/24 options="\"server_id\"=\"172.16.209.1\" \"server_mac\"=\"02:ac:10:ff:02:01\" \"lease_time\"=\"3600\" \"router\"=\"172.16.209.1\""`
# ovn-nbctl lsp-set-dhcpv4-options dmz-vm1 $dhcp
```

查看北向数据
```shell
# ovn-nbctl show
switch 5d027c40-175f-4e41-91a0-24811e171687 (dmz)
    port dmz-tenant1
        addresses: ["02:ac:10:ff:01:29"]
        router-port: tenant1-dmz
    port dmz-vm1
        addresses: ["02:ac:10:ff:01:30 172.16.209.2"]
router 545b3f10-54ab-49cf-8905-68329f63cc9d (tenant1)
    port tenant1-dmz
        mac: "02:ac:10:ff:01:29"
        networks: ["172.16.209.1/24"]
```

### 在ovn151添加伪虚拟机
```shell
# ip netns add vm1
# ovs-vsctl add-port br-int vm1 -- set interface vm1 type=internal
# ip link set vm1 address 02:ac:10:ff:02:02
# ip link set vm1 netns vm1
# ovs-vsctl set Interface vm1 external_ids:iface-id=dmz-vm1
# ip netns exec vm1 dhclient vm1
```

### 查看vm1是否正确获取ip地址
```shell
# ip netns exec vm1 ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
5: vm1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether 02:ac:10:ff:02:02 brd ff:ff:ff:ff:ff:ff
    inet 172.16.209.2/24 brd 172.16.209.255 scope global vm1
       valid_lft forever preferred_lft forever
    inet6 fe80::ac:10ff:feff:202/64 scope link 
       valid_lft forever preferred_lft forever
```

## 清理
1. ovn161
```shell
# ip netns del vm1
# ovs-vsctl --if-exists --with-iface del-port br-int vm1

# ovs-vsctl remove open . external_ids ovn-remote
# ovs-vsctl remove open . external_ids ovn-encap-type
# ovs-vsctl remove open . external_ids ovn-encap-ip
```
2. ovn160
```shell
# ovn-nbctl lr-del denant1
# ovn-nbctl ls-del dmz

# ovn-nbctl del-connection
# ovn-sbctl del-connection

# ovn-nbctl dhcp-options-del $dhcp

# ovn-sbctl show
# ovn-sbctl chassis-del
```