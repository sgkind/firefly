一、空间
----
1.df 查看磁盘空间

2.dd命令快速生成指定大小的文件

```shell
dd if=/dev/zero of=test bs=1M count=1000
```

3.dd命令生成超大文件，但是实际不占用空间

```shell
dd if=/dev/zero of=test bs=1M count=0 seek=10000
```

4.随机生成一百万个1K的文件

```shell
seq 1000000 | xargs -i dd if=/dev/zero of={}.dat bs=1024 count=1
```
