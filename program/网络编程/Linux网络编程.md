# Linux网络编程

### 套接字地址结构

1.通用套接字数据结构
```
struct sockaddr {                   /*套接字地址结构*/
	sa_family_t sa_family;            /*协议族*/
	char        sa_data[14];          /*协议族数据*/
};
```
其中，sa_family的类型为sa_family_t,其实类型为unsigned short。
```
typedef unsigned short sa_family_t;
```

2.struct sockaddr_in套接字数据结构
```
struct sockaddr_in {                 /*以太网套接字地址结构*/
	u8             sin_len;          /*结构struct sockaddr_in的长度， 16*/
	u8             sin_family;       /*通常为AF_INET*/
	u16            sin_port;         /*16位的端口号，网络字节序*/
	struct in_addr sin_addr;         /*IP地址32位*/
	char           sin_zero[8];      /*未用*/
};

struct in_addr {                     /*IP地址结构*/
	u32 s_addr;                      /*32位IP地址，网络字节序*/
}
```