Qt
===
## ubuntu中Qt5.7.0无法输入中文
1. 把libfcitxplatforminputcontextplugin.so复制到安装的Qt目录下的两个文件夹中
```
sudo apt install fcitx-frontend-qt5
sudo cp /usr/lib/x86_64-linux-gnu/qt5/plugins/platforminputcontexts/libfcitxplatforminputcontextplugin.so /opt/Qt5.7.0/5.7/gcc_64/plugins/platforminputcontexts/ 

sudo cp /usr/lib/x86_64-linux-gnu/qt5/plugins/platforminputcontexts/libfcitxplatforminputcontextplugin.so /opt/Qt5.7.0/Tools/QtCreator/lib/Qt/plugins/platforminputcontexts/
```

2. 重启电脑即可

## qmake开发
### 一、安装开发环境
```shell
$ sudo apt install qt5-default qt5-qmake
```

### 二、创建项目
```shell
$ mkdir hello
$ cd hello
```

```shell
$ cat helloworld.cpp
#include <QPushButton>
#include <QApplication>
 
int main(int argc, char *argv[])
{
    QApplication app(argc, argv);
 
    QPushButton btn("hello world");
    btn.resize(200,100);
    btn.show();
    return app.exec();
}
```

```shell
$ qmake -project
```

编辑

QT += core gui

greaterThan(QT_MAJOR_VERSION, 4): QT += widgets

qmake hello.pro

 生成了Makefile

make