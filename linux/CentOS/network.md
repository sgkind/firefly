网络设置
===

## 设置静态ip
/etc/sysconfig/network-scripts

```
# cat /etc/sysconfig/network-scripts/ifcfg-ens3
TYPE=Ethernet
BOOTPROTO=static
DEFROUTE=yes
NAME=ens3
IPADDR=192.168.220.39
PREFIX=24
GATEWAY=192.168.220.1
UUID=6b631d1a-1081-4d34-9d1c-12eb3992b44f
DEVICE=ens3
ONBOOT=yes
PROXY_METHOD=none
BROWSER_ONLY=no
DNS1=8.8.8.8
IPV4_FAILURE_FATAL=no
IPV6INIT=no
# nmcli c reload ens3
```

* 将DEFROUTE设置为yes并且设置GATEWAY后可以自动添加default route

## 添加dns
/etc/resolv.conf
```
# cat /etc/resolv.conf
nameserver 8.8.8.8
nameserver 8.8.4.4
```

## 物理网卡添加vlan
* 物理网卡
```
# cat ens5

```

* vlan
```
# cat ens5.200
PHYSDEV=ens5
VLAN=yes
TYPE=Vlan
VLAN_ID=200
BOOTPROTO=none
IPADDR=192.168.110.4
PREFIX=24
DEFROUTE=yes
NAME=ens5.200
DEVICE=ens5.200
ONBOOT=yes
```

## CentOS 8修改后使生效
 nmcli命令
### 加载配置文件
```
nmcli c reload [ens3]
```

### 关闭interface
```
nmcli c down ens3
```

### 启动interface
```
nmcli c up ens3
```

## CentOS 7修改后使生效
```
systemctl restart network
```

