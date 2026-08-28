### HTTP方法总览

HTTP协议定义了9种标准请求方法，用于指定客户端对服务器资源的操作类型，核心作用如下：

|方法|核心作用|关键特性|
|---|---|---|
|**GET**|获取指定资源的内容|安全、幂等、可缓存，参数通过URL传递|
|**POST**|向服务器提交数据/创建新资源|非幂等，数据放在请求体中，适合敏感/大量数据|
|**PUT**|全量替换指定资源（资源不存在时可创建）|幂等，需要提供完整的资源内容|
|**PATCH**|部分修改指定资源的某个/某些字段|非幂等，只需要提供需要修改的内容|
|**DELETE**|删除指定资源|幂等，多次删除结果一致|
|**HEAD**|获取资源的元信息（响应头），不返回响应体|安全、幂等，常用于检查资源是否存在/获取文件大小|
|**OPTIONS**|查询服务器支持的HTTP方法/跨域预检|安全、幂等，响应中包含`Allow`头列出支持的方法|
|**TRACE**|回显服务器收到的原始请求，用于调试|存在安全风险，生产环境通常禁用|
|**CONNECT**|建立客户端与目标服务器之间的隧道，常用于HTTPS代理|非幂等，代理场景专用|

---

### C语言示例（所有代码监听8080端口，编译运行后浏览器访问`localhost:8080`即可测试）

#### 1. GET：获取资源

最基础的方法，返回固定HTML内容。

```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>

int main() {
    int server_fd = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in addr = {.sin_family=AF_INET, .sin_port=htons(8080), .sin_addr.s_addr=INADDR_ANY};
    bind(server_fd, (struct sockaddr*)&addr, sizeof(addr));
    listen(server_fd, 3);
    printf("GET Server running at http://localhost:8080\n");

    while(1) {
        int client = accept(server_fd, NULL, NULL);
        char buf[1024] = {0};
        read(client, buf, sizeof(buf));
        printf("Received request:\n%s\n", buf);

        const char* resp = "HTTP/1.1 200 OK\r\nContent-Type: text/html; charset=utf-8\r\n\r\n<h1>GET: 这是获取到的资源内容</h1>";
        write(client, resp, strlen(resp));
        close(client);
    }
    return 0;
}
```

#### 2. POST：提交数据

读取请求体中的数据并返回接收到的内容。

```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>

int main() {
    int server_fd = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in addr = {.sin_family=AF_INET, .sin_port=htons(8080), .sin_addr.s_addr=INADDR_ANY};
    bind(server_fd, (struct sockaddr*)&addr, sizeof(addr));
    listen(server_fd, 3);
    printf("POST Server running at http://localhost:8080\n");

    while(1) {
        int client = accept(server_fd, NULL, NULL);
        char buf[4096] = {0};
        read(client, buf, sizeof(buf));
        printf("Received request:\n%s\n", buf);

        // 提取请求体内容（简单按空行分割）
        char* body = strstr(buf, "\r\n\r\n");
        body = body ? body + 4 : "无请求体";

        char resp[4096];
        sprintf(resp, "HTTP/1.1 200 OK\r\nContent-Type: text/html\r\n\r\n<h1>POST: 接收到的提交数据</h1><pre>%s</pre>", body);
        write(client, resp, strlen(resp));
        close(client);
    }
    return 0;
}
```

#### 3. PUT：全量替换资源

接收完整的资源内容，返回替换成功的提示。

```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>

int main() {
    int server_fd = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in addr = {.sin_family=AF_INET, .sin_port=htons(8080), .sin_addr.s_addr=INADDR_ANY};
    bind(server_fd, (struct sockaddr*)&addr, sizeof(addr));
    listen(server_fd, 3);
    printf("PUT Server running at http://localhost:8080\n");

    while(1) {
        int client = accept(server_fd, NULL, NULL);
        char buf[4096] = {0};
        read(client, buf, sizeof(buf));
        printf("Received PUT request:\n%s\n", buf);

        const char* resp = "HTTP/1.1 200 OK\r\nContent-Type: text/html\r\n\r\n<h1>PUT: 资源已全量替换为提交的新内容</h1>";
        write(client, resp, strlen(resp));
        close(client);
    }
    return 0;
}
```

#### 4. PATCH：部分修改资源

只接收需要修改的字段，返回部分更新的提示。

```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>

int main() {
    int server_fd = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in addr = {.sin_family=AF_INET, .sin_port=htons(8080), .sin_addr.s_addr=INADDR_ANY};
    bind(server_fd, (struct sockaddr*)&addr, sizeof(addr));
    listen(server_fd, 3);
    printf("PATCH Server running at http://localhost:8080\n");

    while(1) {
        int client = accept(server_fd, NULL, NULL);
        char buf[4096] = {0};
        read(client, buf, sizeof(buf));
        printf("Received PATCH request:\n%s\n", buf);

        const char* resp = "HTTP/1.1 200 OK\r\nContent-Type: text/html\r\n\r\n<h1>PATCH: 资源已部分修改指定字段</h1>";
        write(client, resp, strlen(resp));
        close(client);
    }
    return 0;
}
```

#### 5. DELETE：删除资源

返回删除成功的提示。

```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>

int main() {
    int server_fd = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in addr = {.sin_family=AF_INET, .sin_port=htons(8080), .sin_addr.s_addr=INADDR_ANY};
    bind(server_fd, (struct sockaddr*)&addr, sizeof(addr));
    listen(server_fd, 3);
    printf("DELETE Server running at http://localhost:8080\n");

    while(1) {
        int client = accept(server_fd, NULL, NULL);
        char buf[1024] = {0};
        read(client, buf, sizeof(buf));
        printf("Received DELETE request:\n%s\n", buf);

        const char* resp = "HTTP/1.1 200 OK\r\nContent-Type: text/html\r\n\r\n<h1>DELETE: 指定资源已删除</h1>";
        write(client, resp, strlen(resp));
        close(client);
    }
    return 0;
}
```

#### 6. HEAD：只返回响应头

不返回响应体，只返回资源元信息（如Content-Length）。

```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>

int main() {
    int server_fd = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in addr = {.sin_family=AF_INET, .sin_port=htons(8080), .sin_addr.s_addr=INADDR_ANY};
    bind(server_fd, (struct sockaddr*)&addr, sizeof(addr));
    listen(server_fd, 3);
    printf("HEAD Server running at http://localhost:8080\n");

    while(1) {
        int client = accept(server_fd, NULL, NULL);
        char buf[1024] = {0};
        read(client, buf, sizeof(buf));
        printf("Received HEAD request:\n%s\n", buf);

        // 只返回响应头，无响应体
        const char* resp = "HTTP/1.1 200 OK\r\nContent-Type: text/html\r\nContent-Length: 0\r\n\r\n";
        write(client, resp, strlen(resp));
        close(client);
    }
    return 0;
}
```

#### 7. OPTIONS：查询支持的方法

返回`Allow`头列出服务器支持的所有HTTP方法。

```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>

int main() {
    int server_fd = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in addr = {.sin_family=AF_INET, .sin_port=htons(8080), .sin_addr.s_addr=INADDR_ANY};
    bind(server_fd, (struct sockaddr*)&addr, sizeof(addr));
    listen(server_fd, 3);
    printf("OPTIONS Server running at http://localhost:8080\n");

    while(1) {
        int client = accept(server_fd, NULL, NULL);
        char buf[1024] = {0};
        read(client, buf, sizeof(buf));
        printf("Received OPTIONS request:\n%s\n", buf);

        const char* resp = "HTTP/1.1 200 OK\r\nAllow: GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS\r\nContent-Length: 0\r\n\r\n";
        write(client, resp, strlen(resp));
        close(client);
    }
    return 0;
}
```

#### 8. TRACE：回显原始请求

将收到的完整请求内容原样返回，用于调试。

```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>

int main() {
    int server_fd = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in addr = {.sin_family=AF_INET, .sin_port=htons(8080), .sin_addr.s_addr=INADDR_ANY};
    bind(server_fd, (struct sockaddr*)&addr, sizeof(addr));
    listen(server_fd, 3);
    printf("TRACE Server running at http://localhost:8080\n");

    while(1) {
        int client = accept(server_fd, NULL, NULL);
        char buf[4096] = {0};
        read(client, buf, sizeof(buf));
        printf("Received TRACE request:\n%s\n", buf);

        // 原样回显请求内容
        char resp[4096];
        sprintf(resp, "HTTP/1.1 200 OK\r\nContent-Type: message/http\r\n\r\n%s", buf);
        write(client, resp, strlen(resp));
        close(client);
    }
    return 0;
}
```

#### 9. CONNECT：建立隧道

返回隧道建立成功的提示（实际用于代理场景）。

```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>

int main() {
    int server_fd = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in addr = {.sin_family=AF_INET, .sin_port=htons(8080), .sin_addr.s_addr=INADDR_ANY};
    bind(server_fd, (struct sockaddr*)&addr, sizeof(addr));
    listen(server_fd, 3);
    printf("CONNECT Server running at http://localhost:8080\n");

    while(1) {
        int client = accept(server_fd, NULL, NULL);
        char buf[1024] = {0};
        read(client, buf, sizeof(buf));
        printf("Received CONNECT request:\n%s\n", buf);

        const char* resp = "HTTP/1.1 200 Connection Established\r\n\r\n";
        write(client, resp, strlen(resp));
        close(client);
    }
    return 0;
}
```

---

### 测试说明

- 编译所有代码：`gcc 文件名.c -o server`，运行：`./server`
- GET/POST/PUT/PATCH/DELETE可直接用浏览器访问`http://localhost:8080`测试（POST/PUT/PATCH需要用curl或Postman发送带请求体的请求）
- HEAD/OPTIONS/TRACE建议用curl测试：
    - HEAD：`curl -I http://localhost:8080`
    - OPTIONS：`curl -X OPTIONS http://localhost:8080`
    - TRACE：`curl -X TRACE http://localhost:8080`

---

需要我把这9个方法整合到一个统一的服务器程序里，通过判断请求方法返回不同响应，方便你一次性测试所有方法吗？