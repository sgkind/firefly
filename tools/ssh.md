ssh
===

## ubuntu

### 安装openssh服务端软件
```
sudo apt install openssh-server
```

### 开启ssh服务
```
sudo systemctl start ssh.service
```

### 查看服务是否开启
```
ps aux | grep ssh
```

### 允许root登录
```shell
$cat /etc/ssh/sshd_config
...
#LoginGraceTime 2m
#PermitRootLogin prohibit-passwords
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10
...
```
更改为
```shell
LoginGraceTime 2m
#PermitRootLogin prohibit-password
PermitRootLogin yes
StrictModes yes
#MaxAuthTries 6
#MaxSessions 10
```
重启服务
```shell
$sudo systemctl restart ssh.service
```

## debian
同ubuntu