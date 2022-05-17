# apue

#### 文件

1. 文件描述符（file descriptor）通常是一个小的非负整数，内核用来标识一个特定进程正在访问的文件。
2. 不带缓冲的I/O：函数open、read、write、lseek以及close提供了不带缓冲的I/O。





3. restrict关键字告诉编译器，哪些指针引用是可以优化的。
4. UNIX系统shell把文件描述符0与进程的标准输入关联，文件描述符1与标准输出关联，文件描述符2与标准错误关联。
5. STDIN_FILENO  STDOUT_FILENO STDERR_FILENO 头文件 unistd.h