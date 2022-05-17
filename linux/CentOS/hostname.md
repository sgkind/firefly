hostname
===

## CentOS 8
与CentOS 7相同

## CentOS 7
centos7里面修改hostname的方式有所改变，修改/etc/hosts和/etc/sysconfig/network两个文件不能生效。使用新的命令是：

```shell
$ sudo hostnamectl set-hostname centos7
```

查看修改结果:
```shell
$ sudo hostname
```

## CentOS 7之前版本
```shell
$ sudo vi /etc/sysconfig/network
# Created by anaconda
NETWORKING=yes
HOSTNAME=centos6
```