# tcpdump使用方法

tcpdump选项可以分为四大类：控制tcpdump程序行为；控制数据怎么显示；控制显示什么数据；过滤命令

## 控制程序行为
### 控制捕获报文的数量
-c 
```shell
$ sudo tcpdump -c 10
```

### 指定接口
-a 
```shell
$ sudo tcpdump -i enp2s0
```