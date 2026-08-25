POSIX Socket API 的头文件分布在多个层级：**核心套接字头文件 + 地址/协议族头文件 + 辅助功能头文件**。下面按"必含 → 常用 → 场景化"三个层级梳理清楚。

---

## 一、 核心套接字头文件（必含）

### `<sys/socket.h>` —— 主套接字头文件

这是 POSIX Socket API 的**最核心头文件**，几乎所有网络程序都要包含。它定义了 ：

- **基础类型**：`socklen_t`、`sa_family_t`
- **核心结构体**：`sockaddr`（通用套接字地址）、`sockaddr_storage`（大容量、可容纳所有协议地址）、`msghdr`、`cmsghdr`、`linger`
- **核心常量**：`SOCK_STREAM`、`SOCK_DGRAM`、`SOCK_RAW`、`SOL_SOCKET`、`SO_*` 等
- **核心函数原型的声明**：`socket()`、`bind()`、`connect()`、`listen()`、`accept()`、`send()`、`recv()`、`sendto()`、`recvfrom()`、`sendmsg()`、`recvmsg()`、`setsockopt()`、`getsockopt()`、`shutdown()`、`getsockname()`、`getpeername()`

> 💡 在 glibc 实现中，`<sys/socket.h>` 内部还会包含 `<sys/types.h>` 并引入操作系统特定的 `<bits/socket.h>`（定义 `AF_*`、`PF_*`、`MSG_*` 等常量）。

### `<sys/types.h>` —— 基础数据类型

定义 `socklen_t`、`size_t`、`ssize_t`、`pid_t` 等被 `<sys/socket.h>` 依赖的基础类型。虽然 glibc 的 `<sys/socket.h>` 会自动包含它，但**显式包含是良好的移植性习惯**​ 。

---

## 二、 地址与协议族头文件（按网络层选择）

### `<netinet/in.h>` —— IPv4 / IPv6 地址族 ★最常用

声明 ：

- `struct sockaddr_in`（IPv4 地址结构，含 `sin_family`、`sin_port`、`sin_addr`）
- `struct sockaddr_in6`（IPv6 地址结构）
- `INADDR_ANY`、`IN6ADDR_ANY_INIT` 等常量
- `htons()`、`htonl()`、`ntohs()`、`ntohl()` 等字节序转换函数（部分实现中也通过此头文件提供）

### `<arpa/inet.h>` —— 地址转换函数

提供 ：

- `inet_addr()`、`inet_ntoa()`（IPv4 字符串与点分十进制互转）
- `inet_pton()`、`inet_ntop()`（IPv4/IPv6 通用，推荐新代码使用）

### `<netinet/tcp.h>` —— TCP 协议选项

当你需要对 TCP 套接字设置协议特定选项（如 `TCP_NODELAY`、`TCP_KEEPALIVE`）时包含。

### `<sys/un.h>` —— Unix 域套接字

定义 `struct sockaddr_un`，用于**本机进程间通信（IPC）**的 Unix Domain Socket 。

### `<net/if.h>` —— 网络接口

提供套接字本地接口的相关定义，用于查询/操作网络接口信息。

---

## 三、 辅助功能头文件（按功能选择）

|头文件|主要用途|关键声明|
|---|---|---|
|**`<netdb.h>`**​|网络数据库操作（DNS 解析、服务名查询）|`getaddrinfo()`、`getnameinfo()`、`gethostbyname()`、`servent` 结构体|
|**`<sys/uio.h>`**​|向量 I/O 操作|`struct iovec`、`readv()`、`writev()`（被 `sendmsg`/`recvmsg` 使用）|
|**`<sys/select.h>`**​|`select()` 多路复用|`fd_set`、`select()`|
|**`<poll.h>`**​|`poll()` 多路复用|`pollfd` 结构体、`poll()`|
|**`<sys/epoll.h>`**​|Linux 特有的 epoll 多路复用|`epoll_create()`、`epoll_ctl()`、`epoll_wait()`|
|**`<unistd.h>`**​|基础 POSIX 函数|`close()`、`read()`、`write()`（套接字读写也可用）|
|**`<fcntl.h>`**​|文件控制|`fcntl()`，用于设置套接字非阻塞模式|
|**`<errno.h>`**​|错误处理|`errno`、`EAGAIN`、`ECONNRESET` 等错误码|

---

## 四、 实战：不同场景的最小头文件组合

### 📡 TCP/IP 网络程序（最常见）

```
#include <sys/types.h>
#include <sys/socket.h>     // socket/bind/connect/listen/accept 等核心函数
#include <netinet/in.h>     // sockaddr_in, htons, INADDR_ANY
#include <arpa/inet.h>      // inet_pton, inet_ntop
#include <netdb.h>          // getaddrinfo（现代写法推荐）
#include <unistd.h>         // close, read, write
```

### 🐧 Unix Domain Socket（本机 IPC）

```
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/un.h>         // sockaddr_un
#include <unistd.h>
```

### 🌐 使用 `select()` 做 I/O 多路复用

```
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/select.h>     // select, fd_set
#include <netinet/in.h>
#include <arpa/inet.h>
```

### ⚡ 使用 Linux `epoll()`

```
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/epoll.h>      // epoll 系列函数
#include <netinet/in.h>
#include <arpa/inet.h>
```

---

## 五、 POSIX 标准头文件清单中的套接字相关项

根据 POSIX 规范，下列头文件与套接字编程直接相关 ：

> `<net/if.h>`、`<netdb.h>`、`<netinet/in.h>`、`<netinet/tcp.h>`、`<sys/socket.h>`、`<sys/un.h>`、`<sys/uio.h>`、`<sys/select.h>`、`<poll.h>`

> ⚠️ **注意**：不同 UNIX 实现（Linux/glibc、FreeBSD、macOS、AIX 等）在头文件的细粒度划分上略有差异。例如在 IBM z/VM 上，当 `_OE_SOCKETS` 被定义时，`<sys/socket.h>` 中的 `accept()`、`bind()` 等函数才可用，但 `<netdb.h>` 中的 `getaddrinfo()` 等函数可能没有原型声明，需要特别注意 。

**可移植性最佳实践**：宁可多包含几个标准头文件，也不要依赖某个实现中"头文件 A 自动包含了头文件 B"的隐性行为。