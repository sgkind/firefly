iptables
===

### iptables提供的表
* filter表：负责过滤功能，防火墙； 对应的内核模块：iptables_filter
* nat：network address translation,网络地址转换。内核模块：iptable_nat
* mangle表：拆解报文，做出修改，并重新封装的功能。内核模块：iptable_mangle
* raw表：关闭nat表上启用的连接追踪机制。内核模块：iptable_raw


### 规则
#### 匹配条件
匹配条件分为基本匹配条件和扩展匹配条件
* 基本匹配条件: 
    
  源地址Source IP，目的地址Destination IP
* 扩展匹配条件:

  除了上面的条件外，其他可以匹配的条件泛称为扩展条件，这些扩展条件其实也是netfilter中的一部分，只是已模块的形式存在，如果想要使用这些条件，则需要依赖对应的扩展模块。如源端口Source Port，目标端口Destination Port
#### 处理动作
处理动作在iptables中被称为target，动作也可以分为基本动作和扩展动作
* ACCEPT：允许数据包通过
* DROP: 直接丢弃数据包，不给任何回应
* REJECT：拒绝数据包通过，必要时会给数据发送端一个响应信息
* SNAT：源地址转换，解决内网用户用同一公网地址上网的问题
* MASQUERADE： SNAT的一种特殊形式，适用于动态的、临时会变的ip上
* DNAT：目标地址转换
* REDIRECT：在本机做端口映射
* LOG：在/var/log/message文件中记录日志信息，然后将数据包传递给下一条规则，也就是说处理记录以外不对数据包做任何其他操作，仍然让下一条规则去匹配
3. 


### 命令
#### 查看iptables规则
```shell
# iptables -t 表名 -L
```
查看对应表的规则，-t选项制定要操作的表，省略“-t 表名”时，默认表示操作filter表，-L表示列出规则，其中表名为raw、mangle、nat、filter
```shell
# iptables -t 表名 -L 链名
```
查看指定表的指定链中的规则
```shell
# iptables -t 表名 -v -L
```
查看指定表的所有规则，并且显示更详细的信息。-v表示verbose，表示详细的，冗长的
```shell
# iptables --line-numbers -t 表名 -L
```
查看标的所有规则，并显示规则的序号
```shell
# iptables -t 表名 -v -x -L
```
计数器中的信息显示为精确的计数值，而不是显示为经过可读优化的计数值，-x选项表示显示计数器的精确值

#### 追加规则
-A或--append会把要添加的规则放在最后一行
```shell
# iptables -A chain rule-specification [options]
```

#### 检查给定的规则是否存在
-C或--check
```shell
# iptables -C chain rule-specification
```

#### 删除规则
-D或--delete
* 删除匹配的规则
```shell
# iptables -D chain rule-specification
```
* 删除指定序号的规则，从1开始计数
```shell
# iptables -D chain rulenum
```

#### 插入规则
-I或--insert会把要添加的规则放在第一行或者在指定的rulenum处
```shell
# iptables -I chain [rulenum] rule-specification [options]
```

#### 替换规则
-R或--replace
```shell
# iptables -R chain rulenum rule-specification
```

#### 列出规则
-L或--list
```shell
# iptables -L [chain [rulenum]] [-t table] [options]
```
举例：
```shell
# iptables -L FORWARD -t filter
Chain FORWARD (policy DROP)
target     prot opt source               destination         
DOCKER-USER  all  --  anywhere             anywhere            
DOCKER-ISOLATION-STAGE-1  all  --  anywhere             anywhere            
ACCEPT     all  --  anywhere             anywhere             ctstate RELATED,ESTABLISHED
DOCKER     all  --  anywhere             anywhere            
ACCEPT     all  --  anywhere             anywhere            
ACCEPT     all  --  anywhere             anywhere  
```

#### 打印规则
-S或--list-rules
```shell
# iptables -S [chain [rulenum]] [-t table]
```

打印的是iptables添加相应规则时的命令

举例：
```shell
# iptables -S FORWARD -t filter
-P FORWARD DROP
-A FORWARD -j DOCKER-USER
-A FORWARD -j DOCKER-ISOLATION-STAGE-1
-A FORWARD -o docker0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -o docker0 -j DOCKER
-A FORWARD -i docker0 ! -o docker0 -j ACCEPT
-A FORWARD -i docker0 -o docker0 -j ACCEPT
```

#### 删除所有规则
-F或--flush
```shell
# iptables -F [chain]
```

#### 重置计数器
-Z或--zero
```shell
# iptables -Z [chain [rulenum]]
```

> Zero counters in chain or all chains

#### 创建用户自定义规则链
-N或--new
```shell
# iptables -N chain [-t table]
```

#### 删除用户自定义规则链
-X或--delete-chain
```shell
# iptables -X [chain] [-t table]
```

#### 设置链的策略(即默认操作)
-P或--policy
```shell
# iptables -P chain target
```

#### 重命名链
-E或--rename-chain
```shell
# iptables -E old-chain new-chain
```

> Change chain name, (moving any references)

### options

取反|长命令|短命令|参数|解释|
---|---|---|---|---
|  |--ipv4|-4 ||Nothing (line is ignored by ip6tables-restore)
||--ipv6|-6|| Error (line is ignored by iptables-restore)
[!]|--protocol|-p|proto|协议代号或协议名
[!]|--source|-s|address[/mask][...]|源ip地址
[!]|--destination|-d|address[/mask][...]|目的IP地址
[!]|--in-interface|-i|input name[+]|network interface name ([+] for wildcard)
||--jump|-j|target |target for rule (may load target extension)
||--goto | -g | chain | jump to chain with no return
|| --match | -m | match | extended match (may load extension)
||--numeric|-n| | numeric output of addresses and ports
[!]|--out-interface| -o | output name[+]|network interface name ([+] for wildcard)
||--table|-t|table|指定table（默认为filter）
||--verbose|-v||verbose mode
||--wait|-w|[seconds]|maximum wait to acquire xtables lock before give up, default is 1 second
||--line-numbers|||print line numbers when listing
||--exact|-x||expand numbers (display exact values)
[!]|--fragment|-f||match second or further fragments only
||--modprobe=<command>|||try to insert modules using this command
||--set-counters PKTS BYTES|||set the counter during insert/append
[!]|--version|-V||打印版本号

参考：
1. http://www.zsythink.net/archives/1199/