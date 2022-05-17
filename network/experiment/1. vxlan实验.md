vxlan实验
===

## 网络拓扑
![](../images/vxlan-1.png)

### linux内核直接实现

#### 采用veth实现
192.168.220.3网卡所在的主机为host1，192.168.220.4网卡所在主机为host2

##### 创建网桥
1. 在host1和host2上分别创建网桥br0
```shell
# ovs-vsctl add-br br0 -- set Bridge br0 fail-mode=standalone
```

##### 创建网络命名空间
1. 在host1上创建网络命名空间vm1和vm2
```shell
# ip netns add vm1
# ip netns add vm2
```

2. 在host2上创建网络命名空间vm3和vm4
```shell
# ip netns add vm3
# ip netns add vm4
```

##### 创建veth端口，
1. 在host1上创建配对端口
```shell
# ip link add eth11 type veth peer name eth12
# ip link add eth21 type veth peer name eth22
```

2. 在host2上创建配对端口
```shell
# ip link add eth31 type veth peer name eth32
# ip link add eth41 type veth peer name eth42
```

##### 将配对端口的一端加入网络命名空间
1. 在host1上
```shell
# ip link set eth11 netns vm1
# ip link set eth21 netns vm2
```

2. 在host2上
```shell
# ip link set eth31 netns vm3
# ip link set eth41 netns vm4
```

##### 配置网络命名空间中的IP地址
1. 在host1上
```shell
# ip netns exec vm1 ip addr add 192.168.0.11/24 dev eth11
# ip netns exec vm2 ip addr add 192.168.0.12/24 dev eth21
```

2. 在host2上
```shell
# ip netns exec vm3 ip addr add 192.168.0.13/24 dev eth31
# ip netns exec vm4 ip addr add 192.168.0.14/24 dev eth41
```

##### 启动各端口
1. 在host1上
```shell
# ip netns exec vm1 ip link set eth11 up
# ip netns exec vm2 ip link set eth21 up
# ip link set eth12 up
# ip link set eth22 up
```

2. 在host2上
```shell
# ip netns exec vm1 ip link set eth31 up
# ip netns exec vm2 ip link set eth41 up
# ip link set eth32 up
# ip link set eth42 up
```

##### 将配对端口的另一端加入网桥
1. 在host1上
```shell
# ovs-vsctl add-port br0 eth12
# ovs-vsctl add-port br0 eth22
```

2. 在host2上
```shell
# ovs-vsctl add-port br0 eth32
# ovs-vsctl add-port br0 eth42
```

##### 验证主机内部网桥各端口的联通性
1. 在host1上
```shell
# ip netns exec vm1 ping -c 3 192.168.0.12
PING 192.168.0.12 (192.168.0.12) 56(84) bytes of data.
64 bytes from 192.168.0.12: icmp_seq=1 ttl=64 time=0.256 ms
64 bytes from 192.168.0.12: icmp_seq=2 ttl=64 time=0.070 ms
64 bytes from 192.168.0.12: icmp_seq=3 ttl=64 time=0.071 ms

--- 192.168.0.12 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2039ms
rtt min/avg/max/mdev = 0.070/0.132/0.256/0.087 ms
```

2. 在host2上
```shell
# ip netns exec vm3 ping -c 3 192.168.0.14
PING 192.168.0.14 (192.168.0.14) 56(84) bytes of data.
64 bytes from 192.168.0.14: icmp_seq=1 ttl=64 time=0.220 ms
64 bytes from 192.168.0.14: icmp_seq=2 ttl=64 time=0.067 ms
64 bytes from 192.168.0.14: icmp_seq=3 ttl=64 time=0.070 ms

--- 192.168.0.14 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2030ms
rtt min/avg/max/mdev = 0.067/0.119/0.220/0.071 ms
```

##### 配置vxlan隧道
1. 在host1上添加vxlan0端口
```shell
# ip link add vxlan0 type vxlan id 42 dstport 4789 remote 192.168.220.4 local 192.168.220.3 dev enp1s0
# ip addr add 20.0.0.1/24 dev vxlan0
# ip link set vxlan0 up
```

2. 在host2上添加vxlan0端口
```shell
# ip link add vxlan0 type vxlan id 42 dstport 4789 remote 192.168.220.3 local 192.168.220.4 dev enp1s0
# ip addr add 20.0.0.2/24 dev vxlan0
# ip link set vxlan0 up
```

##### 验证跨主机的vxlan隧道端口间是否联通
在host1上
```shell
# ping -c 20.0.0.2
PING 20.0.0.2 (20.0.0.2) 56(84) bytes of data.
64 bytes from 20.0.0.2: icmp_seq=1 ttl=64 time=1.58 ms
64 bytes from 20.0.0.2: icmp_seq=2 ttl=64 time=0.892 ms
64 bytes from 20.0.0.2: icmp_seq=3 ttl=64 time=0.622 ms

--- 20.0.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 0.622/1.034/1.589/0.408 ms
```

##### 将vxlan0端口加入网桥
在host1和host2主机上分别执行
```shell
# ovs-vsctl add-port br0 vxlan0
```

##### 验证vxlan网络的联通性
1. 在host1上
```shell
# ip netns exec vm1 ping -c 192.168.0.14
PING 192.168.0.14 (192.168.0.14) 56(84) bytes of data.
64 bytes from 192.168.0.14: icmp_seq=1 ttl=64 time=1.02 ms
64 bytes from 192.168.0.14: icmp_seq=2 ttl=64 time=0.838 ms
64 bytes from 192.168.0.14: icmp_seq=3 ttl=64 time=0.682 ms

--- 192.168.0.14 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 0.682/0.848/1.026/0.144 ms
```

2. 在host2上
```shell
# ip netns exec vm3 ping -c 3 192.168.0.12
PING 192.168.0.12 (192.168.0.12) 56(84) bytes of data.
64 bytes from 192.168.0.12: icmp_seq=1 ttl=64 time=2.06 ms
64 bytes from 192.168.0.12: icmp_seq=2 ttl=64 time=0.927 ms
64 bytes from 192.168.0.12: icmp_seq=3 ttl=64 time=0.751 ms

--- 192.168.0.12 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 0.751/1.248/2.066/0.582 ms
```

#### 采用ovs添加端口实现

##### 创建网桥
1. 在host1和host2上分别创建网桥br0
```shell
# ovs-vsctl add-br br0 -- set Bridge br0 fail-mode=standalone
```

##### 添加端口并设置端口的mac地址
1. 在host1上
```shell
# ovs-vsctl add-port br0 eth1 -- set interface eth1 type=internal
# ip link set eth1 address 00:00:00:00:00:11
# ovs-vsctl add-port br0 eth2 -- set interface eth2 type=internal
# ip link set eth2 address 00:00:00:00:00:12
```

2. 在host2上
```shell
# ovs-vsctl add-port br0 eth3 -- set interface eth3 type=internal
# ip link set eth3 address 00:00:00:00:00:13
# ovs-vsctl add-port br0 eth4 -- set interface eth4 type=internal
# ip link set eth4 address 00:00:00:00:00:14
```

##### 添加网络命名空间
1. 在host1上添加vm1和vm2两个网络命名空间
```shell
# ip netns add vm1
# ip netns add vm2
```

2. 在host2上添加vm3和vm4两个网络命名空间
```shell
# ip netns add vm3
# ip netns add vm4
```

##### 将端口添加到网络命名空间，并设置ip地址
1. 在host1上
```shell
# ip link set eth1 netns vm1
# ip netns exec vm1 ip link set eth1 up
# ip netns exec vm1 ip addr add 192.168.0.11/24 dev eth1
# ip link set eth2 netns vm2
# ip netns exec vm2 ip link set eth2 up
# ip netns exec vm2 ip addr add 192.168.0.12/24 dev eth2
```

2. 在host2上
```shell
# ip link set eth3 netns vm3
# ip netns exec vm3 ip link set eth3 up
# ip netns exec vm3 ip addr add 192.168.0.13/24 dev eth3
# ip link set eth4 netns vm4
# ip netns exec vm4 ip link set eth4 up
# ip netns exec vm4 ip addr add 192.168.0.14/24 dev eth4
```

##### 验证主机内部网桥各端口间的联通性
1. 在host1上
```shell
# ip netns exec vm1 ping -c 3 192.168.0.12
PING 192.168.0.12 (192.168.0.12) 56(84) bytes of data.
64 bytes from 192.168.0.12: icmp_seq=1 ttl=64 time=0.817 ms
64 bytes from 192.168.0.12: icmp_seq=2 ttl=64 time=0.063 ms
64 bytes from 192.168.0.12: icmp_seq=3 ttl=64 time=0.068 ms

--- 192.168.0.12 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2013ms
rtt min/avg/max/mdev = 0.063/0.316/0.817/0.354 ms
```

2. 在host2上
```shell
# ip netns exec vm3 ping -c 3 192.168.0.14
PING 192.168.0.14 (192.168.0.14) 56(84) bytes of data.
64 bytes from 192.168.0.14: icmp_seq=1 ttl=64 time=0.266 ms
64 bytes from 192.168.0.14: icmp_seq=2 ttl=64 time=0.061 ms
64 bytes from 192.168.0.14: icmp_seq=3 ttl=64 time=0.078 ms

--- 192.168.0.14 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2033ms
rtt min/avg/max/mdev = 0.061/0.135/0.266/0.092 ms
```

##### 配置vxlan隧道
1. 在host1上添加vxlan0端口
```shell
# ip link add vxlan0 type vxlan id 42 dstport 4789 remote 192.168.220.4 local 192.168.220.3 dev enp1s0
# ip addr add 20.0.0.1/24 dev vxlan0
# ip link set vxlan0 up
```

2. 在host2上添加vxlan0端口
```shell
# ip link add vxlan0 type vxlan id 42 dstport 4789 remote 192.168.220.3 local 192.168.220.4 dev enp1s0
# ip addr add 20.0.0.2/24 dev vxlan0
# ip link set vxlan0 up
```

##### 验证跨主机的vxlan隧道端口间是否联通
在host2上
```shell
# ping -c 20.0.0.1
PING 20.0.0.1 (20.0.0.1) 56(84) bytes of data.
64 bytes from 20.0.0.1: icmp_seq=1 ttl=64 time=0.534 ms
64 bytes from 20.0.0.1: icmp_seq=2 ttl=64 time=0.758 ms
64 bytes from 20.0.0.1: icmp_seq=3 ttl=64 time=0.696 ms

--- 20.0.0.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2029ms
rtt min/avg/max/mdev = 0.534/0.662/0.758/0.099 ms
```

##### 将vxlan0端口加入网桥
在host1和host2上分别执行
```shell
# ovs-vsctl add-port br0 vxlan0
```

##### 验证vxlan网络的联通性
1. 在host1上
```shell
# ip netns exec vm1 ping -c 192.168.0.13
PING 192.168.0.13 (192.168.0.13) 56(84) bytes of data.
64 bytes from 192.168.0.13: icmp_seq=1 ttl=64 time=1.17 ms
64 bytes from 192.168.0.13: icmp_seq=2 ttl=64 time=0.697 ms
64 bytes from 192.168.0.13: icmp_seq=3 ttl=64 time=0.693 ms

--- 192.168.0.13 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 0.693/0.853/1.171/0.226 ms
``` 

2. 在host2上
```shell
# ip netns exec vm4 ping -c 192.168.0.12
PING 192.168.0.12 (192.168.0.12) 56(84) bytes of data.
64 bytes from 192.168.0.12: icmp_seq=1 ttl=64 time=1.44 ms
64 bytes from 192.168.0.12: icmp_seq=2 ttl=64 time=0.798 ms
64 bytes from 192.168.0.12: icmp_seq=3 ttl=64 time=0.640 ms

--- 192.168.0.12 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 0.640/0.962/1.449/0.351 ms
```


### ovs实现
以下以ovs添加端口为例，采用veth类似

##### 创建网桥
1. 在host1和host2上分别创建网桥br0
```shell
# ovs-vsctl add-br br0 -- set Bridge br0 fail-mode=standalone
```

##### 添加端口并设置端口的mac地址
1. 在host1上
```shell
# ovs-vsctl add-port br0 eth1 -- set interface eth1 type=internal
# ip link set eth1 address 00:00:00:00:00:10
```

2. 在host2上
```shell
# ovs-vsctl add-port br0 eth1 -- set interface eth1 type=internal
# ip link set eth1 address 00:00:00:00:00:20
```

##### 添加网络命名空间
1. 在host1上添加vm1网络命名空间
```shell
# ip netns add vm1
```

2. 在host2上添加vm2网络命名空间
```shell
# ip netns add vm2
```

##### 将端口添加到网络命名空间，并设置ip地址
1. 在host1上
```shell
# ip link set eth1 netns vm1
# ip netns exec vm1 ip link set eth1 up
# ip netns exec vm1 ip addr add 192.168.0.10/24 dev eth1
```

2. 在host2上
```shell
# ip link set eth1 netns vm2
# ip netns exec vm2 ip link set eth1 up
# ip netns exec vm2 ip addr add 192.168.0.20/24 dev eth1
```

##### 设置vxlan
1. 在host1上
```shell
# ovs-vsctl add-port br0 vx1 -- set interface vx1 type=vxlan options:remote_ip=192.168.220.4
```

2. 在host2上
```shell
# ovs-vsctl add-port br0 vx1 -- set interface vx1 type=vxlan options:remote_ip=192.168.220.3
```

##### 测试联通性
在host1上
```shell
# ip netns exec vm1 ping -c 3 192.168.0.20
```
