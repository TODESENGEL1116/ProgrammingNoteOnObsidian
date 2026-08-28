以下是 HTTP 状态码的完整整理，按五大类别分类，涵盖 IETF 标准定义的状态码以及常见的非标准扩展状态码。

---

## HTTP 状态码总览

HTTP 状态码是由三位数字组成的响应代码，首位数字决定类别：

| 类别  | 范围      | 含义                   |
| --- | ------- | -------------------- |
| 1xx | 100–199 | 信息性响应（临时响应，继续处理）     |
| 2xx | 200–299 | 成功（请求被正常接收、理解并处理）    |
| 3xx | 300–399 | 重定向（需要进一步操作才能完成请求）   |
| 4xx | 400–499 | 客户端错误（请求有语法错误或无法完成）  |
| 5xx | 500–599 | 服务器错误（服务器在处理请求时发生错误） |

---

### 1xx — 信息性响应（Informational）

|状态码|英文名称|中文说明|
|---|---|---|
|100|Continue|客户端应继续发送请求体（常用于大文件上传前的试探）|
|101|Switching Protocols|服务器同意切换协议（如 HTTP → WebSocket）|
|102|Processing|服务器已收到请求，正在处理中（WebDAV 扩展）|
|103|Early Hints|允许客户端在服务器准备最终响应时预加载资源|

---

### 2xx — 成功（Successful）

|状态码|英文名称|中文说明|
|---|---|---|
|200|OK|请求成功（最常见），响应体包含请求结果|
|201|Created|请求成功并创建了新资源（常用于 POST/PUT）|
|202|Accepted|请求已接受，但尚未处理完成（常用于异步操作）|
|203|Non-Authoritative Information|请求成功，但返回的元信息来自第三方副本，非原始服务器|
|204|No Content|请求成功，但响应不含实体内容（常用于 DELETE 操作）|
|205|Reset Content|请求成功，客户端应重置文档视图（如清空表单）|
|206|Partial Content|服务器成功处理了部分 GET 请求（用于断点续传、视频流）|
|207|Multi-Status|响应体包含多个独立操作的状态信息（WebDAV 扩展）|
|208|Already Reported|已在之前的响应中枚举过，避免重复（WebDAV 扩展）|
|226|IM Used|服务器已完成对资源的 GET 请求，响应是实例操作的结果（HTTP Delta encoding）|

---

### 3xx — 重定向（Redirection）

|状态码|英文名称|中文说明|
|---|---|---|
|300|Multiple Choices|请求的资源有多种表示，客户端需自行选择|
|301|Moved Permanently|资源已**永久**移动到新 URL，搜索引擎会转移权重|
|302|Found|资源**临时**移动到新 URL，浏览器不缓存跳转|
|303|See Other|建议客户端用 GET 方法访问另一个 URI（常用于 POST 后重定向）|
|304|Not Modified|资源未修改，客户端可使用本地缓存（协商缓存）|
|305|Use Proxy|请求的资源必须通过指定代理访问（已弃用）|
|306|Switch Proxy|已被废弃，保留但不再使用|
|307|Temporary Redirect|临时重定向，要求客户端保持原请求方法不变|
|308|Permanent Redirect|永久重定向，且要求保持原请求方法不变|

---

### 4xx — 客户端错误（Client Error）

|状态码|英文名称|中文说明|
|---|---|---|
|400|Bad Request|请求存在语法错误或参数无效，服务器无法理解|
|401|Unauthorized|请求需要身份认证（未登录或未提供有效凭证）|
|402|Payment Required|保留状态码，预留给将来使用|
|403|Forbidden|服务器理解请求，但拒绝执行（权限不足）|
|404|Not Found|请求的资源不存在（最知名的状态码）|
|405|Method Not Allowed|请求方法不被允许（如用 POST 访问只支持 GET 的接口）|
|406|Not Acceptable|服务器无法按客户端要求的格式返回内容|
|407|Proxy Authentication Required|需要先在代理服务器上进行身份认证|
|408|Request Timeout|客户端在服务器允许的时间内未完成请求（超时）|
|409|Conflict|请求与服务器当前资源状态冲突（如版本冲突）|
|410|Gone|请求的资源已永久删除，且无进一步参考地址|
|411|Length Required|服务器拒绝不含 Content-Length 头的请求|
|412|Precondition Failed|请求头中的先决条件不满足|
|413|Content Too Large|请求实体过大，超出服务器允许的范围|
|414|URI Too Long|请求的 URI 过长，服务器无法处理|
|415|Unsupported Media Type|服务器不支持请求附带的媒体格式|
|416|Range Not Satisfiable|请求的字节范围无效或不可满足|
|417|Expectation Failed|服务器无法满足 Expect 请求头的期望值|
|418|I'm a Teapot|我是一个茶壶（愚人节彩蛋，RFC 2324）|
|421|Misdirected Request|请求被发送到无法生成响应的服务器|
|422|Unprocessable Entity|请求格式正确，但语义错误，服务器无法处理（WebDAV 扩展）|
|423|Locked|请求的资源当前被锁定（WebDAV 扩展）|
|424|Failed Dependency|由于先前的请求失败，当前请求也失败（WebDAV 扩展）|
|425|Too Early|服务器不愿意处理可能被重放的请求|
|426|Upgrade Required|客户端应切换到不同的协议（如 TLS）|
|428|Precondition Required|服务器要求请求必须包含条件头（如 If-Match）|
|429|Too Many Requests|客户端发送请求过于频繁，触发了限流|
|431|Request Header Fields Too Large|请求头字段过大，服务器拒绝处理|
|451|Unavailable for Legal Reasons|因法律原因（如审查、版权）无法提供该资源|

---

### 5xx — 服务器错误（Server Error）

|状态码|英文名称|中文说明|
|---|---|---|
|500|Internal Server Error|服务器内部错误（通用错误，如代码异常、空指针）|
|501|Not Implemented|服务器不支持请求的功能或方法|
|502|Bad Gateway|网关/代理服务器收到上游服务器的无效响应（后端服务挂了）|
|503|Service Unavailable|服务器暂时无法处理请求（过载或维护中）|
|504|Gateway Timeout|网关/代理服务器未及时从上游服务器获得响应（超时）|
|505|HTTP Version Not Supported|服务器不支持请求中指定的 HTTP 版本|
|506|Variant Also Negotiates|服务器存在内部配置错误，选择的资源引用自身进行协商|
|507|Insufficient Storage|服务器没有足够的存储空间来完成请求（WebDAV 扩展）|
|508|Loop Detected|服务器在处理请求时检测到无限循环（WebDAV 扩展）|
|510|Not Extended|客户端的请求需要服务器进行扩展才能处理|
|511|Network Authentication Required|客户端需要进行网络认证才能访问（如 Wi-Fi 门户认证）|

---

### 非标准但常见的扩展状态码

以下状态码不在 IETF 标准中，但被特定服务器或 CDN 广泛使用：

|状态码|来源|中文说明|
|---|---|---|
|499|Nginx|客户端主动关闭连接（通常是后端处理太慢，用户等不及刷新了页面）|
|521|Cloudflare|源站 Web 服务器拒绝连接|
|522|Cloudflare|源站连接超时|
|523|Cloudflare|源站不可达|
|524|Cloudflare|源站已连接，但响应超时|
|599|网络代理|网络超时（非 HTTP 标准，用于代理场景）|

---

### 快速记忆口诀

> **1** 继续处理别着急，**2** 成功返回没问题；  
> **3** 重定向换地址，**4** 你（客户端）的错自己查；  
> **5** 我（服务器）的问题别怪你。

简版：**2 成功，3 跳转，4 你错，5 我错**