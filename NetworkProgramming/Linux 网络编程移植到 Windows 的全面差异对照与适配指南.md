

## 一、头文件与库依赖

Linux 网络编程遵循 POSIX 标准，头文件分散在多个系统头文件中；Windows 则通过 Winsock 2 提供统一的网络 API 封装层，需要显式链接动态库。

### 1.1 头文件对照

| 功能域       | Linux                       | Windows                                             |
| --------- | --------------------------- | --------------------------------------------------- |
| Socket 核心 | `<sys/socket.h>`            | `<winsock2.h>`（**必须在 `<windows.h>` 之前包含**）          |
| 地址族/协议族   | `<netinet/in.h>`            | `<ws2tcpip.h>`（含 IPv6 支持）                           |
| 地址转换      | `<arpa/inet.h>`             | `<ws2tcpip.h>`（含 `inet_pton`/`inet_ntop`）           |
| 网络数据库     | `<netdb.h>`                 | `<ws2tcpip.h>`                                      |
| I/O 控制    | `<fcntl.h>`、`<sys/ioctl.h>` | `<winsock2.h>`（`ioctlsocket`）                       |
| 错误处理      | `<errno.h>`                 | `<winsock2.h>`（`WSAGetLastError`）                   |
| 未定义类型补齐   | —                           | 需手动 `typedef int socklen_t;`、`typedef int ssize_t;` |

### 1.2 链接库

- **Linux**：无需额外链接网络库，`socket`/`bind`/`listen` 等均为 glibc 封装的系统调用，编译时仅需标准 C 库（`-lc`，通常隐式链接）。在早期某些 Unix 变体上可能需要 `-lsocket -lnsl`，但在现代 Linux 中已完全弃用。
- **Windows**：必须显式链接 `ws2_32.lib`（对应运行时 `ws2_32.dll`）。MSVC 中使用 `#pragma comment(lib, "ws2_32.lib")`；GCC/MinGW 中使用 `-lws2_32`。

---

## 二、Winsock 初始化与清理

这是 Windows 网络编程独有的步骤，Linux 完全不存在此环节。

### 2.1 初始化（程序启动时）

Windows 要求在任何 Socket 操作之前调用 `WSAStartup()` 加载 Winsock DLL 并协商版本：

```c
// Windows 特有
WSADATA wsaData;
int result = WSAStartup(MAKEWORD(2, 2), &wsaData);
if (result != 0) {
    // 初始化失败，错误码通过 WSAGetLastError() 获取
}
```

- `MAKEWORD(2, 2)` 请求 Winsock 2.2 版本（当前标准版本）。
- 若不调用此函数，后续所有 `socket()` 调用将返回 `INVALID_SOCKET`，且 `WSAGetLastError()` 返回 `WSANOTINITIALISED`。

### 2.2 清理（程序退出前）

```c
// Windows 特有
WSACleanup();
```

- 释放 Winsock 实现分配的内部资源。
- `WSAStartup` 与 `WSACleanup` 必须成对调用，支持引用计数（多次 `WSAStartup` 需对应多次 `WSACleanup`）。

### 2.3 Linux 侧

Linux 内核在进程启动时自动管理网络协议栈资源，**无需任何初始化/清理操作**。

---

## 三、Socket 类型与句柄语义

### 3.1 类型差异

|项目|Linux|Windows|
|---|---|---|
|Socket 类型|`int`（文件描述符）|`SOCKET`（实际为 `UINT_PTR`，64 位系统上为 64 位无符号整数）|
|失败返回值|`-1`|`INVALID_SOCKET`（即 `(SOCKET)(~0)`）|
|通用错误判断|`fd == -1`|`sock == INVALID_SOCKET`|

### 3.2 语义差异

- **Linux**："一切皆文件"，Socket 是文件描述符（fd），与普通文件 fd 共享同一命名空间。可对 Socket 使用 `read()`/`write()`、`select()`/`poll()`/`epoll()`、`dup()`/`dup2()` 等所有文件操作。
- **Windows**：Socket 是独立的句柄对象，与普通文件句柄（`HANDLE`）**不互通**。不能对 Socket 使用 `ReadFile()`/`WriteFile()`（除非使用重叠 I/O 的特殊模式），也不能使用 `close()` 关闭——必须使用 `closesocket()`。

### 3.3 跨平台兼容宏

```c
#ifdef _WIN32
    typedef SOCKET socket_t;
    #define INVALID_SOCK INVALID_SOCKET
#else
    typedef int socket_t;
    #define INVALID_SOCK (-1)
#endif
```

---

## 四、关闭 Socket

|平台|函数|头文件|
|---|---|---|
|Linux|`close(fd)`|`<unistd.h>`|
|Windows|`closesocket(s)`|`<winsock2.h>`|

**关键区别**：

- Linux 的 `close()` 是通用文件描述符关闭函数，对 Socket fd 同样有效。
- Windows 的 `close()` **不能**用于关闭 Socket，必须使用专用的 `closesocket()`。若误用 `close()` 关闭 Socket，在 Windows 上会导致资源泄漏或行为未定义。
- 在需要优雅关闭时，两端均可先调用 `shutdown(fd, SHUT_WR)` 发送 FIN 包，再调用关闭函数。

---

## 五、错误处理机制

### 5.1 错误码获取方式

|项目|Linux|Windows|
|---|---|---|
|获取方式|全局变量 `errno`|函数调用 `WSAGetLastError()`|
|头文件|`<errno.h>`|`<winsock2.h>`|
|错误描述|`strerror(errno)` 或 `perror()`|`FormatMessage()` 或 `WSAGetLastError()` 配合查表|

### 5.2 错误码体系差异

两平台的错误码**命名不同但语义大部分可映射**：

|错误语义|Linux (errno)|Windows (WSA 错误码)|
|---|---|---|
|操作将阻塞|`EAGAIN` / `EWOULDBLOCK`|`WSAEWOULDBLOCK`|
|连接正在进行|`EINPROGRESS`|`WSAEINPROGRESS`|
|连接被拒绝|`ECONNREFUSED`|`WSAECONNREFUSED`|
|连接被重置|`ECONNRESET`|`WSAECONNRESET`|
|连接超时|`ETIMEDOUT`|`WSAETIMEDOUT`|
|地址已在使用|`EADDRINUSE`|`WSAEADDRINUSE`|
|参数无效|`EINVAL`|`WSAEINVAL`|
|中断的系统调用|`EINTR`|**无直接等价项**|

**特别注意**：

- Linux 的 `EINTR`（系统调用被信号中断）在 Windows Winsock 中**没有直接对应**。Windows 的 Winsock 函数不会被信号中断，因此移植时涉及 `EINTR` 重试逻辑的代码需要条件编译移除。
- Windows 有一些独有的错误码，如 `WSANOTINITIALISED`（未调用 WSAStartup）、`WSAESOCKTNOSUPPORT`（不支持的 Socket 类型）等，在 Linux 侧无对应。

### 5.3 跨平台错误获取宏

```c
#ifdef _WIN32
    #define GET_LAST_ERROR()  WSAGetLastError()
    #define WOULD_BLOCK(err)  ((err) == WSAEWOULDBLOCK)
    #define IN_PROGRESS(err)  ((err) == WSAEINPROGRESS)
#else
    #define GET_LAST_ERROR()  errno
    #define WOULD_BLOCK(err)  ((err) == EAGAIN || (err) == EWOULDBLOCK)
    #define IN_PROGRESS(err)  ((err) == EINPROGRESS)
#endif
```

---

## 六、非阻塞 I/O 模式设置

### 6.1 Linux：`fcntl()`

```c
#include <fcntl.h>

int flags = fcntl(sockfd, F_GETFL, 0);
if (flags == -1) { /* 错误处理 */ }
fcntl(sockfd, F_SETFL, flags | O_NONBLOCK);
```

### 6.2 Windows：`ioctlsocket()`

```c
#include <winsock2.h>

u_long mode = 1;  // 1 = 非阻塞, 0 = 阻塞
ioctlsocket(sockfd, FIONBIO, &mode);
```

### 6.3 差异要点

- Linux 使用 `fcntl()` 操作文件描述符标志，这是通用文件操作接口，对所有 fd 类型有效。
- Windows 使用专用的 `ioctlsocket()`，仅对 Socket 有效。
- 两者功能等价，但函数签名和参数完全不同，无法通过简单宏替换统一。

---

## 七、`send()` / `recv()` 函数参数差异

### 7.1 函数签名对比

```c
// Linux
ssize_t send(int sockfd, const void *buf, size_t len, int flags);
ssize_t recv(int sockfd, void *buf, size_t len, int flags);

// Windows
int send(SOCKET s, const char *buf, int len, int flags);
int recv(SOCKET s, char *buf, int len, int flags);
```

**关键差异**：

|项目|Linux|Windows|
|---|---|---|
|返回值类型|`ssize_t`（有符号，可为 -1）|`int`（可为 `SOCKET_ERROR` 即 -1）|
|缓冲区指针类型|`const void *` / `void *`|`const char *` / `char *`|
|长度参数类型|`size_t`（无符号）|`int`（有符号）|

### 7.2 `flags` 参数的关键差异——`MSG_NOSIGNAL`

**这是移植时最容易被忽略但后果严重的差异之一。**

- **Linux**：当对端关闭连接后，本端继续 `send()` 会触发 `SIGPIPE` 信号，默认行为是**终止整个进程**。必须在 `send()` 的 flags 参数中传入 `MSG_NOSIGNAL` 来抑制此信号：
    
    ```c
    send(sockfd, buf, len, MSG_NOSIGNAL);
    ```
    
    或者在程序初始化时全局忽略 `SIGPIPE`：
    
    ```c
    signal(SIGPIPE, SIG_IGN);
    ```
    
- **Windows**：**不存在 `SIGPIPE` 信号**，`send()` 在对端关闭时仅返回 `SOCKET_ERROR`，`WSAGetLastError()` 返回 `WSAECONNRESET`。flags 参数通常设为 `0`。
    

### 7.3 `read()` / `write()` 的使用

- **Linux**：Socket 是文件描述符，可直接使用 POSIX 文件 I/O 函数 `read()`/`write()` 进行收发。
- **Windows**：Socket 不是文件句柄，**不能**使用 `read()`/`write()`，必须使用 `recv()`/`send()`。

---

## 八、`select()` 函数的细微差异

虽然两端均支持 `select()` 且函数签名看似相同，但存在以下差异：

### 8.1 第一个参数 `nfds`

```c
// Linux：nfds 为监控的最大 fd + 1
int nfds = max_fd + 1;
select(nfds, &readfds, &writefds, &exceptfds, &timeout);

// Windows：nfds 参数被忽略，可传入任意值（通常传 0）
select(0, &readfds, &writefds, &exceptfds, &timeout);
```

### 8.2 `fd_set` 的内部实现

- **Linux**：`fd_set` 是一个位掩码数组（bitmask），通过位操作设置/检查 fd。`FD_SETSIZE` 默认为 1024。
- **Windows**：`fd_set` 是一个**整数数组**（`SOCKET` 数组 + 计数器），不是位掩码。这意味着 Windows 上 `fd_set` 不受 `FD_SETSIZE` 的位宽限制，但仍受数组大小约束。

### 8.3 Socket 类型对 `FD_SET` 的影响

由于 Windows 的 `SOCKET` 是 `UINT_PTR` 而非 `int`，直接使用 Linux 风格的位操作宏操作 `fd_set` 在 Windows 上不可行。必须使用标准的 `FD_SET()`、`FD_CLR()`、`FD_ISSET()`、`FD_ZERO()` 宏，这些宏在两端均有定义但内部实现不同。

---

## 九、I/O 多路复用与高性能事件模型

这是 Linux 与 Windows 网络编程中**最根本的架构性差异**，直接决定了高性能服务器的设计模式。

### 9.1 Linux：epoll（Reactor 模式）

```c
#include <sys/epoll.h>

int epfd = epoll_create1(0);
struct epoll_event ev;
ev.events = EPOLLIN | EPOLLET;  // 边缘触发
ev.data.fd = sockfd;
epoll_ctl(epfd, EPOLL_CTL_ADD, sockfd, &ev);

struct epoll_event events[MAX_EVENTS];
int n = epoll_wait(epfd, events, MAX_EVENTS, timeout_ms);
```

**核心特征**：

- **Reactor 模式**：epoll 通知应用"某个 fd 已就绪（可读/可写）"，应用程序自行执行 `read()`/`write()` 完成实际 I/O 操作。
- 支持**边缘触发（ET）**和**水平触发（LT）**两种模式。
- 通过红黑树管理 fd，事件通知通过回调机制实现，时间复杂度 O(1)（仅返回就绪事件）。
- 无硬性 fd 数量上限（受系统资源限制）。

### 9.2 Windows：IOCP（Proactor 模式）

```c
// 创建完成端口
HANDLE hIOCP = CreateIoCompletionPort(INVALID_HANDLE_VALUE, NULL, 0, 0);

// 将 Socket 绑定到完成端口
CreateIoCompletionPort((HANDLE)sockfd, hIOCP, completionKey, 0);

// 发起异步读操作（需要 OVERLAPPED 结构）
WSAOVERLAPPED overlapped = {0};
WSABUF dataBuf;
dataBuf.buf = buffer;
dataBuf.len = BUFFER_SIZE;
DWORD bytesRecv, flags;
WSARecv(sockfd, &dataBuf, 1, &bytesRecv, &flags, &overlapped, NULL);

// 工作线程等待 I/O 完成
DWORD bytesTransferred;
ULONG_PTR completionKey;
WSAOVERLAPPED *pOverlapped;
GetQueuedCompletionStatus(hIOCP, &bytesTransferred, &completionKey,
                          &pOverlapped, INFINITE);
// 此处 I/O 操作已经完成，数据已在 buffer 中
```

**核心特征**：

- **Proactor 模式**：应用发起异步 I/O 请求后，IOCP 在内核中完成实际的数据读写，完成后通知应用"I/O 已完成，数据已就绪"。
- 内核维护线程池，自动调度工作线程，默认线程数与 CPU 核心数匹配。
- 必须使用 `OVERLAPPED`（重叠 I/O）结构体，每个并发 I/O 操作需要独立的 `OVERLAPPED` 实例。
- 通过 `GetQueuedCompletionStatus()` 获取完成的 I/O 结果。

### 9.3 模型本质对比

|维度|Linux epoll|Windows IOCP|
|---|---|---|
|设计模式|**Reactor**（就绪通知）|**Proactor**（完成通知）|
|通知时机|"fd 可读/可写了，你来读/写"|"读/写已完成，数据在这里"|
|I/O 执行者|应用程序（用户态）|操作系统内核|
|触发模式|ET / LT|仅完成通知|
|线程管理|用户态手动管理线程池|内核自动管理线程池|
|缓冲区管理|应用控制读写缓冲区|应用提供缓冲区，内核填充|
|编程复杂度|中等|高（需管理 OVERLAPPED 生命周期）|

### 9.4 Windows 上的替代方案

Windows 也支持一些与 Linux 更接近的 I/O 模型，可作为过渡方案：

- **`select()`**：两端通用，但性能受限（fd 数量少时可用）。
- **`WSAEventSelect()`**：基于事件对象的通知模型，类似 epoll 的 Reactor 思路，配合 `WSAWaitForMultipleEvents()` 使用。
- **`WSAAsyncSelect()`**：基于窗口消息的异步通知（仅适用于有消息循环的 GUI 程序）。

### 9.5 移植建议

由于 epoll 与 IOCP 的架构差异太大，无法通过简单宏替换统一。工程上的推荐做法：

1. **使用跨平台网络库**：如 libuv（Node.js 底层）、Boost.Asio、Poco Net 等，它们在内部通过条件编译分别对接 epoll 和 IOCP。
2. **自行封装抽象层**：定义统一的 `EventLoop` 接口，Linux 实现基于 epoll，Windows 实现基于 IOCP 或 `WSAEventSelect`。

---

## 十、信号处理——`SIGPIPE` 问题

### 10.1 Linux 的 `SIGPIPE`

在 Linux 上，当向一个已被对端关闭的 TCP 连接执行 `write()` 或 `send()` 时，内核会向进程发送 `SIGPIPE` 信号。该信号的**默认动作是终止进程**。

**处理方式**（二选一）：

```c
// 方式一：全局忽略 SIGPIPE
signal(SIGPIPE, SIG_IGN);

// 方式二：每次 send 时传入 MSG_NOSIGNAL
send(sockfd, buf, len, MSG_NOSIGNAL);
```

### 10.2 Windows 无此问题

Windows 不存在 `SIGPIPE` 信号机制。当对端关闭连接后调用 `send()`，函数直接返回 `SOCKET_ERROR`，`WSAGetLastError()` 返回 `WSAECONNRESET`，进程不会收到任何信号。

### 10.3 跨平台处理

```c
#ifdef __linux__
    signal(SIGPIPE, SIG_IGN);  // 或在 send 时使用 MSG_NOSIGNAL
#endif
```

---

## 十一、数据类型与结构体差异

### 11.1 核心类型对照

|类型/概念|Linux|Windows|
|---|---|---|
|Socket 描述符|`int`|`SOCKET`（`UINT_PTR`）|
|地址长度类型|`socklen_t`（`<sys/socket.h>`）|`int`（无 `socklen_t`，需手动 typedef）|
|有符号尺寸类型|`ssize_t`（`<sys/types.h>`）|无此类型，需手动 `typedef int ssize_t;`|
|`fd_set` 中 fd 类型|`int`|`SOCKET`（`UINT_PTR`）|

### 11.2 地址结构体

`struct sockaddr_in` 在两端**内存布局一致**（`sin_family`、`sin_port`、`sin_addr` 字段顺序和对齐相同），可直接互通。但需注意：

- Windows 上需额外包含 `<ws2tcpip.h>` 以获得完整的 IPv6 支持（`struct sockaddr_in6`）。
- `inet_addr()` 已被废弃（不支持 `255.255.255.255`），两端均应使用 `inet_pton()` / `inet_ntop()`。

### 11.3 跨平台类型补齐宏

```c
#ifdef _WIN32
    typedef int socklen_t;
    typedef int ssize_t;
    #ifndef WIN32_LEAN_AND_MEAN
        #define WIN32_LEAN_AND_MEAN
    #endif
#endif
```

---

## 十二、时间函数差异

网络编程中常需要获取毫秒级时间戳（超时计算、性能测量等），两端使用的函数不同：

|功能|Linux|Windows|
|---|---|---|
|毫秒级时间|`gettimeofday()`（微秒精度）|`GetTickCount()` / `GetTickCount64()`|
|高精度时间|`clock_gettime(CLOCK_MONOTONIC, ...)`|`QueryPerformanceCounter()`|
|头文件|`<sys/time.h>`|`<windows.h>`|

此外，Windows 上缺少 `struct timeval` 的加减宏（`timeradd`、`timersub`），需要手动定义。

---

## 十三、`sleep` 函数差异

|平台|函数|参数单位|头文件|
|---|---|---|---|
|Linux|`sleep(seconds)`|**秒**|`<unistd.h>`|
|Linux|`usleep(microseconds)`|**微秒**|`<unistd.h>`|
|Windows|`Sleep(milliseconds)`|**毫秒**（注意大写 S）|`<windows.h>`|

---

## 十四、多线程模型差异

网络服务器通常需要多线程处理并发连接，两端的线程 API 完全不同：

|功能|Linux (POSIX Threads)|Windows|
|---|---|---|
|头文件|`<pthread.h>`|`<process.h>` 或 `<windows.h>`|
|创建线程|`pthread_create()`|`_beginthreadex()` 或 `CreateThread()`|
|退出线程|`pthread_exit()`|`_endthreadex()`|
|等待线程|`pthread_join()`|`WaitForSingleObject()`|
|互斥锁|`pthread_mutex_t` + `pthread_mutex_lock()`|`CRITICAL_SECTION` 或 `SRWLOCK`|
|条件变量|`pthread_cond_t`|`CONDITION_VARIABLE`|
|线程局部存储|`pthread_key_create()` / `pthread_setspecific()`|`TlsAlloc()` / `TlsSetValue()`|
|链接选项|`-lpthread`|无需额外链接|

**推荐做法**：使用 C++11 标准线程库（`<thread>`、`<mutex>`、`<condition_variable>`），可在两端统一使用，无需条件编译。

---

## 十五、TCP 协议栈默认行为差异

虽然两端均遵循 TCP/IP RFC 标准，但默认配置存在差异：

|选项|Linux 默认|Windows 默认|
|---|---|---|
|Nagle 算法 (`TCP_NODELAY`)|部分发行版默认启用（即 Nagle 关闭）|默认启用 Nagle（`TCP_NODELAY` 关闭）|
|`SO_REUSEADDR`|支持，语义标准|支持，但行为略有差异|
|`SO_KEEPALIVE`|默认关闭，超时 7200s|默认关闭，超时 2h|
|接收/发送缓冲区|内核自动调整|内核自动调整，默认值可能不同|

**建议**：不要依赖默认值，在代码中显式设置所有关键的 Socket 选项：

```c
// 禁用 Nagle 算法（降低延迟）
int flag = 1;
setsockopt(sockfd, IPPROTO_TCP, TCP_NODELAY, (char *)&flag, sizeof(flag));

// 启用地址复用
int opt = 1;
setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, (char *)&opt, sizeof(opt));
```

---

## 十六、零拷贝与高级数据传输

|功能|Linux|Windows|
|---|---|---|
|零拷贝发送文件|`sendfile(out_fd, in_fd, ...)`|`TransmitFile()`|
|通用零拷贝|`copy_file_range()`|`CopyFileEx()`|
|头文件|`<sys/sendfile.h>`|`<mswsock.h>`|

两者功能等价但 API 完全不同，需要条件编译分别实现。

---

## 十七、DNS 解析与地址查询

|功能|Linux|Windows|
|---|---|---|
|协议无关解析（推荐）|`getaddrinfo()` / `freeaddrinfo()`|`getaddrinfo()` / `FreeAddrInfo()`|
|旧式主机名解析|`gethostbyname()`（线程不安全）|`gethostbyname()`（同样不安全）|
|反向解析|`getnameinfo()`|`getnameinfo()`|

两端均支持 `getaddrinfo()`，这是**推荐的跨平台方案**。但需注意：

- 旧式 `gethostbyname()` 在 Linux 上非线程安全（返回指向静态数据的指针），应改用 `getaddrinfo()`。
- Windows 早期版本对 `AI_ADDRCONFIG` 标志支持不完善，需要做兼容性兜底。

---

## 十八、构建系统差异

|项目|Linux|Windows|
|---|---|---|
|主流编译器|GCC / Clang|MSVC / MinGW-w64|
|构建工具|Make / CMake / Ninja|MSBuild / CMake / Ninja|
|链接网络库|无需额外链接|必须链接 `ws2_32.lib`|
|推荐跨平台方案|**CMake**（通过 `find_package` 和条件编译自动适配）||

CMake 示例：

```cmake
cmake_minimum_required(VERSION 3.10)
project(NetworkApp)

if(WIN32)
    target_link_libraries(${PROJECT_NAME} ws2_32)
    target_compile_definitions(${PROJECT_NAME} PRIVATE _WIN32_WINNT=0x0601)
endif()

find_package(Threads REQUIRED)
target_link_libraries(${PROJECT_NAME} Threads::Threads)
```

---

## 十九、完整差异速查表

|差异维度|Linux|Windows|
|---|---|---|
|头文件|`<sys/socket.h>` 等|`<winsock2.h>` + `<ws2tcpip.h>`|
|初始化|无需|`WSAStartup()` / `WSACleanup()`|
|链接库|无需额外链接|`ws2_32.lib`|
|Socket 类型|`int`|`SOCKET`（`UINT_PTR`）|
|失败返回值|`-1`|`INVALID_SOCKET`|
|关闭 Socket|`close()`|`closesocket()`|
|错误获取|`errno`|`WSAGetLastError()`|
|错误描述|`strerror(errno)`|`FormatMessage()`|
|非阻塞设置|`fcntl(F_SETFL, O_NONBLOCK)`|`ioctlsocket(FIONBIO)`|
|发送数据|`send(fd, buf, len, MSG_NOSIGNAL)`|`send(s, buf, len, 0)`|
|接收数据|`recv()` / `read()` 均可|仅 `recv()`|
|SIGPIPE|需处理（忽略或 MSG_NOSIGNAL）|不存在|
|I/O 多路复用|`epoll`（Reactor）|`IOCP`（Proactor）|
|select nfds|最大 fd + 1|忽略（传 0）|
|sleep|`sleep()`（秒）|`Sleep()`（毫秒）|
|时间获取|`gettimeofday()`|`GetTickCount()`|
|线程|`pthread_create()`|`CreateThread()` / `_beginthreadex()`|
|文件传输|`sendfile()`|`TransmitFile()`|
|`socklen_t`|原生支持|需 `typedef int socklen_t`|
|`ssize_t`|原生支持|需 `typedef int ssize_t`|

---

## 二十、跨平台封装最佳实践

### 20.1 条件编译隔离

将所有平台差异集中在少量头文件中，业务代码不直接接触平台 API：

```c
// platform_net.h
#ifdef _WIN32
    #ifndef WIN32_LEAN_AND_MEAN
        #define WIN32_LEAN_AND_MEAN
    #endif
    #include <winsock2.h>
    #include <ws2tcpip.h>
    #pragma comment(lib, "ws2_32.lib")

    typedef SOCKET socket_t;
    #define INVALID_SOCK INVALID_SOCKET
    typedef int socklen_t;
    typedef int ssize_t;
    #define net_close(s) closesocket(s)
    #define net_errno() WSAGetLastError()
    #define net_would_block(e) ((e) == WSAEWOULDBLOCK)
    #define SEND_FLAGS 0
#else
    #include <sys/socket.h>
    #include <netinet/in.h>
    #include <netinet/tcp.h>
    #include <arpa/inet.h>
    #include <unistd.h>
    #include <fcntl.h>
    #include <errno.h>

    typedef int socket_t;
    #define INVALID_SOCK (-1)
    #define net_close(s) close(s)
    #define net_errno() errno
    #define net_would_block(e) ((e) == EAGAIN || (e) == EWOULDBLOCK)
    #define SEND_FLAGS MSG_NOSIGNAL
#endif
```

### 20.2 RAII 资源管理（C++）

```cpp
class SocketGuard {
    socket_t fd_;
public:
    explicit SocketGuard(socket_t fd) : fd_(fd) {}
    ~SocketGuard() { if (fd_ != INVALID_SOCK) net_close(fd_); }
    SocketGuard(const SocketGuard&) = delete;
    SocketGuard& operator=(const SocketGuard&) = delete;
    socket_t get() const { return fd_; }
};
```

### 20.3 推荐使用成熟的跨平台库

对于生产级项目，强烈建议使用以下经过验证的跨平台网络库，而非手动处理所有差异：

|库|语言|特点|
|---|---|---|
|**Boost.Asio**|C++|统一异步模型，内部自动对接 epoll/IOCP|
|**libuv**|C|Node.js 底层，跨平台事件循环|
|**Poco Net**|C++|完整网络框架|
|**ZeroMQ**|多语言|消息总线，屏蔽底层 Socket 细节|
|**gRPC**|多语言|RPC 框架，内置跨平台传输层|

---

> **总结**：Linux 网络编程基于 POSIX/Berkeley Socket 标准，与操作系统内核深度集成（"一切皆文件"）；Windows 通过 Winsock 2 提供兼容层，在基础 API 上高度相似，但在**类型系统、错误处理、I/O 模型架构、信号机制、资源生命周期管理**等方面存在系统性差异。对于简单的阻塞式 TCP/UDP 通信，移植工作量较小（主要是头文件、初始化、类型替换）；但对于基于 epoll 的高并发异步服务器，移植到 Windows 需要**重新设计事件循环核心**（epoll → IOCP），这是最大的工程挑战。采用跨平台抽象层或成熟网络库是降低移植成本的最佳策略。
