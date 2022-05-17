## 系统
ubuntu18.04

## 检查系统是否支持虚拟化
```
grep -E --color 'vmx|svm' /proc/cpuinfo
```

## 安装kubectl
* 下载最新发布版的kubectl
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
```
* 给kubectl赋予可执行权限
```
chmod +x ./kubectl
```
* 将kubectl移动到指定目录
```
sudo mv ./kubectl /usr/local/bin/kubectl
```
* 测试kubectl是否正常安装
```
kubectl version
```

## 安装docker
* 安装docker
```
sudo apt install -y docker.io
```
* 启动docker服务
```
sudo systemctl enable docker.service
```

## 安装minikube
* 下载二进制包
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube
```
* 安装
```
sudo install minikube /usr/local/bin
```
* 测试安装是否成功
```
$ minikube version
minikube version: v1.3.1
commit: ca60a424ce69a4d79f502650199ca2b52f29e631
```

## 运行
```
sudo minikube start --vm-driver=none
```

## 在运行`sudo minikube start --vm-driver=none`时发生的警告和错误
1.警告
kubelet服务未运行，安装提示运行如下命令即可
```
sudo systemctl enable kubelet.service
```
2.错误
因墙的存在，在执行`minikube start --vm-driver=non`命令时，会发生docker image下载错误的情况
* 针对需要的docker image，先在国内阿里云docker仓库中下载
```
sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.2.24
```
* 然后将docker image标签改为指定的
```
sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.2.24 k8s.gcr.io/etcd:3.2.24
```

注意：需要根据提示修改docker image的名称和版本号

参考:
https://kubernetes.io/docs/tasks/tools/install-kubectl/
https://www.jianshu.com/p/70efa1b853f5