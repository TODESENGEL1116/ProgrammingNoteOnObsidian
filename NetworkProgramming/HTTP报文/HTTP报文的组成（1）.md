HTTP报文是客户端与服务器之间传输数据的载体，分为**请求报文（Request）** 和  **响应报文（Response）** 两种类型。它们共享相似的结构框架，但具体字段有所区别。

---

### HTTP 请求报文

请求报文由以下四部分组成：

#### 请求行（Request Line）

请求行是报文的第一行，包含三个要素：

- **方法（Method）**：表示请求的动作类型，常见方法包括 `GET`（获取资源）、`POST`（提交数据）、`PUT`（更新资源）、`DELETE`（删除资源）、`HEAD`（仅获取头部）、`OPTIONS`（查询支持的方法）等。
- **请求目标（Request-URI）**：标识请求的资源路径，例如 `/index.html` 或 `https://www.example.com/api/users`。
- **HTTP 版本（HTTP-Version）**：标明使用的协议版本，如 `HTTP/1.1` 或 `HTTP/2`。

示例：`GET /api/users HTTP/1.1`

#### 请求头部（Request Headers）

请求头部以键值对的形式传递客户端的附加信息，常见的头部字段包括：

- **Host**：指定目标服务器的主机名和端口号，HTTP/1.1 中为必填项。
- **User-Agent**：描述客户端的软件信息，如浏览器类型和版本。
- **Accept**：告知服务器客户端能 够处理的媒体类型，如 `text/html`、`application/json`。
- **Content-Type**：标明请求体的数据格式，如 `application/x-www-form-urlencoded` 或 `multipart/form-data`。
- **Content-Length**：指示请求体的字节长度。
- **Authorization**：携带客户端的身份认证凭据。
- **Cookie**：向服务器发送之前存储的 Cookie 信息。
- **Connection**：控制连接的管理方式，如 `keep-alive` 或 `close`。

#### 空行（Blank Line）

一个空行（即 `\r\n`）用于分隔头部字段和报文主体，是报文结构中的必要分隔符。

#### 请求体（Request Body）

请求体承载客户端向服务器发送的实际数据，并非所有请求都包含请求体。`GET` 请求通常没有请求体，而 `POST` 和 `PUT` 请求则常通过请求体传递表单数据、JSON 对象或文件等内容。

---

### HTTP 响应报文

响应报文同样由四个部分组成：

#### 状态行（Status Line）

状态行是报文的第一行，包含三个要素：

- **HTTP 版本（HTTP-Version）**：标明服务器使用的协议版本，如 `HTTP/1.1`。
- **状态码（Status Code）**：用三位数字表示请求的处理结果，常见的状态码分类如下：
    - **1xx（信息性）**：如 `100 Continue`，表示请求已接收，继续处理。
    - **2xx（成功）**：如 `200 OK`，表示请求成功；`201 Created`，表示资源已创建。
    - **3xx（重定向）**：如 `301 Moved Permanently`，表示资源已永久迁移；`304 Not Modified`，表示资源未修改，可使用缓存。
    - **4xx（客户端错误）**：如 `400 Bad Request`，表示请求格式有误；`401 Unauthorized`，表示需要身份认证；`404 Not Found`，表示资源不存在。
    - **5xx（服务器错误）**：如 `500 Internal Server Error`，表示服务器内部错误；`502 Bad Gateway`，表示网关错误；`503 Service Unavailable`，表示服务暂不可用。
- **原因短语（Reason Phrase）**：对状态码的简短文字描述，如 `OK`、`Not Found`。

示例：`HTTP/1.1 200 OK`

#### 响应头部（Response Headers）

响应头部以键值对的形式传递服务器的附加信息，常见的头部字段包括：

- **Content-Type**：标明响应体的媒体类型，如 `text/html; charset=utf-8` 或 `application/json`。
- **Content-Length**：指示响应体的字节长度。
- **Server**：描述服务器所使用的软件信息。
- **Set-Cookie**：指示客户端存储 Cookie。
- **Cache-Control**：控制缓存行为，如 `max-age=3600`、`no-cache`、`no-store`。
- **Location**：在重定向响应中指定新的资源地址。
- **Date**：标明响应生成的时间。
- **Connection**：控制连接的管理方式。

#### 空行（Blank Line）

与请求报文相同，一个空行用于分隔头部字段和报文主体。

#### 响应体（Response Body）

响应体承载服务器返回给客户端的实际数据，如 HTML 页面内容、JSON 数据、图片二进制流等。部分响应（如 `204 No Content` 或 `304 Not Modified`）不包含响应体。

---

### 总结对比

| 组成部分 | 请求报文                    | 响应报文                        |
| ---- | ----------------------- | --------------------------- |
| 首行   | 请求行（方法 + URI + 版本）      | 状态行（版本 + 状态码 + 原因短语）        |
| 头部   | 请求头部（Host、User-Agent 等） | 响应头部（Content-Type、Server 等） |
| 分隔   | 空行                      | 空行                          |
| 主体   | 请求体（POST/PUT 数据）        | 响应体（HTML、JSON 等）            |

理解 HTTP 报文的结构是进行 Web 开发、接口调试和网络问题排查的基础。在实际开发中，借助浏览器开发者工具或抓包工具（如 Wireshark、Charles）可以直接观察到完整的 HTTP 报文内容。
