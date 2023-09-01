ovn-nbctl命令
===

## 一般命令
### 初始化数据库
```shell
ovn-nbctl init
```

### 打印数据库概要
```shell
ovn-nbctl show
switch 59ca6223-430c-4437-8470-f4cc007bf04d (inside)
    port inside-vm2
        addresses: ["02:ac:10:ff:01:31 10.0.0.2"]
    port inside-router1
        type: router
        addresses: ["02:ac:10:ff:00:01"]
        router-port: router1-inside
    port inside-vm1
        addresses: ["02:ac:10:ff:01:30 10.0.0.1"]
router 22b9b97f-23c1-4e85-a60e-eb568be7b33d (router1)
    port router1-inside
        mac: "02:ac:10:ff:00:01"
        networks: ["10.0.0.100/24"]
```

### 打印数据库中某交换机的概要
```shell
ovn-nbctl show SWITCH
```

### 打印数据库中某路由器的概要
```shell
ovn-nbctl show ROUTER
```

## 逻辑交换机命令
### 创建一个名为SWITCH的逻辑交换机
```shell
ovn-nbctl ls-add [SWITCH]
```

### 删除一个交换机和它所有的端口
```shell
ovn-nbctl ls-del SWITCH
```

### 列出所有的逻辑交换机的名字
```shell
ovn-nbctl ls-list
```

## 逻辑交换机端口命令
### 在交换机SWITCH上增加一个逻辑端口
```shell
ovn-nbctl lsp-add SWITCH PORT
```

### 在交换机SWITCH上增加一个逻辑端口with PARENT on TAG
```shell
ovn-nbctl lsp-add SWITCH PORT PARENT TAG
```
### 从交换机上删除一个端口
```shell
ovn-nbctl lsp-del PORT
```

### 打印交换机SWITCH上所有的逻辑端口
```shell
ovn-nbctl lsp-list SWITCH
```

### 查看端口的parent，如果没设置返回空
```shell
ovn-nbctl lsp-get-parent PORT
```

### 查看端口的tag，如果没设置返回空
```shell
ovn-nbctl lsp-get-tag PORT
```

### 设置端口的MAC地址或MAC+IP
```shell
ovn-nbctl lsp-set-addresses PORT [ADDRESS]...
```

### 查看端口的MAC或MAC+IP的列表
```shell
ovn-nbctl lsp-get-addresses PORT
```

### 设置端口的安全地址
```shell
ovn-nbctl lsp-set-port-security PORT [ADDRS]...
```

### 查看端口的安全地址
```shell
ovn-nbctl lsp-get-port-security PORT
```

### 查看端口的状态(up or down)
```shell
ovn-nbctl lsp-get-up PORT
```

### set administrative state PORT('enabled' or 'disabled')
```shell
ovn-nbctl lsp-set-enabled PORT STATE
```

### get administrative state PORT('enabled' or 'disabled')
```shell
ovn-nbctl lsp-get-enabled PORT
```

### 设置端口类型
```shell
ovn-nbctl lsp-set-type PORT TYPE
```

目前为止见过的类型有：
1. router
2. vtep

### 查看端口类型
```shell
ovn-nbctl lsp-get-type PORT
```

### 设置与端口类型相关的可选项
```shell
ovn-nbctl lsp-set-options PORT KEY=VALUE [KEY=VALUE]...
```

### 查看端口已分配的可选项
```shell
ovn-nbctl lsp-get-options PORT
```

### 设置端口的dhcpv4可选项
```shell
ovn-nbctl lsp-set-dhcpv4-options PORT [DHCP_OPTIONS_UUID]
```

### 查看端口的dhcpv4可选项
```shell
ovn-nbctl lsp-get-dhcpv4-options PORT
```

### 设置端口的dhcpv6可选项
```shell
ovn-nbctl lsp-set-dhcpv6-options PORT [DHCP_OPTIONS_UUID]
```

### 查看端口的dhcpv6可选项
```shell
ovn-nbctl lsp-get-dhcpv6-options PORT
```

## 逻辑路由器命令
### 创建名为ROUTER的逻辑路由器
```shell
ovn-nbctl lr-add [ROUTER]
```

### 删除路由器和它所有的端口
```shell
ovn-nbctl lr-del ROUTER
```

### 打印所有的路由器的名字
```shell
ovn-nbctl lr-list
```

## 逻辑路由器端口命令
### 在路由器上添加端口
```shell
ovn-nbctl lrp-add ROUTER PORT MAC NETWORK... [peer=PEER]
```

### 设置端口PORT的网关chassis
```shell
ovn-nbctl lrp-set-gateway-chassis PORT CHASSIS [PRIORITY]
```

### 从端口PORT上删除网关chassis
```shell
ovn-nbctl lrp-del-gateway-chassis PORT CHASSIS
```

### print the names of all gateway chassis on PORT with PRIORITY
```shell
ovn-nbctl lrp-get-gateway-chassis PORT
```

### 从路由器上删除端口
```shell
ovn-nbctl lrp-del PORT
```

### 列出路由器上的所有端口
```shell
ovn-nbctl lrp-list ROUTER
```

### set administrative state PORT('enabled' or 'disabled')
```shell
ovn-nbctl lrp-set-enabled PORT STATE
```

### get administrative state PORT('enabled' or 'disabled')
```shell
ovn-nbctl lrp-get-enabled PORT
```

## ACL 命令
[--log] [--severity=SEVERITY] [--name=NAME] [--may-exist]
### 向交换机添加一个ACL
```shell
ovn-nbctl acl-add SWITCH DIRECTION PRIORITY MATCH ACTION
```

### 从交换机删除ACLs
```shell
ovn-nbctl acl-del SWITCH [DIRECTION [PRIORITY MATCH]]
```

### 列出交换机的所有ACLs
```shell
ovn-nbctl acl-list SWITCH
```

## 路由命令
### 向路由器添加一条路由
```shell
ovn-nbctl [--policy=POLICY] lr-route-add ROUTER PREFIX NEXTHOP [PORT]
```

举例：
```shell
ovn-nbctl lr-route-add gw1 "192.168.100.0/24" 192.168.255.2
```

### 从路由器删除路由
```shell
ovn-nbctl lr-route-del ROUTER [PREFIX]
```

### 打印路由器的路由
```shell
ovn-nbctl lr-route-list ROUTER
```

## NAT命令
### 向路由器添加一条NAT
```shell
ovn-nbctl lr-nat-add ROUTER TYPE EXTERNAL_IP LOGICAL_IP [LOGICAL_PORT EXTERNAL_MAC]
```

### 删除路由器的NATs
```shell
ovn-nbctl lr-nat-del ROUTER [TYPE [IP]]
```

### 打印路由器的NATs
```shell
ovn-nbctl lr-nat-list ROUTER
```

## LB(load-balancer)命令

### 创建一个负载均衡或添加VIP到已经存在的负载均衡
```shell
ovn-nbctl lb-add LB VIP[:PORT] IP[:PORT]... [PROTOCOL]
```

### 删除一个负载均衡或仅删除VIP
```shell
ovn-nbctl lb-del LB [VIP]
```

### 列出负载均衡
```shell
ovn-nbctl lb-list [LB]
```

### 向路由器添加一个负载均衡
```shell
ovn-nbctl lr-lb-add ROUTER LB
```

### 从路由器删除负载均衡
```shell
ovn-nbctl lr-lb-del ROUTER [LB]
```

### 列出路由器上的所有负载均衡
```shell
ovn-nbctl lr-lb-list ROUTER
```

### 向交换机添加一个负载均衡
```shell
ovn-nbctl ls-lb-add SWITCH LB
```

### 从交换机删除一个负载均衡
```shell
ovn-nbctl ls-lb-del SWITCH [LB]
```

### 列出交换机上的所有负载均衡
```shell
ovn-nbctl ls-lb-list SWITCH
```

## DHCP相关命令
### create a DHCP options row with CIDR
```shell
ovn-nbctl dhcp-options-create CIDR [EXTERNAL_IDS]
```

### 删除一个DHCP options
```shell
ovn-nbctl dhcp-options-del DHCP_OPTIONS_UUID
```

### 列出所有DHCP
```shell
ovn-nbctl dhcp-options-list
```

### set DHCP options for DHCP_OPTIONS_UUID
```shell
ovn-nbctl dhcp-options-set-options DHCP_OPTIONS_UUID  KEY=VALUE [KEY=VALUE]...
```

### displays the DHCP options for DHCP_OPTIONS_UUID
```shell
ovn-nbctl dhcp-options-get-options DHCO_OPTIONS_UUID
```

## 连接命令
### 打印连接
```shell
ovn-nbctl get-connection
```

### 删除连接
```shell
ovn-nbctl del-connection
```

### 设置连接
```shell
ovn-nbctl set-connection TARGET...
```

## SSL命令
### 打印SSL配置
```shell
ovn-nbctl get-ssl
```

### 删除ssl配置
```shell
ovn-nbctl del-ssl
```

### 设置ssl配置
```shell
set-ssl PRIV-KEY CERT CA-CERT [SSL-PROTOS [SSL-CIPHERS]]
```

## 数据库相关命令
### 列出table中的REC记录或所有记录
```shell
ovn-nbctl list TBL [REC]
```

### 列出table中满足给定条件的记录
```shell
ovn-nbctl find TBL CONDITION...
```

### print values of COLumns in RECord in TBL
```shell
ovn-nbctl get TBL REC COL[:KEY]
```

### set COLumn values in RECord in TBL
```shell
ovn-nbctl set TBL REC COL[:KEY]=VALUE
```

### add (KEY=)VALUE to COLumn in RECord in TBL
```shell
ovn-nbctl add TBL REC COL [KEY=]VALUE
```

### remove (KEY=)VALUE from COLumn
```shell
ovn-nbctl remove TBL REC COL [KEY=]VALUE
```

### clear values from COLumn in RECord in TBL
```shell
ovn-nbctl clear TBL REC COL
```

### create and initialize new record
```shell
ovn-nbctl create TBL COL[:KEY]=VALUE
```

### delete RECord from TBL
```shell
destroy TBL REC
```

### wait until condition is true
```shell
ovn-nbctl wait-until TBL REC [COL[:KEY]=VALUE]
```

