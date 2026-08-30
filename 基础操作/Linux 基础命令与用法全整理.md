 
这份整理覆盖了日常开发、运维 90% 的使用场景，按功能模块分类，从入门到进阶均可直接查阅使用。

---

#### 一、基础入门命令（新手必会）

- **`pwd`**：显示当前所在目录的完整绝对路径，刚登录系统不确定位置时优先使用。
- **`ls`**：列出当前目录下的文件和目录，常用参数：
    - `ls -l`：显示文件权限、所有者、大小、修改时间等详细信息（可简写为`ll`）
    - `ls -a`：显示所有文件，包括以`.`开头的隐藏文件
    - `ls -lh`：以 KB/MB/GB 等易读格式显示文件大小
- **`cd`**：切换工作目录，常用用法：
    - `cd /path/to/dir`：进入指定绝对路径目录
    - `cd ..`：返回上一级目录
    - `cd ~`：回到当前用户的主目录
    - `cd -`：切换到上一次所在的目录
- **`clear`**：清空终端屏幕内容，快捷键为`Ctrl + L`。
- **`man`**：查看命令的官方手册，遇到不熟悉的命令优先使用，比如`man ls`，按`q`退出。
- **`history`**：查看历史执行过的命令，可通过`!命令编号`快速重复执行对应命令。

---

#### 二、文件与目录管理

- **`mkdir`**：创建目录，`mkdir -p a/b/c`可递归创建多级目录，无需逐级手动创建。
- **`touch`**：创建空文件，也可用于更新已有文件的时间戳，比如`touch test.txt`。
- **`cp`**：复制文件或目录，复制目录时必须加`-r`参数递归复制，比如`cp -r dir1 dir2`；`cp -a`可保留文件的所有属性（权限、时间戳等）。
- **`mv`**：移动文件或重命名，两种用法：
    - `mv old.txt new.txt`：重命名文件
    - `mv file.txt /backup/`：移动文件到指定目录
- **`rm`**：删除文件或目录，高危命令需谨慎使用：
    - `rm file.txt`：删除单个文件
    - `rm -r dir`：递归删除目录及其中所有内容
    - `rm -f file`：强制删除不提示确认
    - ⚠️ 严禁执行`rm -rf /`，会删除系统根目录所有文件导致系统崩溃。
- **`ln`**：创建链接文件，`ln -s 源文件 链接名`创建软链接（类似 Windows 快捷方式）。
- **`find`**：按条件查找文件，比如`find / -name "*.conf"`查找所有`.conf`配置文件，`find / -size +100M`查找大于 100MB 的文件。


# Linux rm 删除命令

| 命令       | 类比          | 能干啥           |
| -------- | ----------- | ------------- |
| `rm`     | 碎纸机         | 只能碎单张纸（单个文件）  |
| `rm -r`  | 碎纸机+吸尘器     | 能连整个文件夹一起清掉   |
| `rm -ri` | 吸尘器但每吸一个都问你 | 清文件夹，但每删一个都确认 |

### 参数小注解

- `-r`：`recursive` 递归，向下遍历文件夹里所有内容
- `-i`：`interactive` 交互，删除前逐个询问确认，防止手滑删错

> ⚠️ 注意：Linux 的 rm 删除一般不走回收站，删除后很难恢复，谨慎使用 `rm -r`。
---

#### 三、文件查看与文本处理

- **`cat`**：一次性显示文件全部内容，适合查看小型文件，`cat -n`可显示行号。
- **`less`**：分页查看大型文件，支持上下翻页、搜索（按`/`输入关键词搜索），按`q`退出，比`more`功能更强大。
- **`head`/`tail`**：查看文件的头部或尾部内容：
    - `head -n 10 file.txt`：查看文件前 10 行
    - `tail -n 20 file.txt`：查看文件后 20 行
    - `tail -f app.log`：实时追踪文件内容更新，排查日志问题的必备命令
- **`grep`**：在文件中搜索匹配关键词的行，常用参数：
    - `grep "error" log.txt`：查找包含`error`的行
    - `grep -i "error" log.txt`：忽略大小写搜索
    - `grep -v "error" log.txt`：反向匹配，显示不包含`error`的行
    - `grep -r "pattern" /dir/`：递归搜索目录下所有文件
- **`wc`**：统计文件的行数、单词数、字节数，`wc -l file.txt`仅统计行数。
- **`vim`/`nano`**：终端文本编辑器，`nano`操作简单适合新手，`vim`功能更强大，进入后按`i`进入编辑模式，`:wq`保存退出，`:q!`不保存强制退出。

---

#### 四、权限与用户管理

- **`chmod`**：修改文件 / 目录的访问权限，权限数字含义：读=4、写=2、执行=1，三组数字分别对应所有者、所属组、其他用户的权限：
    - `chmod 755 script.sh`：所有者可读写执行，其他用户可读可执行
    - `chmod +x script.sh`：给文件添加执行权限
- **`chown`**：修改文件的所有者和所属组，比如`chown user:group file.txt`，`chown -R user dir`可递归修改目录下所有文件的归属。
- **`useradd`**：创建新用户，`useradd -m -s /bin/bash username`可同时创建用户主目录并指定默认 shell。
- **`passwd`**：修改用户密码，直接执行`passwd`修改当前用户密码，`passwd username`修改指定用户密码（需 root 权限）。
- **`sudo`**：以管理员（root）权限执行命令，普通用户执行系统级操作时使用，比如`sudo apt install nginx`。
- **`su`**：切换到其他用户，`su - root`切换到 root 用户并加载完整环境变量，`su username`仅切换用户不加载环境变量。

---

#### 五、系统监控与进程管理

- **`top`**：实时动态监控系统进程、CPU、内存使用情况，按`q`退出，按`M`按内存排序，按`P`按 CPU 排序；`htop`是增强版，界面更友好，需手动安装。
- **`ps`**：查看当前系统的进程快照：
    - `ps aux`：显示所有进程的详细信息
    - `ps -ef | grep nginx`：过滤查找指定进程
- **`kill`**：终止指定进程，`kill PID`发送正常终止信号，`kill -9 PID`强制终止无响应的进程（PID 可通过`ps`或`top`查看）。
- **`df -h`**：以易读格式显示磁盘分区的使用情况，排查磁盘空间不足时使用。
- **`free -h`**：以易读格式显示内存和交换空间的使用情况，重点关注`available`字段（可用内存）。
- **`uptime`**：显示系统运行时间和平均负载，负载值长期超过 CPU 核心数时说明系统压力较大。
- **`uname -a`**：显示系统内核版本、架构等完整信息。

---

#### 六、网络操作命令

- **`ip addr`**：查看本机网卡和 IP 地址信息（推荐替代老旧的`ifconfig`）。
- **`ping`**：测试与目标主机的网络连通性，`ping -c 4 baidu.com`发送 4 个测试包后自动停止，默认会持续发送，按`Ctrl + C`停止。
- **`ssh`**：远程登录服务器，`ssh user@192.168.1.1`默认使用 22 端口，`ssh -p 2222 user@host`可指定端口。
- **`scp`**：基于 SSH 的安全文件传输命令，`scp file.txt user@host:/path/`将本地文件传到远程服务器，`scp user@host:/path/file.txt ./`从远程下载文件到本地。
- **`curl`/`wget`**：终端下载文件或请求接口：
    - `wget https://example.com/file.zip`：下载文件，支持断点续传`wget -c`
    - `curl -I https://example.com`：查看网页响应头，`curl -O url`下载文件
- **`ss -tulnp`**：查看当前系统的端口监听状态和对应进程（推荐替代`netstat`）。

---

#### 七、压缩与解压

- **`tar`**：Linux 最常用的归档压缩工具：
    - 压缩：`tar -czvf archive.tar.gz dir/`，参数含义：`-c`创建、`-z`gzip 压缩、`-v`显示过程、`-f`指定文件名
    - 解压：`tar -xzvf archive.tar.gz`，`-x`表示解压
- **`zip`/`unzip`**：处理 zip 格式压缩包：
    - 压缩：`zip -r archive.zip dir/`递归压缩目录
    - 解压：`unzip archive.zip -d /target/dir`解压到指定目录

---

#### 八、软件包管理

不同发行版使用不同的包管理工具：

- **Debian/Ubuntu 系**：使用`apt`
    - `sudo apt update`：更新软件源列表
    - `sudo apt install package_name`：安装软件
    - `sudo apt remove package_name`：卸载软件
    - `sudo apt upgrade`：升级所有已安装软件
- **CentOS/RHEL 系**：使用`yum`或新一代的`dnf`
    - `sudo yum install package_name`：安装软件
    - `sudo yum update`：升级软件
    - `sudo yum remove package_name`：卸载软件

---

#### 九、进阶实用技巧

- **管道符`|`**：将前一个命令的输出作为后一个命令的输入，比如`cat log.txt | grep "error" | wc -l`统计日志中错误行数。
- **重定向**：
    - `>`：覆盖写入文件，比如`ls > filelist.txt`将目录列表保存到文件
    - `>>`：追加写入文件，比如`echo "new line" >> file.txt`在文件末尾添加内容
- **`crontab`**：设置定时任务，`crontab -e`编辑当前用户的定时任务列表。
- **`alias`**：设置命令别名简化操作，比如`alias ll='ls -lh'`，写入`~/.bashrc`可永久生效。
- **后台运行**：
    - `command &`：让命令在后台运行，关闭终端后会停止
    - `nohup command &`：让命令后台常驻运行，关闭终端也不会停止

---

#### 十、新手避坑提示

- 所有删除操作优先用`ls`确认路径，避免误删重要文件，可设置`alias rm='rm -i'`开启删除确认提示。
- 修改系统配置文件前先备份，比如`cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak`，出错可快速回滚。
- 日常操作尽量使用普通用户，仅在需要系统级权限时加`sudo`，避免误操作损坏系统。
- 遇到不熟悉的命令参数，优先用`命令 --help`或`man 命令`查看官方说明，无需死记硬背。
