qmake开发
===

## 一、安装开发环境
```shell
$ sudo apt install qt5-default qt5-qmake
```

## 二、创建项目
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