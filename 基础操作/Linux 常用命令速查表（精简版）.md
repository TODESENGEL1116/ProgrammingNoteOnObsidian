#### 目录与文件

| 命令      | 用法                        | 说明                    |
| ------- | ------------------------- | --------------------- |
| `pwd`   | `pwd`                     | 显示当前路径                |
| `ls`    | `ls -lah`                 | 列出全部文件（含隐藏），显示详情和易读大小 |
| `cd`    | `cd ~` / `cd ..` / `cd -` | 回主目录 / 上一级 / 上次目录     |
| `mkdir` | `mkdir -p a/b/c`          | 递归创建多级目录              |
| `touch` | `touch file.txt`          | 创建空文件                 |
| `cp`    | `cp -r src/ dst/`         | 递归复制目录                |
| `mv`    | `mv old.txt new.txt`      | 重命名或移动文件              |
| `rm`    | `rm -rf dir/`             | 强制递归删除（慎用！）           |
| `ln`    | `ln -s src link`          | 创建软链接                 |
| `find`  | `find / -name "*.log"`    | 按名称查找文件               |

---

#### 文件查看与文本处理

|命令|用法|说明|
|---|---|---|
|`cat`|`cat file.txt`|显示文件全部内容|
|`less`|`less file.txt`|分页查看大文件（`/`搜索，`q`退出）|
|`head`|`head -n 20 file.txt`|查看前 20 行|
|`tail`|`tail -f app.log`|实时追踪日志|
|`grep`|`grep -rn "error" /dir/`|递归搜索关键词并显示行号|
|`wc`|`wc -l file.txt`|统计行数|
|`vim`|`vim file.txt`|编辑文件（`i`编辑，`:wq`保存退出）|

---

#### 权限与用户

|命令|用法|说明|
|---|---|---|
|`chmod`|`chmod 755 file`|设置权限（7=读写执行，5=读执行，4=只读）|
|`chown`|`chown user:group file`|修改文件归属|
|`sudo`|`sudo command`|以 root 权限执行|
|`su`|`su - username`|切换用户|
|`useradd`|`useradd -m user`|创建用户|
|`passwd`|`passwd user`|修改密码|

---

#### 系统监控

|命令|用法|说明|
|---|---|---|
|`top`|`top`|实时进程监控（`q`退出）|
|`ps`|`ps aux \| grep nginx`|查找指定进程|
|`kill`|`kill -9 PID`|强制终止进程|
|`df`|`df -h`|磁盘使用情况|
|`free`|`free -h`|内存使用情况|
|`uptime`|`uptime`|系统运行时长与负载|

---

#### 网络

|命令|用法|说明|
|---|---|---|
|`ip addr`|`ip addr`|查看 IP 地址|
|`ping`|`ping -c 4 baidu.com`|测试网络连通性|
|`ssh`|`ssh user@host -p 22`|远程登录|
|`scp`|`scp file user@host:/path/`|远程传输文件|
|`curl`|`curl -O url`|下载文件|
|`wget`|`wget -c url`|下载文件（支持断点续传）|
|`ss`|`ss -tulnp`|查看端口监听状态|

---

#### 压缩解压

|命令|用法|说明|
|---|---|---|
|`tar` 压缩|`tar -czvf archive.tar.gz dir/`|压缩为 `.tar.gz`|
|`tar` 解压|`tar -xzvf archive.tar.gz`|解压 `.tar.gz`|
|`zip`|`zip -r archive.zip dir/`|压缩为 `.zip`|
|`unzip`|`unzip archive.zip -d /target/`|解压到指定目录|

---

#### 软件管理

|发行版|安装|卸载|更新|
|---|---|---|---|
|Ubuntu/Debian|`apt install pkg`|`apt remove pkg`|`apt update && apt upgrade`|
|CentOS/RHEL|`yum install pkg`|`yum remove pkg`|`yum update`|

---

#### 常用技巧速记

|符号/技巧|说明|示例|
|---|---|---|
|`\|` 管道|前者输出传给后者|`cat log \| grep "error"`|
|`>` 覆盖写入|输出写入文件（覆盖）|`ls > list.txt`|
|`>>` 追加写入|输出追加到文件末尾|`echo "hi" >> log.txt`|
|`&&` 顺序执行|前一条成功才执行后一条|`make && make install`|
|`Ctrl+C`|终止当前命令|—|
|`Ctrl+Z`|挂起当前命令到后台|用 `fg` 恢复，`bg` 后台运行|
|`nohup cmd &`|后台常驻运行|`nohup python app.py &`|
|`alias`|设置命令别名|`alias ll='ls -lah'`|

---

这份速查表涵盖了日常使用最高频的命令和技巧，建议收藏随时查阅。如果需要针对某个具体场景（如 Docker、Nginx、数据库等）的命令整理，随时告诉我！