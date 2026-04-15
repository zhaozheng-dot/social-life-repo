
# 整体构思

使用git工具，将PC端的obsidian和Android端的obsidian通过github进行数据同步。

![git工具同步PC与Android端Obsidian示意图](https://obsidian-image-zhaozheng-dot.oss-cn-beijing.aliyuncs.com/git%E5%B7%A5%E5%85%B7%E5%90%8C%E6%AD%A5PC%E4%B8%8EAndroid%E7%AB%AFObsidian%E7%A4%BA%E6%84%8F%E5%9B%BE.png)
# 规范

> [!info] 以下规范是为保证两端数据在进行git 操作时不会发生冲突。

- 操作任何一端的obsidian前，首先从github上pull最新分支

- PC端与Android端同时存在，优先操作PC端obsidian。

- 无PC端，操作Android端obsidian。

- 每次操作完成，需要将增量文件进行git commit并 push到 github，之后才可以操作另一个平台的obsidian。

- 定期 操作Android端obsidian 进行git pull 来对两端数据进行校准
	```bash
	git status
	
	git pull 
	
	git status
	
	```

- commit $msg---提交注释规范
	```bash
	# 数据提交的平台简称 + 说明 + 日期时间
	git commit -m "Android/PC: [简要说明修改内容] - $(date '+%Y-%m-%d %H:%M')"
	```


# PC端环境准备


-  IDEA 打开obsidian 仓库目录，并启用其中的Terminal终端

- 执行以下命令进行git仓库初始化
	```bash
	# 跳转到obsidian仓库根目录
	cd  /f/obsidian_repository/knowledge_repo

	# 添加 README文档 并初始化
	echo "# knowledge_repo" >> README.md

	# 实例化空git仓库
	git init
	
	# git 添加追踪文件
	git add README.md
	
	# git 首次conmmit
	git commit -m "first commit"
	
	# git 创建本地main分支
	git branch -M main
	
	# git 设置 remote 远程仓库地址，这里选用github
	git remote add origin https://github.com/zhaozheng-dot/knowledge_repo.git
	
	# 将创建好的git 仓库 推送到 remote,显示指定远程分支main,
	git push -u origin main
	```


- 如果PC端obsidian仓库中在第一次git提交前，有存量文件，执行以下操作
	```bash
	# 添加本地目录及其子目录中所有变更文件到 git追踪列表中
	git add .
	
	# commit 提交
	git commit -m "knowledge_repo初始化推送"
	
	# 推送
	git push -u origin main
	
	
	```



# Android端环境准备


- 安装Termux
[Termux - 一款可通过多种软件包扩展的 Android 操作系统终端模拟器](https://github.com/termux/termux-app#github)

- 安装 Termux 必要工具：
	```bash
	# 首次运行 ，更新软件包安装器
	pkg update && pkg upgrade
	
	# 安装 git 和 openssh：
	pkg install git -y
	pkg install openssh -y
	```

- 配置 Git 用户信息---拉取或推送时的必要配置
	```bash
	
	# 配置
	git config --global user.name "你的用户名"
	git config --global user.email "你的邮箱@example.com"
	
	# 验证
	git config --list
	
	```

- 配置github 密钥，实现数据保护，非登录状态下推送拉取
	```bash
	# 生成 SSH 密钥
	## 按提示直接回车，默认保存密钥到 `~/.ssh/id_rsa`
    
	## 当提示输入密码时，直接回车两次以实现无密码传输
	## `id_rsa`：私钥（不要泄露）
    
	## `id_rsa.pub`：公钥
    
	## 文件位置：`~/.ssh/` 目录
	ssh-keygen -t rsa
	
	#################################################################
	
	# 复制输出的公钥内容
    
	# 登录 GitHub，进入 **Settings > SSH and GPG keys**
    
	# 点击 **New SSH Key**，将复制的公钥粘贴到 **Key** 输入框中，并保存
	cat ~/.ssh/id_rsa.pub
	
	#################################################################
	
	# 测试连接
	ssh -T git@github.com
	```
	

    
- 设置 Termux运行沙盒访问外部存储的权限
	```bash
	# 外部存储的权限
	termux-setup-storage
	
	# 跳转到手机存储根目录
	cd /sdcard
	
	```


- 当Termux可访问外部存储时，，需授权 Git 可跨用户执行操作，确保Termux沙盒应用可操作手机目录
	```bash
	# /sdcard/obsidian/knowledge_repo 目录为Android端obsidian仓库目录
	git config --global --add safe.directory /sdcard/obsidian/knowledge_repo
	```


- 设置git 中文显示
	```bash
	# 配置 Git 正确显示中文**
	git config --global core.quotepath false
	git config --global i18n.logoutputencoding utf-8
	git config --global i18n.commitencoding utf-8
	
	#######################################################
	
	# 设置终端编码**（Termux）
	export LANG=en_US.UTF-8
	export LC_ALL=en_US.UTF-8
	
	```
	


# Android端操作

## 本次编辑之前

```bash
# 1、打开Termux

# 2、跳转到obsidian仓库目录
cd /sdcard/obsidian/knowledge_repo

# 3、拉取远程分支
git pull 

# 4、查看未追踪文件（理论上应该不存在任何未追踪文件）
git status

```


## 编辑完成之后
```bash

# 1、跳转到obsidian仓库目录
cd /sdcard/obsidian/knowledge_repo

# 2、将obsidian仓库根目录下的所有文件变更 添加到暂存区
git add .

# 3、commit 提交
git commit -m "Android: [简要说明修改内容] - $(date '+%Y-%m-%d %H:%M')"

# 4、推送到远程仓库
git push origin main

# 5、查看当前git 仓库状态（理论上应该不存在任何未追踪文件）
git status
```


# PC端操作

> [!info] 同 Android端操作基本相同

> [!warning] bash语法与powershell语法差异
> PowerShell无法识别  Date 参数，使用以下命令替代
> ```powershell
> git commit -m "PC: [README.md 增加一些信息] - $(Get-Date -Format 'yyyy-MM-dd HH:mm')"
> ```


# 踩坑记录

> [!warning] 踩坑 系统分身安装 termux failed 失败!!!
> ```bash
> # 报错提示：[^1]
> **"Termux can only be run as the primary user."** （Termux 只能作为主用户运行。）
> ```
> 1,realme gt8 双系统，系统分身安装 Termux 无法运行，需要**卸载分身安装的Termux**
> 在主系统中安装，否则会造成应用签名冲突

> [!warning] 踩坑 PC端开代理，向github推送文件 failed失败!!!
> 



  >[!warning] 踩坑 github上下载软件 不匹配手机CPU架构
  
  
# 脚注

[^1]: Termux 是一个在 Android 上模拟 Linux 环境的终端模拟器。它的运行机制非常依赖于 Android 系统的底层用户 ID（UID）。
	
	在 Android 系统中，“主用户”（Primary User，通常也就是机主）的 User ID 是 0。报错触发： Termux 检测到当前的运行环境 不是 User 0。
	
	由于 Termux 需要在特定的系统路径（如 /data/data/com.termux/files/...）下进行读写操作来安装 "bootstrap"（即基础的 Linux 包环境），如果非主用户运行，权限路径会发生变化，导致安装失败



标签： #Termux #GitHub #SSH #Git 