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
