debian10安装
===

## 无线网络驱动

### debian 9
```shell
sudo apt-get update
sudo apt-get install firmware-iwlwifi
modprobe -r iwlwifi && modprobe iwlwifi
```

### debian10
```shell
sudo apt-get update
sudo apt-get install firmware-iwlwifi
sudo reboot
```

## 用户不在sudoers文件中
1. su
2. chmod 740 /etc/sudoers
3. vi /etc/sudoers
4. 找到# Allow members of group sudo to execute any command

               %sudo    ALL=(ALL) ALL

      在下面添加一行，如下

                     xx       ALL=(ALL) ALL  (将此处的XX修改为出现改问题的用户名！）
