qcow2镜像修改密码
===

以下针对ubuntu和debian的云镜像

## 修改镜像大小
```
qemu-img resize debian-10-generic-amd64.qcow2 +98G
```

## 修改密码
安装软件
```
$ sudo apt install libguestfs-tools
```

修改密码
```
$ virt-customize -a debian-10-generic-amd64.qcow2 --root-password password:root
```

参考:
1. https://leux.cn/doc/Debian%E5%AE%98%E6%96%B9qcow2%E9%95%9C%E5%83%8F%E4%BF%AE%E6%94%B9root%E5%AF%86%E7%A0%81.html