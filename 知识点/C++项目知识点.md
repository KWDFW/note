<a name="AmKP9"></a>
# 一、整体思路
<a name="UCLLw"></a>
## 1、什么是WebServer项目
Webserver是一个软件服务器，用来接收用户的HTTP请求，并返回相应的请求的内容。
<a name="eJFbw"></a>
## 2、用户如何与WebServer进行通信
用户输入输入域名，浏览器将域名解析为IP地址，并向对应的服务器发送一个HTTP请求。<br />此过程需要经过TCP三次握手建立连接，HTTP协议生成对应的请求报文，通过TCP协议发送到服务器上。
<a name="eDtG7"></a>
## 3、服务器如何接受客户端的HTTP请求报文
服务器端通过`socket`来监听来自用户的请求。
<a name="i3FiD"></a>
### 3.1 创建监听socket
```cpp
#include <sys/socket.h>
#include <netinet/in.h>
/* 创建监听socket文件描述符 */
int listenfd = socket(PF_INET, SOCK_STREAM, 0);
/* 创建监听socket的TCP/IP的IPV4 socket地址 */
struct sockaddr_in address;
bzero(&address, sizeof(address));
address.sin_family = AF_INET;
address.sin_addr.s_addr = htonl(INADDR_ANY);  /* INADDR_ANY：将套接字绑定到所有可用的接口 */
address.sin_port = htons(port);

int flag = 1;
/* SO_REUSEADDR 允许端口被重复使用 */
setsockopt(listenfd, SOL_SOCKET, SO_REUSEADDR, &flag, sizeof(flag));
/* 绑定socket和它的地址 */
ret = bind(listenfd, (struct sockaddr*)&address, sizeof(address));  
/* 创建监听队列以存放待处理的客户连接，在这些客户连接被accept()之前 */
ret = listen(listenfd, 5);
```
<a name="G29WR"></a>
#### 3.1.1 socket
Socket（套接字）是一种在计算机网络中进行通信的方法，它是通过网络将两个不同计算机上的应用程序连接在一起，使它们能够进行双向通信。在网络编程中，Socket常常被用来实现客户端与服务器之间的通信，包括传输数据、接收数据、连接管理等。<br />使用Socket进行网络编程需要先创建一个Socket对象，通过调用该对象的方法，实现与目标计算机的连接、发送数据和接收数据等操作。
<a name="UbI81"></a>
#### 3.1.2 socket()
socket(PF_INET, SOCK_STREAM, 0) 是一个创建Socket对象的系统调用，它在网络通信中常被用来创建一个TCP套接字。<br />该函数的第一个参数PF_INET指定了使用IPv4协议进行通信，对应的常量为AF_INET。<br />第二个参数SOCK_STREAM指定了使用面向连接的TCP协议，对应的常量为SOCK_STREAM。<br />第三个参数0通常为默认参数，表示使用协议族指定的默认协议，对于IPv4协议，它会使用TCP协议。<br />该函数的返回值是一个整型的Socket文件描述符（file descriptor），用于后续的Socket操作，如果创建Socket失败，返回值为-1。在Linux系统中，Socket文件描述符是一种特殊的文件描述符，它提供了对Socket的访问控制和数据传输等功能。
<a name="IKNhO"></a>
#### 3.1.3 结构体sockaddr_in
struct sockaddr_in 是一个数据结构体，用于在C/C++中表示IPv4套接字地址。它定义在<netinet/in.h>头文件中，通常用于网络编程中，特别是在使用TCP或UDP协议时。<br />该结构体定义如下：
```cpp
struct sockaddr_in {     
	short int sin_family; // 地址族，一般为AF_INET     
	unsigned short int sin_port; // 端口号     
	struct in_addr sin_addr; // IPv4地址     	
	unsigned char sin_zero[8]; // 未使用 
};
```
其中，sin_family 表示地址族，通常使用IPv4协议，因此取值为AF_INET。<br />sin_port 表示端口号，使用网络字节序（大端字节序）表示。sin_addr 结构体表示IPv4地址，定义如下：
```cpp
struct in_addr {     
	uint32_t s_addr; // 存储IPv4地址，使用网络字节序（大端字节序）
};
```
sin_zero 是一个8字节的未使用空间，通常用0填充。<br />struct sockaddr_in 结构体主要用于在Socket编程中表示服务器地址和客户端地址，例如在调用 bind() 和 connect() 函数时需要使用。
<a name="tm7U8"></a>
#### 3.1.4 bzro()
bzero() 是一个函数，用于将一块内存区域清零，其定义在<strings.h>头文件中。bzero() 函数的原型如下：
```cpp
void bzero(void* s, size_t n);
```
该函数将 s 指向的内存区域前 n 个字节的值全部设为 0，也就是将内存清零。<br />在 bzero(&address, sizeof(address)); 中，<br />&address 表示取 address 结构体的地址,<br />sizeof(address) 表示该结构体的大小。<br />这句代码的作用是将 address 结构体清零，确保其各个字段的初始值为 0。<br />在网络编程中，清零 struct sockaddr_in 结构体通常是一种良好的编程习惯，以免在使用时发生意外的错误。
<a name="q2rTZ"></a>
#### 3.1.5 htonl(INADDR_ANY)
这段代码是用于设置一个套接字（socket）的 IP 地址。<br />其中，INADDR_ANY 是一个特殊的 IP 地址常量，它表示绑定到任何可用的网络接口上。这个常量定义在 <netinet/in.h> 头文件中，其值为 0。<br />htonl() 是一个函数，用于将一个 32 位无符号整数从主机字节序转换为网络字节序。主机字节序是指本机 CPU 的字节序，而网络字节序是指大端字节序（Big-Endian），即高位字节在前，低位字节在后。<br />因此，address.sin_addr.s_addr = htonl(INADDR_ANY) 的作用是将 INADDR_ANY 的值转换为网络字节序，然后存储在 address.sin_addr.s_addr 中，以便在绑定套接字时使用。address 是一个 struct sockaddr_in 类型的变量，它包含了 IP 地址和端口号等信息。
<a name="GszUC"></a>
#### 3.1.6 htons(port)
其中 sin_port 是一个 16 位的无符号整数，表示套接字的端口号。在使用套接字进行网络通信时，需要指定一个端口号，以便其他主机能够找到该套接字。如果端口号没有指定，系统会为套接字分配一个随机的端口号。<br />在代码中，htons() 函数将主机字节序的端口号转换为网络字节序，以保证在网络中的不同主机之间进行通信时，字节序的转换是正确的。
<a name="ttr2g"></a>
#### 3.1.7 setsocketopt(listenfd, SOL_SOCKET, SO_REUSEADDR, &flag, sizeof(flag))

- listenfd 是服务器端监听套接字的文件描述符；
- SOL_SOCKET 表示设置的选项属于套接字级别的选项，即通用套接字选项；
- SO_REUSEADDR 是一个布尔类型选项，表示在 bind 过程中可以重用已经处于 TIME_WAIT 状态的地址和端口号；
- &flag 是指向设置选项值的指针，这里将 flag 设置为非零值，表示开启 SO_REUSEADDR 选项；
- sizeof(flag) 表示设置选项值的指针的大小。

在网络编程中，服务器在绑定 IP 地址和端口号创建监听套接字时，可能会遇到因为网络延迟等原因导致之前绑定的地址和端口号仍处于 TIME_WAIT 状态，此时再次绑定同样的地址和端口号会失败。使用 SO_REUSEADDR 选项可以避免这种情况，即使之前的连接还没有完全关闭，也可以立即重新使用该地址和端口号。<br />因此，在服务器端编程中，通常会在绑定套接字之前使用 setsockopt 函数设置 SO_REUSEADDR 选项，以确保能够及时重用之前绑定的地址和端口号。
<a name="bwvj0"></a>
#### 3.1.8 bind()
bind() 函数用于将一个 IP 地址和端口号绑定到套接字上，以便服务器可以监听指定的地址和端口，并接受客户端的连接请求。函数原型如下：
```cpp
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

- sockfd：表示要绑定的套接字的文件描述符；
- addr：表示要绑定的 IP 地址和端口号，是一个指向 sockaddr 结构体的指针；
- addrlen：表示 sockaddr 结构体的长度。

在 bind() 函数中，listenfd 表示服务器监听套接字的文件描述符<br />address 表示要绑定的 IP 地址和端口号，它是一个 sockaddr_in 结构体类型的变量。<br />为了将 sockaddr_in 类型的变量转换为 sockaddr 类型的指针，需要使用强制类型转换 (struct sockaddr*)&address。<br />sizeof(address) 表示 sockaddr_in 结构体的长度，也就是要绑定的地址和端口号的长度。<br />bind() 函数的返回值是一个整数，表示函数执行的结果。如果绑定成功，返回值为 0，否则返回一个负数，表示发生了错误。在服务器端编程中，一般需要检查 bind() 函数的返回值，如果返回值小于 0，则需要根据错误码进行相应的处理。
<a name="AFuiX"></a>
#### 3.1.9 listen()
在 C++ Linux 服务器编程中，listen() 函数用于将指定的套接字设置为监听状态，以便服务器可以接受客户端的连接请求。函数原型如下：
```cpp
int listen(int sockfd, int backlog);
```
sockfd：表示要设置为监听状态的套接字的文件描述符；<br />backlog：表示服务器请求队列的最大长度。<br />在 listen() 函数中，listenfd 表示服务器监听套接字的文件描述符，backlog 表示服务器请求队列的最大长度，它表示服务器可以同时处理的客户端连接请求的最大数量。当请求队列已满时，后续的连接请求将被拒绝，客户端将收到一个 ECONNREFUSED 错误。<br />listen() 函数的返回值是一个整数，表示函数执行的结果。如果成功，返回值为 0，否则返回一个负数，表示发生了错误。在服务器端编程中，一般需要检查 listen() 函数的返回值，如果返回值小于 0，则需要根据错误码进行相应的处理。

<br /> 
