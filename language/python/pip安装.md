# 安装pip

## ubuntu安装pip

### 安装python
```shell
$ sudo apt install python
```

### 安装pip

1. 去[pypi.org](https://pypi.org/project/pip/#files)下载pip-xx.x.x.tar.gz
2. 解压
```shell
$ tar xvf pip-19.3.1.tar.gz
```
3. 安装
```shell
$ cd pip-19.3.1
$ sudo python setup.py install
```

### 安装wheel和setuptools
```shell
$ sudo pip install --upgrade wheel
$ sudo pip install --upgrade setuptools
```
