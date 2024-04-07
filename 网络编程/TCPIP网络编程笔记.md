# TCPIP 网络编程

# 1.1 理解网络编程和套接字

**套接字** 是网络数据传输用的软件设备。

## 构建接电话套接字

* `socket()`​ 函数

    ```cpp
    #include <sys/socket.h>
    int socket(int domain, int type, int protocol);
    
    // 成功时返回文件描述符， 失败时返回-1
    ```

* `bind()`​ 函数

    ```cpp
    #include <sys/socket.h>
    int socket(int domain, int type, int protocol);
    
    // 成功时返回 0，失败时返回-1
    ```

* `listen()`​ 函数

    ```cpp
    #include <sys/socket.h>
    int listen(int sockfd, int backlog);
    
    // 成功时返回 0，失败时返回-1
    ```

* `accept()`​ 函数

    ```cpp
    int accept(int sockfd, struct sockaddr* addr, socklen_t *addrlen);
    
    // 成功时返回文件描述符，失败时返回-1
    ```

网络编程中接受连接请求的套接字创建过程如下：

1. 调用 `socket`​ 函数创建套接字。
2. 调用 `bind`​ 函数分配 IP 地址和端口号。
3. 调用 `listen`​ 函数转为可接受请求状态。
4. 调用 `accept`​ 函数受理连接请求。

* 构建打电话套接字

    ```cpp
    #include <sys/socket.h>
    int connect(int sockfd, struct sockaddr *serv_addr, socklen_t addrlen);
    
    // 成功时返回 0，失败时返回-1
    ```

# 1.2 基于 Linux 的文件操作

* 打开文件

    ```cpp
    #include <sys/types.h>
    #include <sys/stat.h>
    #include <fcntl.h>
    
    // path: 文件名的字符串地址
    // flag: 文件打开模式信息
    int open(const char* path, int flag>
    // 成功时返回文件描述符，失败时返回-1
    ```

| 打开模式   |            含义            |
| ---------- | :------------------------: |
| `O_CREAT`​  |       必要时创建文件       |
| `O_TRUNC`​  |      删除全部现有数据      |
| `O_APPEND`​ | 维持现有数据，保存到其后面 |
| `O_RDONLY`​ |          只读打开          |
| `O_WRONLY`​ |          只写打开          |
| `O_RDWR`​   |          读写打开          |

* 关闭文件

    ```cpp
    #include <unistd.h>
    
    // fd: 需要关闭的文件或套接字的文件描述符
    int close(int fd);
    // 成功时返回 0，失败时返回-1
    ```

* 将数据写入文件

    ```c
    #include <unistd.h>
    
    // fd: 显示数据传输对象的文件描述符
    // buf: 保存要传输数据的缓冲地址值
    // nbytes: 要传输数据的字节数
    ssize_t write(int fd, const void* buf, size_t nbytes);
    // 成功时返回写入的字节数，失败时返回-1
    ```

* 读取文件中的数据

    ```c
    #include <unistd.h>
    
    ssize_t read(int fd, void *buf, size_t nbytes);
    // 成功时返回写入的字节数（遇到文件结尾返回 0），失败时返回-1
    ```

# 2.1 套接字协议及其数据传输特性

**协议：计算机间对话必备通信规则。**

* 创建套接字

    ```c
    #include <sys/socket.h>
    
    // domain: 套接字中使用的协议族信息
    // type: 套接字数据传输类型信息
    // protocol: 计算机间通信中使用的协议信息
    int socket(int domain, int type, int protocol);
    // 成功时返回文件描述符，失败时返回-1
    ```

| 名称        | 协议族                 |
| ----------- | ---------------------- |
| `PF_INET`​   | IPv4 互联网协议族      |
| `PF_INET6`​  | IPv6 互联网协议族      |
| `PF_LOCAL`​  | 本地通信的 UNIX 协议族 |
| `PF_PACKET`​ | 底层套接字的协议族     |
| `PF_IPX`​    | IPX Novell 协议族      |

## 套接字类型

* 套接字类型 1：面向连接的套接字 (SOCK_STREAM)————*可靠的、按序传递的、基于字节的面向连接的数据传输方式的套接字*
* 套接字类型 2：面向消息的套接字（SOCK_DGRAM）————不可靠的、不按序传递的、以数据的高速传输为目的的套接字

## 协议的最终选择

* 只有在 **同一协议族中存在多个数据传输方式相同的协议** 时向 `socket`​ 的第三个参数指定协议信息。

    * 其实对于 IPv4 而言就以下两种。

```c
// IPv4 + SOCK_STREAM
int tcp_socket = socket(PF_INET, SOCK_STREAM, IPPROTO_TCP);

// IPv4 + SOCK_DGRAM
int udp_socket = socket(PF_INET, SOCK_DGRAM, IPPROTO_UDP);
```

# 3.1 分配给套接字的 IP 地址与端口号

IP(*Internet Protocol*)：网络协议，是为收发网络数据而分配给计算机的值。

端口号：是为区分程序中创建的套接字而分配给套接字的序号。

## 网络地址

![image](./../myNote/assets/image-20240224230254-rfk4uj6.png "Ipv4地址族")

## 网络地址分类与主机地址边界

只需通过 IP 地址的第一个字节即可判断网络地址占用的字节数：

* A 类地址：0~127（以 0 开头）
* B 类地址：128~191（以 10 开头）
* C 类地址：192~223（以 110 开头）

## 用于区分套接字的端口号

IP 用于区分计算机，只要有 IP 地址就能向目标主机传输数据。

计算机中一般配有 NIC（网络接口卡）。通过 NIC 接受的数据内有端口号，OS 通过端口号把数据传输给相应的套接字。

![image](./../myNote/assets/image-20240224231249-j6bp49x.png)

端口号由 16 位组成（即 0\~65535)，其中 0~1023 知名端口(*Well-known PORT)，分配给特定应用程序。

TCP 套接字和 UDP 套接字不会共用端口号，所以允许重复。

# 3.2 地址信息的表示

## 表示 IPv4 地址的结构体

```c
struct sockaddr_in {
	sa_family_t sin_family;		//地址族
	uint16_t sin_port;			//16 位 TCP/UDP 端口号
	struct in_addr sin_addr;	//32 位 IP 地址
	char sin_zero[8];			//不使用
}
```

其中的 `in_addr`​ 结构体定义如下：

```c
struct in_addr {
	In_addr_t s_addr;		//32 位 IPv4 地址
}
```

![image](./../myNote/assets/image-20240224234046-2i4ejin.png)

## `sockaddr_in`​ 的成员分析

* `sin_family`​

![image](https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240224234300-f0tk4pn.png)​

* `sin_port`​

    该成员 **以网络字节序** 保存 16 位端口号。

* `sin_addr`​

    该成员 **以网络字节序** 保存 32 位 IP 地址信息。

* `sin_zero`​

    无特殊含义。只是为了保证 `sockaddr_in`​ 和 `sockaddr`​ 大小相同。

# 3.3 网络字节序与地址变换

## 字节序与网络字节序

CPU 分为大端序和小端序：

* 大端序：高位字节存放到 **低位** 地址。
* 小端序：高位字节存放到 **高位** 地址。

![image](https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240224235034-8swljct.png)​

网络字节序（*Network Byte Order*）统一为大端序。

## 字节序转换

转换字节序的函数：

* `unsigned short htons(unsigned short);`​
* `unsigned short ntohs(unsigned short);`​
* `unsigned long htonl(unsigned long);`​
* `unsigned long ntohl(unsigned long);`​

其中 h 代表主机（host）字节序，n 代表网络（network）字节序。

# 3.4 网络地址的初始化与分配

## 将字符串信息转换为网络字节序的整数型

下面的函数可以将字符串形式的 IP 地址转换成 32 位整数型数据。

```c
#include <arpa/inet.h>

in_addr_t inet_addr(const char* string);
// 成功时返回 32 位大端序整数型值，失败时返回 INADDR_NONE
```

`inet_aton`​ 函数与 `inet_addr`​ 函数在功能上完全相同，该函数利用了 `in_addr`​ 结构体。

```c
#include <arpa/inet.h>

// string: 含有需转换的 IP 地址信息的字符串地址值
// addr: 将保存转换结果的 in_addr 结构体变量的地址值
int inet_aton(const char *string, struct in_addr * addr);
// 成功时返回 1 (true), 失败时返回 0 (false)
```

⚠️: 实际编程中若要调用 `inet_addr`​ 函数, 需将转换后的 IP 地址信息代入 `sockaddr_in`​ 结构体中声明的 `in_addr`​ 结构体变量。

`inet_ntoa`​ 函数：把网络字节序整数型 IP 地址转换成我们熟悉的字符串形式。

```c
#incldue <arpa/inet.h>

char* inet_ntoa(struct in_addr adr);
// 成功时返回转换的字符串地址值，失败时返回-1
```

## 网络地址初始化

网络地址初始化方法：

```c
struct sockaddr_in addr;					// 
char* serv_ip = "211.217.168.13";			// 声明 IP 地址字符串
char* serv_port = "9190";					// 声明端口号字符串
memset(&addr, 0, sizeof(addr));				// 结构体变量 addr 的所有成员初始化为 0
addr.sin_family = AF_INET;					// 指定地址族
addr.sin_addr.s_addr = inet_addr(serv_ip);	// 基于字符串的 IP 地址初始化
addr.sin_port = htons(atoi(serv_port));		// 基于字符串的端口号初始化
```

## 客户端地址信息初始化

服务器端声明 `sockaddr_in`​ 结构体变量，将其初始化为赋予服务器端 IP 和套接字的端口号，然后调用 `bind`​ 函数；

而客户端则声明 `sockaddr_in`​ 结构体，并初始化为要与之连接的服务器端套接字的 IP 和端口号，然后调用 `connect`​ 函数

## INADDR_ANY

可以直接用 `addr.sin_addr.s_addr = htonl(INADDR_ANY)`​ 初始化地址信息。

采用这种方式可以自动获取运行服务器端的计算机 IP 地址，不必亲自输入。

## 向套接字分配网络地址

```c
#include <sys/socket.h>

// sockfd：要分配地址信息（IP 地址和端口号）的套接字文件描述符
// myaddr：存有地址信息的结构体变量地址值
// addrlen：第二个结构体变量的长度
int bind(int sockfd, struct sockaddr* myaddr, socklen_t addrlen);
// 成功时返回 0， 失败时返回-1
```

# 4.1 理解 TCP 和 UDP

TCP 是面向连接的，又称基于流的套接字。

## TCP/IP 协议栈

![image](https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240228214137-jajdrwz.png "TCP/IP协议栈")

## 链路层

链路层是物理链接领域标准化的结果，也是最基本的领域，专门定义 LAN，WAN，MAN 等网络标准。

若两台主机通过网络进行数据交换，则需要下图所示的物理连接，链路层就负责这些标准。

![image](https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240228215704-lz9k24c.png)

## IP 层

IP 本身是面向消息的、不可靠的协议。每次传输数据时会帮我们选择路径，但并不一致。如果传输中发生路径错误，则选择其他路径；但如果发生数据丢失或错误，则无法解决。

## TCP/UDP 层

TCP 和 UDP 层以 IP 层提供的路径信息为基础完成实际的数据传输，故该层又称传输层。

TCP 可以保证可靠的数据传输，但它发送数据时以 IP 层为基础。

如果数据交换过程中可以确认对方已收到数据，并重传丢失的数据，那么即便 IP 层不保证数据传输，这类通信也是可靠的。

## 应用层

编写软件的过程中，需要根据程序特点决定服务器端和客户端之间的数据传输规则，这便是应用层协议。

# 4.2 实现基于 TCP 的服务器端/客户端

## 进入等待连接请求状态

```c
#include <sys/socket.h>

// sock: 希望进入等待连接请求状态的套接字文件描述符，传递的描述符套接字参数成为服务器端套接字
// backlog: 连接请求等待队列的长度
int listen(int sock, int backlog);
// 成功时返回 0，失败时返回-1
```

## 受理客户端连接请求

```c
#include <sys/socket.h>

// sock: 服务器套接字的文件描述符
// addr: 保存发起连接请求的客户端地址信息的变量地址值，调用函数后向传递懒得地址变量参数填充客户端地址信息
// addrlen: 参数 addr 结构体的长度，但是存有长度的变量地址。函数调用完成后，该变量即被填入客户端地址长度
int accept(int sock, struct sockaddr* addr, socklen_t *addrlen);
// 成功时返回创建的套接字文件描述符，失败时返回-1
```

## TCP 客户端的默认函数调用顺序

```c
#include <sys/socket.h>

// sock: 客户端套接字文件描述符
// servaddr: 保存目标服务器端地址信息的变量地址值
// addrlen: 以字节为单位传递已传递给第二个结构体参数 servaddr 的地址变量长度
int connect(int sock, struct sockaddr *servaddr, socklen_t addrlen);
// 成功时返回 0，失败时返回-1
```

客户端调用 `connect`​函数后，发生以下情况之一才会返回：

* 服务端接收连接请求。
* 发生断网等异常情况而中断连接请求。

## 基于 TCP 的服务器端/客户端函数调用关系

![image](https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240229012116-7qy35fc.png)

# 4.3 实现迭代服务器端/客户端

## 实现迭代服务器端

方法：插入循环语句反复调用 `accept`​ 函数。

![image](https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240304001804-x70dvxg.png)

缺点：目前这个迭代服务器只能同一时刻服务于一个客户端。

## 迭代回声服务器端/客户端

程序的基本运行方式：

1. 服务器端在同一时刻只与一个客户端相连，并提供回声服务。
2. 服务器端依次向 5 个客户端提供服务并退出。
3. 客户端接收用户输入的字符串并发送到服务器端。
4. 服务器端将接收的字符串数据传回客户端，即“回声”。
5. 服务器端与客户端之间的字符串回声一直执行到客户端输入 Q 为止。

## 回声客户端存在的问题

以下代码存在错误：

```c
write(sock, message, strlen(message));
str_len = read(sock, message, BUF_SIZE - 1);
message[str_len] = 0;
printf("Message from server : %s", message);
```

假设：每次调用 `read`​、`write`​ 函数时都会以字符串为单位执行实际的 I/O 操作。

错误原因：上述客户端是基于 TCP 的，因此，多次调用 `write`​ 函数传递的字符串有可能一次性传递到服务器端。此时客户端有可能从服务器端收到多个字符串。

# 5.1 回声客户端的完美实现

## 回声服务器端没有问题，只有回声客户端有问题？

回声客户端传输的是字符串，而且是通过调用 `write`​ 函数一次性发送的。之后还调用一次 `read`​ 函数，期待接收自己传输的字符串。

**解决方法：** 提前确定接收数据的大小。

## 如果问题不在于回声客户端：定义应用层协议

若无法预知接收数据长度，就需要定义应用层协议。

服务器端/客户端实现过程中逐步定义的这些规则集合就是应用层协议。

# 5.2 TCP 原理

## TCP 套接字中的 I/O 缓冲

TCP 套接字的数据收发无边界。

`write`​ 函数调用后并非立即传输数据，`read`​ 函数调用后也并非马上接收数据。

更准确的说：`write`​ 函数调用瞬间，数据将移至输出缓冲；`read`​ 函数调用瞬间，从输入缓冲读取数据。

I/O 缓冲特性整理如下：

* I/O 缓冲在每个 TCP 套接字中单独存在。
* I/O 缓冲在创建套接字时自动生成。
* 即使关闭套接字也会继续传递输出缓冲中遗留的数据。
* 关闭套接字将丢失输入缓冲中的数据。

**重要结论：** 不会发生超过输入缓冲大小的数据传输。

原理：TCP 中有滑动窗口协议。

## TCP 内部工作原理 1：与对方套接字的链接

TCP 套接字从创建到消失所经过程分为如下 3 步：

1. 与对方套接字进行连接。
2. 与对方套接字进行数据交换。
3. 断开与对方套接字的连接。

‍

![image](https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240308205916-wne91it.png)

## TCP 内部工作原理 2：与对方主机的数据交换

![image](https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240308210427-sez1z8w.png)

## TCP 内部工作原理 3：断开与套接字的连接

# 6.1 理解 UDP

## UDP 套接字的特点

为了提供可靠的数据传输服务，TCP 在不可靠的 IP 层进行流控制，而 UDP 就缺少这种流控制机制。

## UDP 内部工作原理

![image](https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240308211201-dygyd99.png)

UDP 最重要的作用就是根据端口号将传到主机的数据包交付给最终的 UDP 套接字。

## UDP 的高效使用

UDP 并非每次都快于 TCP，TCP 比 UDP 慢的原因通常有以下两点：

1. 收发数据前后进行的连接设置及清除过程。
2. 收发数据过程中为保证可靠性而添加的流控制。

如果收发的数据量小但需要频繁连接时，UDP 比 TCP 更高效。

# 6.2 实现基于 UDP 的服务器端/客户端

## UDP 中的服务器端和客户端没有连接

UDP 服务器端/客户端无需经过连接过程。也就是说，不必调用 TCP 连接过程中调用的 `listen`​ 和 `accept`​ 函数。UDP 中只有创建套接字的过程和数据交换过程。

## UDP 服务器端和客户端均只需 1 个套接字

在 UDP 中，不管是服务器还是客户端都只需要 1 个套接字就可以和多台主机通信。

## 基于 UDP 的数据 I/O 函数

UDP 不会保持连接状态，因此每次传输数据都要添加目标地址信息。

```c
#include <sys/socket.h>

// sock: 用于传输数据的 UDP 套接字文件描述符
// buff: 保存待传输数据的缓冲地址值
// nbytes: 待传输的数据长度，以字节为单位
// flags: 可选项参数，若没有则传递 0
// to: 存有目标地址信息的 sockaddr 结构体变量的地址值
// addrlen: 传递给参数 to 的地址值结构体变量长度
ssize_t sendto(int sock, void *buff, size_t nbytes, int flags, struct sockaddr *to, socklen_t addrlen);
// 成功时返回传输的字节数，失败时返回-1
```

UDP 数据的发送端并不固定，因此该函数定义为可接受发送端信息的形式，也就是将同时返回 UDP 数据包中的发送端信息。

```c
#include <sys/socket.h>

// sock: 用于接收数据的 UDP 套接字文件描述符
// buff: 保存接收数据的缓冲地址值
// nbytes: 可接收的最大字节数，故无法超过参数 buff 所指的缓冲大小
// flags: 可选项参数，若没有则传递 0
// from: 存有发送端地址信息的 sockaddr 结构体变量的地址值
// addrlen: 传递给参数 to 的地址值结构体变量长度
ssize_t recvfrom(int sock, void* buff, size_t nbytes, int flags, struct sockaddr* from, socklen_t *addrlen);
// 成功时返回接收的字节数，失败时返回-1
```

## UDP 客户端套接字的地址分配

UDP 中缺少把 IP 和端口分配给套接字的过程。

UDP 程序中，调用 `sendto`​ 函数传输数据前应完成对套接字的地址分配工作，因此调用 `bind`​ 函数。

因此，调用 `sendto`​ 函数时自动分配 IP 和端口号，UDP 客户端中通常无需额外的地址分配过程。

# 6.3 UDP 的数据传输特性和调用 connect 函数

## 存在数据边界的 UDP 套接字

UDP 是具有数据边界的协议，传输中调用 I/O 函数的次数非常重要。因此，输入函数的调用次数应和输出函数的调用次数完全一致，这样才能保证接收全部已发生数据。

## 已连接 UDP 套接字与未连接 UDP 套接字

TCP 套接字中需注册待传输数据的目标 IP 和端口号，而 UDP 中则无需注册。因此，通过 `sendto`​ 函数传输数据的过程分为以下三个阶段：

1. 向 UDP 套接字注册目标 IP 和端口号。
2. 传输数据
3. 删除 UDP 套接字中注册的目标地址信息。

每次调用 `sento`​ 函数时重复上述过程。每次都变更目标地址，因此可以重复利用同一 UDP 套接字向不同目标传输数据。这种未注册目标地址信息的套接字称为未连接套接字，反之称为连接套接字。

## 创建已连接 UDP 套接字

创建已连接 UDP 套接字只需针对 UDP 套接字调用 `connect`​ 函数。

```c
sock = socket(PF_INET, SOCK_DGRAM, 0);
memset(&adr, 0, sizeof(adr));
adr.sin_family = AF_INEF;
adr.sin_addr.s_addr = ...;
adr.sin_port = ...;
connect(sock, (struct sockaddr*)&adr, sizeof(adr));
```

针对 UDP 套接字调用 `connect`​ 函数并不意味着要与对方 UDP 套接字连接，这只是向 UDP 套接字注册目标 IP 和端口信息。

# 7.1 基于 TCP 的半关闭

TCP 中的断开连接过程比建立连接过程更重要，因为连接过程中一般不会出现大的变数，但断开过程有可能发生预想不到的情况。

## 单方面断开连接带来的问题

Linux 的 `close`​ 函数和 Windows 的 `closesocket`​ 函数意味着完全断开连接。完全断开后就不能传输、接收数据。

半关闭 (*Half-close*)：可以传输数据但是无法接收，或者可以接收无法传输。

## 套接字和流

两台主机通过套接字建立了连接后进入可交换数据的状态，又称“流(*Stream*)形成的状态”。

![image](https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240310151207-3ac64h5.png)

一旦两台主机建立了套接字连接，每个主机就会拥有单独的输入流和输出流。

## 针对优雅断开的 shutdown 函数

```c
#include <sys/socket.h>

// sock: 需要断开的套接字文件描述符
// howto: 传递断开方式信息
int shutdown(int sock, int howto);
// 成功时返回 0， 失败时返回-1
```

第二个参数的可能值如下：

|  SHUT_RD  |   断开输入流    |
| :-------: | :-------------: |
|  SHUT_WR  |   断开输出流    |
| SHUT_RDWR | 同时断开 I/O 流 |

## 为何需要半关闭

略。看书 P120

# 8.1 域名系统

DNS 是对 IP 地址和域名进行相互转换的系统，其核心是 DNS 服务器。

域名：将容易记、易表述的域名分配并取代 IP 地址。

## DNS 服务器

域名和 IP 地址的接入过程不同。域名是赋予服务器端的虚拟地址，而非实际地址。因此，需要将虚拟地址转化为实际地址。

所有计算机中都记录着默认 DNS 服务器地址，就是通过这个默认 DNS 服务器得到相应域名的 IP 信息。在浏览器地址栏输入域名后，浏览器通过默认 DNS 服务器获取该域名对应的 IP 地址信息，之后才真正接入该网站。

计算机内置的默认 DNS 服务器并不知道网络上所有域名的 IP 地址信息。若该 DNS 服务器无法解析，则会询问其他 DNS 服务器，并提供给用户。

![image](https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240310160000-u4dmkze.png)

# 8.2 IP 地址和域名之间的转换

## 利用域名获取 IP 地址

```c
#include <netdb.h>

struct hostent *gethostbyname(const char* hostname);
// 成功时返回 hostent 结构体地址，失败时返回 NULL 指针
```

hostent 结构体定义如下：

```c
struct hosent {
	char *h_name;			// official name: 官方域名
	char **h_aliases;		// alias list: 可以通过多个域名访问同一主页， 同一 IP 可以绑定多个域名
	int h_addrtype;			// host address type: 通过此变量获取保存在 h_addr_list 的 IP 地址的地址族信息
	int h_length;			// address length: 保存 IP 地址长度
	char **h_addr_list;		// address list: 通过此变量以整数形式保存域名对应的 IP 地址
}
```

![image](https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240312192657-2x83cf1.png)

## 利用 IP 地址获取域名

```c
#include <netdb.h>

// addr: 含有 IP 地址信息的 in_addr 结构体指针
// len: 向第一个参数传递的地址信息的字节数
// family: 传递地址族信息
struct hostent* gethostbyaddr(const char* addr, socklen_t len, int family);
// 成功时返回 hostent 结构体变量地址值，失败时返回 NULL 指针
```

# 9.1 套接字可选项和 I/O 缓冲大小

## 套接字多种可选项

![套接字可选项](https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/202403122046231.png)

套接字可选项是分层的。IPPROTO_IP层是IP协议相关事项，IPPROTO_TCP层可选项是TCP协议相关的事项，SOL_SOCKET层是套接字相关的通用可选项。

## getsockopt & setsockopt

```c
#include <sys/socket.h>

// sock: 用于查看选项套接字文件描述符
// level: 要查看的可选项的协议层
// optname: 要查看的可选项名
// optval: 保存查看结果的缓冲地址值
// optlen: 向第四个参数optval传递的缓冲大小
int getsockopt(int sock, int level, int optname, void *optval, socklen_t *optlen);
// 成功时返回0，失败时返回-1
```

```c
#include <sys/socket.h>

// sock: 用于更改可选项的套接字文件描述符
// level: 要更改的可选项协议层
// optname: 要更改的可选项名
// optval: 保存要更改的选项信息的缓冲地址值
// optlen: 向第四个参数optval传递的可选项信息的字节数
int setsockopt(int sock, int level, int optname, const void *optval, socklen_t optlen);
// 成功时返回0，失败时返回-1
```

## SO_SNDBUF & SO_RCVBUF

SO_RCVBUF是输入缓冲大小相关可选项，SO_SNDBUF是输出缓冲大小相关可选项。

# 9.2 SO_REUSEADDR

## 发生地址分配错误（Binding Error）

如果客户端率先终止连接不会出现问题。

但如果服务器端使用 <kbd>CTRL+C</kbd> 时，如果在短时间内立即使用相同端口重启服务端，则会出现 `bind() error`，并且无法再次运行。

## Time-wait 状态

<img src="https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240312234620142.png" alt="image-20240312234620142" style="zoom: 50%;" />

套接字经过四次握手过程后并非立即消除，而是要经过一段时间的 `Time-wait` 状态。因此，若服务器先断开连接，则无法立即重新运行。套接字处在 `Time-wait` 过程时，相应端口是正在使用的状态。

## 地址再分配

<img src="https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240312235234654.png" alt="image-20240312235234654" style="zoom: 50%;" />

如图所示，在主机A的四次挥手过程中，如果最后的数据丢失，则主机B会认为主机A未能收到自己发送的FIN消息，因此重传。这时，收到FIN消息的主机A将重启 `Time-wait` 计时器。因此，如果网络状态不理想，`Time-wait` 状态将持续。

解决方案就是在套接字的可选项中更改SO_REUSEADDR的状态。适当调整该参数，可将 `Time-wait` 状态下的套接字端口号重新分配给新的套接字。

SO_REUSEADDR的默认值为0，意味着无法分配 `Time-wait` 状态下的套接字端口号。因此需要将这个值改成1。

# 9.3 TCP_NODELAY

## Nagle算法

Nagle算法用于防止因数据包过多而发生网络过载。它应用于TCP层，非常简单。

![Nagle算法](https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240313233848695.png)

>   只有收到前一数据的ACK消息时，Nagle算法才发送下一数据。

TCP套接字默认使用Nagle算法交换数据，因此最大限度地进行缓冲，直到收到ACK。

根据传输数据的特性，网络流量未受太大影响时，不使用Nagle算法要比使用它时传输速度快。

一般情况下，不使用Nagle算法可以提高传输速度。但如果无条件放弃使用Nagle算法，就会增加过多的网络流量，反而会影响传输。因此，未准确判断数据特性时不应禁用Nagle算法。

## 禁用Nagle算法

>   Nagle算法使用与否在网络流量上差别不大，使用Nagle算法的传输速度更慢。

禁用方法：将套接字可选项TCP_NODELAY改为1（TRUE）即可。

```c
int opt_val = 1;
setsockopt(sock, IPPROTO_TCP, TCP_NODELAY, (void*)&opt_val, sizeof(opt_val));
```

可以通过TCP_NODELAY的值查看Nagle算法的设置状态。

```c
int opt_val;
socklen_t opt_len;
opt_len = sizeof(opt_val);
getsockopt(sock, IPPROTO_TCP, TCP_NODELAY, (void*)&opt_val, &opt_len);
```

# 10.1 进程概念及应用

## 并发服务器端的实现方法

-   多进程服务器：通过创建多个进程提供服务
-   多路复用服务器：通过捆绑并统一管理I/O对象提供服务
-   多线程服务器：通过生成与客户端等量的线程提供服务

## 理解进程（Process）

进程：占用内存空间的正在运行的程序。

从OS的角度看，进程是程序流的基本单位，若创建多个进程，则操作系统将同时运行。

## 进程ID

无论进程是如何创建的，所有进程都会从操作系统分配到ID。此ID称为“进程ID”，其值为大于2的整数。1要分配给OS启动后的（用于协助OS）首个进程，因此用户进程无法得到ID值1。

## 通过调用fork函数创建进程

```c
#include <unistd.h>

pid_t fork(void);
// 成功时返回进程ID，失败时返回-1
```

`fork` 函数将创建调用的进程副本。也就是说，并非根据完全不同的程序创建进程，而是复制正在运行的、调用 `fork` 函数的进程。

利用 `fork` 函数的如下特点区分程序执行流程：

-   父进程：`fork` 函数返回子进程ID
-   子进程：`fork` 函数返回0

![fork函数的调用](https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240314004126274.png)

# 10.2 进程和僵尸进程

如果未认真对待进程销毁，它们将变成僵尸进程困扰各位。

## 僵尸（Zombie）进程

进程完成工作后（执行完main函数中的程序后）应该被销毁，但有时这些进程将变成僵尸进程，占用系统中的重要资源。这种状态下的进程称作“僵尸进程”，这也是给系统带来负担的原因之一。

## 产生僵尸进程的原因

首先利用如下两个实例展示调用 `fork` 函数产生子进程的终止方式。

-   传递参数并调用 `exit` 函数
-   `main` 函数中执行 `return` 语句并返回值

向 `exit` 函数传递的参数值和 `main` 函数的 `return` 语句返回的值都会传递给操作系统。而操作系统不会销毁子进程，直到把这些值传递给产生该子进程的父进程。处在这种状态下的进程就是僵尸进程。

## 销毁僵尸进程1：利用wait函数

如前所述，为了销毁子进程，父进程应该主动请求获取子进程的返回值。

```c
#include <sys/wait.h>

pid_t wait(int *statloc);
// 成功时返回终止的子进程ID，失败时返回-1
```

调用此函数时如果已有子进程终止，那么子进程终止时传递的返回值（`exit` 函数的参数值、`main`函数的`return`返回值）将保存到该函数的参数所指内存空间。但函数参数指向的单元中还包含其他信息，因此需要通过下列宏进行分离：

-   `WIFEXITED` 子进程正常终止时返回 `TRUE`
-   `WEXITSTATUS`返回子进程的返回值

也就是说，向 `wait` 函数传递变量 `status` 的地址时，调用 `wait` 函数后应编写如下代码：

```c
if (WIFEXITED(status)) {
    puts("Normal termination!");
    printf("Child pass num: %d", WEXITSTATUS(status));
}
```

调用 `wait` 函数时，如果没有已终止的子进程，那么程序将阻塞直到有子进程终止。

## 销毁僵尸进程2：使用waitpid函数

使用 `waitpid` 函数可以防止阻塞。

```c
#include <sys/wait.h>

// pid: 等待终止的目标子进程ID，若传递-1，则与wait函数相同
// statloc: 与wait函数的statloc参数具有相同含义
// options: 传递头文件中声明的常量，即使没有终止的子进程也不会进入阻塞状态，而是返回0并退出函数
pid_t waitpid(pid_t pid, int * statloc, int options);
// 成功时返回终止的子进程ID（或0），失败时返回-1
```

# 10.3 信号处理

## 向操作系统求助

子进程终止的识别主体是操作系统，因此，若操作系统能把停止信息告诉正忙于工作的父进程，将有助于构建高效的程序。

引入信号处理（*Signal Handling*）机制。此处的“信号”是在特定事件发生时由操作系统向进程发送的信息。

另外，为了响应该消息，执行与消息相关的自定义操作的过程称为“处理”或“信号处理”。

## 信号与Signal函数

进程发现自己的子进程结束时，请求操作系统调用特定函数。该请求通过如下函数调用完成（因此称此函数为信号注册函数）。

```c
#include <signal.h>

void (*signal(int signo, void (*func)(int)))(int);
// 为了在产生信号时调用，返回之前注册的函数指针。
```

调用上述函数时，第一个参数为特殊情况信息，第二个参数为特殊情况下将要调用的函数的地址值（指针）。发生第一个参数代表的情况时，调用第二个参数所指的指针。

下面给出可以在signal函数中注册的部分特殊情况和对应的常数。

-   SIGALRM: 已到通过调用 `alarm` 函数注册的时间。
-   SIGINT: 输入 <kbd>CTRL+C</kbd>。
-   SIGCHLD: 子进程终止。

```c
#include <unistd.h>

unsigned int alarm(unsigned int seconds);
// 返回0或以秒为单位的距SIGALRM信号发生所剩时间
```

如果调用该函数的同时向它传递一个正整形参数，相应时间后将产生SIGALRM信号。若向该函数传递0，则之前对SIGALRM信号的预约将取消。如果通过该函数预约信号后未指定该信号对应的处理函数，则终止进程，不做任何处理。

>   发生信号时将唤醒由于调用sleep函数而进入阻塞状态的进程。

## 利用sigaction函数进行信号处理

`sigaction` 函数类似于 `signal` 函数，而且可以完全代替后者，也更稳定。

>   signal 函数在 UNIX 系列的不同操作系统可能存在区别，但 sigaction 函数完全相同。

```c
#include <signal.h>

// signo: 与signal函数相同，传递信号信息
// act: 对应第一个参数的信号处理函数信息
// oldact: 通过此参数获取之前注册的信号处理函数指针
int sigaction(int signo, const struct sigaction* act, struct sigaction *oldact);
// 成功时返回0，失败时返回-1
```

声明并初始化 `sigaction` 结构体变量以调用上述函数，该结构体定义如下：

```c
struct sigaction {
	void (*sa_handler)(int);		// 保存信号处理函数的指针值
    sigset_t sa_mask;				// 指定信号相关的选项和特性
    int sa_flags;					// 同上，初始化为0即可
};
```

# 10.4 基于多任务的并发服务器

## 基于进程的并发服务器模型

<img src="https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240317105419055.png" alt="image-20240317105419055" style="zoom:50%;" />

从上图可以看出，每当有客户端请求服务时，回声服务器端都创建子进程以提供服务。请求服务的客户端若有5个，则创建5个子进程提供服务。为了完成这些任务，需要经过如下过程：

1.   回声服务器端（父进程）通过调用 `accept` 函数受理连接请求。
2.   此时获取的套接字文件描述符创建并传递给子进程。
3.   子进程利用传递来的文件描述符提供服务。

## 通过fork函数复制文件描述符

调用 `fork` 函数后，2个文件描述符将指向同一套接字。

<img src="https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240317114458925.png" alt="image-20240317114458925" style="zoom:50%;" />

如上图所示，1个套接字中存在2个文件描述符时，只有2个文件描述符都终止后，才能销毁套接字。如果维持图中看的连接状态，即使子进程销毁了与客户端连接的套接字文件描述符，也无法完全销毁套接字。因此，调用 `fork` 函数后，要将无关的套接字文件描述符关掉。

<img src="https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240317115320794.png" alt="image-20240317115320794" style="zoom:50%;" />

# 10.5 分割TCP的I/O程序

 ## 分割I/O程序的优点

目前已实现的回声客户端的数据回声方式如下：

>   向服务器端传输数据，并等待服务器端回复。无条件等待，直到接受完服务器端的回声数据后，才能传输下一批数据。

传输数据后需要等待服务器端返回的数据，因为程序代码中重复调用了 `read` 和 `write` 函数。但现在可以创建多个进程，因此可以分割数据收发过程。

<img src="https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240317140526718.png" alt="image-20240317140526718" style="zoom:50%;" />

从上图可以看出，客户端的父进程负责接收数据，额外创建的子进程负责发生数据。分割后，不同进程分别负责输入和输出，这样，无论客户端是否从服务器端接收完数据都可以进行传输。

分割I/O程序的另一个好处是，可以提高频繁交换数据的程序性能。

<img src="https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240317140817055.png" alt="image-20240317140817055" style="zoom:50%;" />

上图左侧演示的是之前的回声客户端数据交换方式，右侧演示的是分割I/O后的客户端数据传输方式。分割I/O后的客户端发送数据时不必考虑接收数据的情况，因此可以连续发生数据。

# 11.1 进程间通信的基本概念

进程间通信意味着两个不同进程间可以交换数据，为了完成这一点，操作系统应提供两个进程可以同时访问的内存空间。

## 通过管道实现进程间通信

<img src="https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240318160356199.png" alt="image-20240318160356199" style="zoom:50%;" />

上图表示基于管道（PIPE）的进程间通信结构模型。管道并非属于进程的资源，而是和套接字一样，属于操作系统。所以，两个进程通过操作系统提供的内存空间进行通信。

```c
#include <unistd.h>

// filedes[0]：通过管道接收数据时使用的文件描述符，即管道出口
// filedes[1]：通过管道传输数据时使用的文件描述符，即管道入口
int pipe(int filedes[2]);
// 成功时返回0，失败时返回-1
```

父进程调用该函数时将创建管道，同时获取对应于出入口的文件描述符，此时父进程可以读写同一管道。

示例代码 `pipe1.c` 的通信方法及路径如下图所示。重点在于，父子进程都可以访问管道的I/O路径，但子进程仅用于输入路径，父进程仅用于输出路径。

<img src="https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240318161640774.png" alt="image-20240318161640774" style="zoom:50%;" />

## 通过管道进行进程间双向通信

<img src="https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240318161902391.png" alt="image-20240318161902391" style="zoom:50%;" />

如果注释掉 `pipe2.c` 中的 `sleep(2)` 后再运行将会引发运行错误。

>   向管道传递数据时，先读的进程会把数据取走。

数据进入管道后成为无主数据。也就是通过 `read` 函数先读取数据的进程将得到数据，即使该进程将数据传到了管道。

1个管道无法完成双向通信任务，因此需要创建2个管道，各自负责不同的数据流动即可。

<img src="https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240318163412040.png" alt="image-20240318163412040" style="zoom:50%;" />

# 11.2 运用进程间通信

## 保存消息的回声服务器端

给第10章中的 `echo_mpserv.c` 添加如下功能：

>   将回声客户端传输的字符串按序保存到文件中。

我希望将该任务委托给另外的进程。换言之，另行创建进程，从向客户端提供服务的进程读取字符串信息。

# 12.1 基于I/O复用的服务器端

## 多进程服务器端的缺点和解决方法

I/O复用技术可以在不创建进程的同时向多个客户端提供服务。

## 理解复用

复用：在一个通信频道中传递多个数据（信号）的技术。

>   为了提高物理设备的效率，用最少的物理要素传递最多数据时使用的技术。

## 复用技术在服务端的应用

服务器端引入复用技术可以减少所需进程数。

第10章的多进程服务器端模型如下：

![image-20240319135511636](https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240319135511636.png)

引入复用技术后，服务器端模型如下，无论连接多少客户端，提供服务的进程只有1个：

![image-20240319135720833](https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240319135720833.png)

# 12.2 理解 select 函数并实现服务器端

## select 函数的功能和调用顺序

使用 `select` 函数时可以将多个文件描述符集中到一起统一监视，功能如下：

-   是否存在套接字接收数据？
-   无需阻塞传输数据的套接字有哪些？
-   哪些套接字发生了异常？

`select` 函数的使用方法与一般函数区别较大，很难使用。下图是 `select` 函数的调用方法和顺序。

<img src="https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240319141531357.png" alt="image-20240319141531357" style="zoom:50%;" />

## 设置文件描述符

利用 `select` 函数可以同时监视多个文件描述符。监视文件描述符可以视为监视套接字。此时首先需要将要监视的文件描述符集中到一起。集中时也要按照监视现进行区分——即接收、传输、异常。

使用 `fd_set` 数组变量执行此项操作，该数组是存有0和1的位数组。

![image-20240319142739115](https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240319142739115.png)

上图中最左端的位表示文件描述符0（所在位置）。如果该位设为1，则表示该文件描述符是监视对象。

在 `fd_set` 变量中注册或更改值的操作都由下列宏完成。

-   `FD_ZERO(fd_set* fdset)`：将 `fd_set` 变量的所有位初始化为0。
-   `FD_SET(int fd, fd_set* fdset)`：在参数 `fdset` 指向的变量中注册文件描述符 `fd` 的信息。
-   `FD_CLR(int fd, fd_set* fdset)`：从参数 `fdset` 指向的变量中清除文件描述符 `fd` 的信息。
-   `FD_ISSET(int fd, fd_set* fdset)`：若参数 `fdset` 指向的变量中包含文件描述符 `fd` 的信息，则返回 `TRUE`。

<img src="https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240319144417461.png" alt="image-20240319144417461" style="zoom:50%;" />

## 设置检查（监视）范围及超时

```c
#include <sys/select.h>
#include <sys/time.h>

// maxfd：监视对象文件描述符数量
// readset：将所有关注“是否存在待读取数据”的文件描述符注册到fd_set型变量，并传递其地址值。
// writeset：将所有关注“是否可传输无阻塞数据”的文件描述符注册到fd_set型变量，并传递其地址值。
// exceptset：将所有关注“是否发生异常”的文件描述符注册到fd_set型变量，并传递其地址值。
// timeout：调用select函数后，为防止陷入无限阻塞的状态，传递超时信息。
int select(int maxfd, fd_set* readset, fd_set* writeset, fd_set* exceptset, const struct timeval* timeout);
// 发生错误返回-1，超时返回0，因发生关注的事件返回大于0的值，该值是发生事件的文件描述符。
```

`select` 函数用来验证3种监视项的变化情况。根据监视项声明3个 `fd_set` 型变量，分别向其注册文件描述符信息，并把变量的地址值传递到上述函数的第二到第四个参数。

在调用 `select` 函数之前需要决定下面两件事：

1.   文件描述符的监视（检查）范围是？
2.   如何设定 `select` 函数的超时时间？

第一，文件描述符的监视范围与 `select` 函数的第一个参数有关。`select` 函数要求通过第一个参数传递监视对象文件描述符的数量。

第二，`select` 函数的超时时间与 `select` 函数的最后一个参数有关，其中 `timeval` 结构体定义如下。

```c
struct timeval {
	long tv_sec;			// seconds
    long tv_usec;			// microseconds
};
```

如果不想设置超时，则传递NULL参数。

## 调用select函数后查看结果

`select` 函数返回正整数时，怎样获知哪些文件描述符发生了变化？向 `select` 函数的第二到第四个参数传递的 `fd_set` 变量中将产生下图所示变化：

![image-20240319151704870](https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240319151704870.png)

由上图可知，`select` 函数调用完成后，向其传递的 `fd_set` 变量中将发生变化。原来为1的所有位均变为0，但发生变化的文件描述符对应位除外。因此，可以认为值仍为1的位置上的文件描述符发生了变化。

# 13.1 send & recv 函数

## Linux中的 send & recv

```c
#include <sys/socket.h>

// sockfd：表示与数据传输对象的连接的套接字文件描述符。
// buf：保存待传输数据的缓冲地址值。
// nbytes：待传输的字节数。
// flags：传输数据时指定的可选项信息。
ssize_t send(int sockfd, const void *buf, size_t nbytes, int flags);
// 成功时返回发送的字节数，失败时返回-1
```

```c
#include <sys/socket.h>

// sockfd：表示数据接收对象的连接的套接字文件描述符。
// buf：保存接收数据的缓冲地址值。
// nbytes：可接受的最大字节数。
// flags：接收数据时指定的可选项信息。
ssize_t recv(int sockfd, void *buf, size_t nbytes, int flags);
// 成功时返回接收的字节数（收到EOF时返回0），失败时返回-1
```

`send` 函数和 `recv` 函数的最后一个参数是收发数据时的可选项。该可选项可利用位或（bit OR）运算同时传递多个信息。

![image-20240320145926158](https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240320145926158.png)

## MSG_OOB：发送紧急信息

MSG_OOB可选项用于发送“带外数据”紧急消息。

MSG_OOB可选项就用于创建特殊发送方式和通道以发生紧急消息。

``` c
fcntl(recv_sock, F_SETOWN, getpid());
```

`fcntl` 函数用于控制文件描述符，上述调用语句的含义如下：

>   将文件描述符 `recv_sock` 指向的套接字拥有者 （F_SETOWN）改为把 `getpid` 函数返回值用作 ID 的进程。 

>   文件描述符 `recv_sock` 指向的套接字引发的 SIGURG 信号处理进程变为将 `getpid` 函数返回值用作ID的进程。

通过 MSG_OOB 可选项传递数据时不会加快数据传输速度，而且通过信号处理函数 `urg_handler` 读取数据时也只能读1个字节。剩余数据只能通过未设置 MSG_OOB 可选项的普通输入函数读取。这是因为 TCP 不存在真正意义上的“带外数据”。

>   带外数据：通过完全不同的通信路径传输的数据。

即真正意义上的 Out-of-band 需要通过单独的通信路径高速传输数据，但 TCP 不另外提供，只能利用 TCP 的紧急模式进行传输。

## 紧急模式工作原理

MSG_OOB的真正意义在于督促数据接收对象尽快处理数据。这是紧急模式的全部内容，而且TCP“保持传输顺序”的传输特性依然成立。

后略。

# 14.1 多播

多播（*Multicast*）方式的数据传输是基于UDP完成的。区别在于，UDP数据传输以单一目标进行，而多播数据同时传递到加入（注册）特定组的大量主机。

## 多播的数据传输方式及流量方面的优点

多播的数据传输特点：

-   多播服务器端针对特定多播组，只发送一次数据。
-   即使只发送一次数据，但该组内的所有客户端都会接收数据。
-   多播组数可在IP地址范围内任意增加。
-   加入特定组即可接收发往该多播组的数据。

多播是基于UDP完成的，也就是说，多播数据包的格式和UDP数据包相同。只是与一般的UDP数据包不同，向网络传递一个多播数据包时，路由器将复制该数据包并传递到多个主机。

<img src="https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240322011600070.png" alt="image-20240322011600070" style="zoom:50%;" />

## 路由（*Routing*）和TTL（*Time To Live*）以及加入组的方法

为了传递多数据包，必须设置TTL。TTL是决定“数据包传递距离”的主要因素。TTL用整数表示，并且每进过一个路由器就减一。TTL变成0时，该数据包无法再被传递，只能销毁。

TTL的值设置过大将影响网络流量；设置过小会无法传递到目标。

<img src="https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240322012630768.png" alt="image-20240322012630768" style="zoom: 67%;" />

TTL设置方法。程序中的TTL设置是通过套接字可选项完成的。与设置TTL相关的协议层为 IPPROTO_IP，选项名为IP_MULTICAST_TTL。

可以用如下代码把TTL设置为64。

```c
int send_sock;
int time_live = 64;
....
send_sock = socket(PF_INET, SOCK_DGRAM, 0);
setsockopt(send_sock, IPPROTO_IP, IP_MULTICAST_TTL, (void*)&time_live, sizeof(time_live));
...
```

另外，加入多播组也通过设置套接字选项完成。加入多播组相关的协议层为IPPROTO_IP，选项名为 IP_ADD_MEMBERSHIP。可通过如下代码加入多播组。

``` c
int recv_sock;
struct ip_mreq join_adr;
....
recv_sock = socket(PF_INET, SOCK_DGRAM, 0);
....
join_adr.imr_multiaddr.s_addr = "多播组地址信息";
join_adr.imr_interface.s_addr = "加入多播组的主机地址信息";
setsockopt(recv_sock, IPPROTO_IP, IP_ADD_MEMBERSHIP, (void*)&join_adr, sizeof(join_adr));
....
```

`ip_mreq` 结构体如下：

```c
struct ip_mreq {
    struct in_addr imr_multiaddr;		// 写入加入的组IP地址
    struct in_addr imr_interface;		// 加入该组的套接字所属主机的IP地址
}
```

## 实现多播 Sender 和 Receiver

多播中用“Sender”和“Receiver”替代服务器端和客户端。此处的 Sender 是多播数据的发送主体，Receiver 是需要多播组加入过程的数据接收主体。

示例代码中，运行场景如下：

-   Sender：向AAA组广播文件中保存的新闻信息。
-   Receiver：接收传递到AAA组的新闻信息。

# 14.2 广播

多播在跨越不同网络的情况下，只要加入多播组就能接收数据。相反，广播只能向同一网络中的主机传输数据。

## 广播的理解及实现方法

广播是向同一网络中的所有主机传输数据的方法。与多播相同，广播也是基于UDP完成的。

根据传输数据时使用的IP地址的形式，可以分为以下两种：

-   直接广播（*Directed Broadcast*）
-   本地广播（*Local Broadcast*）

二者的主要差别在于 IP 地址。

-   直接广播的 IP 地址中除了网络地址外，其余主机地址全部设置为1。
-   本地广播中使用的 IP 地址限定为 255.255.255.255。

在代码中调用 `setsockopt` 函数，将 `SO_BROADCAST` 选项设置为 `bcast` 变量中的值1，这意味着可以进行数据广播。

# 15.1 标准 I/O 函数的优点

## 标准 I/O 函数的两个优点

标准 I/O 函数的两大优点：

-   标准 I/O 函数具有良好的移植性。
-   标准 I/O 函数可以利用缓冲提高性能。

<img src="https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240403150012600.png" alt="image-20240403150012600" style="zoom:67%;" />

设置缓冲的主要目的是为了提高性能，但套接字中的缓冲主要是为了实现 TCP 协议而设立的。

实际上，缓冲并非在所有情况下都能带来卓越的性能。但需要传输的数据越多，有无缓冲带来的性能差异越大。

可以通过以下两种角度说明性能的提高：

-   传输的数据量
-   数据向输出缓冲移动的次数

## 标准 I/O 函数的几个缺点

-   不容易进行双向通信
-   有时可能频繁调用 `fflush` 函数
-   需要以 `FILE` 结构体指针的形式返回文件描述符

# 15.2 使用标准 I/O 函数

## 利用 `fdopen` 函数转换为 `FILE` 结构体指针

```c
#include <stdio.h>

// fildes: 需要转换的文件描述符
// mode: 将要创建的 FILE 结构体指针的模式信息
FILE* fdopen(int fildes, const char* mode);
// 成功时返回转换的 FILE 结构体指针，失败时返回 NULL
```

## 利用 `fileno` 函数转换为文件描述符

```c
#include <stdio.h>

int fileno(FILE* stream);
// 成功时返回转换后的文件描述符，失败时返回-1
```

# 16.1 分离 I/O 流

## 2 次 I/O 流分离

第 10 章的“TCP/IP 过程分离”，通过调用 `fork` 函数复制出 1 个文件描述符，以区分输入和输出中使用的文件描述符。

第 15 章的分离，通过 2 次 `fdopen` 函数的调用，创建读模式 `FILE` 指针（`FILE` 结构体指针）和写模式 `FILE` 指针。

## 分离“流”的好处

第 10 章的“流”分离目的：

-   通过分开输入过程和输出过程降低实现难度。
-   与输入无关的输出操作可以提高速度。

第 15 章的“流”分离目的：

-   为了将 `FILE` 指针按读模式和写模式加以区分。
-   可以通过区分读写模式降低实现难度。
-   通过区分 I/O 缓冲提高缓冲性能。

# 16.2 文件描述符的复制和半关闭

## 终止“流”时无法半关闭的原因

下图是 `sep.serv.c` 示例中的 2 个 `FILE` 指针、文件描述符及套接字之间的关系。

<img src="https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240406222211967.png" alt="image-20240406222211967" style="zoom:50%;" />

从图中可以看到，示例中的读模式 `FILE` 指针和写模式 `FILE` 指针都是基于同一文件描述符创建的。

因此，针对任意一个 `FILE` 指针调用 `fclose` 函数时都会关闭文件描述符，也就终止套接字。

<img src="https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240406222445921.png" alt="image-20240406222445921" style="zoom:50%;" />

从上图可以看到，销毁套接字时再也无法进行数据交换。

<img src="https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240406222611580.png" alt="image-20240406222611580" style="zoom:50%;" />

如上图所示，复制后另外创建一个文件描述符，然后利用各自的文件描述符生成读模式 `FILE` 指针和写模式 `FILE` 指针。

也就是说，针对写模式 `FILE` 指针调用 `fclose` 函数时，只能销毁与该 `FILE` 指针相关的文件描述符，无法销毁套接字。

<img src="https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240406222756926.png" alt="image-20240406222756926" style="zoom:50%;" />

## 复制文件描述符

![image-20240406232551509](https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20240406232551509.png)

上图给出的是同一进城内存在 2 个文件描述符可以同时访问文件的情况。

此处的“复制”具有如下含义：

>   为了访问同一文件或套接字，创建另一个文件描述符。

## dup & dup2

```c
#include <unistd.h>

// fildes: 需要复制的文件描述符
// fildes2: 明确指定的文件描述符整数值
int dup(int fildes);
int dup2(int fildes, int fildes2);
// 成功时返回复制的文件描述符，失败时返回-1
```

`dup2` 函数明确指定复制的文件描述符整数值。向其传递大于 0 且小于进程能生成的最大文件描述符值时，该值将成为复制出的文件描述符值。

# 17.1 epoll 理解及应用

## 基于 select 的 I/O 复用技术速度慢的原因

-   调用 `select` 函数后常见的针对所有文件描述符的循环语句。
-   每次调用 `select` 函数时都需要向该函数传递监视对象信息。

调用 `select` 函数后，并不是把发生变化的文件描述符单独集中到一起，而是通过观察作为监视对象的 `fd_set` 变量的变化，找出发生变化的文件描述符，因此无法避免针对所有监视对象的循环语句。

`select` 函数的这一缺点可以通过如下方式弥补：

>   仅向操作系统传递 1 次监视对象，监视范围或内容发生变化时只通知发生变化的事项。

这样就无需每次调用 `select` 函数时都向操作系统传递监视对象信息，但前提是操作系统支持这种处理方式。

## select 也有优点

只要满足或要求如下两个条件，即使在 Linux 平台也不应拘泥于 epoll。

-   服务器端接入者少。
-   程序应具有兼容性。

## 实现 epoll 时必要的函数和结构体

能够克服 `select` 函数缺点的 `epoll` 函数具有如下优点，这些优点正好与之前的 `select` 函数缺点相反。

-   无需编写以监视状态变化为目的的针对所有文件描述符的循环语句。
-   调用对应于 `select` 函数的 `epoll_wait` 函数时无需每次传递监视对象信息。

下面是 `epoll` 服务器端实现中需要的 3 个函数：

-   `epoll_create`：创建保存 `epoll` 文件描述符的空间。
-   `epoll_ctl`：向空间注册并注销文件描述符。
-   `epoll_wait`：与 `select` 函数类似，等待文件描述符发生变化。

`select` 方式中为了保存监视对象文件描述符，直接声明了 `fd_set` 变量。但 `epoll` 方式下由操作系统负责保存监视对象文件描述符，因此需要向操作系统请求创建保存文件描述符的空间，此时使用的函数就是 `epoll_create`。

此外，为了添加和删除监视对象文件描述符，`select` 方式中需要 `FD_SET`, `FD_CLR` 函数。但在 `epoll` 方式中，通过 `epoll_ctl` 函数请求操作系统完成。最后，`select` 方式下调用 `select` 函数等待文件描述符的变化，而 `epoll` 中调用 `epoll_wait` 函数。还有，`seletc` 方式中通过 `fd_set` 变量查看监视对象的状态变化，而`epoll` 方式中通过如下结构体 `epoll_event` 将发生变化的文件描述符单独集中到一起。

```c
struct epoll_event
{
    __uint32_t events;
    epoll_data_t data;
}

typedef union epoll_data{
    void* ptr;
    int fd;
    __uint32_t u32;
    __uint64_t u64;
} epoll_data_t;
```

## epoll_create

```c
#include <sys/epoll.h>

// size：epoll 实例的大小
int epoll_create(int size);
// 成功时返回 epoll 文件描述符，失败时返回-1
```

调用 `epoll_create` 函数时创建的文件描述符保存空间称为“epoll 例程”。

`epoll_create` 函数创建的资源与套接字相同，也由操作系统管理。因此，该函数和创建套接字的情况相同，也会返回文件描述符。

## epoll_ctl

```c
#include <sys/epoll.h>

// epfd：用于注册监视对象的 epoll 例程的文件描述符。
// op：用于指定监视对象的添加、删除或更改等操作。
// fd：需要注册的监视对象文件描述符。
// event：监视对象的事件类型。
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
// 成功时返回0，失败时返回-1。
```

示例：`epoll_ctl(A, EPOLL_CTL_ADD, B, C);`

含义：“epoll 例程 A 中注册文件描述符 B，主要目的是监视参数 C 中的事件。”

示例：`epoll_ctl(A, EPOLL_CTL_DEL, B, NULL);`

含义：“从 epoll 例程 A 中删除文件描述符 B。”

以下是向 `epoll_ctl` 第二个参数传递的常量及含义：

-   `EPOLL_CTL_ADD`：将文件描述符注册到 epoll 例程。
-   `EPOLL_CTL_DEL`：从 epoll 例程中删除文件描述符。
-   `EPOLL_CTL_MOD`：更改注册的文件描述符的关注事件发生情况。

下面是 `epoll_event` 的成员 `events` 中可以保存的常量及所指的事件类型：

-   `EPOLLIN`：需要读取数据的情况。
-   `EPOLLOUT`：输出缓冲为空，可以立即发送数据的情况。
-   `EPOLLPRI`：收到 OOB 数据的情况。
-   `EPOLLREHUP`：断开连接或半关闭的情况。
-   `EPOLLERR`：发生错误的情况。
-   `EPOLLET`：以边缘触发的方式得到事件通知。
-   `EPOLLONESHOT`：发生一次事件后，相应文件描述符不再收到事件通知。

## epoll_wait

```c
#include <sys/epoll.h>

// epfd：表示事件发生监视范围的 epoll 例程的文件描述符。
// events：保存发生事件的文件描述符集合的结构体地址值。
// maxevents：第二个参数中可以保存的最大事件数。
// timeout：以 1/1000 秒为单位的等待时间，传递 -1 时，一直等待直到发生事件。
int epoll_wait(int epfd, struct epoll_event* events, int maxevents, int timeout);
// 成功时返回发生事件的文件描述符数，失败时返回-1。
```

