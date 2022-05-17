
## ubuntu开启ssh服务
1.安装openssh服务端软件
```
sudo apt install openssh-server
```

2.开启ssh服务
```
sudo systemctl start ssh.service
```

3.查看服务是否开启
```
ps aux | grep ssh
```