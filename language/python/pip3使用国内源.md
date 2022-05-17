### pip3使用国内源

Python开发的时候需要安装各种模块，而pip是很强大的模块安装工具，但是由于国外官方pypi经常被墙，导致不可用，所以我们最好是将自己使用的pip源更换一下。

例如豆瓣：<http://pypi.douban.com/simple/> 
清华：https://pypi.tuna.tsinghua.edu.cn/simple

个人喜欢清华大学的pip源，它是官网pypi的镜像，每隔5分钟同步一次，地址为： https://pypi.tuna.tsinghua.edu.cn/simple

使用命令:

```python
pip -i https://pypi.tuna.tsinghua.edu.cn/simple
```

例如:

```python
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple numpy
```

