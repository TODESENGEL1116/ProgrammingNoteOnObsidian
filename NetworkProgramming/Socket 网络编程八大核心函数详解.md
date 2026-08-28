### 1. socket

#### 定义

创建一个通信端点，返回用于后续网络操作的套接字文件描述符，是所有网络通信的起点。

#### 函数原型

```c
#include <sys/socket.h>
int socket(int domain, int type, int protocol);
```

#### 参数说明

| 参数         | 说明                                                                     |
| ---------- | ---------------------------------------------------------------------- |
| `domain`   | 通信协议族，常用值：`AF_INET`（IPv4）、`AF_INET6`（IPv6）、`AF_UNIX`（本地进程通信）           |
| `type`     | 套接字类型，常用值：`SOCK_STREAM`（TCP字节流）、`SOCK_DGRAM`（UDP数据报）、`SOCK_RAW`（原始套接字） |
| `protocol` | 指定具体协议，通常填`0`由内核自动匹配对应协议族的默认协议，也可填`IPPROTO_TCP`、`IPPROTO_UDP`等         |

#### 返回值

- 成功：返回非负整数，为新创建的套接字文件描述符。
- 失败：返回`-1`，同时`errno`记录错误原因。

---

### 2. bind

#### 定义

将套接字绑定到指定的本地协议地址（IP+端口），服务端必须调用该函数固定监听端口，客户端通常无需调用。

#### 函数原型

```c
#include <sys/socket.h>
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

#### 参数说明

|参数|说明|
|---|---|
|`sockfd`|由`socket`函数返回的套接字文件描述符|
|`addr`|指向要绑定的本地地址结构体的指针，IPv4通常用`struct sockaddr_in`强转而来|
|`addrlen`|地址结构体的字节长度，通常填`sizeof(struct sockaddr)`|

#### 返回值

- 成功：返回`0`。
- 失败：返回`-1`，常见错误：`EADDRINUSE`（端口已被占用）、`EACCES`（无权限绑定特权端口）。

---

### 3. listen

#### 定义

将已绑定地址的套接字设置为被动监听状态，等待客户端的连接请求，仅适用于面向连接的`TCP`套接字。

#### 函数原型

```c
#include <sys/socket.h>
int listen(int sockfd, int backlog);
```

#### 参数说明

|参数|说明|
|---|---|
|`sockfd`|已绑定地址的套接字文件描述符|
|`backlog`|内核中该套接字对应已完成连接队列的最大长度，超过该值的连接请求会被拒绝|

#### 返回值

- 成功：返回`0`。
- 失败：返回`-1`，同时`errno`记录错误原因。

---

### 4. accept

#### 定义

从监听套接字的已完成连接队列中取出一个连接，返回新的套接字文件描述符用于和对应客户端通信，原监听套接字保持不变。

#### 函数原型

```c
#include <sys/socket.h>
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

#### 参数说明

|参数|说明|
|---|---|
|`sockfd`|监听套接字的文件描述符|
|`addr`|用于存储客户端地址信息的结构体指针，不需要时可填`NULL`|
|`addrlen`|传入时为`addr`结构体的大小，返回时被内核修改为实际存储的地址长度，不需要时可填`NULL`|

#### 返回值

- 成功：返回新的非负整数，为用于和客户端通信的已连接套接字文件描述符。
- 失败：返回`-1`，同时`errno`记录错误原因。

---

### 5. connect

#### 定义

客户端用于和指定服务端地址建立连接的函数，`TCP`协议下会触发三次握手流程。

#### 函数原型

```c
#include <sys/socket.h>
int connect(int sockfd, const struct sockaddr *serv_addr, socklen_t addrlen);
```

#### 参数说明

|参数|说明|
|---|---|
|`sockfd`|由`socket`函数返回的客户端套接字文件描述符|
|`serv_addr`|指向服务端地址结构体的指针，包含服务端IP和端口信息|
|`addrlen`|地址结构体的字节长度，通常填`sizeof(struct sockaddr)`|

#### 返回值

- 成功：返回`0`。
- 失败：返回`-1`，常见错误：`ECONNREFUSED`（服务端拒绝连接）、`ETIMEDOUT`（连接超时）。

---

### 6. send

#### 定义

在已连接的套接字上向对端发送数据，将用户态缓冲区的数据拷贝到内核态套接字发送缓冲区，由内核协议栈负责实际发送。

#### 函数原型

```c
#include <sys/socket.h>
ssize_t send(int sockfd, const void *buf, size_t len, int flags);
```

#### 参数说明

|参数|说明|
|---|---|
|`sockfd`|已连接的套接字文件描述符|
|`buf`|指向要发送的数据缓冲区的指针|
|`len`|要发送的数据的最大字节数|
|`flags`|发送控制标志，常用值：`0`（默认行为）、`MSG_DONTWAIT`（非阻塞发送）、`MSG_OOB`（发送带外数据）|

#### 返回值

- 成功：返回实际发送的字节数，可能小于`len`。
- 失败：返回`-1`，同时`errno`记录错误原因。

---

### 7. recv

#### 定义

在已连接的套接字上从对端接收数据，将内核态套接字接收缓冲区的数据拷贝到用户态缓冲区。

#### 函数原型

```c
#include <sys/socket.h>
ssize_t recv(int sockfd, void *buf, size_t len, int flags);
```

#### 参数说明

|参数|说明|
|---|---|
|`sockfd`|已连接的套接字文件描述符|
|`buf`|用于存储接收数据的缓冲区指针|
|`len`|缓冲区的最大可接收字节数|
|`flags`|接收控制标志，常用值：`0`（默认行为）、`MSG_DONTWAIT`（非阻塞接收）、`MSG_OOB`（接收带外数据）|

#### 返回值

- 成功：返回实际接收到的字节数；若返回`0`则表示对端已正常关闭连接。
- 失败：返回`-1`，同时`errno`记录错误原因。

---

### 8. close

#### 定义

关闭套接字文件描述符，释放对应资源。`TCP`协议下会触发四次挥手流程，将套接字标记为已关闭，不可再用于收发数据。

#### 函数原型

```c
#include <unistd.h>
int close(int sockfd);
```

#### 参数说明

|参数|说明|
|---|---|
|`sockfd`|要关闭的套接字文件描述符|

#### 返回值

- 成功：返回`0`。
- 失败：返回`-1`，同时`errno`记录错误原因。

---

需要我帮你整理一份TCP三次握手/四次挥手的时序图，方便你理解connect和close的底层过程吗？