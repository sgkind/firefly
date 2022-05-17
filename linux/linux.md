## linux

#### 一、

1. linux系统下改变windows换行符（^M):

   vim打开文件，在命令模式下输入`:w ++ff=unix`即可

2. msys使用全路径：`msys2_shell.cmd -use-full-path`

#### 二、linux命令

1. `telnet ipaddr port` 模拟tcp客户端连接

#### 三、linux挂载NTFS文件系统只读
1. `sudo ntfsfix /dev/sda3`

### 查看内核版本
```
$ cat /proc/version
```
