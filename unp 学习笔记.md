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
#include <netinet/in.h>struct sockaddr_storage {    uint8_t 		ss_len;			// length of this struct    sa_family_t 	ss_family;		// AF_XXX}
```

如所有套接字函数都是内核的系统调用（System V 中作为库函数）：

从进程到内核传递套接字地址结构的函数：bind、connect、sendto。

从内核到进程传递套接字地址结构的函数：accept、recvfrom、getsockname、getpeername。

传递套接字地址结构的函数：recvmsg、sendmsg。



字节序转换：

```c
#include <netinet/in.h>uint16_t htons(uint16_t host16bitvalue);uint32_t htonl(uint32_t host32bitvalue);	// 返回网络序的值uint16_t ntohs(uint16_t net16bitvalue);uint32_t ntohl(uint32_t net32bitvalue);		// 返回主机序的值
```



注：在 TCP/IP 发展早期，Byte（8 bits）一般不使用，使用 octet 作为 1 Byte。





## Chapter Four

AF_XXX：表示地址簇

PF_XXX：表示协议簇

```c
#include <sys/socket.h>int connect(int sockfd, const struct sockaddr *servaddr, socklen_t addrlen);	// 成功返回0，失败返回1
```



产生 RST 的条件：

1. 目的地某端口收到 SYN 到达，但该端口上没有正在监听的服务器
2. TCP 想取消一个已有连接
3. TCP 接收到一个根本不存在的连接上的报文。





```c
#include <sys/socket.h>int bind(int sockfd, const struct sockaddr *servaddr, socklen_t addrlen);	// 成功返回0，失败返回1
```

调用 bind 可以指定 IP 地址或端口，可以两者都指定，也可以都不指定

```
                               +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                               |          进程指定          |                                    |                               +-+-+-+-+-+-+-+-+-+-+-+-+-+|			  结果				      |                               |   IP 地址       |   端口   |              			             |                               +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                               |     通配地址     |     0    |         内核选择IP地址和端口          |                               |     通配地址     |    非0   |     内核选择IP地址，进程指定端口        |                               |    本地IP地址    |     0    |     进程指定IP地址，内核选择端口        |                               |    本地IP地址    |    非0   |         进程指定IP地址和端口          |                               +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

注：对于 IPv4，通配地址由 INADDR_ANY 来指定，其值一般为 0。对于 IPv6，则是 IN6ADDR_ANY。

为得到内核所选择的临时端口值，必须调用函数 getsockname 来返回协议地址。

```c
#include <sys/socket.h>int listen(int sockfd, int backlog);	// 成功返回0，出错返回-1
```

做两件事：

1. 当 socket 函数创建一个套接字时，其被假设为一个主动套接字，也就是其是客户端套接字。listen 函数把 一个未连接的套接字转换成一个被动套接字。
2. 该函数的第二个参数规定内核为该套接字排队的最大连接个数。



为理解 backlog 参数：内核为监听每个套接字维护两个队列：

1. 未完成连接队列（incomplete connection queue）：正处于 TCP 三次握手中的套接字。处于SYN_RCVD。
2. 已完成连接队列（completed connection queue）：已完成 TCP 三次握手的套接字，处于 ESTABLISHED。



[SYN 泛洪攻击](https://datatracker.ietf.org/doc/html/rfc4987)



```c
#include <sys/socket.h>int accept(int sockfd, struct sockaddr *cliaddr, socklen_t *addrlen);	// 成功返回非负描述符，出错返回-1
```

accept 函数由 TCP 服务器调用，用于从已完成连接队列队头中取出一个已完成连接。如已完成连接队列为空，则进程进入睡眠（阻塞方式）。

accept 函数第一个参数是监听套接字（listening socket）描述符，返回值是已连接套接字（connected socket）描述符。



```c
#include <unistd.h>pit_t fork(void);	// 子进程中返回0，父进程则返回子进程ID，出错返回-1
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
int execve(const char *path, char *const argv[], char *const envp[]);	// // 只有 execve 是内核中的系统调用
```


​                       

每个文件或套接字都有一个引用计数，引用计数在文件表项中维护。

fork() 后套接字的引用计数会变成 2。



```c
#include <unistd.h>
int close(int sockfd);
```

close() 一个套接字的默认行为是把该套接字标记成已关闭，然后立即返回到调用进程，然后该套接字描述符不能再由调用进程使用。

TCP 尝试把未发送的数据发送完后，再发送正常的终止序列。

SO_LINGER 套接字选项可以改变这种默认行为。



若父进程对于已连接套接字都不调用 close()，则会导致父进程耗尽可用描述符，并且没有一个客户连接被终止。





```c
#include <sys/socket.h>

int getsockname(int sockfd, struct sockaddr *localaddr, socket_len *addrlen);	// 返回某个套接字关联的本地协议地址
int getpeername(int sockfd, struct sockaddr *peeraddr, socket_len *addrlen);	// 返回某个套接字关联的外地协议地址
// 成功返回0，失败返回-1
```

用途：

1. 在 TCP 客户端上，connect 成功后，getsockname 用于返回内核赋予的套接字信息。
2. 用端口号 0 调用 bind（告知内核去选择本地端口号），getsockname 用于返回内核赋予的本地端口号。
3. getsockname 用于获取某个套接字的地址族。
4. 用通配 IP 地址调用 bind，与某个客户的连接一旦建立（accept 成功返回），getsockname 可用于返回由内核赋予该连接的本地 IP 地址。
5. 当一个服务器是由调用过 accept 的某个进程通过调用 exec 执行程序时，它能够获得客户身份的唯一途径就是调用 getpeername。





## Chapter Five

信号（signal）：告知某个进程发生了某个事件的通知，有时也成为软件中断（software interrupt）。

信号通常是异步发生的。也就是进程预先不知道信号准确发生的时刻。



信号：

1. 由一个进程发送给另一个进程（或自身）。
2. 由内核发送给某个进程。



每个信号都有一个与之关联的处置（disposition），也称为行动（action）。



1. 我们可以提供信号处理函数（signal handler），只要有特定的信号发送它就被调用。这种行为称为捕获（catching）信号。但有两个信号不能被捕获，他们是 SIGKILL 和 SIGSTOP。信号处理函数由信号值这个单一的整数参数来调用，且没返回值。函数原型如下：$void handler(int signo);$
2. 可以把某个信号的处置设定为 SIG_IGN 来忽略（ignore）它。SIGKILL 和 SIGSTOP 这两个信号不能被忽略。
3. 可以把某个信号处置设定为 SIG_DFL 来启用它的默认（default）处置。默认处置通常是在收到信号后终止进程，其中某些信号还在当前工作目录产生一个进程的核心映像（core image）。另外有个别信号的默认处置是忽略，例如 SIGCHLD 和 SIGURG。



僵死（zombie）状态：维护子进程的信息，以便父进程在之后某个时候获取。如果一个进程终止，且该进程有子进程处于僵死状态，那么它所有僵死子进程都给 init 进程（进程号为 1）。init 进程将清理这些僵死进程（init 进程将 wait 它们，从而去除它们的僵死状态）。



处理僵死进程：

无论何时我们 fork 子进程都得 wait 它们，防止其变成僵死进程。为止，需要建立一个捕获 SIGCHLD 信号的信号处理函数，在函数体中调用 wait。$Signal(SIGCHLD, sig_chld)$



sigaction 函数：

 signal 函数的使用方法简单，但并不属于 POSIX 标准，在各类 UNIX 平台上的实现不尽相同，因此其用途受

到了一定的限制。而 POSIX 标准定义的信号处理接口是 sigaction 函数，其接口头文件及原型如下：

```c
#include <signal.h>
int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);
/**
  * signum：要操作的信号。
  * act：要设置的对信号的新处理方式。
  * oldact：原来对信号的处理方式。
  * 返回值：0 表示成功，-1 表示有错误发生。
  **/
```



struct sigaction 类型用来描述对信号的处理：

```c
struct sigaction {
    void (*sa_handler)(int);
    void (*sa_sigaction)(int, siginfo_t *, void *);
    sigset_t sa_mask;
    int sa_flags;
    void (*sa_restorer)(void);
}
/**
  * sa_handler 是一个函数指针
  * sa_sigaction 是另一个信号处理函数，有三个参数，可以获得关于信号的更详细信息。
  *  sa_mask 成员用来指定在信号处理函数执行期间需要被屏蔽的信号，特别是当某个信号被处理时，它自身会被自动放入进程的信号掩码，因此在信号处理函数执行期间这个信号不会再度发生。
  *  sa_flags 成员用于指定信号处理的行为，它可以是一下值的“按位或”组合。
  *  re_restorer 成员则是一个已经废弃的数据域，不要使用。
  **/
```

sa_flags 的值包含了 SA_SIGINFO 标志时，系统将用 sa_sigaciton 函数作为信号处理函数，否则使用 sa_handler 作为信号处理函数。在某些系统中，成员 sa_handler 与 sa_sigaction 被放在联合体中，因此使用时不要同时设置。



- SA_RESTART：使被信号打断的系统调用自动重新发起。
- SA_NOCLDSTOP：使父进程在它的子进程暂停或继续运行时不会收到 SIGCHLD 信号、
- SA_NOCLDWAIT：使父进程在它的子进程退出时不会收到 SIGCHLD 信号，这时子进程如果退出也不会成为僵尸进程。
- SA_NODEFER：使对信号的屏蔽无效，即在信号处理函数执行期间仍能发出这个信号。
- SA_RESETHAND：信号处理之后重新设置为默认的处理方式。
- SA_SIGINFO：使用 sa_sigaction 成员而不是 sa_handler 作为信号处理函数。



