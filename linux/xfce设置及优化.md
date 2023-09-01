xfce设置及优化.md
===

## xfce4编译
### 安装依赖包
#+BEGIN_SRC
sudo apt install libgudev-1.0-dev
sudo apt install libwnck-3-dev
sudo apt install libdbusmenu-gtk3-dev
sudo apt install libnotify-dev
sudo apt install libupower-glib-dev
#+END_SRC

### 编译命令
#+BEGIN_SRC
./configure --prefix=/usr/local && make && sudo make install
#+END_SRC

### 编译顺序
* xfce4-dev-tools (开发工具)
* libxfce4util
* xfconf
* libxfce4ui
* garcon exo
* thunar thunar-volman
* xfce4-panel xfce4-settings xfce4-session xfdesktop xfwm4 xfce4-appfinder tumbler xfce4-power-manager



## xfce触摸板设置
### 前提条件
确保已经安装了synaptics驱动
```shell
$ sudo apt install xserver-xorg-input-synaptics
```

### 复制配置文件
将`/usr/share/X11/xorg.conf.d`文件夹复制到`/etc/X11`内
```shell
$ sudo cp -R /usr/share/X11/xorg.conf.d/ /etc/X11
```

### 修改配置文件
修改配置文件`/etc/X11/xorg.conf.d/70-synaptics.conf`
```shell
$ cat 70-synaptics.conf
...
Section "InputClass"
        Identifier "touchpad catchall"
        Driver "synaptics"
        MatchIsTouchpad "on"
# This option is recommend on all Linux systems using evdev, but cannot be
# enabled by default. See the following link for details:
# http://who-t.blogspot.com/2010/11/how-to-ignore-configuration-errors.html
        MatchDevicePath "/dev/input/event*"
	Option "TapButton1" "1"
	Option "TapButton3" "2"
	Option "TapButton2" "3"
EndSection
...
```
其中：
1. Option "TapButton1" "1" 表示单点点击触摸板模拟鼠标左键单击
2. Option "TapButton3" "2" 表示两点点击触摸板模拟鼠标右键单击
3. Option "TapButton2" "3" 表示三点点击触摸板模拟鼠标中键单击


## 连接无线网络
```shell
nmcli dev wifi connect <wifi-name> password <password>
```
