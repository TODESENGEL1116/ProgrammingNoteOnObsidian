以下是基于 IANA 官方注册表及 RFC 9110 等标准整理的**所有 HTTP 状态码对应的状态文本（Status Text / Reason Phrase）**完整列表。

---

## HTTP 状态码与状态文本（Status Text）完整对照表

HTTP 响应状态行格式为：`HTTP/版本 状态码 状态文本`，例如 `HTTP/1.1 200 OK`，其中 **OK** 即为状态文本。

---

### 1xx — 信息性响应（Informational）

|状态码|状态文本（Status Text）|中文含义|标准来源|
|---|---|---|---|
|100|Continue|继续|RFC 9110|
|101|Switching Protocols|切换协议|RFC 9110|
|102|Processing|正在处理|RFC 2518（WebDAV）|
|103|Early Hints|提前提示|RFC 8297|

---

### 2xx — 成功（Successful）

|状态码|状态文本（Status Text）|中文含义|标准来源|
|---|---|---|---|
|200|OK|成功|RFC 9110|
|201|Created|已创建|RFC 9110|
|202|Accepted|已接受|RFC 9110|
|203|Non-Authoritative Information|非权威信息|RFC 9110|
|204|No Content|无内容|RFC 9110|
|205|Reset Content|重置内容|RFC 9110|
|206|Partial Content|部分内容|RFC 9110|
|207|Multi-Status|多状态|RFC 4918（WebDAV）|
|208|Already Reported|已报告|RFC 5842（WebDAV）|
|226|IM Used|IM 已使用|RFC 3229|

---

### 3xx — 重定向（Redirection）

|状态码|状态文本（Status Text）|中文含义|标准来源|
|---|---|---|---|
|300|Multiple Choices|多种选择|RFC 9110|
|301|Moved Permanently|永久移动|RFC 9110|
|302|Found|已找到|RFC 9110|
|303|See Other|查看其他|RFC 9110|
|304|Not Modified|未修改|RFC 9110|
|305|Use Proxy|使用代理（已弃用）|RFC 9110|
|306|(Unused)|未使用（保留）|RFC 9110|
|307|Temporary Redirect|临时重定向|RFC 9110|
|308|Permanent Redirect|永久重定向|RFC 9110|

---

### 4xx — 客户端错误（Client Error）

|状态码|状态文本（Status Text）|中文含义|标准来源|
|---|---|---|---|
|400|Bad Request|错误请求|RFC 9110|
|401|Unauthorized|未授权|RFC 9110|
|402|Payment Required|需要付款（保留）|RFC 9110|
|403|Forbidden|禁止访问|RFC 9110|
|404|Not Found|未找到|RFC 9110|
|405|Method Not Allowed|方法不允许|RFC 9110|
|406|Not Acceptable|不可接受|RFC 9110|
|407|Proxy Authentication Required|需要代理认证|RFC 9110|
|408|Request Timeout|请求超时|RFC 9110|
|409|Conflict|冲突|RFC 9110|
|410|Gone|已删除|RFC 9110|
|411|Length Required|需要内容长度|RFC 9110|
|412|Precondition Failed|先决条件失败|RFC 9110|
|413|Content Too Large|内容过大|RFC 9110|
|414|URI Too Long|URI 过长|RFC 9110|
|415|Unsupported Media Type|不支持的媒体类型|RFC 9110|
|416|Range Not Satisfiable|范围不满足|RFC 9110|
|417|Expectation Failed|期望失败|RFC 9110|
|418|(Unused)|未使用（原 I'm a Teapot）|RFC 9110|
|421|Misdirected Request|错误定向请求|RFC 9110|
|422|Unprocessable Content|不可处理的内容|RFC 9110|
|423|Locked|已锁定|RFC 4918（WebDAV）|
|424|Failed Dependency|依赖失败|RFC 4918（WebDAV）|
|425|Too Early|过早|RFC 8470|
|426|Upgrade Required|需要升级|RFC 9110|
|428|Precondition Required|需要先决条件|RFC 6585|
|429|Too Many Requests|请求过多|RFC 6585|
|431|Request Header Fields Too Large|请求头字段过大|RFC 6585|
|451|Unavailable For Legal Reasons|因法律原因不可用|RFC 7725|

---

### 5xx — 服务器错误（Server Error）

|状态码|状态文本（Status Text）|中文含义|标准来源|
|---|---|---|---|
|500|Internal Server Error|内部服务器错误|RFC 9110|
|501|Not Implemented|未实现|RFC 9110|
|502|Bad Gateway|错误网关|RFC 9110|
|503|Service Unavailable|服务不可用|RFC 9110|
|504|Gateway Timeout|网关超时|RFC 9110|
|505|HTTP Version Not Supported|HTTP 版本不支持|RFC 9110|
|506|Variant Also Negotiates|变体也在协商|RFC 2295|
|507|Insufficient Storage|存储空间不足|RFC 4918（WebDAV）|
|508|Loop Detected|检测到循环|RFC 5842（WebDAV）|
|510|Not Extended|未扩展|RFC 2774|
|511|Network Authentication Required|需要网络认证|RFC 6585|

---

### 非标准但广泛使用的状态文本

以下状态文本不在 IETF 标准中，但在特定服务器/CDN 中广泛使用：

|状态码|状态文本（Status Text）|来源|中文含义|
|---|---|---|---|
|449|Retry With|Microsoft IIS|使用…重试|
|499|Client Closed Request|Nginx|客户端关闭请求|
|509|Bandwidth Limit Exceeded|Apache（非官方）|带宽超限|
|521|Web Server Is Down|Cloudflare|Web 服务器宕机|
|522|Connection Timed Out|Cloudflare|连接超时|
|523|Origin Is Unreachable|Cloudflare|源站不可达|
|524|A Timeout Occurred|Cloudflare|发生超时|
|525|SSL Handshake Failed|Cloudflare|SSL 握手失败|
|526|Invalid SSL Certificate|Cloudflare|SSL 证书无效|
|527|Railgun Error|Cloudflare|Railgun 错误|
|530|1XXX 错误|Cloudflare|Cloudflare 内部错误|

---

### 新旧标准状态文本差异说明

部分状态文本在 RFC 9110（2022）中相比旧版 RFC 2616/7231 有更新：

|状态码|旧状态文本（RFC 2616/7231）|新状态文本（RFC 9110）|
|---|---|---|
|413|Request Entity Too Large|**Content Too Large**|
|414|Request-URI Too Long|**URI Too Long**|
|416|Requested Range Not Satisfiable|**Range Not Satisfiable**|
|422|Unprocessable Entity|**Unprocessable Content**|

实际开发中，不同服务器实现可能仍使用旧版文本，两者均被广泛接受。

---

### 补充说明

- **306** 在早期 HTTP/1.1 规范中定义为 `Switch Proxy`，但在 RFC 9110 中已标记为 `(Unused)`，不再分配用途。
- **418** 源自 RFC 2324（1998 年愚人节彩蛋 "I'm a Teapot"），在 RFC 9110 中已被标记为 `(Unused)`。
- **104** 是 IANA 临时注册的状态码（Upload Resumption Supported），用于可恢复上传扩展，预计 2026-11-13 到期。
