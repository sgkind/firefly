linux命令
===

## 进程
### pgrep
查找进程
```
$ pgrep -u sgk  # 列出用户所用的进程

$ pgrep code # 列出程序的进程
```

### pstree
以树形方式列出系统的所有进程
```
$ pstree
systemd─┬─WeChatWeb.exe───11*[{WeChatWeb.exe}]
        ├─agetty
        ├─bluetoothd
        ├─crashpad_handle───2*[{crashpad_handle}]
        ├─crashpad_handle───{crashpad_handle}
        ├─cron
        ├─2*[dbus-daemon]
        ├─deepin-wine───WeChat.exe───47*[{WeChat.exe}]
        ├─dhclient
        ├─explorer.exe───3*[{explorer.exe}]
        ├─fcitx
        ├─fcitx-dbus-watc
        ├─haveged
        ├─lightdm─┬─Xorg───3*[{Xorg}]
        │         ├─lightdm─┬─xfce4-session─┬─Thunar───2*[{Thunar}]
        │         │         │               ├─blueman-applet───3*[{blueman-applet}]
        │         │         │               ├─light-locker───3*[{light-locker}]
        │         │         │               ├─plank───3*[{plank}]
        │         │         │               ├─polkit-gnome-au───2*[{polkit-gnome-au}]
        │         │         │               ├─ssh-agent
        │         │         │               ├─xfce4-panel─┬─code─┬─code───code───6*[{code+
        │         │         │               │             │      ├─code
        │         │         │               │             │      ├─code───5*[{code}]
        │         │         │               │             │      ├─code───19*[{code}]
        │         │         │               │             │      ├─code─┬─code─┬─code───7+
        │         │         │               │             │      │      │      ├─cpptools+++
        │         │         │               │             │      │      │      └─18*[{cod+
        │         │         │               │             │      │      ├─code───11*[{cod+
        │         │         │               │             │      │      └─20*[{code}]
        │         │         │               │             │      ├─code─┬─code───17*[{cod+
        │         │         │               │             │      │      ├─2*[code───11*[{+
        │         │         │               │             │      │      └─21*[{code}]
        │         │         │               │             │      └─30*[{code}]
        │         │         │               │             ├─msedge─┬─2*[cat]
        │         │         │               │             │        ├─msedge───msedge───7*+
        │         │         │               │             │        ├─msedge───8*[{msedge}+
        │         │         │               │             │        ├─msedge-sandbox───mse+
        │         │         │               │             │        └─79*[{msedge}]
        │         │         │               │             ├─panel-2-actions───2*[{panel-2+
        │         │         │               │             ├─panel-6-systray───2*[{panel-6+
        │         │         │               │             └─2*[{xfce4-panel}]
        │         │         │               ├─xfce4-power-man───2*[{xfce4-power-man}]
        │         │         │               ├─xfdesktop───2*[{xfdesktop}]
        │         │         │               ├─xfsettingsd───2*[{xfsettingsd}]
        │         │         │               ├─xfwm4───3*[{xfwm4}]
        │         │         │               └─2*[{xfce4-session}]
        │         │         └─2*[{lightdm}]
        │         └─2*[{lightdm}]
        ├─plugplay.exe───2*[{plugplay.exe}]
        ├─polkitd───2*[{polkitd}]
        ├─rsyslogd───3*[{rsyslogd}]
        ├─rtkit-daemon───2*[{rtkit-daemon}]
        ├─services.exe───6*[{services.exe}]
        ├─sshd
        ├─systemd─┬─(sd-pam)
        │         ├─at-spi-bus-laun─┬─dbus-daemon
        │         │                 └─3*[{at-spi-bus-laun}]
        │         ├─at-spi2-registr───2*[{at-spi2-registr}]
        │         ├─bamfdaemon───2*[{bamfdaemon}]
        │         ├─dbus-daemon
        │         ├─dconf-service───2*[{dconf-service}]
        │         ├─gnome-terminal-─┬─zsh───pstree
        │         │                 └─3*[{gnome-terminal-}]
        │         ├─gpg-agent
        │         ├─gvfs-udisks2-vo───2*[{gvfs-udisks2-vo}]
        │         ├─gvfsd─┬─gvfsd-trash───2*[{gvfsd-trash}]
        │         │       └─2*[{gvfsd}]
        │         ├─gvfsd-metadata───2*[{gvfsd-metadata}]
        │         ├─obexd
        │         ├─pulseaudio───2*[{pulseaudio}]
        │         ├─xfce4-notifyd───2*[{xfce4-notifyd}]
        │         └─xfconfd───2*[{xfconfd}]
        ├─systemd-journal
        ├─systemd-logind
        ├─systemd-timesyn───{systemd-timesyn}
        ├─systemd-udevd
        ├─udisksd───4*[{udisksd}]
        ├─upowerd───2*[{upowerd}]
        ├─2*[winedevice.exe───3*[{winedevice.exe}]]
        ├─wineserver.real
        └─wpa_supplicant
```

## 文件
### lsof
可以列出打开了的文件
```
$ lsof | grep TCP
wineserve 1165                             sgk  294u     sock                0,9       0t0      25845 protocol: TCP
wineserve 1165                             sgk  422u     sock                0,9       0t0     167529 protocol: TCP
code      1584                             sgk   33u     sock                0,9       0t0     183127 protocol: TCP
code      1584 1585 ThreadPoo              sgk   33u     sock                0,9       0t0     183127 protocol: TCP
...
```

### split
切割文件
```
$ ls -lh 计算机组成与设计.硬件软件接口-v5.pdf 
-rw-r--r-- 1 sgk sgk 150M 11月 17 09:44 计算机组成与设计.硬件软件接口-v5.pdf
$ split -b 50m 计算机组成与设计.硬件软件接口-v5.pdf computer_
$ ls -lh
总用量 367M
-rwxr-xr-x 1 sgk sgk  68M 11月 17 09:44 '操作系统精髓与设计原理(原书第6版).pdf'
-rw-r--r-- 1 sgk sgk 150M 11月 17 09:44  计算机组成与设计.硬件软件接口-v5.pdf
-rw-r--r-- 1 sgk sgk  50M 11月 27 13:54  computer_aa
-rw-r--r-- 1 sgk sgk  50M 11月 27 13:54  computer_ab
-rw-r--r-- 1 sgk sgk  50M 11月 27 13:54  computer_ac
```

合并被切割的文件
```
$ cat computer_a*>computer.pdf
$ ls -lh
总用量 516M
-rwxr-xr-x 1 sgk sgk  68M 11月 17 09:44 '操作系统精髓与设计原理(原书第6版).pdf'
-rw-r--r-- 1 sgk sgk 150M 11月 17 09:44  计算机组成与设计.硬件软件接口-v5.pdf
-rw-r--r-- 1 sgk sgk  50M 11月 27 13:54  computer_aa
-rw-r--r-- 1 sgk sgk  50M 11月 27 13:54  computer_ab
-rw-r--r-- 1 sgk sgk  50M 11月 27 13:54  computer_ac
-rw-r--r-- 1 sgk sgk 150M 11月 27 13:55  computer.pdf
```

### nl
显示文件并打上行号

### col
可以把man文件转换成纯文本文件

1.df 查看磁盘空间

2.dd命令快速生成指定大小的文件

```shell
dd if=/dev/zero of=test bs=1M count=1000
```

3.dd命令生成超大文件，但是实际不占用空间

```shell
dd if=/dev/zero of=test bs=1M count=0 seek=10000
```

4.随机生成一百万个1K的文件

```shell
seq 1000000 | xargs -i dd if=/dev/zero of={}.dat bs=1024 count=1
```