ovn编译
===

**未成功**

## 编译ovs
### 环境
1. debian 10

### 安装依赖包
```
# apt -y update
# apt -y upgrade
# apt -y install -R \
                build-essential dpkg-dev lintian devscripts fakeroot \
                debhelper dh-autoreconf uuid-runtime \
                autoconf automake libtool \
                python3-all  \
                xdg-utils groff graphviz netcat curl ethtool \
                libcap-ng-dev libssl-dev python3-dev openssl \
                python3-pyftpdlib python3-flake8 \
                linux-headers-`uname -r` \
                lftp
# apt install python3-twisted python-wget python3-wget
# apt install python3-pip
# pip3 install tftpy
# apt install python3-sphinx libunbound-dev libunwind-dev 
```

### 克隆仓库
```
# cd /root
# git clone https://github.com/openvswitch/ovs.git
```


### 配置ovs
```
cd /root/ovs
./boot.sh
[ -f Makefile ] && ./configure && make distclean
mkdir -p ~/build
cd ~/build
/root/ovs/configure --with-linux=/lib/modules/`uname -r`/build --enable-silent-rules
```

### 编译ovs
```
cd ~/build
make
```

### 打包
```
# cd ~/build
# PACKAGE_VERSION=`autom4te -l Autoconf -t 'AC_INIT:$2' /root/ovs/configure.ac`
# make dist
# cd ~/
# ln -sf ~/build/openvswitch-$PACKAGE_VERSION.tar.gz openvswitch_$PACKAGE_VERSION.orig.tar.gz
# rm -rf ~/openvswitch-$PACKAGE_VERSION
# dpkg-buildpackage -us -uc
```


### 将系统默认python修改为python3
```
update-alternatives --install /usr/bin/python python /usr/bin/python3 1
```

### 安装需要的包
```
apt-get -y install build-essential fakeroot graphviz autoconf automake bzip2 \
                   debhelper dh-autoreconf libssl-dev libtool openssl procps \
                   python-all python-qt4 python-twisted-conch python-zopeinterface \
                   libcap-ng-dev libunbound-dev

apt -y install python3-all python3-sphinx python3-twisted python3-zope.interface libunwind-dev
```

### 克隆仓库
```
# mkdir /vagrant
# cd /vagrant
# git clone https://github.com/openvswitch/ovs.git
# git clone https://github.com/ovn-org/ovn.git
```

### 配置ovs
```
# cd /vagrant/ovs
# ./boot.sh
# [ -f Makefile ] && ./configure && make distclean
# mkdir -p ~/build/ovs
# cd ~/build/ovs
# /vagrant/ovs/configure --with-linux=/lib/modules/`uname -r`/build --enable-silent-rules
```

### 编译ovs
```
# cd ~/build/ovs
# make
```

### 打包ovs
```
# cd ~/build/ovs
# PACKAGE_VERSION=`autom4te -l Autoconf -t 'AC_INIT:$2' /vagrant/ovs/configure.ac`
# make dist
# mkdir ~/deb/ovs
# cd ~/deb/ovs
# ln -sf ~/build/ovs/openvswitch-$PACKAGE_VERSION.tar.gz openvswitch_$PACKAGE_VERSION.orig.tar.gz
# rm -rf openvswitch-$PACKAGE_VERSION
# dpkg-buildpackage -us -uc
```

### 配置ovn
```
# cd /vagrant/ovn
# ./boot.sh
# [ -f Makefile ] && ./configure --with-ovs-source=/vagrant/ovs --with-ovs-build=${HOME}/build/ovs && make distclean
# mkdir -pv ~/build/ovn
# cd ~/build/ovn
# /vagrant/ovn/configure --with-ovs-source=/vagrant/ovs --with-ovs-build=${HOME}/build/ovs
```

### 编译
```
# cd ~/build/ovn
# make
```

### 打包
```
# cd ~/build/ovn
# PACKAGE_VERSION=`autom4te -l Autoconf -t 'AC_INIT:$2' /vagrant/ovn/configure.ac`
# make dist
# mkdir ~/deb/ovn
# cd ~/deb/ovn
# ln -sf ~/build/ovn/ovn-$PACKAGE_VERSION.tar.gz ovn-$PACKAGE_VERSION.orig.tar.gz
# rm -rf ovn-$PACKAGE_VERSION
# tar xzf ovn-$PACKAGE_VERSION.orig.tar.gz
# cd ovn-$PACKAGE_VERSION.orig.tar.gz
