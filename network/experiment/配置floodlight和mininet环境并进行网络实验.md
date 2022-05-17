## 系统
ubuntu 16.04

## 安装 java

在安装floodlight时，原来采用的是openjdk，在编译floodlight时一直报错，后面采用甲骨文的java后能正常编译

1.下载linu下的java安装包

下载jdk-8u221-linux-x86.tar.gz

注：官方说明floodlightv1.2的采用jdk8，另外221会随时间的推移而更新。

2.解压安装包
```
tar -zxvf jdk-8u221-linux-x86.tar.gz
```

3.将解压后的文件夹移动到/usr/lib目录下
```
sudo mkdir /usr/lib/jdk
sudo mv <安装包所在的文件夹路径>/jdk1.8.0_221/ /usr/lib/jdk
```

4.配置环境变量
* 打开/etc/profile文件
```
sudo vim /etc/profile
```
* 在/etc/profile文件末尾添加下面几行
```
#set java env
export JAVA_HOME=/usr/lib/jdk/jdk1.8.0_221
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```
* 执行命令使环境变量生效
```
source /etc/profile
```

5.测试是否安装成功
```
$ java -version
java version "1.8.0_221"
Java(TM) SE Runtime Environment (build 1.8.0_221-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.221-b11, mixed mode)
$ javac -version
javac 1.8.0_221
```

参考：https://blog.csdn.net/smile_from_2015/article/details/80056297

## 编译 floodlight

1.安装编译工具及依赖包
```
sudo apt-get install build-essential ant python-dev
```

2.从github上克隆代码
```
git clone gith://github.com/floodlight/floodlight.git
```

3.编译floodlight
```
cd floodlight
git submodule init
git submodule update
ant
```

4.测试安装是否成功
```
java -jar target/floodlight.jar
```
在另一台主机浏览器中打开网址:http://ip:8080/ui/index.html
查看是否能否访问管理界面

## 安装mininet
建议在另外一台ubuntu上安装
```
sudo apt-get install mininet
```

参考：
https://www.cnblogs.com/pullself/p/10259420.html
https://www.sdnlab.com/19189.html