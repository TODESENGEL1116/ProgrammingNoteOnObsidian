### HTTP方法总览

HTTP方法（也称HTTP动词）定义了客户端对服务器资源的操作意图，不同方法有明确的安全、幂等属性区分，以下是完整的方法说明及最简C语言示例（基于原始Socket实现，仅保留体现方法核心特点的代码，省略错误处理逻辑）。

---

#### 1. GET：获取资源

**作用**：从服务器读取指定资源，不修改服务器状态，是**安全、幂等**的方法，常用于页面请求、数据查询。

```c
#include <stdio.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>

int main() {
    int sock = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in server = {
        .sin_family = AF_INET,
        .sin_port = htons(80),
        .sin_addr.s_addr = inet_addr("127.0.0.1")
    };
    connect(sock, (struct sockaddr*)&server, sizeof(server));
    
    // 构造GET请求：仅请求资源，无请求体
    char req[] = "GET /index.html HTTP/1.1\r\nHost: 127.0.0.1\r\nConnection: close\r\n\r\n";
    send(sock, req, strlen(req), 0);
    
    char resp[1024] = {0};
    recv(sock, resp, sizeof(resp)-1, 0);
    printf("%s\n", resp);
    close(sock);
    return 0;
}
```

#### 2. POST：提交数据/创建资源

**作用**：向服务器提交数据，通常用于创建新资源、表单提交，**不幂等**，会修改服务器状态，数据放在请求体中。

```c
#include <stdio.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>

int main() {
    int sock = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in server = {
        .sin_family = AF_INET,
        .sin_port = htons(80),
        .sin_addr.s_addr = inet_addr("127.0.0.1")
    };
    connect(sock, (struct sockaddr*)&server, sizeof(server));
    
    // 构造POST请求：携带请求体提交数据
    const char* body = "username=test&password=123";
    char req[1024];
    snprintf(req, sizeof(req), 
        "POST /api/login HTTP/1.1\r\n"
        "Host: 127.0.0.1\r\n"
        "Content-Type: application/x-www-form-urlencoded\r\n"
        "Content-Length: %ld\r\n"
        "Connection: close\r\n\r\n"
        "%s", strlen(body), body);
    send(sock, req, strlen(req), 0);
    
    char resp[1024] = {0};
    recv(sock, resp, sizeof(resp)-1, 0);
    printf("%s\n", resp);
    close(sock);
    return 0;
}
```

#### 3. PUT：全量更新/替换资源

**作用**：用请求体的完整内容替换目标资源，若资源不存在则创建，**幂等**，需要提交完整的资源信息。

```c
#include <stdio.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>

int main() {
    int sock = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in server = {
        .sin_family = AF_INET,
        .sin_port = htons(80),
        .sin_addr.s_addr = inet_addr("127.0.0.1")
    };
    connect(sock, (struct sockaddr*)&server, sizeof(server));
    
    // 构造PUT请求：提交完整的资源数据做全量更新
    const char* body = "{\"id\":1,\"name\":\"new_name\",\"age\":25}";
    char req[1024];
    snprintf(req, sizeof(req),
        "PUT /api/users/1 HTTP/1.1\r\n"
        "Host: 127.0.0.1\r\n"
        "Content-Type: application/json\r\n"
        "Content-Length: %ld\r\n"
        "Connection: close\r\n\r\n"
        "%s", strlen(body), body);
    send(sock, req, strlen(req), 0);
    
    char resp[1024] = {0};
    recv(sock, resp, sizeof(resp)-1, 0);
    printf("%s\n", resp);
    close(sock);
    return 0;
}
```

#### 4. DELETE：删除资源

**作用**：删除服务器指定的资源，**幂等**，多次请求效果相同，通常无请求体。

```c
#include <stdio.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>

int main() {
    int sock = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in server = {
        .sin_family = AF_INET,
        .sin_port = htons(80),
        .sin_addr.s_addr = inet_addr("127.0.0.1")
    };
    connect(sock, (struct sockaddr*)&server, sizeof(server));
    
    // 构造DELETE请求：仅指定要删除的资源路径，无请求体
    char req[] = "DELETE /api/users/1 HTTP/1.1\r\nHost: 127.0.0.1\r\nConnection: close\r\n\r\n";
    send(sock, req, strlen(req), 0);
    
    char resp[1024] = {0};
    recv(sock, resp, sizeof(resp)-1, 0);
    printf("%s\n", resp);
    close(sock);
    return 0;
}
```

#### 5. PATCH：部分更新资源

**作用**：对资源做局部修改，仅提交需要变更的字段，节省带宽，**不强制幂等**。

```c
#include <stdio.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>

int main() {
    int sock = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in server = {
        .sin_family = AF_INET,
        .sin_port = htons(80),
        .sin_addr.s_addr = inet_addr("127.0.0.1")
    };
    connect(sock, (struct sockaddr*)&server, sizeof(server));
    
    // 构造PATCH请求：仅提交需要修改的部分字段
    const char* body = "{\"age\":30}";
    char req[1024];
    snprintf(req, sizeof(req),
        "PATCH /api/users/1 HTTP/1.1\r\n"
        "Host: 127.0.0.1\r\n"
        "Content-Type: application/json\r\n"
        "Content-Length: %ld\r\n"
        "Connection: close\r\n\r\n"
        "%s", strlen(body), body);
    send(sock, req, strlen(req), 0);
    
    char resp[1024] = {0};
    recv(sock, resp, sizeof(resp)-1, 0);
    printf("%s\n", resp);
    close(sock);
    return 0;
}
```

#### 6. HEAD：获取资源元信息

**作用**：与GET请求逻辑一致，但服务器仅返回响应头，不返回响应体，用于检查资源是否存在、大小等，节省带宽。

```c
#include <stdio.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>

int main() {
    int sock = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in server = {
        .sin_family = AF_INET,
        .sin_port = htons(80),
        .sin_addr.s_addr = inet_addr("127.0.0.1")
    };
    connect(sock, (struct sockaddr*)&server, sizeof(server));
    
    // 构造HEAD请求：仅获取响应头，无响应体
    char req[] = "HEAD /index.html HTTP/1.1\r\nHost: 127.0.0.1\r\nConnection: close\r\n\r\n";
    send(sock, req, strlen(req), 0);
    
    char resp[1024] = {0};
    recv(sock, resp, sizeof(resp)-1, 0);
    printf("%s\n", resp);
    close(sock);
    return 0;
}
```

#### 7. OPTIONS：查询服务器支持的请求方法

**作用**：获取目标资源支持的HTTP方法、跨域权限等通信选项，常用于CORS预检请求。

```c
#include <stdio.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>

int main() {
    int sock = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in server = {
        .sin_family = AF_INET,
        .sin_port = htons(80),
        .sin_addr.s_addr = inet_addr("127.0.0.1")
    };
    connect(sock, (struct sockaddr*)&server, sizeof(server));
    
    // 构造OPTIONS请求：查询资源支持的HTTP方法
    char req[] = "OPTIONS /api/users HTTP/1.1\r\nHost: 127.0.0.1\r\nConnection: close\r\n\r\n";
    send(sock, req, strlen(req), 0);
    
    char resp[1024] = {0};
    recv(sock, resp, sizeof(resp)-1, 0);
    printf("%s\n", resp);
    close(sock);
    return 0;
}
```

#### 8. TRACE：回显请求内容

**作用**：将客户端的请求原样回显，用于诊断网络链路问题，存在安全风险，生产环境通常禁用。

```c
#include <stdio.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>

int main() {
    int sock = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in server = {
        .sin_family = AF_INET,
        .sin_port = htons(80),
        .sin_addr.s_addr = inet_addr("127.0.0.1")
    };
    connect(sock, (struct sockaddr*)&server, sizeof(server));
    
    // 构造TRACE请求：服务器会原样回显该请求内容
    char req[] = "TRACE / HTTP/1.1\r\nHost: 127.0.0.1\r\nConnection: close\r\n\r\n";
    send(sock, req, strlen(req), 0);
    
    char resp[1024] = {0};
    recv(sock, resp, sizeof(resp)-1, 0);
    printf("%s\n", resp);
    close(sock);
    return 0;
}
```

#### 9. CONNECT：建立隧道

**作用**：与目标服务器建立透明隧道，常用于HTTPS代理场景，代理服务器会代为转发后续加密流量。

```c
#include <stdio.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>

int main() {
    int sock = socket(AF_INET, SOCK_STREAM, 0);
    // 连接代理服务器端口（通常为8080）
    struct sockaddr_in proxy = {
        .sin_family = AF_INET,
        .sin_port = htons(8080),
        .sin_addr.s_addr = inet_addr("127.0.0.1")
    };
    connect(sock, (struct sockaddr*)&proxy, sizeof(proxy));
    
    // 构造CONNECT请求：请求建立到目标服务器的隧道
    char req[] = "CONNECT www.example.com:443 HTTP/1.1\r\nHost: www.example.com:443\r\n\r\n";
    send(sock, req, strlen(req), 0);
    
    char resp[1024] = {0};
    recv(sock, resp, sizeof(resp)-1, 0);
    printf("%s\n", resp);
    close(sock);
    return 0;
}
```

---

需要我帮你整理一份HTTP方法的安全性和幂等性对照表吗？实际开发中选型时很关键。