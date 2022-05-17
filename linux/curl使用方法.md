# curl使用方法

### get请求

```shell
$ curl http://baidu.com/name=shuming&age=20
```

### post请求

```shell
curl -d "name=shuming&age=20" http://www.baidu.com
```

### curl 查看响应头信息
使用`-I`参数

```shell
$ curl -I https://www.baidu.com

```

### curl设置cookies
使用`cookie "COOKIES"`选项来指定cookie，多个cookie使用分号分割
```shell
$ curl http://man.linuxde.net --cookie "user=root;pass=123456"
```