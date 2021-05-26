## Chapter Two



1. TCP 的 TIME_WAIT 状态会造成网络编程人员的混淆。
2. IP 协议其他版本信息：[IPv0 到 IPv9](https://wander.science/articles/ip-version/)

## Chapter Two



1. TCP 的 TIME_WAIT 状态会造成网络编程人员的混淆。
2. IP 协议其他版本信息：[IPv0 到 IPv9](https://wander.science/articles/ip-version/)





## Chapter Three

```c
#include <netinet/in.h>
   	struct in_addr {
    	in_addr_t s_addr; 	// 32-bits IPv4 address
	}
	struct sockaddr_in {
        uint8_t 		sin_len;		// length of structure (16)
        sa_family_t 	sin_family;		// AF_INEF
        in_port_t 		sin_port;		// 16-bits TCP or UDP ports
        struct in_addr 	sin_addr;		// 32-bits IPv4 address
        char 			sin_zero[8];	// unused
    }

	
```

通用套接字地址结构

```c
#include <sys/socket.h>
    struct sockaddr {
        uint8_t 		sa_len;		// length of structure (16)
        sa_family_t 	sa_family;		// AF_INEF
        char			sa_data[14];
    }
```



新通用套接字地址结构

```c
#include <netinet/in.h>
struct sockaddr_storage {
    uint8_t 		ss_len;			// length of this struct
    sa_family_t 	ss_family;		// AF_XXX
}
```

如所有套接字函数都是内核的系统调用（System V 中作为库函数）：

从进程到内核传递套接字地址结构的函数：bind、connect、sendto。

从内核到进程传递套接字地址结构的函数：accept、recvfrom、getsockname、getpeername。

传递套接字地址结构的函数：recvmsg、sendmsg。



字节序转换：

```c
#include <netinet/in.h>
uint16_t htons(uint16_t host16bitvalue);
uint32_t htonl(uint32_t host32bitvalue);	// 返回网络序的值

uint16_t ntohs(uint16_t net16bitvalue);
uint32_t ntohl(uint32_t net32bitvalue);		// 返回主机序的值
```



注：在 TCP/IP 发展早期，Byte（8 bits）一般不使用，使用 octet 作为 1 Byte。

注：ANSI C 的 fun(const void *ptr) 表示是该指针所指向的内容不会被函数修改。



IPv4 地址 ASCII 字符串与网络字节序二进制值相互转换：

```c
#include <arpa/inet.h>

int inet_aton(const char *strptr, struct in_addr *addrptr);	// 将 C 字符串转换为 32 位二进制网络字节序，成功返回 1，失败返回 0
// 注：若 addrptr 为空，则任会对 strptr 进行有效性检查，但不存结果

in_addr_t inet_addr(const char *strptr);	// 返回 32 位二进制网络字节序的 IPv4 地址
// 该函数出错返回 TNADDR_NONE：全 1，故该函数无法处理 255.255.255.255
// 此函数已被废弃，改用 inet_aton 函数

char* inet_ntoa(struct in_addr inaddr);		// 返回一个点分十进制字符串的指针
```



```c
#include <arpa/inet.h>
int inet_pton(int family, const char *strptr, void *addrptr);	// 成功返回1，错误返回-1，格式错误返回0

const char* inet_ntop(int family, const void *addrptr, char *strptr, size_t len);	// 成功返回指向结果的指针，出错返回NULL

// p 代表表达（presentation 通常指 ASCII 字符串）和 n 代表数值（numeric）
// family 可以是 AF_INET 或 AF_INET6
```





## Chapter Four

AF_XXX：表示地址簇

PF_XXX：表示协议簇

```c
#include <sys/socket.h>
int connect(int sockfd, const struct sockaddr *servaddr, socklen_t addrlen);	// 成功返回0，失败返回1
```



产生 RST 的条件：

1. 目的地某端口收到 SYN 到达，但该端口上没有正在监听的服务器
2. TCP 想取消一个已有连接
3. TCP 接收到一个根本不存在的连接上的报文。





```c
#include <sys/socket.h>
int bind(int sockfd, const struct sockaddr *servaddr, socklen_t addrlen);	// 成功返回0，失败返回1
```

调用 bind 可以指定 IP 地址或端口，可以两者都指定，也可以都不指定

```
                               +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                               |          进程指定          |                                    |
                               +-+-+-+-+-+-+-+-+-+-+-+-+-+|			  结果				      |
                               |   IP 地址       |   端口   |              			             |
                               +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                               |     通配地址     |     0    |         内核选择IP地址和端口          |
                               |     通配地址     |    非0   |     内核选择IP地址，进程指定端口        |
                               |    本地IP地址    |     0    |     进程指定IP地址，内核选择端口        |
                               |    本地IP地址    |    非0   |         进程指定IP地址和端口          |
                               +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

注：对于 IPv4，通配地址由 INADDR_ANY 来指定，其值一般为 0。对于 IPv6，则是 IN6ADDR_ANY。

为得到内核所选择的临时端口值，必须调用函数 getsockname 来返回协议地址。

```c
#include <sys/socket.h>
int listen(int sockfd, int backlog);	// 成功返回0，出错返回-1
```

做两件事：

1. 当 socket 函数创建一个套接字时，其被假设为一个主动套接字，也就是其是客户端套接字。listen 函数把 一个未连接的套接字转换成一个被动套接字。
2. 该函数的第二个参数规定内核为该套接字排队的最大连接个数。



为理解 backlog 参数：内核为监听每个套接字维护两个队列：

1. 未完成连接队列（incomplete connection queue）：正处于 TCP 三次握手中的套接字。处于SYN_RCVD。
2. 已完成连接队列（completed connection queue）：已完成 TCP 三次握手的套接字，处于 ESTABLISHED。



[SYN 泛洪攻击](https://datatracker.ietf.org/doc/html/rfc4987)



```c
#include <sys/socket.h>
int accept(int sockfd, struct sockaddr *cliaddr, socklen_t *addrlen);	// 成功返回非负描述符，出错返回-1
```

accept 函数由 TCP 服务器调用，用于从已完成连接队列队头中取出一个已完成连接。如已完成连接队列为空，则进程进入睡眠（阻塞方式）。

accept 函数第一个参数是监听套接字（listening socket）描述符，返回值是已连接套接字（connected socket）描述符。



```c
#include <unistd.h>
pit_t fork(void);	// 子进程中返回0，父进程则返回子进程ID，出错返回-1
```

为什么子进程不返回父进程 ID 而返回 0？

因为每个进程都有一个父进程，且可以通过 getppid() 来获得父进程 ID，而父进程可能有多个子进程，所以会无法得知具体某个子进程 ID。



fork 两个典型用法：

1. 一个进程创建一个自身的副本。
2. 一个进程想要执行另一个程序。



存放在硬盘上的可执行文件能够被 Unix 执行的唯一方法：由一个现有进程调用 6 个 exec 函数中的某一个。exec 把当前进程映像替换成新的程序文件，进程 ID 并不改变。

我们称调用 exec 的进程为调用进程（calling process），称新执行的程序为新程序（new program）。



```c
#include <unistd.h>

extern char **environ;

int execl(const char *path, const char *arg0, ...);
int execlp(const char *file, const char *arg, ...);
int execle(const char *path, const char *arg , ..., char * const envp[]);
int execv(const char *path, char *const argv[]);
int execvp(const char *file, char *const argv[]);
int execve(const char *path, char *const argv[], char *const envp[]);
```



+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|    execlp(file, arg, ..., 0)      |                        
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                          
                       





## Chapter Three

```c
#include <netinet/in.h>
   	struct in_addr {
    	in_addr_t s_addr; 	// 32-bits IPv4 address
	}
	struct sockaddr_in {
        uint8_t 		sin_len;		// length of structure (16)
        sa_family_t 	sin_family;		// AF_INEF
        in_port_t 		sin_port;		// 16-bits TCP or UDP ports
        struct in_addr 	sin_addr;		// 32-bits IPv4 address
        char 			sin_zero[8];	// unused
    }

	
```

通用套接字地址结构

```c
#include <sys/socket.h>
    struct sockaddr {
        uint8_t 		sa_len;		// length of structure (16)
        sa_family_t 	sa_family;		// AF_INEF
        char			sa_data[14];
    }
```



新通用套接字地址结构

```c
#include <netinet/in.h>
struct sockaddr_storage {
    uint8_t 		ss_len;			// length of this struct
    sa_family_t 	ss_family;		// AF_XXX
}
```

如所有套接字函数都是内核的系统调用（System V 中作为库函数）：

从进程到内核传递套接字地址结构的函数：bind、connect、sendto。

从内核到进程传递套接字地址结构的函数：accept、recvfrom、getsockname、getpeername。

传递套接字地址结构的函数：recvmsg、sendmsg。



字节序转换：

```c
#include <netinet/in.h>
uint16_t htons(uint16_t host16bitvalue);
uint32_t htonl(uint32_t host32bitvalue);	// 返回网络序的值

uint16_t ntohs(uint16_t net16bitvalue);
uint32_t ntohl(uint32_t net32bitvalue);		// 返回主机序的值
```



注：在 TCP/IP 发展早期，Byte（8 bits）一般不使用，使用 octet 作为 1 Byte。
