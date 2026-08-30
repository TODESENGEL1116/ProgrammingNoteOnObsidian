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


为了让你更直观地理解，我为你准备了一个最经典的 **HTTP 请求报文**和一个 **HTTP 响应报文**的示例。

假设场景：你在浏览器地址栏输入 `www.example.com` 并回车，浏览器向服务器请求首页。

---

### 一、 HTTP 请求报文（浏览器发给服务器）

```http
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: text/html,application/xhtml+xml
Accept-Language: zh-CN,zh;q=0.9
Connection: keep-alive
```

#### 🔍 逐行拆解分析：

1. **第一行：请求行 (Request Line)**
    
    - `GET`：**请求方法**。告诉服务器“我要获取资源”。
    - `/index.html`：**请求路径 (URI)**。告诉服务器“我要你这里的 `index.html` 这个文件”。
    - `HTTP/1.1`：**协议版本**。告诉服务器“我用的是 HTTP  1.1 版本的规则跟你通信”。
2. **中间部分：请求头部 (Request Headers)**
    
    - `Host: www.example.com`：**主机名**。告诉服务器“我要访问的是你管理的 `www.example.com` 这个网站”（因为一台服务器可能托管了多个网站）。
    - `User-Agent: ...`：**客户端身份**。告诉服务器“我是一个运行在 Windows 10 上的现代浏览器”。
    - `Accept: ...`：**接受的数据类型**。告诉服务器“我能看懂 HTML 格式的内容”。
    - `Accept-Language: zh-CN,zh;q=0.9`：**接受的语言**。告诉服务器“我首选中文，其次也是中文”。
    - `Connection: keep-alive`：**连接控制**。告诉服务器“传完这个文件后先别断开连接，我等会儿可能还要请求网页里的图片”。
3. **空行 (Blank Line)**
    
    - 注意，在 `Connection: keep-alive` 后面有一个**空行**。这是 HTTP 协议的硬性规定，用来告诉服务器：“我的头部信息说完了，下面没有更多头部了。”（在这个 GET 请求中，空行后面没有请求体）。

---

### 二、 HTTP 响应报文（服务器发给浏览器）

服务器收到上面的请求后，会回复一个响应报文：

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 138
Server: nginx
Date: Sat, 29 Aug 2026 12:00:00 GMT

<!DOCTYPE html>
<html>
<body>
  <h1>Hello World</h1>
</body>
</html>
```

#### 🔍 逐行拆解分析：

1. **第一行：状态行 (Status Line)**
    
    - `HTTP/1.1`：**协议版本**。
    - `200`：**状态码**。代表“请求成功，你要的东西我找到了”。（如果是 `404` 就是没找到）。
    - `OK`：**原因短语**。对状态码的简短文字解释。
2. **中间部分：响应头部 (Response Headers)**
    
    - `Content-Type: text/html; charset=utf-8`：**数据类型**。告诉浏览器“我给你的内容是 HTML 格式，编码是 UTF-8，你按网页来渲染就行”。
    - `Content-Length: 138`：**数据长度**。告诉浏览器“下面的正文内容一共有 138 个字节”。
    - `Server: nginx`：**服务器软件**。告诉浏览器“我是用 Nginx 这个软件搭建的”。
    - `Date: ...`：**响应时间**。告诉浏览器“这个响应是 2026年8月29日 12点整生成的”。
3. **空行 (Blank Line)**
    
    - 同样，在 `Date` 后面有一个**空行**，表示头部结束，接下来是真正的数据。
4. **最后部分：响应体 (Response Body)**
    
    - 空行之后的 `<!DOCTYPE html>...` 就是**响应体**。
    - 这就是网页的“真面目”（HTML 代码）。浏览器拿到这段代码后，会在屏幕上画出带有 "Hello World" 标题的网页。

---

### 💡 核心总结

- **请求报文** = 我要什么（方法 + 路径） + 我的情况（头部） + [可选：我提交的数据（请求体）]
- **响应报文** = 处理结果（状态码） + 数据说明（头部） + 实际给的数据（响应体）
- **空行** = 永远存在于头部和主体之间的“分割线”。
