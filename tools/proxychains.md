## ShadowsocksR设置
在ShadowsocksR右键菜单选项设置中勾选**允许来自局域网的连接**，本地端口号为1080，设置用户名:<usr>，设置密码:<user>

## linux
1.安装proxychains
```
sudo apt-get install -y proxychains
```
2.配置代理地址
proxychains会按如下顺序查找配置文件
* ./proxychains.conf 
* $(HOME)/.proxychains/proxychains.conf
* /etc/proxychains

```
sudo vim /etc/proxychains.conf
```

添加服务器配置信息，具体添加格式见文档中说明

3.解决保存问题
> ERROR: ld.so: object 'libproxychains.so.3' from LD_PRELOAD cannot be preloaded (cannot open shared object file): ignored.

* 查找libproxychains.so.3的路径
```
$ find /usr/ -name libproxychains.so.3 -print
/usr/lib/x86_64-linux-gnu/libproxychains.so.3
```
* 修改/usr/bin/proxychains中LD_PRELOAD的值
```
sudo vim /usr/bin/proxychains
```
将LD_PRELOAD的值修改为libproxychains.so.3的绝对路径

## 使用代理
```
proxychains curl -s www.google.com
```

参考：
https://www.jianshu.com/p/a8739782a801
https://www.jianshu.com/p/3f392367b41f