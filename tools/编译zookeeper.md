1. 安装ant
```
apt-get install ant
```
2. 安装java
```
apt-get install openjdk-8-jdk
```
3. 安装编译环境
```
apt-get install build-essential
```
4. 安装libtool
```
apt-get install libtool
```
5. 编译并安装cppunit
* 下载源代码
```
git clone https://github.com/epronk/cppunit.git
```
我是下载zip文件后用unzip解压
* 在cppunit源代码根目录执行
```
./configure LDFLAGS='-ldl'
```
* 编译
```
make
```
* 安装
```
make install
```
6. 编译zookeeper-client-c
* 下载源代码
```
git clone https://github.com/apache/zookeeper.git
```
我是下载zip文件后用unzip解压
* 在根目录执行
```
ant compile_jute
```
* 切换到c客户端代码目录
```
cd zookeeper-client/zookeeper-client-c
```
* 执行autoreconf
```
autoreconf -if
```
* 配置
```
./configure
```
* 编译
```
make
```
* 安装
```
make install
```