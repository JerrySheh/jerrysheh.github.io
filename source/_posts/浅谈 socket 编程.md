---
title: 浅谈 socket 编程
categories: 计算机网络
tags:
  - linux
  - C/C++
  - 计算机网络
abbrlink: bfa70c14
date: 2017-10-18 00:04:01
---

# 计算机通信和socket

计算机程序之间的通信，是进程与进程之间的通信。之前在 Linux 学习中就遇到用管道（PIPE）来让一个进程的输出和另一个进程的输入连接起来，从而利用文件操作API来管理进程间通信。

Unix/Linux的基本哲学之一就是“一切皆文件”，一切都可以用`打开(open) –> 读写(read/write) –> 关闭(close)`的模式来操作。**socket可以看成一个进程向socket的一端写入或读取文本流，而另一个进程可以从另一端读取或写入。比较特别的是，这两个建立socket通信的进程可以分别属于两台不同的计算机**，可以简单地把socket理解为一种特殊的文件，我们通过操作socket函数，实现文本流在多台计算机之间传输，同时也就实现了计算机之间的通信。

一个socket包含四个地址信息: **两台计算机的IP地址和两个进程所使用的端口(port)**。IP地址用于定位计算机，而端口用于定位进程 (一台计算机上可以有多个进程分别使用不同的端口)。

# TCP和UDP

在开始 socket 编程之前，有必要简单了解一下TCP和UDP协议。

TCP（传输控制协议）提供**面向连接、可靠的字节流服务**。客户端和服务器彼此交换数据前，必须先在双方之间建立一个TCP连接，之后才能传输数据。TCP提供超时重发，丢弃重复数据，检验数据，流量控制等功能，从而**保证数据能从一端传到另一端**。

UDP（用户数据报协议）是一个简单的面向数据报的运输层协议。**UDP不提供可靠性**，它只是把应用程序传给IP层的数据报发送出去，但是并不能保证它们能到达目的地。由于UDP在传输数据报前不需在客户端和服务器之间建立一个连接，且没有超时重发等机制，故而传输速度很快。

<!-- more -->

# socket函数

## 1. socket()

`socket()`可以理解成初始化：

```c
int socket(int domain, int type, int protocol);
```

参数|内容|释义|举例
---|---|---|---
1|domain|协议族（family）|常用的 domain 有 **AF_INET（ipv4网络通信）**、AF_INET6（ipv6网络通信）、AF_LOCAL（或称AF_UNIX，本地通信，将绝对路径名作为地址）等
2|type|socket类型|常用的 socket type 有 **SOCK_STREAM（字节流）**、SOCK_DGRAM（数据报）、SOCK_RAW（原始）、SOCK_PACKET（分组）、SOCK_SEQPACKET（有序分组）等
3|protocol|指定协议|常用的协议有，**IPPROTO_TCP（TCP传输协议）**、IPPTOTO_UDP（UDP传输协议）、IPPROTO_SCTP、IPPROTO_TIPC等

- type 和 protocol 并不是可以随意组合，比如使用`SOCK_STREAM`（这是TCP相关类型）的时候就不能用`IPPROTO_UDP`（UDP相关类型）。

## 2. bind()

`bind()`函数赋予socket以固定的地址和端口，例如AF_INET就是把一个 **ipv4地址和端口号组合** 赋给socket。

```c
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

参数|内容|描述
---|---|---
1|sockfd|socket文件描述符
2|addr|指向要绑定给sockfd的协议地址的指针，这个地址结构根据地址创建socket时的地址协议族的不同而不同
3|addrlen|地址的长度


ipv4地址结构的具体实现

```c
struct sockaddr_in {
    sa_family_t    sin_family; // address family: AF_INET
    in_port_t      sin_port;   // port in network byte order
    struct in_addr sin_addr;   // internet address
};

// Internet address.
struct in_addr {
    uint32_t       s_addr;     // address in network byte order
};

```

* 创建socket时的地址协议族, 如果不是 AF_INET（ipv4），那么上面这个结构体的内容也是不一样的。第二个参数的指针就是指向你创建socket时对应协议族的协议地址。


## 3. listen()、connect()


`listen()`用于服务器监听，`connect()`用于客户端连接。


```c
int listen(int sockfd, int backlog);
```

参数|内容|描述
---|---|---
1|sockfd|要监听的socket描述字
2|backlog|最大连接个数

```c
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

参数|内容|描述
---|---|---
1|sockfd|客户端的socket描述字
2|addr|指针指向服务器的socket地址
3|addrlen|socket地址的长度


## 4. accept()

TCP服务器端依次调用`socket()`、`bind()`、`listen()`之后，就会开始监听指定的socket地址。

TCP客户端依次调用`socket()`、`connect()`之后就向TCP服务器发送了一个连接请求。

TCP服务器监听到这个请求之后，就会调用`accept()`函数取接收这个请求，这样连接就建立好了。

之后就可以开始网络I/O操作，类同于普通文件的读写I/O操作。

```c
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

参数|内容|描述
---|---|---
1|sockfd|服务器的socket描述字
2|addr|指向struct sockaddr \*的指针，用于返回**客户端的协议地址**
3|addrlen|协议地址的长度

* `accept()`的第一个参数为服务器的socket描述字，是服务器开始调用`socket()`函数生成的，称为监听socket描述字；而`accept()`返回的是已连接的socket描述字。


## 5. recvmsg() 、sendmsg()

经过上面的几个步骤，服务器与客户端已经建立好连接了。现在就可以开始调用网络I/O进行读写操作。

网络I/O操作有下面几组：

* read() / write()
* recv() / send()
* readv() / writev()
* recvmsg() / sendmsg()
* recvfrom() / sendto()

它们的声明如下：
```c
#include <unistd.h>

       ssize_t read(int fd, void *buf, size_t count);
       ssize_t write(int fd, const void *buf, size_t count);

       #include <sys/types.h>
       #include <sys/socket.h>

       ssize_t send(int sockfd, const void *buf, size_t len, int flags);
       ssize_t recv(int sockfd, void *buf, size_t len, int flags);

       ssize_t sendto(int sockfd, const void *buf, size_t len, int flags,
                      const struct sockaddr *dest_addr, socklen_t addrlen);
       ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags,
                        struct sockaddr *src_addr, socklen_t *addrlen);

       ssize_t sendmsg(int sockfd, const struct msghdr *msg, int flags);
       ssize_t recvmsg(int sockfd, struct msghdr *msg, int flags);
```

具体用法可参见man文档

## 6. close()

在服务器与客户端建立连接之后，会进行一些读写操作，完成读写操作之后要关闭相应的socket描述字。

```c
int close(int fd);
```

---

# 一个客户端与服务器通信的例子（C语言）

## 服务器端

```c
#include <sys/types.h>
#include <sys/socket.h>
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>
#include <unistd.h>
#include <netinet/in.h>
#define SERVPORT 14001
#define BACKLOG 10
#define MAX_CONNECTED_NO 10
#define MAXDATASIZE 50

int main()
{
    // 声明服务器和客户端的socket结构体
	struct sockaddr_in server_sockaddr,client_sockaddr;

    // 声明数据大小
	int sin_size,recvbytes;

    // 声明socket文件描述符
	int sockfd,client_fd;

    // 当前时间
	time_t currentTime;
	char timebuffer[MAXDATASIZE+1];

    // 缓存
	char buf[MAXDATASIZE];

    // 初始化socket
	if((sockfd = socket(AF_INET,SOCK_STREAM,0))==-1){
		perror("socket");
		exit(1);
	}
	printf("socket success!,sockfd=%d\n",sockfd);

    // 声明协议
	server_sockaddr.sin_family=AF_INET;
	server_sockaddr.sin_port=htons(SERVPORT);
	server_sockaddr.sin_addr.s_addr=INADDR_ANY;
	bzero(&(server_sockaddr.sin_zero),8);

    // 绑定
	if(bind(sockfd,(struct sockaddr *)&server_sockaddr,sizeof(struct sockaddr))==-1){
		perror("bind");
		exit(1);
	}
	printf("bind success!\n");

    // 监听
	if(listen(sockfd,BACKLOG)==-1){
		perror("listen");
		exit(1);
	}
	printf("listening....\n");

    sin_size = sizeof(struct sockaddr);

    // 接收
	if((client_fd=accept(sockfd,(struct sockaddr *)&client_sockaddr,&sin_size))==-1){
		perror("accept");
		exit(1);
	}
	currentTime = time(NULL);
	snprintf(timebuffer, MAXDATASIZE, "%s\n", ctime(&currentTime));

    // 向socket客户端写数据
	if ((recvbytes =write(client_fd, timebuffer, strlen(timebuffer))) <0 ) {
		perror("write");
		exit(1);
	}
	while (1) {
		if((recvbytes=recv(client_fd,buf,MAXDATASIZE,0))==-1){
			perror("recv");
			exit(1);
		}
		buf[recvbytes] = '\0';
	 	printf("received msg from clients :%s\n",buf);

	    currentTime = time(NULL);
	    snprintf(timebuffer, MAXDATASIZE, "%s\n", ctime(&currentTime));
		write(client_fd, timebuffer, strlen(timebuffer));
	}

    // 关闭socket
	close(sockfd);
}
```

## 客户端

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>
#include <netdb.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>
#define SERVPORT 14001
#define MAXDATASIZE 300
main(int argc,char *argv[])
{
	int sockfd,sendbytes;
	char buf[MAXDATASIZE];
	struct hostent * host;
	struct sockaddr_in serv_addr;
	if(argc < 2){
		fprintf(stderr,"Please enter the server's hostname!\n");
		exit(1);
	}
	if((host=gethostbyname(argv[1]))==NULL){
		perror("gethostbyname");
		exit(1);
	}
	if((sockfd=socket(AF_INET,SOCK_STREAM,0))==-1){
		perror("socket");
		exit(1);
	}
	serv_addr.sin_family=AF_INET;
	serv_addr.sin_port=htons(SERVPORT);
	serv_addr.sin_addr=* ((struct in_addr * )host->h_addr);
	bzero(&(serv_addr.sin_zero),8);

    // 连接
	if(connect(sockfd,(struct sockaddr *)&serv_addr,\
		sizeof(struct sockaddr))==-1){
		perror("connect");
		exit(1);
	}

    // 接收数据
	if((sendbytes=recv(sockfd,buf,MAXDATASIZE,0))==-1){
		perror("recv");
		exit(1);
	}
	buf[sendbytes] = '\0';
 	printf("received msg from server:%s\n",buf);
	while (1)
	{
        	printf("Enter the message  : ");
        	if(fgets(buf, sizeof(buf) - 1, stdin) == NULL)
        	{
        		break;
        	}
		if((sendbytes=send(sockfd,buf,strlen(buf),0))==-1){
			perror("send");
			exit(1);
		}
		printf("sent %d bytes \n", sendbytes);

		if((sendbytes=recv(sockfd,buf,MAXDATASIZE,0))==-1){
		perror("recv");
		exit(1);
	    }

		buf[sendbytes] = '\0';
		printf("received time from server:%s\n",buf);
	}

    // 关闭socket
	close(sockfd);
}
```

---

# 参考链接

* [Linux Socket编程](https://www.cnblogs.com/skynet/archive/2010/12/12/1903949.html)
* [Python应用01 原始Python服务器](http://www.cnblogs.com/vamei/archive/2012/10/30/2744955.html)
* [Linux进程间通信](http://www.cnblogs.com/vamei/archive/2012/10/10/2715398.html)
