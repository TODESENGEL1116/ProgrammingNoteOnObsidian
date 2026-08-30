## 什么是 Content-Type？

### 定义

**Content-Type**（内容类型）是 HTTP 协议中的一个头部字段（Header），用于指示 HTTP 请求或响应中**实体内容的数据格式**。它遵循 **MIME**（Multipurpose Internet Mail Extensions，多用途互联网邮件扩展）标准，最初设计用于电子邮件系统，后来被 HTTP 协议采纳。

### 语法格式

```
Content-Type: type/subtype; parameter=value
```

|组成部分|说明|
|---|---|
|**type**（主类型）|数据的大类别，如 `text`、`image`、`application` 等|
|**subtype**（子类型）|具体的数据格式，如 `html`、`json`、`png` 等|
|**parameter**（可选参数）|附加信息，最常见的是 `charset=UTF-8`（字符编码）和 `boundary`（多部分分隔符）|

示例：

```
Content-Type: text/html; charset=UTF-8
Content-Type: application/json; charset=utf-8
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW
```

### 核心作用

- **数据解析**：浏览器根据 Content-Type 自动选择合适的解析器（如渲染 HTML、执行 JS、显示图片）
- **内容协商**：客户端和服务器可协商最优的内容类型
- **安全性**：防止恶意文件被错误解析执行
- **浏览器行为**：浏览器使用 MIME 类型（而非文件扩展名）来决定如何处理文件

### 两个重要的默认类型

- `text/plain` — 文本文件的默认类型
- `application/octet-stream` — 未知/二进制文件的默认类型（浏览器会特别谨慎处理）

---

## Content-Type 完整整理

### 一、text 类型 — 文本数据

| Content-Type        | 文件扩展名       | 说明                                              |
| ------------------- | ----------- | ----------------------------------------------- |
| `text/plain`        | .txt        | 纯文本                                             |
| `text/html`         | .htm, .html | HTML 文档                                         |
| `text/css`          | .css        | CSS 样式表                                         |
| `text/javascript`   | .js, .mjs   | JavaScript（规范已更新，旧写法为 `application/javascript`） |
| `text/xml`          | .xml        | XML 文本                                          |
| `text/csv`          | .csv        | CSV 逗号分隔值                                       |
| `text/calendar`     | .ics        | iCalendar 日历格式                                  |
| `text/markdown`     | .md         | Markdown 文本                                     |
| `text/event-stream` | —           | SSE（Server-Sent Events）数据流                      |

---

### 二、application 类型 — 应用数据

#### 通用/数据交换

|Content-Type|文件扩展名|说明|
|---|---|---|
|`application/json`|.json|JSON 数据（前后端交互最常用）|
|`application/xml`|.xml|XML 数据|
|`application/xhtml+xml`|.xhtml|XHTML 文档|
|`application/atom+xml`|—|Atom XML 聚合格式|
|`application/ld+json`|.jsonld|JSON-LD 格式|
|`application/pdf`|.pdf|PDF 文档|
|`application/rtf`|.rtf|RTF 富文本格式|
|`application/octet-stream`|.bin|通用二进制流（文件下载）|

#### 表单提交（请求体编码）

|Content-Type|说明|
|---|---|
|`application/x-www-form-urlencoded`|表单默认编码，键值对格式（`key=value&key=value`）|
|`multipart/form-data`|多部分表单数据，支持文件上传|
|`application/json`|JSON 格式提交（RESTful API 常用）|

#### Office 文档

|Content-Type|文件扩展名|说明|
|---|---|---|
|`application/msword`|.doc|Microsoft Word（旧格式）|
|`application/vnd.openxmlformats-officedocument.wordprocessingml.document`|.docx|Word（OpenXML）|
|`application/vnd.ms-excel`|.xls|Microsoft Excel（旧格式）|
|`application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`|.xlsx|Excel（OpenXML）|
|`application/vnd.ms-powerpoint`|.ppt|PowerPoint（旧格式）|
|`application/vnd.openxmlformats-officedocument.presentationml.presentation`|.pptx|PowerPoint（OpenXML）|

#### 压缩包与归档

|Content-Type|文件扩展名|说明|
|---|---|---|
|`application/zip`|.zip|ZIP 压缩包（Windows/macOS 上传时可能使用非标准类型 `application/x-zip-compressed`）|
|`application/gzip`|.gz|GZip 压缩包（Windows/macOS 上传时可能使用非标准类型 `application/x-gzip`）|
|`application/x-bzip`|.bz|BZip 压缩包|
|`application/x-bzip2`|.bz2|BZip2 压缩包|
|`application/x-tar`|.tar|TAR 归档|
|`application/x-rar-compressed`|.rar|RAR 压缩包|
|`application/x-7z-compressed`|.7z|7-Zip 压缩包|
|`application/x-freearc`|.arc|FreeArc 归档|

#### 其他应用类型

|Content-Type|文件扩展名|说明|
|---|---|---|
|`application/java-archive`|.jar|Java 归档|
|`application/epub+zip`|.epub|电子书|
|`application/vnd.amazon.ebook`|.azw|Kindle 电子书|
|`application/vnd.ms-fontobject`|.eot|MS 嵌入式 OpenType 字体|
|`application/x-httpd-php`|.php|PHP 脚本|
|`application/x-sh`|.sh|Shell 脚本|
|`application/x-csh`|.csh|C-Shell 脚本|
|`application/x-shockwave-flash`|.swf|Flash 文件|
|`application/manifest+json`|.webmanifest|Web 应用清单|
|`application/vnd.apple.installer+xml`|.mpkg|Apple 安装包|
|`application/vnd.oasis.opendocument.text`|.odt|OpenDocument 文本|
|`application/vnd.oasis.opendocument.spreadsheet`|.ods|OpenDocument 表格|
|`application/vnd.oasis.opendocument.presentation`|.odp|OpenDocument 演示|
|`application/vnd.visio`|.vsd|Microsoft Visio|
|`application/ogg`|.ogx|Ogg 容器|

---

### 三、image 类型 — 图像数据

|Content-Type|文件扩展名|说明|
|---|---|---|
|`image/jpeg`|.jpg, .jpeg|JPEG 图片|
|`image/png`|.png|PNG 图片|
|`image/gif`|.gif|GIF 动图|
|`image/svg+xml`|.svg|SVG 矢量图|
|`image/webp`|.webp|WebP 图片|
|`image/avif`|.avif|AVIF 图片|
|`image/apng`|.apng|动态 PNG|
|`image/bmp`|.bmp|BMP 位图|
|`image/tiff`|.tif, .tiff|TIFF 图片|
|`image/vnd.microsoft.icon`|.ico|图标|
|`image/x-icon`|.ico|图标（非标准写法）|

---

### 四、audio 类型 — 音频数据

|Content-Type|文件扩展名|说明|
|---|---|---|
|`audio/mpeg`|.mp3|MP3 音频|
|`audio/wav`|.wav|WAV 音频|
|`audio/ogg`|.oga, .opus|Ogg 音频|
|`audio/webm`|.weba|WebM 音频|
|`audio/aac`|.aac|AAC 音频|
|`audio/midi`|.mid, .midi|MIDI 音频|
|`audio/opus`|.opus|Opus 音频|
|`audio/flac`|.flac|FLAC 无损音频|
|`audio/3gpp`|.3gp|3GPP 音频（不含视频时）|

---

### 五、video 类型 — 视频数据

|Content-Type|文件扩展名|说明|
|---|---|---|
|`video/mp4`|.mp4|MP4 视频|
|`video/mpeg`|.mpeg|MPEG 视频|
|`video/webm`|.webm|WebM 视频|
|`video/ogg`|.ogv|Ogg 视频|
|`video/x-msvideo`|.avi|AVI 视频|
|`video/quicktime`|.mov|QuickTime 视频|
|`video/mp2t`|.ts|MPEG 传输流|
|`video/3gpp`|.3gp|3GPP 视频容器|
|`video/3gpp2`|.3g2|3GPP2 视频容器|

---

### 六、font 类型 — 字体数据

|Content-Type|文件扩展名|说明|
|---|---|---|
|`font/woff`|.woff|Web 开放字体格式|
|`font/woff2`|.woff2|Web 开放字体格式 2|
|`font/ttf`|.ttf|TrueType 字体|
|`font/otf`|.otf|OpenType 字体|
|`font/collection`|.ttc|TrueType 字体集合|

---

### 七、multipart 类型 — 多部分数据

|Content-Type|说明|
|---|---|
|`multipart/form-data`|表单数据（支持文件上传），需配合 `boundary` 参数|
|`multipart/mixed`|混合数据（如邮件附件）|
|`multipart/related`|关联数据（如 HTML 邮件内嵌图片）|
|`multipart/alternative`|替代数据（如同时提供纯文本和 HTML 版本的邮件）|
|`multipart/byteranges`|字节范围响应（用于断点续传）|

---

### 八、model 类型 — 3D 模型数据

|Content-Type|文件扩展名|说明|
|---|---|---|
|`model/gltf-binary`|.glb|glTF 二进制格式|
|`model/gltf+json`|.gltf|glTF JSON 格式|
|`model/obj`|.obj|Wavefront OBJ|
|`model/stl`|.stl|STL 3D 打印格式|
|`model/3mf`|.3mf|3D 制造格式|

---

### 九、常用 charset 参数

|字符集|说明|
|---|---|
|`UTF-8`|最常用，支持全球字符集（推荐默认值）|
|`ISO-8859-1`|拉丁字母，旧标准默认值|
|`GBK` / `GB2312`|简体中文|
|`UTF-16`|双字节编码|

---

### 十、开发中最常接触的 5 种 Content-Type

|Content-Type|使用场景|
|---|---|
|`application/json`|RESTful API 请求/响应|
|`application/x-www-form-urlencoded`|HTML 表单默认提交|
|`multipart/form-data`|文件上传|
|`text/html`|网页响应|
|`application/octet-stream`|文件下载|

---

### 最佳实践

- **始终明确指定 Content-Type**，避免依赖默认值
- 文本类 MIME 建议加上 `charset=UTF-8`，避免中文乱码
- 使用 IANA 官方注册的 MIME 类型，避免自定义非标准类型
- 已知二进制文件应使用专属 MIME（如 PDF 用 `application/pdf`），而非统一用 `application/octet-stream`

