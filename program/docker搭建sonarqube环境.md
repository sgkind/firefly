docker搭建sonarqube环境
===

## 获取镜像
```shell
$ sudo docker pull postgres:10
$ sudo docker pull sonarqube:7.9.1-community
```

## 启动镜像
* 启动postgres镜像
```shell
$ sudo docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=1 --name postgres postgres:10
```
* 采用psql登录postgres，并创建sonar数据库(用户名postgres，密码：1)
```shell
$ psql -h 192.168.200.222 -U postgres
用户 postgres 的口令：<1>
postgres=# create database sonar;
$
```
* 设置虚拟机最大map count
```shell
$ sudo sysctl -w vm.max_map_count=262144
```
* 启动sonarqube镜像
```shell
$ sudo docker run -d -p 9000:9000 -e "SONARQUBE_JDBC_URL=jdbc:postgresql://192.168.200.222:5432/sonar" -e "SONARQUBE_JDBC_USERNAME=postgres" -e "SONARQUBE_JDBC_PASSWORD=1" --name sonarqube sonarqube:7.9.1-community
```

## 下载sonar scanner
1. 去[sonarqube sonarscanner](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/)下载linux版本的sonarscanner
2. 用`unzip sonar-scanner-cli-4.3.0.2102-linux.zip`命令解压sonar-scanner-cli-4.3.0.2102-linux.zip
3. 将sonar-scanner-4.3.0.2102-linux移动到/usr/share/下
4. 用命令`ln -s /bin/sonar-scanner /usr/share/sonar-scanner-4.3.0.12102-linux/bin/sonar-scanner`创建软连接

## 扫描项目
1. 编辑$install_directory/conf/sonar-scanner.properties
```
#Configure here general information about the environment, such as SonarQube server connection details for example
#No information about specific project should appear here

#----- Default SonarQube server
sonar.host.url=http://192.168.200.222:9000

#----- Default source code encoding
sonar.sourceEncoding=UTF-8
```
2. 浏览器打开http://192.168.200.222:9000网页，用admin:admin登录，
3. 在项目目录下执行
```shell
$ cd ~/Project/mars
$ sonar-scanner -Dsonar.projectKey=onos -Dsonar.sources=/home/ubuntu/Project/mars -Dsonar.language=java -Dsonar.java.binaries=/home/ubuntu/Project/mars/buck-out/gen
```

参考：
1. https://www.cnblogs.com/zhi-leaf/p/11538413.html
2. 