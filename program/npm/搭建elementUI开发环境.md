element ui环境搭建
================
## 安装npm
```shell
sudo apt install -y npm
```

## npm设置淘宝镜像
1. 永久使用
```
npm config set registry https://registry.npm.taobao.org
```
2.直接安装使用
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

3.安装cnpm
```
npm install -g cnpm
```

4.安装全局vue-cli脚手架
```
cnpm install --global vue-cli
```

5.初始化vue项目
```
vue init webpack projectname
```
注：
  因网络问题，此步骤在安装依赖包的时候会卡住。此时直接輸入`Ctrl+C`終止命令，然後進入項目，輸入 `cnpm install`即可
  
6.安装elemntUI
```
npm i element-ui -S
```
