fio硬盘测试
===

## 参数

参数名 | 说明 | 取值样例
---|---|---
bs | 每次请求的块大小。取值包括4k,8k,16k等 | 4k
ioengine | I/O引擎。推荐使用Linux的异步I/O引擎。| libaio
iodepth | 请求的I/O队列深度 | 1
direct | 指定direct模式。1表示指定O_DIRECT标识符，忽略I/O缓存，数据直写;0表示不指定O_DIRECT标识符。默认为True(1) | 1
rw | 读写模式。取值包括顺序读(read)、顺序写(write)、随机读(randread)、随机写(randwrite)、混合随机读写(randrw)和混合顺序读写(rw, readwrite)。| read
time_based | 指定采用时间模式。无需设置该参数值，只要FIO基于时间来运行| N/A
runtime | 指定测试时长，即FIO运行时长 | 600
refill_buffers | FIO将在每次提交时重新填充I/O缓冲区。默认设置是仅在初始化时填充并重用该数据 | N/A
norandommap | 进行随机I/O时，FIO将覆盖文件的每个块。若给出此参数，则将选择新的偏移量而不查看I/O历史记录。| N/A
randrepeat | 随机序列是否可重复，True(1)表示随机序列可重复，False(0)表示随机序列不可重复。默认为True(1) | 0
group_reporting | 多个job并发时，打印整个group的统计值 | N/A
name | job的名称 | fio-read
size | I/O测试的寻址空间 | 100G
filename | 测试对象，即待测试的磁盘设备名称 | /dev/sda

参考:
1. [https://cloud.tencent.com/document/product/362/6741](https://cloud.tencent.com/document/product/362/6741)

