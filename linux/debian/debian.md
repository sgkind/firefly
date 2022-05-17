debian系统
===

## 设置
### 关闭笔记本盖子不休眠
1. 修改/etc/systemd/logind.conf文件
将下面的参数改为ignore
HandleLidSwitch=ignore
2. 重启服务
```
$ sudo service systemd-logind restart
```
参考: https://www.cnblogs.com/yanglai/p/6874496.html

## 命令
### 设置相关
1. 窗口管理器: xfwm4-settings
2. 设置:       xfce4-settings-manager

3. 命令行调节声音

查看设备
```shell
$ amixer
```
调节音量
```shell
$ amixer set Master 30%
```

## 主题
### 窗口主题
推荐
* [Arc-Flatabulous-Dark](https://github.com/andreisergiu98/arc-flatabulous-theme/)

尝试过
* Arc

不推荐
* Paper


### 图表主题
推荐
* Papirus-Dark

其他
1. [paper](https://github.com/snwh/paper-icon-theme)
2. elementary
3. [moka](https://github.com/snwh/moka-icon-theme)

## 问题
### 一、命令行出现如下错误
```
Gtk-Message: Failed to load module "atk-bridge"
```
解决办法：
```
$ sudo apt install libatk-adaptor
```

```
Gtk-Message: Failed to load module "gail"
```
解决办法：
```
$ sudo apt install libatk-adaptor
```

### 
