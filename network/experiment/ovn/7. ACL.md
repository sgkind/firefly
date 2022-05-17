OVN访问控制列表--ACL
===

## 介绍
OVN中的ACL规则存储于北向数据库的ACL表中，并且可以使用ovn-nbctl的acl命令进行配置。目前，ACL只能应用于逻辑交换机，但是未来将支持应用到逻辑路由器。

在出站和入站方向都支持使用ACL：
* 入站：从工作负载（to-lport）进入逻辑端口
* 出站：从逻辑端口出去到工作负载（from-lport）

此外，为每个ACL分配一个优先级，以确定它们的匹配顺序，最高优先级首先被匹配。另外ACL可以被赋予相同的优先级。 然而，在两个具有相同优先级并且都匹配同一个分组的ACL的情况下，将仅匹配一个ACL。确切地说，哪一个ACL最终将被匹配是不确定的，你不能真正地确定哪个规则将应用于给定的情况。所以我建议：在大多数情况下尽量使用不同的优先级。

ACL中的匹配规则基于OVS中的流语法，对于具有编程背景的人都会觉得很熟悉。该语法在ovn-sb手册页的“Logical_Flow表”部分中进行了说明。它值得一读，特别是介绍“！=”匹配规则的那部分内容。

另外值得强调的一点是，**您不能在具有type = router的端口上创建ACL匹配规则**。为了减少ACL表中的条目数量，可以使用定义相同类型地址组的地址集。例如，一组IPv4地址/网络，一组MAC地址或一组IPv6地址可以放置在一个可命名的地址集内。为了减少ACL表中的条目数，可以使用定义相同类型地址组的地址集。 例如，一组IPv4地址/网络，一组MAC地址或一组IPv6地址可以放置在命名地址集内。 然后地址集可以被ACL的match子句内的“name”引用（以$ name的形式）。

## 实验网络拓扑
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
* node2 192.168.209.111/19 作为OVN Host  vm1 vm3 vm5
* node3 192.168.209.112/19 作为OVN Host  vm2 vm4 vm6

### 逻辑网络
                           ____ ____ 
                          |  router |    port 'router1-ls1':  172.18.100.1/24
                           ---------     port 'router1-ls2':  172.18.200.1/24
                               |         port 'router1-ls3':  172.18.250.1/24
             -----------------------------------------------------------------------
         ____|____                         ____|____                           ____|____
        | switch  | ls1 172.18.100.0/24   |  switch | ls2 172.18.200.0/24     |  switch | ls3 172.18.250.0/24
         ---------                         ---------                           ---------
        /       \                          /       \                           /       \
  _____/_       _\_______          _______/_       _\_______           _______/_       _\_______ 
 |  vm1  |     |   vm2   |        |  vm3    |     |   vm4   |         |  vm5    |     |   vm6   |
 ---------      ---------          ---------       ---------           ---------       ---------
172.18.100.10  172.18.100.20     172.18.200.10    172.18.200.20       172.18.250.10    172.18.250.20

## 实验过程

### 阻止外部主机访问vm1
```shell
# ip netns exec vm2 ping -c 1  172.18.100.10
PING 172.18.100.10 (172.18.100.10) 56(84) bytes of data.
64 bytes from 172.18.100.10: icmp_seq=1 ttl=64 time=8.34 ms

--- 172.18.100.10 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 8.342/8.342/8.342/0.000 ms

# ovn-nbctl acl-add ls1 to-lport 900 "outport == \"ls1-vm1\" && ip" drop

# ip netns exec vm2 ping -c 3 172.18.100.10
PING 172.18.100.10 (172.18.100.10) 56(84) bytes of data.

--- 172.18.100.10 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2025ms
```

### 恢复vm2的访问
```shell
# ovn-nbctl acl-del ls1
# ovn-nbctl acl-add ls1 to-lport 900 "outport == \"ls1-vm1\" && ip" allow-related

# ip netns exec vm3 ping -c 3 172.18.100.10
PING 172.18.100.10 (172.18.100.10) 56(84) bytes of data.
64 bytes from 172.18.100.10: icmp_seq=1 ttl=64 time=2.69 ms
64 bytes from 172.18.100.10: icmp_seq=2 ttl=64 time=1.03 ms
64 bytes from 172.18.100.10: icmp_seq=3 ttl=64 time=0.545 ms

--- 172.18.100.10 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 0.545/1.422/2.690/0.918 ms
```

清理
```shell
# ovn-nbctl acl-del ls1
```

### 仅允许到vm1和vm2的https连接
```shell
# ovn-nbctl acl-add ls1 to-lport 900 "outport == \"ls1-vm1\" && ip" drop
# ovn-nbctl acl-add ls1 to-lport 900 "outport == \"ls1-vm2\" && ip" drop
# ovn-nbctl acl-add ls1 to-lport 1000 "outport == \"ls1-vm1\" && tcp.dst == 443" allow-related
# ovn-nbctl acl-add ls1 to-lport 1000 "outport == \"ls1-vm2\" && tcp.dst == 443" allow-related
```

在vm3中ping主机vm1，不通
```shell
# ip netns exec vm3 ping -c 3 172.18.100.10
PING 172.18.100.10 (172.18.100.10) 56(84) bytes of data.

--- 172.18.100.10 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2056ms
```

在node2和node3上安装nmap
```shell
# apt install nmap
```

在vm1和vm2的443端口运行监听程序
```shell
# ip netns exec vm1 ncat -l -p 443
```

```shell
# ip netns exec vm2 ncat -l -p 443
```

在vm3中连接vm1和vm2的443端口
```shell
# ip netns exec vm3 ncat -w 1 172.10.100.10 443
^C
# ip netns exec vm3 ncat -w 1 172.10.100.20 443
^C
```
如果连接失败，程序会在1s后退出，如连接成功程序保持打开状态。可以看出443端口可以连通，ping失败。

清理
```shell
# ovn-nbctl acl-del ls1
```

### 仅允许从指定IP的指定端口访问vm1和vm2
创建ACL规则
```shell
# ovn-nbctl create Address_Set name=client addresses="172.18.200.0/24"

# ovn-nbctl acl-add ls1 to-lport 1000 'outport == "ls1-vm1" && ip4.src == $client && tcp.dst == 3306' allow-related
# ovn-nbctl acl-add ls1 to-lport 1000 'outport == "ls1-vm2" && ip4.src == $client && tcp.dst == 3306' allow-related

# ovn-nbctl acl-add ls1 to-lport 900 'outport == "ls1-vm1" && ip' drop
# ovn-nbctl acl-add ls1 to-lport 900 'outport == "ls1-vm2" && ip' drop
```

vm3主机ping vm1，不通
```shell
# ip netns exec vm3 ping -c 3 172.18.100.10
PING 172.18.100.10 (172.18.100.10) 56(84) bytes of data.

--- 172.18.100.10 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2037ms
```

vm6和vm1之间3306之间的连接
```shell
# ip netns exec vm1 ncat -l -p 3306
```

```shell
# ip netns exec vm6 ncat -w 1 172.18.100.10 3306
cat: Connection timed out.
```

```shell
# ip netns exec vm4 ncat -w 1 172.18.100.10 3306
^C
```

vm6与vm1之间3306端口不通，vm4与vm1之间3306端口通