onos开发环境搭建
===

使用bazel编译onos因需要下载一些依赖包，因为中国特色的网络环境，依赖包下载很慢并且经常报错，因此安装时需要翻墙。另外proxychains也不会起作用。我在家安装时最后尝试出来的方法是在路由器上翻墙，凡是国外的IP都翻墙，最终编译成功。

在onos wiki页面有使用代理编译onos的方法，参考[代理](https://wiki.onosproject.org/display/ONOS/Developer+Quick+Start)

```bash
$ export HTTPS_PROXY=https://<proxy address>:<proxy port>
$ export HTTP_PROXY=http://<proxy address>:<proxy port>
$ bazel build onos --action_env=HTTP_PROXY=$HTTP_PROXY
```

## 安装依赖的软件
```shell
$ sudo apt install git zip curl unzip python2.7 python3
```

## 安装Bazel
参考https://docs.bazel.build/versions/master/install-ubuntu.html，采用二进制安装

### 安装编译器
```shell
$ sudo apt install g++
```

### 安装OpenJDK
```shell
$ sudo apt install openjdk-11-jdk
```

### 下载Bazel二进制安装包
去Bazel正式版发布页面`https://github.com/bazelbuild/bazel/releases`下载安装包

当前时间(2020.02.11)我下载的安装包为bazel-1.2.0-installer-linux-x86_64.sh

### 安装
```shell
$ sudo chmod +x bazel-<version>-installer-linux-x86_64.sh
$ ./bazel-<version>-installer-linux-x86_64.sh --user
```

### 设置环境变量
在~/.bashrc或~/.zshrc中添加
```
export PATH="$PATH:$HOME/bin"
```

推荐使用bash，原因如下：

如果需要添加`source $HOME/.bazel/bin/bazel-complete.bash`，
则要添加到~/.bashrc中，如果用zsh会报错
```
/home/sgk/.bazel/bin/bazel-complete.bash:503: command not found: complete
/home/sgk/.bazel/bin/bazel-complete.bash:504: command not found: complete
```

## 编译onos

### 克隆代码
```shell
$ git clone https://gerrit.onosproject.org/onos
```

### 设置环境变量
将下面的内容添加到~/.bashrc中
```
export ONOS_ROOT=$HOME/github/onos
source $ONOS_ROOT/tools/dev/bash_profile
```

执行命令（采用bash）
```
$ source ~/.bashrc
```

### 编译
```bash
$ cd $ONOS_ROOT
$ bazel build onos
```

### 启动onos
To run ONOS locally on the development machine, simply run the following command:
```bash
$ bazel run onos-local [-- [clean] [debug]]
```
Or simpler one, if you have added the ONOS developer environment to your bash profile:
```bash
$ ok [clean] [debug]
```

### 连接onos
访问onos图形界面，打开浏览器访问

[http://localhost:8181/onos/ui](http://localhost:8181/onos/ui)

或者使用`onos-gui localhost`命令

访问onos客户端，运行
```bash
$ onos localhost
```

## 安装IDE
onos推荐采用IntelliJ IDEA集成开发环境。

安装IntelliJ IDEA（方法为下载安装包，解压，运行bin目录中的idea.sh即可），并安装Bazel插件

安装Bazel插件的方法：
1. 在IntelliJ IDEA->File->Settings->Plugins中搜索Bazel
2. 点击'Install'按钮
3. 等待IntelliJ IDEA安装完成
4. 重启IntelliJ IDEA

注：（安装时间为2020年2月11日）
安装IntelliJ IDEA 2019.3导入onos失败，安装IntelliJ IDEA 2019.2成功，onos wiki上面也建议安装2019.2。

## 使用IDE打开onos项目
参考[https://wiki.onosproject.org/pages/viewpage.action?pageId=28836246](https://wiki.onosproject.org/pages/viewpage.action?pageId=28836246)


1. 使用命令`onos-gen-bazel-project > /tmp/onos_bazelproject`生成最小.bazelproject文件
2. 启动Intellij IDEA，选择Import Bazel Project
3. 点击Workspace最后面的'...'按钮，在弹出的文件对话框中选择onos源代码所在的文件夹，单击'Next'按钮
4. 在`Select a project view`窗口中选择`Copy external`，将第一步生成的文件路径`/tmp/onos_bazelproject`填入
5. 单击`Finish`