ovn-sbctl命令
===
## 一般命令
### 显示数据库概要内容
```shell
ovn-sbctl show
```

## chassis命令
### 创建一个名为chassis with ENCAP-TYPE tunnels and ENCAP-IP
```shell
ovn-sbctl chassis-add CHASSIS ENCAP-TYPE ENCAP-IP
```

### 删除一个chassis及所有的encaps和gateway_ports
```shell
ovn-sbctl chassis-del CHASSIS
```

## 端口绑定命令
### 绑定逻辑端口PORT到CHASSIS
```shell
ovn-sbctl lsp-bind PORT CHASSIS
```

### 重置逻辑端口PORT的绑定
```shell
ovn-sbctl lsp-unbind PORT
```

## 逻辑流命令
### 列出DATAPATH的逻辑流
```shell
ovn-sbctl lflow-list [DATAPATH] [LFLOW...]
```

```shell
ovn-sbctl dump-flow [DATAPATH] [LFLOW...]
```

## 连接命令
### 打印连接
```shell
ovn-sbctl get-connection
```

### 删除连接
```shell
ovn-sbctl del-connection
```

### 设置连接到target
```shell
ovn-sbctl set-connection TARGET...
```

## SSL命令
### 打印SSL配置
```shell
ovn-sbctl get-ssl
```

### 删除SSL配置
```shell
ovn-sbctl del-ssl
```

### 配置SSL
```shell
ovn-sbctl set-ssl PRIV-KEY CERT CA-CERT [SSL-PROTOS [SSL-CIPHERS]]
```

## 数据库相关命令
**参见ovn-nbctl数据库相关命令**