###套接字编程简介

1 IPv4套接字地址结构
   sockaddr_in    <netinet/in.h>
   '''
    struct in_addr {
        in_addr_t    s_addr;          /* 32-bit IPv4 address */
    };                                /* network byte ordered */
    '''

    '''
    struct sockaddr_in {
    	uint8_t        sin_len;       /* length of structure (16) */
    	sa_family_t    sin_family;    /* AF_INET */
    	in_port_t      sin_port;      /* 16-bit TCP or UDP port number */

        struct in_addr sin_addr;      /* 32-bit IPv4 address */
                                      /* network byte ordered */
        char           sin_zero[8];
    };
    ```
2 32位IPv4地址存在两种不同的访问方法。如果serv定义为某个网际套接字地址结构，那么serv.sin_addr将按in_addr结构引用其中的32位IPv4地址，而serv.sin_addr.s_addr将按in_addr_t（通常是一个无符号的32位整数）引用同一个32位IPv4地址。

3 通用套接字地址结构sockaddr，定义在头文件<sys/socket.h>中
  '''
  struct sockaddr {
      uint8_t    sa_len;
      sa_family  sa_family;          /* address family: AF_xxx value */
      char       sa_data[14];        /* protocol-specific address */
  };
  '''

  4 IPv6套接字地址结构<netinet/in.h>
    '''
    struct in6_addr {
    	uint8_t   s6_addr[14];        /* 128-bit IPv6 address */
    };                                /* network byte ordered */

    #define SIN6_LEN

    struct sockaddr_in6 {
    	uint8_t          sin6_len;         /* length of this struct (28) */
    	sa_family_t      sin6_family;      /* AF_INET6 */
    	in_port_t        sin6_port;        /* transport layer port# */
                                           /* network byte ordered */
    	uint32_t         sin6_flowinfo;    /* flow information, undefined */
    	struct in6_addr  sin6_addr;        /* IPv6 address */
                                           /* network byte ordered */
    	uint32_t         sin6_scope_id;    /* set of interfaces for a scope */
    };
    '''

5 新的通用套接字地址结构
  struct sockaddr_storage    头文件<netinet/in.h>
  '''
  struct sockaddr_storage {
      uint8_t        ss_len;      /* length of this struct */
      sa_family_t    ss_family;   /* address family:AF_xxx value */
  };
  '''

6 字节序转换函数
   '''
   #include <netinet/in.h>

   uint16_t  htons(uint16_t host16bitvalue);
   uint32_t  htonl(uint32_t host32bitvalue);

   uint16_t  ntohs(uint16_t net16bitvalue);
   uint32_t  ntohl(uint32_t net32bitvalue);
   '''

7 字节操纵函数
  BSD函数
  '''
  #include <strings.h>
  void bzero(void *dest, size_t nbytes);
  void bcopy(const void *src, void *dest, size_t nbytes);
  int bcmp(const void *ptr1, const void *ptr2, size_t nbytes);
  '''
  ANSI C函数
  '''
  #include <strings.h>
  void *memset(void *dest, int c, size_t len);
  void *memcpy(void *dest, const void *src, size_t nbytes);
  int memcmp(const void *ptr1, const void *ptr2, size_t nbytes);
  '''

8 inet_aton inet_addr inet_ntoa
   '''
   #include <arpa/inet.h>
   int inet_aton(const char *strptr, struct in_addr *addrptr);

   in_addr_t inet_addr(const char *strptr);   //已被废弃

   char *inet_ntoa(struct in_addr inaddr);
   '''

9 inet_pton inet_ntop
   '''
   #include <arpa/inet.h>
   int inet_pton(int family, const char *strptr, void *addrptr);

   const char *inet_ntop(int family, const void *addrptr, char *strptr, size_t len);
   '''

10 socket函数
   '''
   #include <sys/socket.h>
   int socket(int family, int type, int protocol);
   '''
   AF_前缀表示地址族，PF_前缀表示协议族

11 connect函数
    '''
    #include <sys/socket.h>
    int connect(int sockfd, const struct sockaddr *servaddr, socklen_t addrlen);
    '''

12 bind函数
   '''
   #include <sys/socket.h>
   int bind(int sockfd, const struct sockaddr *myaddr, socklen_t addrlen);
   '''

13 listen函数
   '''
   #include <sys/socket.h>
   int listen(int sockfd, int backlog);
   '''
14 accept函数
   '''
   #include <sys/socket.h>
   int accept(int sockfd, struct sockaddr *cliaddr, socklen_t *addrlen);
   '''

15 fork function
    '''
    #include <unistd.h>
    pid_t fork(void);
    '''

16 exec function
   '''
   #include <unistd.h>

   int execl(const char *pathname, const char *arg0, .../* (char *) 0 */);

   int execv(const char *pathname, char *const *argv[]);

   int execle(const char *pathname, const char *arg0, ... /* (char *) 0, char *const envp[] */);

   int execve(const char *pathname, char *const argv[], char *const envp[]);

   int execlp(const char *filename, const char *arg0, ... /* (char *) 0 */ );

   int execvp(const char *filename, char *const argv[]);
   '''

17 close
   '''
   #include <unistd.h>
   int close(int sockfd);
   '''
   