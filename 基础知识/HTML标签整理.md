HTML 标签按功能分门别类整理如下，可直接作为速查表使用。

### 一、文档结构与元数据（头部信息）

|标签|说明|
|---|---|
|`<!DOCTYPE html>`|文档类型声明，HTML5 固定写法|
|`<html>`|根元素|
|`<head>`|元数据容器|
|`<title>`|页面标题（SEO 关键）|
|`<meta>`|字符集、视口、关键词等元信息|
|`<link>`|引入外部资源（CSS、favicon、预加载）|
|`<style>`|内联 CSS|
|`<base>`|页面所有相对 URL 的基准地址|
|`<script>`|JavaScript（可放 head 或 body 末尾）|
|`<noscript>`|浏览器禁用脚本时的替代内容|
|`<body>`|页面可见内容主体|

### 二、分区与布局（语义化结构）

|标签|说明|
|---|---|
|`<header>`|页眉、区块头部|
|`<nav>`|导航链接区域|
|`<main>`|页面主要内容（每页唯一）|
|`<article>`|独立可复用的内容（文章、帖子）|
|`<section>`|有主题的内容区块|
|`<aside>`|侧边栏、附属内容|
|`<footer>`|页脚|
|`<div>`|无语义通用容器（布局用）|
|`<h1>`–`<h6>`|标题，层级不可跳级|
|`<address>`|联系信息|
|`<hr>`|主题分隔线|

### 三、文本语义

|标签|说明|
|---|---|
|`<p>`|段落|
|`<span>`|无语义行内容器|
|`<br>`|换行|
|`<blockquote>`|块级引用|
|`<q>`|行内短引用|
|`<cite>`|作品标题|
|`<code>`|代码片段|
|`<pre>`|预格式化文本（保留空格换行）|
|`<kbd>`|键盘输入|
|`<samp>`|程序输出样例|
|`<var>`|变量名|
|`<abbr>`|缩写（配合 title）|
|`<dfn>`|术语定义|
|`<time>`|日期时间（配合 datetime）|
|`<data>`|机器可读值|
|`<mark>`|高亮标记|
|`<strong>`|重要（加粗，有语义）|
|`<em>`|强调（斜体，有语义）|
|`<b>`|仅视觉加粗无语义|
|`<i>`|仅视觉斜体无语义|
|`<u>`|下划线（常用于拼写错误标注）|
|`<s>` / `<del>`|删除线；`<del>` 带修改语义|
|`<ins>`|新增内容|
|`<sub>` / `<sup>`|下标 / 上标|
|`<small>`|附属小字（版权、注释）|
|`<wbr>`|可选换行点|
|`<ruby>` / `<rt>` / `<rp>`|注音（拼音、假名）|
|`<bdi>` / `<bdo>`|双向文本隔离 / 方向覆盖|
|`<details>` / `<summary>`|折叠面板|
|`<dialog>`|对话框/模态框|

### 四、列表

|标签|说明|
|---|---|
|`<ul>`|无序列表|
|`<ol>`|有序列表（start、reversed、type 属性）|
|`<li>`|列表项|
|`<dl>` / `<dt>` / `<dd>`|描述列表（键值对、术语解释）|
|`<menu>`|命令菜单（现多用于工具栏上下文菜单）|

### 五、表格

|标签|说明|
|---|---|
|`<table>`|表格|
|`<caption>`|表格标题|
|`<thead>` / `<tbody>` / `<tfoot>`|表头 / 表体 / 表尾分组|
|`<tr>`|行|
|`<th>`|表头单元格（scope 属性标明作用范围）|
|`<td>`|数据单元格|
|`<colgroup>` / `<col>`|列分组与列样式|

### 六、表单与输入

**容器与控件：**

|标签|说明|
|---|---|
|`<form>`|表单（action、method、enctype）|
|`<fieldset>` / `<legend>`|控件分组及分组标题|
|`<label>`|控件标签（for 绑定 id，提升可访问性）|
|`<input>`|输入框（见下方 type）|
|`<textarea>`|多行文本|
|`<select>` / `<option>` / `<optgroup>`|下拉选择及选项分组|
|`<button>`|按钮（type 默认 submit，注意区分）|
|`<datalist>`|输入建议列表|
|`<output>`|计算结果输出|
|`<progress>`|进度条|
|`<meter>`|标量测量值（如磁盘用量）|

**input 常用 type：** `text`、`password`、`email`、`tel`、`url`、`number`、`range`、`date`、`time`、`datetime-local`、`month`、`week`、`color`、`checkbox`、`radio`、`file`、`hidden`、`search`、`submit`、`reset`、`button`、`image`。

### 七、链接与媒体嵌入

|标签|说明|
|---|---|
|`<a>`|超链接（href、target、download、rel）|
|`<img>`|图片（alt 必填、loading="lazy"、srcset 响应式）|
|`<picture>` / `<source>`|响应式图片源选择|
|`<figure>` / `<figcaption>`|独立图文内容及说明|
|`<svg>`|矢量图形|
|`<canvas>`|位图画布（JS 绘制）|
|`<video>` / `<audio>`|音视频（controls、autoplay、muted、loop、poster）|
|`<track>`|字幕轨道（WebVTT）|
|`<iframe>`|内嵌网页（sandbox、loading="lazy"）|
|`<embed>` / `<object>` / `<param>`|插件或外部对象（已少用）|
|`<map>` / `<area>`|图片热区映射|

### 八、交互与脚本化

|标签|说明|
|---|---|
|`<script>`|JS 脚本（defer、async、type="module"）|
|`<template>`|客户端模板（不渲染，供 JS 克隆）|
|`<slot>`|Web Components 插槽|
|`<canvas>`|绘图上下文|
|`<portal>`|（实验性）嵌入式预渲染页面|

### 九、已废弃 / 避免使用的标签

`<font>`、`<center>`、`<big>`、`<strike>`、`<tt>`、`<acronym>`、`<applet>`、`<basefont>`、`<bgsound>`、`<blink>`、`<marquee>`、`<frame>`、`<frameset>`、`<noframes>`、`<isindex>`、`<xmp>`。

> 提示：`<table>` 用于布局、`<br>` 制造间距、`<i>/<b>` 代替 CSS 都是常见反模式。

### 十、速记要点

1. **语义优先**：能用 `<nav>`、`<article>`、`<button>` 就别用 `<div>` + class。这直接影响 SEO、无障碍阅读（屏幕阅读器）和代码可维护性。
2. **自闭合规则**：HTML5 中 `<img>`、`<br>`、`<input>`、`<meta>`、`<link>`、`<hr>`、`<source>`、`<track>`、`<area>`、`<base>`、`<col>`、`embed>`、`<param>`、`<wbr>` 无结束标签，写成 `<img />` 也合法但非必需。
3. **块级 vs 行内**：`<p>` 里不能放 `<div>`、`<h1>` 等块级元素；`<a>` 在 HTML5 中可包裹块级内容（但不能嵌套另一个 `<a>`）。
4. **必填属性别省**：`<img alt="">`、`<label for="">`、`<th scope="">`、`<html lang="zh-CN">`。
5. **表单可用性**：`<button type="submit">` 比 `<div onclick>` 更可靠（支持回车提交、原生 disabled、表单验证）。