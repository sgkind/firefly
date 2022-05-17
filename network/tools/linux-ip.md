linux网络管理工具ip
===

## 简介

ip是linux中查看和修改路由、网络设备、网卡接口和隧道的工具。

### ip语法
```
  ip [ OPTIONS ] OBJECT { COMMAND | help }
  ip [ --force ] -batch filename

  OBJECT := { link | address | addrlabel | route | rule | neigh | ntable | tunnel | tuntap | maddress | mroute | mrule | monitor | xfrm | netns | l2tp | tcp_metrics | token | macsec }

  OPTIONS := { -V[ersion] | -h[uman-readable] | -s[tatistics] | -d[etails] | -r[es‐olve] | -iec | -f[amily] { inet | inet6 | ipx | dnet | link } | -4 | -6 | -I | -D | -B | -0 | -l[oops] { maximum-addr-flush-attempts } | -o[neline] | -rc[vbuf] [size] | -t[imestamp] | -ts[hort] | -n[etns] name | -a[ll] | -c[olor] | -br[ief] | -j[son] | -p[retty] }
```

### ip选项

#### -V, -Version
  打印ip工具的版本号

#### -h, -human, -human-readable
  以添加后缀的人易读的方式输出数据

#### -b, -batch <FILENAME>
  从文件或标准输入中读取命令并执行，遇到错误时终止执行ip命令

#### -force
  在batch模式下遇到错误也不终止执行。在执行过程中有错误发生，程序将返回非0值

#### -s, -stats, -statistics
  输出更多的信息。如果此选项出现两次或更多次，则信息数量增加。

#### -d, -details
  输出更多详细的信息

#### -l, -loops <COUNT>
  Specify maximum number of loops the 'ip address flush' logic will attemp before giving up. The default is 10. Zero (0) means loop until all addresses are removed.

#### -f, -family <FAMILY>
  指定使用的协议族。协议族可以是下面中的一个[inet, inet6, bridge, ipx, dnet, mpls, link]。如果此选项不存在，则协议族从其他选项中猜测。如果其余的参数不能确定使用的协议族，则ip使用默认的（inet或any）。link是一个特殊的协议族，表示不使用网络协议。

#### -4
  -family inet的缩写

#### -6
  -family inet6的缩写

#### -B
  -family bridge的缩写

#### -D
  -family decnet的缩写

#### -I
  -family ipx的缩写

#### -M
  -family mpls的缩写

#### -0
  -family link的缩写

#### -o, -oneline
  一条记录输出一行，换行符用'\'代替

#### -r, -resolve
  使用系统的name resolver打印DNS名而不是主机地址

#### -n, -netns
  在网络命名空间NETNS中执行ip，下面两种语法是等价的
```
  ip netns exec NETNS ip [ OPTIONS ] OBJECT { COMMAND | help }
  ip -n[etns] NETNS [ OPTIONS ] OBJECT { COMMAND | help }
```

#### -a, -all
  在所有对象上执行给定的命令

#### -c[olor][={always|auto|never}]
  配置颜色输出。
  如果不带参数或参数为always，则输出带颜色

#### -t, -timestamp
  使用监控选项时显示当前时间

#### -ts, -tshort
  同-timestamp，使用短格式

#### -rc, -rcvbuf <SIZE>
  设置netlink套接字接收缓冲区大小，默认是1MB

#### -iec
  打印人易读的速率

#### -br, -brief
  以表格形式打印基本的信息，方便阅读。目前此选项仅仅支持ip addr show和ip link show命令

#### -j， -json
  以json格式输出信息

#### -p, -pretty
  默认的JSON格式是压缩的，易于解析但很难阅读。此选项增加json中的缩进，方便阅读

### 对象

#### address
  设备的协议地址(IP或IPv6)

#### addrlabel
  label configuration for protocol address selection
  协议地址选择的标签配置

#### l2tp
  tunnel ethernet over IP (L2TPv3)

#### link
  网络设备

#### maddress
  多播地址

#### monitor
  监控netlink消息

#### mroute
  多播路由缓存项

#### mrule
  rule in multicast routing policy database
  多播路由策略数据库中的规则

#### neighbour
  管理ARP或NDISC缓存项

#### netns
  管理网络命名空间

#### ntable
  manage the neighbor cache's operation

#### route
  路由表项

#### rule
  rule in routing policy database
  路由策略数据库中的规则

#### tcp_metrics/tcpmetrics
  管理TCP测量

#### token
  manage tokenized interface identifiers

#### tunnel
  tunnel over IP

#### tuntap
  管理TUN/TAP设备

#### xfrm
  管理IPSec策略

## 常用的命令

### address
可简写为a或addr，查看帮助`ip a/addr/address help`

#### 查看IP地址
* 查看所有设备的IP地址
```
$ ip a/addr/address
$ ip a/addr/address sh/show
```
* 指定设备的IP地址
```
$ ip a/addr/address sh/show dev eth1
$ ip a/addr/address sh/show eth1
```

#### 添加IP地址
```
$ sudo ip a/addr/address add 192.168.1.2/24 dev eth1
```

注:
在ubuntu上`ip a/addr/address [change/replace]`都会给指定设备新添加一个ip地址

#### 删除IP地址
```
$ sudo ip a/addr/address del 192.168.1.2/24 dev eth1
```

#### 删除所有IP地址
```
$ sudo ip a/addr/address flush [ dev eth1 ]
```

### route

查看帮助信息`ip route help`

#### 路由表
所谓路由表，指的是路由器或者其他互联网网络设备上存储的表，该表中存有到达特定网络终端的路径，在某些情况下，还有一些与这些路径相关的度量。路由器的主要工作就是经过路由器的每个数据包寻找一条最佳的传输路径，并将该数据有效地传送到目的站点。由此可见，选择最佳路径的策略即路由算法是路由器的关键所在。为了完成这项工作，在路由器中保存着各种传输路径的相关数据--路由表。

在linux系统中，可以自定义从1-252个路由表，其中，linux系统维护了4个路由表：
* 0号表：系统保留表
* 253号表： default 没有特别指定的**默认**路由都放在该表
* 254号表： main 没有指明路由表的所有路由都放在该表
* 255号表： local 保存本地接口地址、广播地址和NAT地址，由系统维护，用户不得更改

#### 查询路由表
```
$ ip route list table table_number
$ ip route list table table_name
```

如果省略了table [ table_number | table_name ]，则默认查询main表

#### 添加路由表
* 添加默认路由
```
$ sudo ip route add default via 192.168.1.1 table 1
```

* 添加一般路由
```
$ sudo ip route add 192.168.0.0/24 via 192.168.1.2 table 1
```

#### 删除一条路由规则
```
$ sudo ip route del 192.168.0.0/24
```
  