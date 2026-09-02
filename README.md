编程基础知识笔记（持续更新中）

## 配置obsidian 

#### bash命令
git --version

git config --global user.name "TODESENGEL1116"
git config --global user.email " ***@gmail.com"  (github注册邮箱)

git init
git config --local commit.gpgsign false
git remote add origin https://github.com/TODESENGEL1116/ProgrammingNoteOnObsidian.git  （github上自建的仓库）
git add .
git commit -m "Initial vault setup"
git push -u origin main


## 配置 Obsidian Git 插件

### 1. 安装插件

**设置 → 第三方插件 → 关闭安全模式 → 浏览 → 搜索 "Git" → 找到作者 Vinzent 的 Obsidian Git → 安装 → 启用**

### 2.打开命令面板 （win+P）

**Git: Commit all changes 
Git: Push
