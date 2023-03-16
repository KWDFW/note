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
<a name="VF84P"></a>
### 3.2 通过epoll来实现对于监听socket和connect的同时监听
| ```cpp
#include <sys/epoll.h>
/* 将fd上的EPOLLIN和EPOLLET事件注册到epollfd指示的epoll内核事件中 */
void addfd(int epollfd, int fd, bool one_shot) {
    epoll_event event;
    event.data.fd = fd;
    event.events = EPOLLIN | EPOLLET | EPOLLRDHUP;
    /* 针对connfd，开启EPOLLONESHOT，因为我们希望每个socket在任意时刻都只被一个线程处理 */
    if(one_shot)
        event.events |= EPOLLONESHOT;
    epoll_ctl(epollfd, EPOLL_CTL_ADD, fd, &event);
    setnonblocking(fd);
}
/* 创建一个额外的文件描述符来唯一标识内核中的epoll事件表 */
int epollfd = epoll_create(5);  
/* 用于存储epoll事件表中就绪事件的event数组 */
epoll_event events[MAX_EVENT_NUMBER];  
/* 主线程往epoll内核事件表中注册监听socket事件，当listen到新的客户连接时，listenfd变为就绪事件 */
addfd(epollfd, listenfd, false);  
/* 主线程调用epoll_wait等待一组文件描述符上的事件，并将当前所有就绪的epoll_event复制到events数组中 */
int number = epoll_wait(epollfd, events, MAX_EVENT_NUMBER, -1);
/* 然后我们遍历这一数组以处理这些已经就绪的事件 */
for(int i = 0; i < number; ++i) {
    int sockfd = events[i].data.fd;  // 事件表中就绪的socket文件描述符
    if(sockfd == listenfd) {  // 当listen到新的用户连接，listenfd上则产生就绪事件
        struct sockaddr_in client_address;
        socklen_t client_addrlength = sizeof(client_address);
        /* ET模式 */
        while(1) {
            /* accept()返回一个新的socket文件描述符用于send()和recv() */
            int connfd = accept(listenfd, (struct sockaddr *) &client_address, &client_addrlength);
            /* 并将connfd注册到内核事件表中 */
            users[connfd].init(connfd, client_address);
            /* ... */
        }
    }
    else if(events[i].events & (EPOLLRDHUP | EPOLLHUP | EPOLLERR)) {
        // 如有异常，则直接关闭客户连接，并删除该用户的timer
        /* ... */
    }
    else if(events[i].events & EPOLLIN) {
        /* 当这一sockfd上有可读事件时，epoll_wait通知主线程。*/
        if(users[sockfd].read()) { /* 主线程从这一sockfd循环读取数据, 直到没有更多数据可读 */
            pool->append(users + sockfd);  /* 然后将读取到的数据封装成一个请求对象并插入请求队列 */
            /* ... */
        }
        else
            /* ... */
            }
    else if(events[i].events & EPOLLOUT) {
        /* 当这一sockfd上有可写事件时，epoll_wait通知主线程。主线程往socket上写入服务器处理客户请求的结果 */
        if(users[sockfd].write()) {
            /* ... */
        }
        else
            /* ... */
            }
}
```
 |
| --- |

<a name="o3UQE"></a>
#### 3.2.1 epoll
I/O 事件通知机制是指操作系统提供的一种机制，通过这种机制，可以让应用程序监听多个文件描述符（FD）上的 I/O 事件，以便及时地响应这些事件。<br />epoll 就是多路复用 I/O 模型中的一种实现。它通过在内核中创建一个红黑树（或者一个双向链表），将所有需要监听的文件描述符都挂在这个红黑树上，当有 I/O 事件到达时，只需要取出发生事件的文件描述符即可，不需要遍历整个 FD 集合，这样可以大大提高效率。<br />在 epoll 中，可以使用两种模式来监听文件描述符：水平触发（Level Triggered，即默认模式）和边缘触发（Edge Triggered）。在水平触发模式下，当一个文件描述符上的 I/O 事件被处理后，如果这个文件描述符上还有未处理完的数据，那么下次该文件描述符上的 I/O 事件到来时，操作系统还会通知应用程序；而在边缘触发模式下，当一个文件描述符上的 I/O 事件被处理后，如果这个文件描述符上还有未处理完的数据，那么下次该文件描述符上的 I/O 事件到来时，操作系统不会再次通知应用程序，需要应用程序自己去轮询文件描述符上的数据是否全部处理完毕。
<a name="hRvtO"></a>
#### 3.2.2 文件描述符
在 Linux 中，所有的 I/O 操作都是通过文件描述符（File Descriptor，FD）来实现的。文件描述符是一个非负整数，用来标识一个打开的文件或者一个网络连接。<br />在程序中，当需要访问一个文件或者进行网络通信时，首先需要通过打开该文件或者建立该连接来获得一个文件描述符。然后，通过操作该文件描述符，程序就可以进行读写、关闭等操作。<br />在 Linux 中，文件描述符是由内核管理的，每个进程都有自己独立的一套文件描述符。当一个进程打开一个文件或者建立一个网络连接时，操作系统会为该进程分配一个未使用的文件描述符，并返回该文件描述符给进程。程序可以使用该文件描述符对文件或网络连接进行操作。当程序不再需要使用该文件描述符时，应该及时关闭该文件描述符，释放系统资源。
<a name="yvEjN"></a>
#### 3.2.3 addfd()
该函数是一个添加文件描述符到 epoll 中的函数，函数名为 addfd，函数的参数包括 epollfd、fd 和 one_shot。<br />epollfd：指向 epoll 实例的文件描述符。<br />fd：需要添加到 epoll 实例中的文件描述符。<br />one_shot：一个布尔值，表示是否启用 EPOLLONESHOT 事件，一般情况下为 true。<br />在函数中，首先创建一个 epoll_event 结构体 event，用于描述需要添加到 epoll 实例中的文件描述符的事件类型和数据。然后设置该结构体的成员变量：<br />event.data.fd：设置为需要添加到 epoll 实例中的文件描述符。<br />event.events：设置为需要监听的事件类型，包括 EPOLLIN（可读事件）、EPOLLET（边缘触发模式）和 EPOLLRDHUP（对端关闭连接事件）。<br />如果 one_shot 为 true，则再设置 event.events 为 EPOLLONESHOT，表示开启 EPOLLONESHOT 事件。EPOLLONESHOT 事件是指在一个 socket 被处理之前，其他线程不能同时处理该 socket，确保同一时刻只有一个线程处理该 socket。<br />最后，使用 epoll_ctl 函数将文件描述符添加到 epoll 实例中，并设置为非阻塞模式（使用 setnonblocking 函数）。
<a name="LJq9z"></a>
##### 3.2.3.1 epoll_ctl
epoll_ctl 是一个系统调用函数，用于向 epoll 实例中添加、修改或删除一个文件描述符和对应的事件。它的原型如下：
```cpp
csharpCopy code
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```
其中：<br />epfd：是一个 epoll 实例的文件描述符。<br />op：表示 epoll_ctl 执行的操作，有三个值：EPOLL_CTL_ADD、EPOLL_CTL_MOD 和 EPOLL_CTL_DEL，分别用于添加、修改和删除一个文件描述符。<br />fd：表示需要添加、修改或删除的文件描述符。<br />event：是一个 epoll_event 结构体，用于描述需要添加、修改或删除的文件描述符的事件类型和数据。<br />在使用 epoll_ctl 函数时，需要注意以下几点：<br />如果 op 为 EPOLL_CTL_ADD，则 epoll_ctl 会将 fd 和 event 添加到 epoll 实例中，如果该 fd 已经在 epoll 实例中存在，则会返回 EEXIST 错误。<br />如果 op 为 EPOLL_CTL_MOD，则 epoll_ctl 会修改 epoll 实例中已经存在的 fd 对应的事件为 event，如果该 fd 在 epoll 实例中不存在，则会返回 ENOENT 错误。<br />如果 op 为 EPOLL_CTL_DEL，则 epoll_ctl 会删除 epoll 实例中已经存在的 fd，如果该 fd 在 epoll 实例中不存在，则会返回 ENOENT 错误。<br />epoll_ctl 函数可以方便地向 epoll 实例中添加、修改或删除一个文件描述符和对应的事件，使得程序可以高效地监听多个文件描述符上的事件，并及时响应事件。
<a name="DSbwl"></a>
##### 3.2.3.2 event
在使用 epoll 系统调用时，我们需要使用 epoll_ctl 函数向 epoll 实例中注册需要监听的文件描述符及其关注的事件类型。而 event 结构体就是用来描述一个文件描述符及其关注的事件类型的。<br />event 结构体定义如下：
```cpp
cCopy code
struct epoll_event {     uint32_t events;  // 表示需要关注的事件类型     epoll_data_t data;  // 表示需要关注的数据 };
```
其中：<br />events 是需要关注的事件类型，可以包括以下几种类型：<br />EPOLLIN：表示对应的文件描述符可以读取。<br />EPOLLOUT：表示对应的文件描述符可以写入。<br />EPOLLRDHUP：表示对端关闭连接或半关闭连接。<br />EPOLLPRI：表示有紧急数据可读。<br />EPOLLERR：表示对应的文件描述符发生错误。<br />EPOLLHUP：表示对应的文件描述符被挂起。<br />EPOLLET：表示开启边缘触发模式。<br />EPOLLONESHOT：表示一次事件被触发后，对应的文件描述符将从 epoll 实例中删除。<br />data 是需要关注的数据，通常包含一个指向数据结构的指针或一个整数值。在事件触发时，内核会返回该 data 字段的值，以便用户程序能够知道哪个文件描述符上触发了事件。<br />因此，我们在使用 epoll 系统调用时，需要先构造一个 epoll_event 结构体，然后使用 epoll_ctl 函数将其添加到 epoll 实例中。在 epoll_wait 函数返回时，我们可以从返回的 epoll_event 结构体数组中读取每个事件对应的文件描述符和事件类型，以便进行相应的处理。
<a name="jaZPa"></a>
##### 3.2.3.3 setnonblocking()
在 Linux 中，每个文件描述符都有一个阻塞模式和非阻塞模式。在阻塞模式下，当我们对一个文件描述符进行 I/O 操作时，如果该操作无法立即完成，进程将会被阻塞，直到 I/O 操作完成为止。这意味着在阻塞模式下，如果我们需要同时处理多个文件描述符，我们需要为每个文件描述符创建一个线程，以便在 I/O 操作时不会被其他文件描述符的 I/O 操作所阻塞。<br />相比之下，非阻塞模式下的文件描述符在执行 I/O 操作时不会阻塞进程，而是立即返回一个错误码。因此，我们可以使用单个线程处理多个文件描述符，以便在进行 I/O 操作时不会被其他文件描述符的 I/O 操作所阻塞。<br />setnonblocking 函数是一个设置文件描述符为非阻塞模式的函数，通常会在程序初始化时被调用。在网络编程中，将套接字设置为非阻塞模式是非常常见的操作，以便能够在等待客户端连接或读取数据时不会阻塞程序的执行。<br />设置文件描述符为非阻塞模式有以下几个好处：<br />1提高程序的并发性能<br />在阻塞模式下，当一个文件描述符正在执行 I/O 操作时，如果该操作无法立即完成，进程将会被阻塞，直到 I/O 操作完成为止。这意味着在阻塞模式下，如果我们需要同时处理多个文件描述符，我们需要为每个文件描述符创建一个线程，以便在 I/O 操作时不会被其他文件描述符的 I/O 操作所阻塞。而在非阻塞模式下，单个线程可以轮询多个文件描述符，当发现某个文件描述符有 I/O 事件时，即可立即处理该事件，而不会被其他文件描述符的 I/O 操作所阻塞，从而提高了程序的并发性能。<br />2避免进程陷入永久阻塞<br />在阻塞模式下，当一个文件描述符正在执行 I/O 操作时，如果该操作无法立即完成，进程将会被阻塞，直到 I/O 操作完成为止。如果出现了某个文件描述符上的 I/O 操作永久阻塞的情况，那么整个进程都会被阻塞，从而导致整个应用程序无法响应。而在非阻塞模式下，当发现某个文件描述符的 I/O 操作无法立即完成时，进程会立即返回一个错误码，从而避免了进程陷入永久阻塞的风险。<br />3方便实现超时机制<br />在非阻塞模式下，当一个文件描述符的 I/O 操作无法立即完成时，进程会立即返回一个错误码。这使得我们可以方便地实现超时机制，即在一定时间内等待文件描述符的 I/O 事件发生，如果超过了指定的时间仍然没有事件发生，就可以进行其他处理，从而避免了一直等待的情况。
<a name="J0ES4"></a>
#### 3.2.4 epoll_create()
在这段代码中，使用 epoll_create 函数创建了一个新的 epoll 实例，该实例将被用于存储所有的文件描述符和 I/O 事件。参数 5 指定了 epoll 实例所能处理的最大文件描述符数量（这个参数在Linux 2.6.8之后已经被忽略，但是需要传递一个大于0的值，一般传入5即可）。<br />创建 epoll 实例后，系统会为该实例分配一块内存用于存储事件列表（即 epoll 事件表），同时返回一个新的文件描述符，该文件描述符用于操作 epoll 实例。<br />使用 epoll 机制的步骤大概如下：<br />使用 epoll_create 函数创建一个 epoll 实例，并得到一个 epoll 文件描述符。<br />使用 epoll_ctl 函数向 epoll 实例注册文件描述符，指定需要监听的事件类型（如读事件、写事件等）。	<br />使用 epoll_wait 函数等待事件的发生，并得到所有已经发生的事件。
<a name="M4gjj"></a>
#### 3.2.5 epoll_wait()
epoll_wait() 是使用 epoll 进行 I/O 事件监听的核心函数之一。它的主要作用是等待事件的发生，并返回已经发生的事件。<br />该函数的原型如下：
```cpp
cCopy code
int epoll_wait(int epollfd, struct epoll_event *events, int maxevents, int timeout);
```
其中：<br />epollfd：是 epoll 的文件描述符。<br />events：是用于存储发生事件的数组。<br />maxevents：是事件数组 events 的大小。<br />timeout：是等待超时时间，单位为毫秒，如果为 -1，表示无限等待，直到有事件发生。<br />调用 epoll_wait() 函数会一直阻塞，直到事件发生或者超时，如果有事件发生，就会把事件写入到 events 数组中，函数返回已经发生事件的文件描述符数量，如果发生错误，返回 -1。<br />对于返回的每一个事件，都包含了以下成员：<br />events：指出了哪些事件已经发生，如可读事件、可写事件等等。<br />data：这个字段的类型是 union epoll_data，它的值是注册时所指定的文件描述符或指针等等。<br />一般来说，在使用 epoll 时，可以将 epoll_wait() 放在一个循环中，以等待多个事件的发生。当事件发生时，可以通过判断 events[i].events 来确定具体是什么类型的事件，然后根据相应的事件类型进行处理。<br />epoll_wait() 的返回值是一个整数，表示已经发生的事件数目。具体来说，返回值的含义为：<br />若返回值大于 0，则表示有文件描述符上有 I/O 事件发生，返回值就是这些事件的数目。<br />若返回值等于 0，则表示在等待的超时时间 timeout 内没有任何 I/O 事件发生。<br />若返回值等于 -1，则表示 epoll_wait() 函数出错，此时需要查看 errno 来确定错误的原因。<br />当 epoll_wait() 函数返回后，可以遍历 events 数组来处理发生的 I/O 事件，数组中的每个元素表示一个发生的事件，包括发生事件的文件描述符和事件类型等信息。
<a name="jwGM8"></a>
#### 3.2.6 if()中的条件
这两行代码是用来处理 listenfd 上的就绪事件的。当有新的客户端连接时，listenfd 就会产生可读事件，此时 epoll_wait() 函数返回，返回的 epoll_event 结构体中的 data.fd 就是 listenfd，因此代码中的 sockfd 就等于 listenfd。<br />这里的就绪事件指的是文件描述符上的 I/O 事件已经就绪，可以进行读写等操作了。在这个例子中，listenfd 用于监听新的客户端连接，因此它的就绪事件就表示有新的客户端连接请求到达了，此时程序需要通过 accept() 函数来处理新连接请求。<br />需要注意的是，这里的 listenfd 是用 epoll_ctl() 函数注册到 epoll 实例中的，设置为关注 EPOLLIN 事件的文件描述符，因此只有当 listenfd 上有可读事件时才会产生就绪事件，调用 epoll_wait() 函数才会返回。而在事件就绪之前，程序将阻塞在 epoll_wait() 函数上，等待事件就绪。<br />如果该条件不成立则说明已经连接的客户端有数据到达，需要处理这些数据。
<a name="KnoQG"></a>
#### 3.2.7 存储客户端地址信息
这段代码定义了一个 sockaddr_in 类型的结构体变量 client_address，用于存储客户端的地址信息。同时，client_addrlength 变量用于存储客户端地址结构体的长度。<br />在 Linux 系统中，通过 accept() 函数接受新的客户端连接时，需要传入一个指向 sockaddr 类型的指针和一个指向 socklen_t 类型的整数指针。sockaddr 类型的结构体用于存储客户端的地址信息，socklen_t 类型的整数指针用于传入 sockaddr 结构体的长度，以便 accept() 函数能够正确地读取地址信息。<br />因此，在这段代码中，client_address 结构体变量用于存储客户端的地址信息，client_addrlength 变量用于传递客户端地址结构体的长度。
<a name="K7FWY"></a>
#### 3.2.8 accept()
用于接受客户端连接的函数，其原型如下：
```cpp
c++Copy code
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```
其中：<br />sockfd：表示服务器的套接字描述符，通过调用 socket() 函数创建得到。<br />addr：用于存储客户端的地址信息，类型为 struct sockaddr。<br />addrlen：表示 addr 结构体的长度。<br />accept() 函数的作用是阻塞等待客户端的连接请求，一旦有客户端连接请求到来，accept() 函数会创建一个新的套接字描述符，并返回该套接字描述符。通过该套接字描述符可以与客户端进行通信。<br />注意：<br />accept() 函数是一个阻塞函数，直到有客户端连接请求到来才会返回。<br />accept() 函数返回的套接字描述符是新的，与原来的服务器套接字描述符不同。<br />在多线程或多进程服务器中，每个线程或进程应该有自己的套接字描述符，即每个线程或进程都应该调用 accept() 函数。
<a name="xAjeW"></a>
#### 3.2.9 init()
在库http_conn里面实现的
<a name="ZiBh9"></a>
#### 3.2.10 events事件的不同类型
EPOLLRDHUP 表示套接字的远程端关闭了连接，即 "TCP 连接的一端已经关闭了它的写操作（half-close）"；<br />EPOLLHUP 表示套接字发生了错误，且该错误通常由对端异常关闭连接引起；<br />EPOLLERR 表示套接字发生了错误，且该错误通常不是由对端关闭引起的。<br />EPOLLIN：表示可读事件。当某个套接字上有数据可以读取时，会触发该事件。<br />EPOLLOUT：表示可写事件。当某个套接字可以写入数据时，会触发该事件。
<a name="OCrfN"></a>
## 4、服务器如何处理及响应请求报文
该项目使用线程池（半同步半反应堆模式）并发处理用户请求，主线程负责读写，工作线程（线程池中的线程）负责处理逻辑（HTTP请求报文的解析等等）。通过之前的代码，我们将listenfd上到达的connection通过 accept()接收，并返回一个新的socket文件描述符connfd用于和用户通信，并对用户请求返回响应，同时将这个connfd注册到内核事件表中，等用户发来请求报文。这个过程是：通过epoll_wait发现这个connfd上有可读事件了（EPOLLIN），主线程就将这个HTTP的请求报文读进这个连接socket的读缓存中users[sockfd].read()，然后将该任务对象（指针）插入线程池的请求队列中pool->append(users + sockfd);，线程池的实现还需要依靠锁机制以及信号量机制来实现线程同步，保证操作的原子性。<br />process_read()函数的作用就是将类似上述例子的请求报文进行解析，因为用户的请求内容包含在这个请求报文里面，只有通过解析，知道用户请求的内容是什么，是请求图片，还是视频，或是其他请求，我们根据这些请求返回相应的HTML页面等。项目中使用主从状态机的模式进行解析，从状态机（parse_line）负责读取报文的一行，主状态机负责对该行数据进行解析，主状态机内部调用从状态机，从状态机驱动主状态机。每解析一部分都会将整个请求的m_check_state状态改变，状态机也就是根据这个状态来进行不同部分的解析跳转的<br />经过上述解析，当得到一个完整的，正确的HTTP请求时，就到了do_request代码部分，我们需要首先对GET请求和不同POST请求（登录，注册，请求图片，视频等等）做不同的预处理，然后分析目标文件的属性，若目标文件存在、对所有用户可读且不是目录时，则使用mmap将其映射到内存地址m_file_address处，并告诉调用者获取文件成功。
<a name="u2ZTt"></a>
### 4.1 read())
| ```cpp
void http_conn::process() {
    HTTP_CODE read_ret = process_read();
    if(read_ret == NO_REQUEST) {
        modfd(m_epollfd, m_sockfd, EPOLLIN);
        return;
    }
    bool write_ret = process_write(read_ret);
    if(!write_ret)
        close_conn();
    modfd(m_epollfd, m_sockfd, EPOLLOUT);
}
```
 |
| --- |

<a markdown="1" name="process"></a>

<a name="hVo8f"></a>
#### 4.1.1 process_read()
对发送过来的报文进行解析，获取需要知道的信息
<a name="xtaD2"></a>
#### 4.1.2 NO_REQUEST
如果 process_read 返回的结果是 NO_REQUEST，说明当前并没有完整的 HTTP 请求可以处理，那么就调用 modfd 函数，将当前的套接字（m_sockfd）添加到 epoll 实例（m_epollfd）中，监听该套接字的读事件（EPOLLIN），然后退出函数。<br />在使用 epoll 实现高并发网络服务器时，我们通常采用边缘触发模式（EPOLLET），这种模式下，每个事件只会被触发一次，需要在处理完事件后重新向 epoll 实例注册相应的事件。<br />在上述代码中，当 process_read 函数返回 NO_REQUEST 时，说明当前套接字上并没有可读事件，因此我们需要将其重新加入 epoll 实例中，等待下一次读事件的到来。为了保证正确性，我们只需要监听该套接字的读事件（EPOLLIN），而不需要监听写事件（EPOLLOUT）或其他事件。<br />因此，我们调用 modfd 函数，将 m_sockfd 套接字重新注册到 m_epollfd epoll 实例中，并设置监听事件为 EPOLLIN。这样，当下一次有数据可读时，epoll_wait 函数会返回该套接字的读事件，从而触发对应的事件处理函数。
<a name="zwxkV"></a>
#### 4.1.3 process_write()
如果 process_read 返回的结果不是 NO_REQUEST，说明当前已经有完整的 HTTP 请求可以处理，那么就调用 process_write 函数，该函数用于生成 HTTP 响应并发送给客户端。如果 process_write 函数返回 false，说明发送失败，那么就调用 close_conn 函数关闭连接。<br />如果发送成功，就调用 modfd 函数，将当前的套接字（m_sockfd）添加到 epoll 实例（m_epollfd）中，监听该套接字的写事件（EPOLLOUT）。这样，就可以等待客户端发送新的 HTTP 请求，或者等待当前的 HTTP 响应发送完毕。



<a name="PncnL"></a>
## 5、数据库连接池是如何运行的
数据库连接是什么：数据库连接是指应用程序和数据库之间的通信通道，应用程序通过这个通道与数据库进行交互，包括发送查询语句、获取查询结果等。当应用程序需要访问数据库时，它需要先建立一个数据库连接，然后才能向数据库发送请求。<br />在处理用户注册，登录请求的时候，我们需要将这些用户的用户名和密码保存下来用于新用户的注册及老用户的登录校验，相信每个人都体验过，当你在一个网站上注册一个用户时，应该经常会遇到“您的用户名已被使用”，或者在登录的时候输错密码了网页会提示你“您输入的用户名或密码有误”等等类似情况，这种功能是服务器端通过用户键入的用户名密码和数据库中已记录下来的用户名密码数据进行校验实现的。若每次用户请求我们都需要新建一个数据库连接，请求结束后我们释放该数据库连接，当用户请求连接过多时，这种做法过于低效，所以类似**线程池**的做法，我们构建一个数据库连接池，预先生成一些数据库连接放在那里供用户请求使用。<br />(找不到mysql/mysql.h头文件的时候，需要安装一个库文件：sudo apt install libmysqlclient-dev)<br />我们首先看单个数据库连接是如何生成的：

1. 使用mysql_init()初始化连接
2. 使用mysql_real_connect()建立一个到mysql数据库的连接
3. 使用mysql_query()执行查询语句
4. 使用result = mysql_store_result(mysql)获取结果集
5. 使用mysql_num_fields(result)获取查询的列数，mysql_num_rows(result)获取结果集的行数
6. 通过mysql_fetch_row(result)不断获取下一行，然后循环输出
7. 使用mysql_free_result(result)释放结果集所占内存
8. 使用mysql_close(conn)关闭连接

对于一个数据库连接池来讲，就是预先生成多个这样的数据库连接，然后放在一个链表中，同时维护最大连接数MAX_CONN，当前可用连接数FREE_CONN和当前已用连接数CUR_CONN这三个变量。同样注意在对连接池操作时（获取，释放），要用到锁机制，因为它被所有线程共享。
<a name="mikvG"></a>
## 6、什么是CGI校验
对用户的登录及注册等POST请求，服务器是如何做校验的。当点击新用户按钮时，服务器对这个POST请求的响应是：返回用户一个登录界面；当你在用户名和密码框中输入后，你的POST请求报文中会连同你的用户名密码一起发给服务器，然后我们拿着你的用户名和密码在数据库连接池中取出一个连接用于mysql_query()进行查询，逻辑很简单，同步线程校验SYNSQL方式相信大家都能明白，但是这里社长又给出了其他两种校验方式，CGI什么的，就很容易让小白一时摸不到头脑，接下来就简单解释一下CGI是什么。<br />CGI（通用网关接口），它是一个运行在Web服务器上的程序，在编译的时候将相应的.cpp文件编程成.cgi文件并在主程序中调用即可（通过社长的makefile文件内容也可以看出）。这些CGI程序通常通过客户在其浏览器上点击一个button时运行。这些程序通常用来执行一些信息搜索、存储等任务，而且通常会生成一个动态的HTML网页来响应客户的HTTP请求。我们可以发现项目中的sign.cpp文件就是我们的CGI程序，将用户请求中的用户名和密码保存在一个id_passwd.txt文件中，通过将数据库中的用户名和密码存到一个map中用于校验。在主程序中通过execl(m_real_file, &flag, name, password, NULL);这句命令来执行这个CGI文件，这里CGI程序仅用于校验，并未直接返回给用户响应。这个CGI程序的运行通过多进程来实现，根据其返回结果判断校验结果（使用pipe进行父子进程的通信，子进程将校验结果写到pipe的写端，父进程在读端读取）。
<a name="CTqBU"></a>
## 7、生成HTTP请求并返回给用户
通过以上操作，我们已经对读到的请求做好了处理，然后也对目标文件的属性作了分析，若目标文件存在、对所有用户可读且不是目录时，则使用mmap将其映射到内存地址m_file_address处，并告诉调用者获取文件成功FILE_REQUEST。 接下来要做的就是根据读取结果对用户做出响应了，也就是到了process_write(read_ret);这一步，该函数根据process_read()的返回结果来判断应该返回给用户什么响应，我们最常见的就是404错误了，说明客户请求的文件不存在，除此之外还有其他类型的请求出错的响应，具体的可以去百度。然后呢，假设用户请求的文件存在，而且已经被mmap到m_file_address这里了，那么我们就将做如下写操作，将响应写到这个connfd的写缓存m_write_buf中去
<a name="FMkvS"></a>
### 7.1 写操作
| ```cpp
case FILE_REQUEST: {
    add_status_line(200, ok_200_title);
    if(m_file_stat.st_size != 0) {
        add_headers(m_file_stat.st_size);
        m_iv[0].iov_base = m_write_buf;
        m_iv[0].iov_len = m_write_idx;
        m_iv[1].iov_base = m_file_address;
        m_iv[1].iov_len = m_file_stat.st_size;
        m_iv_count = 2;
        bytes_to_send = m_write_idx + m_file_stat.st_size;
        return true;
    }
    else {
        const char* ok_string = "<html><body></body></html>";
        add_headers(strlen(ok_string));
        if(!add_content(ok_string))
            return false;
    }
}
```
 |
| --- |

<a name="ISjLs"></a>
#### 7.1.1 add_status_line()
add_status_line(200, ok_200_title) 是用来添加 HTTP 响应报文中状态行的函数。HTTP 响应报文包含三部分：状态行、响应头和响应体。状态行是 HTTP 响应报文的第一行，用来指定服务器的响应状态，包括响应码和原因短语。HTTP 响应状态码是三位数字，用来表示服务器响应的状态，例如 200 表示请求成功，而 404 表示请求的资源未找到。ok_200_title 是一个字符串，表示响应码为 200 时的原因短语，即 OK。<br />因此，add_status_line(200, ok_200_title) 的作用是添加状态行，将状态码和原因短语添加到 HTTP 响应报文的状态行中，使客户端能够了解服务器对请求的处理结果。
<a name="deTUU"></a>
#### 7.1.2 m_file_stat.st_size
m_file_stat.st_size 是一个 Linux 系统下的 C++ 函数或结构体成员，用于获取文件的大小，即文件字节数。<br />具体来说，m_file_stat 是一个 struct stat 类型的结构体变量，用于存储文件的元数据信息，包括文件类型、文件权限、文件所有者、文件大小、最后一次访问时间等。st_size 是 struct stat 结构体中的一个成员变量，用于存储文件的大小，单位为字节。<br />在服务器开发中，可以使用 m_file_stat.st_size 来获取服务器上某个文件的大小，进而判断该文件是否可以进行下载或者进行其他的操作。例如，在实现文件下载功能时，需要获取文件的大小以计算文件下载进度或者限制文件下载的大小等。

<a name="gdO4H"></a>
#### 7.1.3 add_headers()
add_headers(m_file_stat.st_size) 的作用是向 HTTP 响应报文中添加响应头部分的内容，其中包括标准的 HTTP 头部信息和自定义的 HTTP 头部信息。而通过添加 Content-Length 头部信息，可以让客户端知道要接收的数据量，从而确保传输的完整性。

<a name="WSNO6"></a>
#### 7.1.4 if语句
用于处理文件请求并构建 HTTP 响应报文的函数中的一部分代码。具体来说，这段代码用于判断请求的文件是否为空，如果不为空，则向 HTTP 响应报文中添加响应头部分的内容，并将文件内容写入响应报文中，以便发送给客户端。<br />代码中的 if 语句判断文件的大小是否为 0。如果文件不为空，则调用 add_headers(m_file_stat.st_size) 函数向 HTTP 响应报文中添加响应头部分的内容，包括 Content-Length 头部信息等。然后，使用 struct iovec 类型的变量 m_iv 来构建响应报文的消息体。m_iv[0] 表示要发送的数据的头部，即 m_write_buf 缓冲区中的数据，而 m_iv[1] 表示要发送的数据的主体，即文件的内容。其中，m_iv[0].iov_base 和 m_iv[1].iov_base 分别指向 m_write_buf 和 m_file_address，表示数据的起始地址，而 m_iv[0].iov_len 和 m_iv[1].iov_len 分别为数据的长度。m_iv_count 变量表示 iov 数组中元素的个数，即 2。<br />最后，代码中的 bytes_to_send 变量表示要发送的数据的总长度，即 m_write_buf 缓冲区中的数据和文件的大小之和。在此之后，代码会将构建好的响应报文发送给客户端，完成文件下载的功能。
<a name="yg49c"></a>
#### 7.1.5 else语句
如果请求的文件为空，代码将会返回一个 HTTP 200 OK 响应，但是不会向客户端发送任何文件内容。具体实现是，首先定义一个 ok_string 字符串，表示要返回的 HTTP 正文内容。然后，调用 add_headers 函数向 HTTP 响应报文中添加响应头部分的内容，包括 Content-Length 头部信息等。接着，调用 add_content 函数向 HTTP 响应报文中添加响应正文部分的内容，即 ok_string 字符串中的内容。如果添加成功，则返回 true，否则返回 false。<br />这段代码的作用是在处理文件为空的情况下，向客户端返回一个 HTTP 200 OK 响应，并在响应正文部分中返回一个空的 HTML 页面，以告知客户端请求已成功完成，但是没有文件内容需要返回。如果请求的文件不为空，则会跳过这段代码，并执行前面提到的将文件内容写入响应报文中的代码。

<a name="JiHy3"></a>
## 8、定时器处理非活动连接
<a name="Vkitf"></a>
### 8.1 定时器相关参数
| ```cpp
/* 定时器相关参数 */
static int pipefd[2];
static sort_timer_lst timer_lst

/* 每个user（http请求）对应的timer */
client_data* user_timer = new client_data[MAX_FD];
/* 每隔TIMESLOT时间触发SIGALRM信号 */
alarm(TIMESLOT);
/* 创建管道，注册pipefd[0]上的可读事件 */
int ret = socketpair(PF_UNIX, SOCK_STREAM, 0, pipefd);
/* 设置管道写端为非阻塞 */
setnonblocking(pipefd[1]);
/* 设置管道读端为ET非阻塞，并添加到epoll内核事件表 */
addfd(epollfd, pipefd[0], false);

addsig(SIGALRM, sig_handler, false);
addsig(SIGTERM, sig_handler, false);
```
 |
| --- |

<a name="ja5az"></a>
### 8.2 定时器主循环
| ```cpp
/* 处理信号 */
else if(sockfd == pipefd[0] && (events[i].events & EPOLLIN)) {
    int sig;
    char signals[1024];
    ret = recv(pipefd[0], signals, sizeof(signals), 0);
    if(ret == -1) {
        continue;  // handle the error
    }
    else if(ret == 0) {
        continue;
    }
    else {
        for(int j = 0; j < ret; ++j) {
            switch (signals[j]) {
                case SIGALRM: {
                    timeout = true;
                    break;
                }
                case SIGTERM: {
                    stop_server = true;
                }
            }
        }
    }
}
```
 |
| --- |

 
<a name="ZgMwM"></a>
## 9、日志

<a name="AP0DO"></a>
## 10、压测
<a name="ITZmo"></a>
### 10.1 运行服务
找到项目文件夹<br />make server<br />./server port(自定义端口号)<br />浏览器端：ip:port
<a name="LfpAV"></a>
### 10.2 压力测试
找到压力测试文件夹<br />./webbench -c 10001 -t 5 ip:port

 

<a name="YIefi"></a>
# 二、线程同步机制封装类
目的：实现多线程同步，通过锁机制，确保任一时刻只能有一个线程能进入关键代码段。
<a name="cNvMS"></a>
## 1、封装信号量
```cpp
class sem{
    public:
    //构造函数
    sem()
    {
        //信号量初始化
        if(sem_init(&m_sem,0,0)!=0){
            throw std::exception();
        }
    }
    //析构函数
    ~sem()
    {
        //信号量销毁
        sem_destroy(&m_sem);
    }
    private:
    sem_t m_sem;
};
```
<a name="lMtZB"></a>
### 1.1 总结
这是一个名为sem的类，其中包含一个信号量变量m_sem。该类定义了一个构造函数和一个析构函数。<br />构造函数用于初始化信号量m_sem，它使用了sem_init函数来初始化信号量。如果初始化失败，则抛出了一个std::exception异常。<br />析构函数用于销毁信号量m_sem，它使用了sem_destroy函数来销毁信号量。<br />该类的作用是封装了信号量的操作，通过使用该类的对象，可以方便地实现信号量的使用和管理。同时，由于构造函数和析构函数的存在，可以确保在对象创建和销毁时，信号量的状态正确地被初始化和销毁，避免了资源泄露和内存泄露等问题。
<a name="v1KtC"></a>
### 1.2 构造与析构
在C++中，构造函数和析构函数是两个特殊的成员函数，用于对象的初始化和清理。<br />**构造函数（Constructor）**<br />构造函数是一种特殊的成员函数，其名称与类名相同，没有返回类型，也不需要显式调用。当创建一个对象时，构造函数会被自动调用，用于初始化对象的数据成员。<br />构造函数有以下几种特性：

   - 构造函数的访问权限可以是public、private或protected。
   - 构造函数可以有参数，也可以没有参数。
   - 构造函数可以有多个，但是参数列表必须不同。
   - 构造函数可以使用成员初始化列表（member initialization list）来初始化成员变量，也可以在函数体中进行初始化。

构造函数的作用是初始化对象的状态，保证对象在创建后可以正常使用。<br />**析构函数（Destructor）**<br />析构函数是一种特殊的成员函数，其名称与类名相同，前面加上一个波浪号（~），没有返回类型，也不需要显式调用。当对象被销毁时，析构函数会自动调用，用于清理对象的资源。<br />析构函数有以下几种特性：

   - 析构函数的访问权限可以是public、private或protected。
   - 每个类只能有一个析构函数，不能有参数，也不能重载。
   - 析构函数不需要显式调用，在对象被销毁时会自动调用。
   - 析构函数的作用是释放对象的资源，包括动态分配的内存、打开的文件、建立的网络连接等，防止资源泄露和内存泄露。

总的来说，构造函数用于初始化对象的状态，而析构函数用于清理对象的资源，两者都是C++中重要的特性，有助于提高程序的可靠性和健壮性。
<a name="pP4N5"></a>
### 1.3 sem_init()
这句话是在使用sem_init函数初始化信号量m_sem的过程中进行错误检查。sem_init函数用于初始化一个命名或未命名的信号量，其参数依次为：<br />第一个参数为指向要初始化的信号量的指针。<br />第二个参数为0或非0，表示信号量是否在进程间共享。如果为0，则信号量只能在当前进程中使用；如果非0，则可以在多个进程中共享使用。<br />第三个参数为信号量的初始值。如果为0，则表示所有等待该信号量的进程将被阻塞；如果大于0，则表示所有等待该信号量的进程将不被阻塞，继续执行。<br />因此，该语句的含义是：如果sem_init函数的返回值不等于0（即初始化失败），则执行下面的语句，抛出一个std::exception异常。<br />这里使用了!=0来判断sem_init函数返回值的大小，因为sem_init函数的返回值类型为int，如果返回0表示初始化成功，如果返回值小于0则表示初始化失败，返回的值表示具体的错误码。由于通常错误码是负数，因此使用!=0来判断返回值不等于0更方便。
<a name="v4xMJ"></a>
### 1.4 sem_t
sem_t 是 C 语言中 POSIX 信号量的类型，它通常用于线程间同步或进程间同步。POSIX 是可移植操作系统接口（Portable Operating System Interface）的缩写，是一种跨平台的API规范，定义了许多标准的系统函数。<br />信号量是一种计数器，用于多个进程或线程之间进行同步和通信，它通常用于保护共享资源的访问。POSIX 信号量是一种线程安全的信号量，可以通过 sem_init() 来初始化信号量， sem_wait() 来阻塞等待信号量的值， sem_post() 来增加信号量的值， sem_destroy() 来销毁信号量。
<a name="Ve0sk"></a>
## 2、封装条件变量
```cpp
//条件变量的使用机制需要配合锁来使用
//内部会有一次加锁和解锁
//封装起来会使得更加简洁
bool wait()
{
   int ret=0;
   pthread_mutex_lock(&m_mutex);
   ret=pthread_cond_wait(&m_cond,&m_mutex);
   pthread_mutex_unlock(&m_mutex);
   return ret==0;
}
bool signal()
{
   return pthread_cond_signal(&m_cond)==0;
}
```
<a name="RZnfS"></a>
### 2.1 总结
上面的代码是使用条件变量实现同步的示例代码。条件变量是一种多线程同步机制，通常用于一个线程需要等待另一个线程满足某个条件后才能继续执行。使用条件变量时需要配合互斥锁来使用，以保证对共享资源的访问是线程安全的。<br />wait() 函数实现了条件变量的等待操作。在等待前，先通过 pthread_mutex_lock() 加锁，以保证在等待时不会有其他线程修改共享资源。然后调用 pthread_cond_wait() 函数等待条件变量。这个函数会自动解锁互斥锁，并且在等待期间会阻塞当前线程，直到条件变量被其它线程发送信号唤醒。等待完成后，再通过 pthread_mutex_unlock() 解锁互斥锁，以允许其他线程继续访问共享资源。<br />signal() 函数实现了条件变量的唤醒操作。它通过调用 pthread_cond_signal() 函数向等待该条件变量的线程发送信号，通知它们可以继续执行了。<br />这里的 wait() 和 signal() 函数都返回一个 bool 类型的值，表示函数执行是否成功。如果返回值为 true，则表示函数执行成功；如果返回值为 false，则表示函数执行失败。<br />这里的示例代码中，条件变量的使用被封装到了 wait() 和 signal() 两个函数中，使得代码更加简洁和易于维护。
<a name="OU7YU"></a>
### 2.2 pthread_cond_wait()
pthread_cond_wait() 函数是 POSIX 线程库中用于等待条件变量的函数。它的函数原型如下：
```cpp
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);
```
该函数有两个参数：第一个参数 cond 是指向条件变量的指针；第二个参数 mutex 是指向互斥锁的指针。在调用 pthread_cond_wait() 函数前，需要先使用 pthread_mutex_lock() 函数来加锁互斥锁，以保证等待过程中不会有其它线程修改共享资源。<br />调用 pthread_cond_wait() 函数时，它会将当前线程加入到条件变量的等待队列中，并释放已加锁的互斥锁，以允许其它线程访问共享资源。然后等待其它线程调用 pthread_cond_signal() 或 pthread_cond_broadcast() 函数发送信号，唤醒等待队列中的线程。<br />当线程被唤醒时，它会重新获得已加锁的互斥锁，并从 pthread_cond_wait() 函数返回。如果线程被信号唤醒，则 pthread_cond_wait() 函数返回0，否则返回一个非零值。<br />在上面的示例代码中，pthread_cond_wait(&m_cond, &m_mutex) 的返回值被赋值给了 ret 变量，以便在函数执行失败时抛出异常。因为如果 pthread_cond_wait() 函数返回一个非零值，则表示等待操作失败。此时应该抛出异常，以便上层代码能够捕获并处理这种异常情况。
<a name="mrFAM"></a>
### 2.3 pthread_cond_signal()
pthread_cond_signal() 函数是 POSIX 线程库中用于发送条件变量信号的函数。它的函数原型如下：
```cpp
int pthread_cond_signal(pthread_cond_t *cond);
```
该函数只有一个参数：cond 是指向条件变量的指针。调用 pthread_cond_signal() 函数时，它会向等待在条件变量上的某个线程发送信号，以通知它可以继续执行了。<br />如果成功发送了信号，则 pthread_cond_signal() 函数返回0；否则返回一个非零值，表示发送信号失败。<br />在上面的示例代码中，pthread_cond_signal(&m_cond) 的返回值被作为 signal() 函数的返回值，以便上层代码能够判断信号是否发送成功。如果 pthread_cond_signal() 函数返回0，则说明信号发送成功；否则说明信号发送失败。通过将这个返回值作为 signal() 函数的返回值，上层代码可以根据需要进行错误处理。
<a name="zgW6X"></a>
# 三、半同步半反应堆线程池
<a name="nWG3k"></a>
## 1、线程池类定义
```cpp
 template<typename T>
 class threadpool{
     public:
         //thread_number是线程池中线程的数量
         //max_requests是请求队列中最多允许的、等待处理的请求的数量
         //connPool是数据库连接池指针
         threadpool(connection_pool *connPool, int thread_number = 8, int max_request = 10000);
         ~threadpool();
 
        //像请求队列中插入任务请求
        bool append(T* request);

    private:
        //工作线程运行的函数
        //它不断从工作队列中取出任务并执行之
        static void *worker(void *arg);

        void run();

    private:
        //线程池中的线程数、
        int m_thread_number;

        //请求队列中允许的最大请求数
        int m_max_requests;

        //描述线程池的数组，其大小为m_thread_number
        pthread_t *m_threads;

        //请求队列
        std::list<T *>m_workqueue;    

        //保护请求队列的互斥锁    
        locker m_queuelocker;

        //是否有任务需要处理
        sem m_queuestat;

        //是否结束线程
        bool m_stop;

        //数据库连接池
        connection_pool *m_connPool;  
};
```
<a name="b7myl"></a>
## 2、线程池创建与回收
```cpp
template<typename T>
threadpool<T>::threadpool( connection_pool *connPool, int thread_number, int max_requests) : m_thread_number(thread_number), m_max_requests(max_requests), m_stop(false), m_threads(NULL),m_connPool(connPool){

    if(thread_number<=0||max_requests<=0)
        throw std::exception();

    //线程id初始化
    m_threads=new pthread_t[m_thread_number];
    // 初始化线程数组
    if(!m_threads)
        throw std::exception();
    for(int i=0;i<thread_number;++i)
    {
        //循环创建线程，并将工作线程按要求进行运行
        if(pthread_create(m_threads+i,NULL,worker,this)!=0){
           delete [] m_threads;
            throw std::exception();
        }

        //将线程进行分离后，不用单独对工作线程进行回收
        if(pthread_detach(m_threads[i])){
            delete[] m_threads;
            throw std::exception();
        }
    }
}
```
<a name="ggWZ1"></a>
### 2.1 pthread_create()
pthread_create()是一个POSIX线程库函数，用于创建一个新的线程。该函数的函数原型如下：
```cpp
c++Copy code
int pthread_create(pthread_t *thread, const pthread_attr_t *attr,                    void *(*start_routine)(void*), void *arg);
```
该函数接受4个参数：<br />thread参数是一个指向pthread_t类型的指针，用于存储新线程的ID。<br />attr参数是一个指向pthread_attr_t类型的指针，用于指定线程的属性。如果为NULL，则使用默认属性。<br />start_routine参数是一个指向线程函数的指针，该函数接受一个void*类型的参数，并返回一个void*类型的结果。这个函数是新线程的入口点。<br />arg参数是一个指向传递给线程函数的参数的指针，可以是任何类型的指针。<br />pthread_create()函数的作用是创建一个新的线程，并将它添加到进程中。新线程从start_routine指向的函数开始执行，并将arg指向的参数传递给它。线程会一直运行，直到完成任务或者被取消。<br />pthread_create()函数的返回值是一个整型值，表示创建线程的状态。如果返回值为0，则表示创建线程成功；如果返回值不为0，则表示创建线程失败，具体错误原因可以通过errno变量来获取。
<a name="ohjjm"></a>
### 2.2 pthread_detach()
pthread_detach()函数是 POSIX 线程库提供的一个函数，用于将一个线程标记为“可分离”的状态。线程在被标记为可分离状态后，意味着该线程的资源可以在线程退出时自动释放，而不需要其他线程调用pthread_join()函数等待该线程退出并释放资源。<br />pthread_detach()函数的返回值表示函数执行的状态，它的返回值为 0 表示函数执行成功，否则表示函数执行失败。通常情况下，pthread_detach()函数调用失败的原因可能是由于线程已经被标记为可分离状态，或者线程已经结束并且资源已经被自动回收。
<a name="UEj5D"></a>
## 3、向请求队列中添加任务
```cpp
template<typename T>
bool threadpool<T>::append(T* request)
{
    m_queuelocker.lock();

    //根据硬件，预先设置请求队列的最大值
    if(m_workqueue.size()>m_max_requests)
    {
        m_queuelocker.unlock();
        return false;
    }

    //添加任务
    m_workqueue.push_back(request);
    m_queuelocker.unlock();

    //信号量提醒有任务要处理
    m_queuestat.post();
    return true;
}
```
<a name="Sy5oz"></a>
### 3.1 m_queuelocker.lock()
locker类中定义过<br />m_queuelocker是一个互斥量（mutex），lock()是该互斥量对象提供的一个成员函数，用于对互斥量进行加锁。在多线程环境下，为了避免多个线程同时对共享资源进行访问，需要对共享资源进行加锁保护，以确保同一时间只有一个线程可以访问该资源。<br />在这里，m_queuelocker.lock()的作用是对一个线程池中的任务队列进行加锁，防止其他线程同时访问该队列。具体来说，当一个线程调用m_queuelocker.lock()函数时，如果互斥量处于未加锁状态，则该线程将互斥量加锁；如果互斥量已经处于加锁状态，则该线程会被阻塞，直到互斥量被解锁为止。
<a name="AGmzY"></a>
### 3.2 m_queuestat.post()
locker类中定义过
<a name="cpfPg"></a>
## 4、线程处理函数
```cpp
template<typename T>
void* threadpool<T>::worker(void* arg){

    //将参数强转为线程池类，调用成员方法
    threadpool* pool=(threadpool*)arg;
    pool->run();
    return pool;
}
```
<a name="ywMJE"></a>
## 5、run执行任务
```cpp
template<typename T>
void threadpool<T>::run()
{
    while(!m_stop)
    {    
        //信号量等待
        m_queuestat.wait();

        //被唤醒后先加互斥锁
        m_queuelocker.lock();
        if(m_workqueue.empty())
        {
            m_queuelocker.unlock();
            continue;
        }

        //从请求队列中取出第一个任务
        //将任务从请求队列删除
        T* request=m_workqueue.front();
        m_workqueue.pop_front();
        m_queuelocker.unlock();
        if(!request)
            continue;

        //从连接池中取出一个数据库连接
        request->mysql = m_connPool->GetConnection();

        //process(模板类中的方法,这里是http类)进行处理
        request->process();

        //将数据库连接放回连接池
        m_connPool->ReleaseConnection(request->mysql);
    }
}
```
<a name="kLc0t"></a>
### 5.1 m_queuestat.wait()
m_queuestat.wait()表示等待信号量的触发。信号量是一个计数器，用于协调多个线程之间的并发访问，保证同一时刻只有一个线程能够访问共享资源。当信号量的计数器大于0时，线程可以继续执行。当计数器等于0时，线程会被阻塞，直到有其他线程发出信号量的信号，计数器加1，该线程才能够被唤醒。<br />在线程池的实现中，当新的任务到达时，会先将该任务加入到请求队列中，并发出一个信号量的信号，通知工作线程有新的任务需要处理。工作线程在执行时，会先等待信号量的信号，等待到信号后再从请求队列中取出任务进行处理。这样可以避免工作线程的空闲和任务队列的阻塞，提高了线程池的效率和并发能力。
<a name="v3ec5"></a>
### 5.2 request->process()
在该函数中，request->process()表示对一个请求进行处理，即执行请求对应的处理函数。由于threadpool是一个通用的线程池类，它并不知道具体请求的类型和如何处理请求，因此在实现时，使用了模板类的方式，将请求和请求处理函数作为模板参数进行传递。这样，在处理任务时，可以将任务强制转换为对应的请求类型，再调用请求的处理函数进行处理。<br />该函数定义了对一个HTTP请求进行处理的逻辑，包括解析请求内容、处理请求参数、生成响应等。由于线程池中的工作线程只负责处理请求，而不关心具体请求的类型和实现，因此将请求处理函数定义为模板类中的方法，这样可以将请求和请求处理函数作为模板参数进行传递，提高了代码的通用性和灵活性。
<a name="eC3SM"></a>
# 四、HTTP连接处理
<a name="zXhUQ"></a>
## 4.1 HTTP类及请求接收
浏览器端发出http连接请求，主线程创建http对象接收请求并将所有数据读入对应buffer，将该对象插入任务队列，工作线程从任务队列中取出一个任务进行处理。
<a name="DYOEi"></a>
### 4.1.1 HTTP类
```cpp
class http_conn{
    public:
        //设置读取文件的名称m_real_file大小
        static const int FILENAME_LEN=200;
        //设置读缓冲区m_read_buf大小
        static const int READ_BUFFER_SIZE=2048;
        //设置写缓冲区m_write_buf大小
        static const int WRITE_BUFFER_SIZE=1024;
        //报文的请求方法，本项目只用到GET和POST
        enum METHOD{GET=0,POST,HEAD,PUT,DELETE,TRACE,OPTIONS,CONNECT,PATH};
        //主状态机的状态
        enum CHECK_STATE{CHECK_STATE_REQUESTLINE=0,CHECK_STATE_HEADER,CHECK_STATE_CONTENT};
        //报文解析的结果
        enum HTTP_CODE{NO_REQUEST,GET_REQUEST,BAD_REQUEST,NO_RESOURCE,FORBIDDEN_REQUEST,FILE_REQUEST,INTERNAL_ERROR,CLOSED_CONNECTION};
        //从状态机的状态
        enum LINE_STATUS{LINE_OK=0,LINE_BAD,LINE_OPEN};

    public:
        http_conn(){}
        ~http_conn(){}

    public:
       //初始化套接字地址，函数内部会调用私有方法init
        void init(int sockfd,const sockaddr_in &addr);
        //关闭http连接
        void close_conn(bool real_close=true);
        void process();
        //读取浏览器端发来的全部数据
        bool read_once();
        //响应报文写入函数
        bool write();
        sockaddr_in *get_address(){
            return &m_address;  
        }
        //同步线程初始化数据库读取表
        void initmysql_result();
        //CGI使用线程池初始化数据库表
        void initresultFile(connection_pool *connPool);
    private:
        void init();
       //从m_read_buf读取，并处理请求报文
        HTTP_CODE process_read();
        //向m_write_buf写入响应报文数据
       bool process_write(HTTP_CODE ret);
        //主状态机解析报文中的请求行数据
        HTTP_CODE parse_request_line(char *text);
       //主状态机解析报文中的请求头数据
        HTTP_CODE parse_headers(char *text);
        //主状态机解析报文中的请求内容
        HTTP_CODE parse_content(char *text);
        //生成响应报文
        HTTP_CODE do_request();

        //m_start_line是已经解析的字符
        //get_line用于将指针向后偏移，指向未处理的字符
        char* get_line(){return m_read_buf+m_start_line;};

        //从状态机读取一行，分析是请求报文的哪一部分
        LINE_STATUS parse_line();

        void unmap();

        //根据响应报文格式，生成对应8个部分，以下函数均由do_request调用
        bool add_response(const char* format,...);
        bool add_content(const char* content);
        bool add_status_line(int status,const char* title);
        bool add_headers(int content_length);
        bool add_content_type();
        bool add_content_length(int content_length);
        bool add_linger();
        bool add_blank_line();

    public:
        static int m_epollfd;
        static int m_user_count;
        MYSQL *mysql;

    private:
        int m_sockfd;
        sockaddr_in m_address;

        //存储读取的请求报文数据
        char m_read_buf[READ_BUFFER_SIZE];
        //缓冲区中m_read_buf中数据的最后一个字节的下一个位置
        int m_read_idx;
        //m_read_buf读取的位置m_checked_idx
        int m_checked_idx;
        //m_read_buf中已经解析的字符个数
        int m_start_line;

        //存储发出的响应报文数据
        char m_write_buf[WRITE_BUFFER_SIZE];
        //指示buffer中的长度
        int m_write_idx;

        //主状态机的状态
        CHECK_STATE m_check_state;
        //请求方法
        METHOD m_method;

        //以下为解析请求报文中对应的6个变量
        //存储读取文件的名称
        char m_real_file[FILENAME_LEN];
        char *m_url;
        char *m_version;
        char *m_host;
        int m_content_length;
        bool m_linger;

        char *m_file_address;        //读取服务器上的文件地址
        struct stat m_file_stat;
        struct iovec m_iv[2];        //io向量机制iovec
        int m_iv_count;
        int cgi;                    //是否启用的POST
        char *m_string;                //存储请求头数据
        int bytes_to_send;          //剩余发送字节数
        int bytes_have_send;        //已发送字节数
};
```
<a name="BI8um"></a>
### 4.1.2 read_once()
```cpp
//循环读取客户数据，直到无数据可读或对方关闭连接
bool http_conn::read_once()
{
    if(m_read_idx>=READ_BUFFER_SIZE)
    {
        return false;
    }
    int bytes_read=0;
    while(true)
    {
        //从套接字接收数据，存储在m_read_buf缓冲区
        bytes_read=recv(m_sockfd,m_read_buf+m_read_idx,READ_BUFFER_SIZE-m_read_idx,0);
        if(bytes_read==-1)
        {    
            //非阻塞ET模式下，需要一次性将数据读完
            if(errno==EAGAIN||errno==EWOULDBLOCK)
                break;
            return false;
        }
        else if(bytes_read==0)
        {
            return false;
        }
        //修改m_read_idx的读取字节数
        m_read_idx+=bytes_read;
    }
    return true;
}
```
read_once读取浏览器端发送来的请求报文，直到无数据可读或对方关闭连接，读取到m_read_buffer中，并更新m_read_idx。
<a name="hExsB"></a>
#### 4.1.2.1 recv()
recv() 函数是一个用于接收数据的系统调用函数。它的语法如下：
```cpp
ssize_t recv(int sockfd, void *buf, size_t len, int flags);
```
其中，<br />sockfd：需要接收数据的套接字文件描述符。<br />buf：指向一个缓冲区的指针，用于存放接收到的数据。<br />len：缓冲区的大小，即最多接收多少字节的数据。<br />flags：用于控制接收数据的行为，通常设置为 0。<br />recv() 函数会一直阻塞，直到有数据到达或者出现错误。当函数返回时，它会返回已经接收到的数据的字节数。如果返回值为 0，则表示对端已经关闭连接。<br />在你的示例中，bytes_read=recv(m_sockfd,m_read_buf+m_read_idx,READ_BUFFER_SIZE-m_read_idx,0); 表示从套接字 m_sockfd 中读取数据，最多读取 READ_BUFFER_SIZE-m_read_idx 字节数据，并将其存放到 m_read_buf 数组中，从 m_read_buf + m_read_idx 的位置开始存储。函数返回实际读取到的字节数，并将其赋值给 bytes_read 变量。

<a name="ndvyC"></a>
#### 4.1.2.2 if语句
上述代码是在对套接字进行非阻塞读取时的错误处理部分。其中，bytes_read 表示实际读取到的字节数。<br />如果 bytes_read 的值为 -1，则表示读取数据时出现了错误，此时需要根据具体的错误类型进行相应的处理。如果错误类型为 EAGAIN 或 EWOULDBLOCK，则表示当前没有数据可读（在非阻塞模式下），应该退出循环并等待下一次读取操作。如果错误类型为其他值，则表示发生了其他类型的错误，此时应该直接返回 false，并在调用方进行进一步的错误处理。<br />如果 bytes_read 的值为 0，则表示对端已经关闭了连接，此时应该直接返回 false，并在调用方进行相应的处理。<br />errno 是一个全局的错误码变量，用于表示最近一次系统调用（例如 open()、read()、write() 等）返回的错误码。在 C 语言中，如果系统调用返回一个负数，通常表示出现了错误，并且 errno 变量会被设置为相应的错误码。<br />errno 变量在头文件 <errno.h> 中定义，通常使用宏函数 errno 来获取其值。在多线程程序中，errno 的值是线程本地的，即每个线程都有自己的 errno 变量。
<a name="l4Z3e"></a>
### 4.1.3 epoll相关代码
<a name="oT3P2"></a>
#### 4.1.3.1 非阻塞模式
```cpp
//对文件描述符设置非阻塞
int setnonblocking(int fd)
{
    int old_option = fcntl(fd, F_GETFL);
    int new_option = old_option | O_NONBLOCK;
    fcntl(fd, F_SETFL, new_option);
    return old_option;
}
```
上述代码定义了一个函数 setnonblocking()，用于将指定的文件描述符设置为非阻塞模式。其实现步骤如下：<br />使用 fcntl(fd, F_GETFL) 获取指定文件描述符的当前状态标志。<br />将当前状态标志与 O_NONBLOCK 按位或，得到新的状态标志。<br />使用 fcntl(fd, F_SETFL, new_option) 将新的状态标志应用到文件描述符上，即将文件描述符设置为非阻塞模式。<br />返回旧的状态标志，以便后续恢复为阻塞模式时使用。<br />其中，fcntl() 是一个用于对文件描述符进行操作的系统调用函数，它可以获取或设置文件描述符的状态标志、文件锁等信息。在这里，我们使用它来获取和设置文件描述符的状态标志。<br />通过将文件描述符设置为非阻塞模式，可以使得读写操作在没有数据可读写时立即返回，而不是一直等待数据的到来。这在一些场景中比较有用，例如在网络编程中，我们通常会使用非阻塞模式来实现异步 IO。<br />**为什么要设置new和old**<br />在将文件描述符设置为非阻塞模式时，我们需要先获取其当前的状态标志，然后将其与 O_NONBLOCK 标志进行按位或操作，得到新的状态标志，最后将新的状态标志应用到文件描述符上。<br />在 setnonblocking() 函数中，我们通过调用 fcntl(fd, F_GETFL) 获取指定文件描述符的当前状态标志，并将其保存到变量 old_option 中。接着，我们根据 old_option 计算出新的状态标志 new_option，并使用 fcntl(fd, F_SETFL, new_option) 将其应用到文件描述符上，以实现将其设置为非阻塞模式。<br />由于我们需要在后续的恢复操作中使用旧的状态标志，因此需要将其保存到变量 old_option 中，并在函数的最后返回。这样，当需要将文件描述符恢复为阻塞模式时，就可以使用 fcntl(fd, F_SETFL, old_option) 来恢复其状态标志。<br />**按位或操作**<br />在 Linux 中，文件描述符的状态标志可以使用多个标志位来表示，这些标志位的取值可以通过按位或操作来组合成不同的状态。例如，常用的文件描述符状态标志包括：<br />O_RDONLY：以只读方式打开文件。<br />O_WRONLY：以只写方式打开文件。<br />O_RDWR：以读写方式打开文件。<br />O_CREAT：如果文件不存在，则创建文件。<br />O_TRUNC：如果文件存在，则将其长度截短为 0。<br />O_APPEND：在写入数据时，始终将数据添加到文件末尾。<br />O_NONBLOCK：将文件描述符设置为非阻塞模式。<br />在将文件描述符设置为非阻塞模式时，我们需要将其当前的状态标志与 O_NONBLOCK 进行按位或操作，以设置该标志位。<br />**为什么要或操作而不是直接修改**<br />使用按位或操作可以将多个标志位组合成一个二进制数，从而方便进行状态的修改。如果直接对状态进行修改，可能会导致一些状态信息的丢失或错误。例如，如果我们需要将一个文件描述符设置为非阻塞模式，同时又想保留其它状态信息（比如读写模式），如果直接修改状态，可能会将读写模式的信息覆盖掉，导致读写操作出错。而通过按位或操作，我们可以将新的状态信息与当前状态信息进行合并，保留原有的状态信息，同时设置新的状态信息，从而避免了状态信息的丢失或错误。ONBLOCK    04000 按位或只修改第四位的值，其他值不变。
<a name="zIrVy"></a>
#### 4.1.3.2 内核时间表注册新事件
```cpp
void addfd(int epollfd, int fd, bool one_shot)
{
    epoll_event event;
    event.data.fd = fd;

#ifdef ET
    event.events = EPOLLIN | EPOLLET | EPOLLRDHUP;
#endif

#ifdef LT
    event.events = EPOLLIN | EPOLLRDHUP;
#endif

    if (one_shot)
        event.events |= EPOLLONESHOT;
    epoll_ctl(epollfd, EPOLL_CTL_ADD, fd, &event);
    setnonblocking(fd);
}
```
<a name="jHjy5"></a>
##### 4.1.3.2.1 #ifdef #endif
#ifdef #endif是根据定义的预处理宏选择不同的事件触发模式来设置 epoll 监听事件的类型。在 Linux 的 epoll 机制中，有两种事件触发模式：<br />边缘触发（EPOLLET）：只有在文件描述符状态变化时才触发事件，也就是说只有当数据从无到有时才会触发一次读事件，否则不会触发。该模式需要使用非阻塞 IO 操作。<br />水平触发（EPOLLRDHUP）：当文件描述符关闭时会触发一次 EPOLLRDHUP 事件，此时需要关闭连接。该模式可以使用阻塞 IO 操作。<br />因此，根据预处理宏 ET 和 LT 的不同，上述代码分别设置事件类型为 EPOLLIN | EPOLLET | EPOLLRDHUP 或 EPOLLIN | EPOLLRDHUP，即指定 epoll 监听时关注的事件类型。如果预处理宏 ET 被定义，则使用边缘触发模式；如果预处理宏 LT 被定义，则使用水平触发模式。
<a name="dPbUr"></a>
##### 4.1.3.2.1 one_shot
如果参数 one_shot 为 true，则将事件设置为 EPOLLONESHOT 模式，表示当该文件描述符上有一个事件被触发后，该文件描述符将被自动从 epoll 监听列表中删除，以防止多个线程同时处理同一个事件的情况
<a name="Yz2Uv"></a>
#### 4.1.3.3 内核事件表删除事件
```cpp
\void removefd(int epollfd, int fd)
{
    epoll_ctl(epollfd, EPOLL_CTL_DEL, fd, 0);
    close(fd);
}
```
close()，关闭文件描述符对应的的套接字
<a name="YFQTc"></a>
#### 4.1.3.4 重置EPOLLONESHOT事件
```cpp
void modfd(int epollfd, int fd, int ev)
{
    epoll_event event;
    event.data.fd = fd;

#ifdef ET
    event.events = ev | EPOLLET | EPOLLONESHOT | EPOLLRDHUP;
#endif

#ifdef LT
    event.events = ev | EPOLLONESHOT | EPOLLRDHUP;
#endif

    epoll_ctl(epollfd, EPOLL_CTL_MOD, fd, &event);
}
```
ev是需要修改的事件类型
<a name="ZRzfT"></a>
### 4.1.4 服务器接收HTTP请求
```cpp
//创建MAX_FD个http类对象
http_conn* users=new http_conn[MAX_FD];

//创建内核事件表
epoll_event events[MAX_EVENT_NUMBER];
epollfd = epoll_create(5);
assert(epollfd != -1);

//将listenfd放在epoll树上
addfd(epollfd, listenfd, false);

//将上述epollfd赋值给http类对象的m_epollfd属性
http_conn::m_epollfd = epollfd;

while (!stop_server)
{
    //等待所监控文件描述符上有事件的产生
    int number = epoll_wait(epollfd, events, MAX_EVENT_NUMBER, -1);
    if (number < 0 && errno != EINTR)
    {
        break;
    }
    //对所有就绪事件进行处理
    for (int i = 0; i < number; i++)
    {
        int sockfd = events[i].data.fd;

        //处理新到的客户连接
        if (sockfd == listenfd)
        {
            struct sockaddr_in client_address;
            socklen_t client_addrlength = sizeof(client_address);
//LT水平触发
#ifdef LT
            int connfd = accept(listenfd, (struct sockaddr *)&client_address, &client_addrlength);
            if (connfd < 0)
            {
                continue;
            }
            if (http_conn::m_user_count >= MAX_FD)
            {
                show_error(connfd, "Internal server busy");
                continue;
            }
            users[connfd].init(connfd, client_address);
#endif

//ET非阻塞边缘触发
#ifdef ET
            //需要循环接收数据
            while (1)
            {
                int connfd = accept(listenfd, (struct sockaddr *)&client_address, &client_addrlength);
                if (connfd < 0)
                {
                    break;
                }
                if (http_conn::m_user_count >= MAX_FD)
                {
                    show_error(connfd, "Internal server busy");
                    break;
                }
                users[connfd].init(connfd, client_address);
            }
            continue;
#endif
        }

        //处理异常事件
        else if (events[i].events & (EPOLLRDHUP | EPOLLHUP | EPOLLERR))
        {
            //服务器端关闭连接
        }

        //处理信号
        else if ((sockfd == pipefd[0]) && (events[i].events & EPOLLIN))
        {
        }

        //处理客户连接上接收到的数据
        else if (events[i].events & EPOLLIN)
        {
            //读入对应缓冲区
            if (users[sockfd].read_once())
            {
                //若监测到读事件，将该事件放入请求队列
                pool->append(users + sockfd);
            }
            else
            {
               //服务器关闭连接
            }
        }

    }
}
```
浏览器端发出http连接请求，主线程创建http对象接收请求并将所有数据读入对应buffer，将该对象插入任务队列，工作线程从任务队列中取出一个任务进行处理。
<a name="qaTeS"></a>
#### 4.1.4.1 assert()
assert 是 C++ 中的一个宏，用于判断某个条件是否为真。如果条件为假，assert 会输出错误信息并终止程序运行。在代码中，assert(epollfd != -1) 的含义是判断 epoll_create 函数是否成功返回了 epoll 文件描述符，如果失败则终止程序运行并输出错误信息。这样可以在程序中及时发现错误，并避免程序因为错误状态的存在而导致意想不到的行为。
<a name="z2F73"></a>
#### 4.1.4.2 pipe
定时器相关
<a name="n7HZt"></a>
## 4.2 请求报文解析
工作线程取出任务后，调用process_read函数，通过主、从状态机对请求报文进行解析。
<a name="eRosM"></a>
### 4.2.1 process()
```cpp
 void http_conn::process()
 {
     HTTP_CODE read_ret=process_read();
 
     //NO_REQUEST，表示请求不完整，需要继续接收请求数据
     if(read_ret==NO_REQUEST)
     {
         //注册并监听读事件
         modfd(m_epollfd,m_sockfd,EPOLLIN);
        return;
    }

    //调用process_write完成报文响应
    bool write_ret=process_write(read_ret);
    if(!write_ret)
    {
        close_conn();
    }
    //注册并监听写事件
    modfd(m_epollfd,m_sockfd,EPOLLOUT);
}
```
见：[跳转到锚点](#process)


<a name="qLeiz"></a>
## 4.3 请求报文响应
解析完之后，跳转do_request函数生成响应报文，通过process_write写入buffer，返回给浏览器端。
