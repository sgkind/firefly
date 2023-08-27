# vnc

## 查看vnc端口号
```
$ netstat -lp | grep -i vnc
```

## ubuntu18.04安装vnc解决灰屏问题

### 安装相关软件包
```
$ sudo apt-get update
$ sudo apt-get install xfce4 xfce4-goodies
$ sudo apt-get install tightvncserver
```

### 修改配置文件
```
# cat ~/.vnc/xstartup
#!/bin/sh
xrdb $HOME/.Xresources
unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS
startxfce4 &
```

### 启动vncserver
```
$ vncserver
```
或
```
$ vncserver :10
```


## 启动vnc
```
$ vncserver :99
```

其中：
99是端口号，请修改为自己要设置的端口号