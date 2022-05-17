sudo apt install libgudev-1.0-dev
sudo apt install libwnck-3-dev
sudo apt install libdbusmenu-gtk3-dev
sudo apt install libnotify-dev
sudo apt install libupower-glib-dev

tar -xjf 
编译顺序：
* xfce4-dev-tools (开发工具)
* libxfce4util
* xfconf
* libxfce4ui
* garcon exo
* thunar thunar-volman
* xfce4-panel, xfce4-settings, xfce4-session, xfdesktop, xfwm4, xfce4-appfinder, tumbler, xfce4-power-manager

编译命令
./configure --prefix=/usr/local/ && make && sudo make install
