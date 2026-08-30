### cookie 是什么？

Cookie 是服务器通过 HTTP 响应报文发送给浏览器的一小段数据，浏览器会把它**保存在本地**。之后每次再访问同一个网站时，浏览器会自动在请求报文中把这段数据"带上"，发回给服务器。

你可以把 Cookie 理解成服务器发给你的**一张"会员卡"**：

- 第一次去商店（访问网站），商店给你一张会员卡（Cookie），上面写着你的会员编号。
- 以后每次去，你出示会员卡（浏览器自动携带 Cookie），商店就知道你是谁了。

---

### Cookie 的工作流程

以一个登录场景为例：

**第一次访问（登录）**

```
→ 浏览器发送请求：
POST /login HTTP/1.1
Host: www.example.com
（携带用户名和密码）

← 服务器验证成功后返回响应：
HTTP/1.1 200 OK
Set-Cookie: session_id=abc123; Path=/; HttpOnly
（服务器说："给你一张会员卡，编号是 abc123"）
```

浏览器收到后，把 `session_id=abc123` 保存到本地。

**之后的每次访问**

```
→ 浏览器自动携带 Cookie：
GET /dashboard HTTP/1.1
Host: www.example.com
Cookie: session_id=abc123
（浏览器自动出示会员卡）

← 服务器识别身份后返回响应：
HTTP/1.1 200 OK
（"欢迎回来，你已经登录了！"）
```

这就是为什么**登录一次后，浏览其他页面不需要重新登录**——因为浏览器每次都自动帮你"出示会员卡"。

---

### Host 和 Cookie 如何配合？

Host 和 Cookie 的配合，核心在于解决一个问题：**一台服务器托管了多个网站时，Cookie 该给谁、该带在哪？**

#### 场景：一台服务器同时托管 `www.shop.com` 和 `www.blog.com`

假设两个网站的 IP 都是 `1.2.3.4`，浏览器向这个 IP 发送请求时：

```
→ 请求 A：
GET / HTTP/1.1
Host: www.shop.com
Cookie: cart_id=xyz789

→ 请求 B：
GET / HTTP/1.1
Host: www.blog.com
Cookie: token=def456
```

服务器收到请求后，处理逻辑是这样的：

|步骤|服务器做了什么|
|---|---|
|**先看 Host**|`Host: www.shop.com` → 知道你要访问的是商城网站|
|**再看 Cookie**|`Cookie: cart_id=xyz789` → 知道你的购物车编号|
|**匹配验证**|服务器检查这个 `cart_id` 是不是属于 `www.shop.com` 的会话数据|

#### 为什么需要配合？

- **如果没有 Host**：服务器不知道你要访问哪个网站，就无法判断 Cookie 属于哪个站点，可能把商城的 Cookie 错误地用在博客上。
- **如果没有 Cookie**：服务器虽然知道你要访问哪个网站（通过 Host），但不知道你是谁，每次都要重新登录。

简单来说：

- **Host 解决的是"你去哪家店"**
- **Cookie 解决的是"你是谁"**

两者配合，服务器才能准确地知道**"哪个用户访问了哪个网站的哪些数据"**。

---

### 补充：Cookie 的域限制

浏览器还有一个安全机制：**Cookie 是按域名隔离的**。

- `www.shop.com` 设置的 Cookie，浏览器**只会**在请求 `www.shop.com` 时携带。
- 请求 `www.blog.com` 时，浏览器**不会**把商城的 Cookie 带过去。

这就保证了不同网站之间的 Cookie 互不干扰，即使它们共享同一个 IP 地址。