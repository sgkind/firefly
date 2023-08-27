docker添加加速器
===

通过修改daemon配置文件`/etc/docker/daemon.json`来使用加速器
```shell
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://iwry9txw.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

其中，加速器可用在阿里云上申请，网址为`https://cr.console.aliyun.com/cn-shanghai/instances/mirrors`