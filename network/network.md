network interface
==

## data structure

* struct sockaddr
  ```
  struct sockaddr {
      u_char  sa_len;
	  u_short sa_family;
	  char    sa_data[14];
  }
  ```
* struct sockaddr_in
  ```
  struct sockaddr_in {
      u_char  sin_len;
	  u_short sin_family;
	  u_short sin_port;
	  struct  in_addr sin_addr;
	  char    sin_zero[8];
  }
  ```
  * struct in_addr
    ```
	struct in_addr {
        uint32_t s_addr;
	}
	```
* struct hostent
  ```
  struct hostent {
      char *h_name;
	  char **h_aliases;
	  int  h_addrtype;
	  int  h_length;
	  char **h_addr_list;
  };
  ```
* struct servent
  ```
  struct servent {
      char *s_name;
	  char **s_aliases;
	  int  s_port;
	  char *s_proto;
  }
  ```


## sock APIs

* socket
  ```
  #include <sys/types.h>
  #include <sys/socket.h>
  int socket(int domain, int type, int protocol);
  ```
* bind
  ```
  #include <sys/types.h>
  #include <sys/socket.h>
  int bind(int sockfd, struct sockaddr *my_addr, int addrlen);
  ```
* connect
  ```
  #include <sys/types.h>
  #include <sys/socket.h>
  int connect(int sockfd, struct sockaddr *serv_addr, int addrlen);
  ```
* listen
  ```
  #include <sys/socket.h>
  int listen(int sockfd, int backlog);
  ```
* accept
  ```
  #include <sys/types.h>
  #include <sys/socket.h>
  int accept(int sockfd, struct sockaddr *add, int *addrlen);
  ```

* send
  ```
  int send(int sockfd, const void *msg, int len, int flags);
  ```
* recv
  ```
  int recv(int sockfd, void *buf, int len, unsigned int flags);
  ```
* write
  ```
  #include <unistd.h>
  ssize_t write(int fd, const void *buf, size_t count);
  ```
* read
  ```
  #include <unistd.h>
  ssize_t read(int fd, void *buf, size_t count);
  ```
* close
  ```
  #include <unistd.h>
  int close(int sockfd);
  ```
* shutdown
  ```
  int shutdown(int sockfd, int how);
  ```
  how is one of the following.
  * 0 - Further receives are disallowed.
  * 1 - Further sends are disallowed.
  * 2 - Further sends and receives are disallowed(like close()).
* sendto
  ```
  int sendto(int sockfd, const void *msg, int len, unsigned int flags, const struct sockaddr *to, int tolen);
  ```
* recvfrom
  ```
  int recvfrom(int sockfd, void *buf, int len, unsigned int flags, struct sockaddr *from, int *fromlen);
  ```

* gethostbyname()
  ```
  #include <netdb.h>
  extern int h_errno;
  struct hostent *gethostbyname(const char *name, const char *proto);
  ```

* getservbyname()
  ```
  #include <netdb.h>
  struct servent *getservbyname(const char *name);
  ```
* getprotobyname()
  ```
  #include <netdb.h>
  struct protoent *getprotobyname(const char *name);
  ```
  ```
  struct protoent {
      char *p_name;
	  char **p_aliases;
	  int  p_proto;
  }
  ```
* getpeername()
  ```
  #include <sys/socket.h>
  int getpeername(int sockfd, struct sockaddr *addr, int *addrlen);
  ```


## code
* address reuse
  ```
  int yes = 1
  if (setsockopt(listener, SOL_SOCKET, SO_REUSEADDR, &yes, sizeof(int)) == -1)
  {
      perror("setsockopt() error");
	  exit(1);
  }
  else
      printf("setsockopt() is OK.\n");
  ```

* A sample of the client socket call flow
  ```
  socket()
  connect()
  while(x)
  {
      write()
	  read()
  }
  close()
  ```
* A sample of the server socket call flow
  ```
  socket()
  bind()
  listen()
  while(1)
  {
      accept()
	  while(x)
	  {
	      read()
		  write()
	  }
	  close()
   }
   close()

