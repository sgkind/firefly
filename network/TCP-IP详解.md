# TCP-IP详解

1. TCP/IP通常被认为是一个四层协议系统：
	* 链路层，有时也称数据链路层或网络接口层；
	* 网络层，有时也称作互联网层；
	* 运输层；
	* 应用层；
2. IGMP是Internet组管理协议; ARP：地址解析协议; RARP；逆地址解析协议。
3. internet意思是用一个共同的协议组把多个网络连接在一起。而Internet指的是世界范围内通过TCP/IP互相通信的所有主机集合。
4. 在TCP/IP协议族中，链路层主要有三个目的：
	* 为IP模块发送和接收IP数据报；
	* 为ARP模块发送ARP请求和接收ARP应答；
	* 为RARP发送RARP请求和接收RARP应答。
5. 以太网和802.3对数据帧的长度都有一个限制，其最大值分别是1500和1492字节。链路层的这个特性称作MTU，最大传输单元。
6. IP提供不可靠、无连接的数据报传送服务。
 不可靠的意思是它不能保证IP数据报能成功地到达目的地；无连接是指IP并不维护任何关于后续数据报的状态信息。
7. ARP为IP地址到对应的硬件地址之间提供动态映射。
8. 知道主机的IP地址并不能让内核发送一帧数据给主机。内核（如以太网驱动程序）必须知道目的端的硬件地址才能发送数据。ARP的功能是在32bit的IP地址和采用不同网络技术的硬件地址之间提供动态映射。
9. 广播和多播仅应用与UDP。TCP是一个面向连接的协议，它意味着分别运行于两主机（由IP地址确定）内的两进程（由端口号确定）间存在一条连接。
10. Internet组管理协议（IGMP）用于支持主机和路由器进行多播。
11. 对每个连接，TCP管理4个不同的定时器
	* 重传定时器使用于当希望收到另一端的确认。
	* 坚持（persist）定时器使窗口大小信息保持不断流动，即使另一端关闭了其接收窗口。
	* 保活（keepalive）定时器可检测到一个空闲连接的另一端何时崩溃或重启。
	* 2MSL定时器测量一个连接处于TIME_WAIT状态的时间。

这是一段普通的文本，
直接回车不能换行，<br>
要使用\<br>

	Hello，大家好，我是果冻虾仁

欢迎到访
	很高兴见到您
	祝您，早上好，中午好，下午好

Thank `You`. Please `Call` me `Coder`

![baidu](http://www.baidu.com/img/bdlogo.gif "百度logo")
[baidu](http://www.baidu.com "百度")

* 昵称：果冻虾仁
* 别名：隔壁老王
* 英文名：Jelly


* 编程语言
	* 脚本语言
		* Python

>数据结构
>>树
>>>平衡二叉数
>>>>满二叉数

代码片段：
```c
int main(int argc, char *argv[])
```
```Bash
echo "hello GitHub"
```
```cpp
string &operator+(const string& A, const string& B)
```

图片带超链接
[![baidu]](http://www.baidu.com)
[baidu]: http://www.baidu.com/img/bdlog.gif "百度Logo"

protk_dllmd.lib
psapi.lib
mpr.lib
Netapi32.lib