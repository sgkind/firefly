显示设置-xrandr
===

## 设置显示器
如果笔记本安装外接显示器，要设置笔记本显示器和外接显示器显示不同的内容，则可以用xrandr设置

### 查看所有显示器
```shell
$ xrandr
Screen 0: minimum 8 x 8, current 3840 x 1080, maximum 32767 x 32767
eDP1 connected 1920x1080+0+0 (normal left inverted right x axis y axis) 340mm x 190mm
   960x540       59.82  
   864x486       60.00    59.92    59.57  
   640x480       59.94  
HDMI1 connected 1920x1080+1920+0 (normal left inverted right x axis y axis) 440mm x 240mm
   1920x1080     60.00*+
   1680x1050     59.88  
   1280x1024     75.02    60.02  
   1440x900      59.90  
   1280x960      60.00  
   1280x720      60.00  
   1024x768      75.03    70.07    60.00  
   832x624       74.55  
   800x600       72.19    75.00    60.32    56.25  
   640x480       75.00    72.81    66.67    59.94  
   720x400       70.08  
VIRTUAL1 disconnected (normal left inverted right x axis y axis)
  1920x1080 (0x47) 138.700MHz +HSync -VSync
        h: width  1920 start 1968 end 2000 total 2080 skew    0 clock  66.68KHz
        v: height 1080 start 1083 end 1088 total 1111           clock  60.02Hz
```

目前此系统有两个显示器，一个为eDP1，另一个为外接的HDMI1

### 同步模式
```
$ xrandr --output HDMI1 --mode 1280x800 --same-as eDP1 --auto
```

### 设置HDMI1在eDP1的右侧
```shell
$ xrandr --output HDMI1 --right-of eDP1
```

* --left-of HDMI1在eDP1的左边
* --right-of HDMI1在eDP1的右边
* --above HDMI1在eDP1的上边
* --below HDMI1在eDP1的下边

## 手动添加显示器分辨率
在某些情况下，系统不能正确显示屏幕支持的分辨率，需要手动设置

### 获取分辨率mode
```shell
$ cvt 1920 1080
# 1920x1080 59.96 Hz (CVT 2.07M9) hsync: 67.16 kHz; pclk: 173.00 MHz
Modeline "1920x1080_60.00"  173.00  1920 2048 2248 2576  1080 1083 1088 1120 -hsync +vsync
```

### 创建新的分辨率mode
```shell
$ xrandr --newmode "1920x1080_60.00"  173.00  1920 2048 2248 2576  1080 1083 1088 1120 -hsync +vsync
```

其中，--newmode后面的内容为上一步中cvt的输出

### 给显示器添加新创建的分辨率mode
```shell
$ xrandr --addmode eDP1 1920x1080_60.00
```

### 设置显示器使用新创建的分辨率
```shell
$ xrandr --output eDP1 --mode 1920x1080_60.00
```