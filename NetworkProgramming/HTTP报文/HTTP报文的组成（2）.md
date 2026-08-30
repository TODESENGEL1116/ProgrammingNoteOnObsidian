 ### HTTP 报文总览

HTTP 报文分为**请求报文**和**响应报文**两种，分别由客户端和服务器发送，构成一次完整的"一问一答"通信。

---

### 请求报文（客户端 → 服务器）

用户输入网址并回车，浏览器发送的就是一个 **GET 请求报文**，由四部分组成：

| 组成部分 | 说明               | 示例                                                            |
| ---- | ---------------- | ------------------------------------------------------------- |
| 请求行  | 方法 + 请求路径 + 协议版本 | `GET /index.html HTTP/1.1`                                    |
| 请求头部 | 客户端附加信息（键值对）     | `Host: www.example.com`、`User-Agent: ...`、`Accept: text/html` |
| 空行   | 分隔头部与主体          | `\r\n`                                                        |
| 请求体  | 携带的数据（GET 请求通常无） | —                                                             |

常见请求方法：

- **GET**：获取资源（输入网址、点击链接）
- **POST**：提交数据（登录、上传文件）
- **PUT**：更新资源
- **DELETE**：删除资源

---

### 响应报文（服务器 → 客户端）

服务器收到请求后返回**响应报文**，同样由四部分组成：

|组成部分|说明|示例|
|---|---|---|
|状态行|协议版本 + 状态码 + 原因短语|`HTTP/1.1 200 OK`|
|响应头部|服务器附加信息（键值对）|`Content-Type: text/html`、`Content-Length: 1256`、`Server: nginx`|
|空行|分隔头部与主体|`\r\n`|
|响应体|实际返回的数据|`<HTML>...</HTML>`、JSON、图片等|

常见状态码分类：

- **1xx**：信息性（如 `100 Continue`）
- **2xx**：成功（如 `200 OK`、`201 Created`）
- **3xx**：重定向（如 `301 Moved Permanently`、`304 Not Modified`）
- **4xx**：客户端错误（如 `400 Bad Request`、`401 Unauthorized`、`404 Not Found`）
- **5xx**：服务器错误（如 `500 Internal Server Error`、`502 Bad Gateway`、`503 Service Unavailable`）

---

### 完整交互流程

```
浏览器（客户端）                        服务器
     |                                   |
     |  -------- 请求报文 -------->      |
     |  GET /index.html HTTP/1.1         |
     |  Host: www.example.com            |
     |  ...                              |
     |                                   |
     |  <------- 响应报文 ---------      |
     |  HTTP/1.1 200 OK                  |
     |  Content-Type: text/html          |
     |  ...                              |
     |  <HTML>...</HTML>                 |
     |                                   |
```

**一句话总结**：客户端发请求报文（告诉服务器"我要什么"），服务器回响应报文（告诉客户端"给你什么"），一来一回完成一次 HTTP 通信。


### Host 的具体含义

Host 是请求头部中的一个字段，用来告诉服务器"你要访问的是哪个网站"。例如：

```
GET /index.html HTTP/1.1
Host: www.example.com
```

这里的 `Host: www.example.com` 表示客户端要访问的是 `www.example.com` 这台服务器上的资源。

### 为什么需要 Host？

一台服务器（一个 IP 地址）上可能运行着**多个网站**，这叫做**虚拟主机**。比如同一个 IP `1.2.3.4` 上同时托管了 `www.siteA.com` 和 `www.siteB.com`，服务器收到请求后怎么知道你要访问哪个网站呢？就靠 Host 字段来区分：

```
请求 A：Host: www.siteA.com  →  服务器返回 A 网站的内容
请求 B：Host: www.siteB.com  →  服务器返回 B 网站的内容
```


GET /index.html HTTP/1.1
Host: ‘www.example.com’ 

`www.example.com` 是你**输入**的网址，而 `index.html` 是浏览器为了显示这个网址，**自动向服务器请求**的具体文件。

### 🤔 为什么输入网址，请求的却是文件？

这背后是浏览器和服务器的分工合作：

1. **你输入网址**：你在浏览器地址栏输入 `www.example.com` 并回车。
2. **浏览器“翻译”请求**：浏览器知道，要显示一个网站，它需要获取这个网站的“首页”文件。按照约定，这个首页文件通常叫 `index.html`、`index.htm` 或 `default.html` 等。
3. **发送HTTP请求**：于是，浏览器会自动向 `www.example.com` 这台服务器发送一个 HTTP 请求，请求的路径就是这个默认首页文件。

所以，你看到的请求报文是：

```
GET /index.html HTTP/1.1
Host: www.example.com
```

- **`Host: www.example.com`**：告诉服务器“我要访问的是你这个域名下的网站”。
- **`GET /index.html`**：告诉服务器“请把这个网站的默认首页文件 `index.html` 发给我”。

### 📄 文件与内容的关系

服务器收到请求后，会找到 `index.html` 这个文件，读取它的内容，然后把内容放在 **HTTP 响应报文**的**响应体（Response Body）**里发回给浏览器。

根据你提供的网页内容，`index.html` 文件里的内容大致是这样的：

```html
<!DOCTYPE html>
<html>
<head><title>Example Domain</title></head>
<body>
  <h1>Example Domain</h1>
  <p>This domain is for use in documentation examples without needing permission. Avoid use in operations.</p>
  <p><a href="">Learn more</a></p>
</body>
</html>
```

浏览器接收到这段 HTML 代码后，会进行“解析”和“渲染”，最终把你看到的标题、段落等元素展示出来，形成了我们看到的网页。

### 总结一下

|你看到的/操作的|背后实际发生的|关系|
|:--|:--|:--|
|输入网址 `www.example.com`|浏览器请求文件 `/index.html`|**网址是入口，文件是内容**|
|看到一个漂亮的网页|浏览器解析并渲染了 HTML 代码|**文件是原料，网页是成品**|

所以，`www.example.com` 和 `index.html` 的关系就是：**前者是用户访问的地址，后者是服务器为了响应该地址而提供的默认内容文件。**