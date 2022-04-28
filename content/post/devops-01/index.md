---
title: 运维笔记01：Linux 基础
description: Linux 简介，文件与目录管理，用户与权限管理，Vim 简介，Bash 基础
slug: "devops-01"
date: 2022-03-27
tags: 
    - DevOps
---

要好好准备找工作！

## Linux 简介

### 历史

在计算机刚发明的年代，由于计算机造价昂贵，人们希望能高效利用宝贵的上机时间，于是分时操作系统诞生了。当时的计算机能提供大约 16 个文字终端使用，然而这还远远不能满足需求。

1963 年，MIT、贝尔实验室和通用电气公司启动了一项雄心勃勃的计划：打造一个能提供 300 个以上文字终端使用的计算机系统（称为“Multics”）。计划的参与者之一、贝尔实验室的 Ken Thompson 受到启发，在 1969 年一个偶然的假期里[^1]，为 PDP-7 电脑编写了一个简单的操作系统（戏称为“Unics”），后来这个系统逐渐演变成大名鼎鼎的 Unix。

[^1]: 其实只是为了打游戏……

Unix 逐渐展示出了足够的商业价值，引发了一系列著作权官司，于是贝尔实验室母公司 AT&T 在 1979 年决定禁止向学生开放源码。1984 年，为了方便教学，Andrew Tanenbaum[^2] 决定自己动手编写一套精简版的 Unix，称为 Minix。同年，Richard Mathew Stallman 发起了 GNU 计划，然后成立了自由软件基金会。

[^2]: 《计算机网络》、《操作系统：设计与实现》和《现代操作系统》的作者。Andrew Tanenbaum 保留了 Minix 的相关权利，但购买教材的人可以免费得到一份 Minix 源代码拷贝。

自由软件基金会发布了一系列方便的软件（GCC, Emacs, BASH, ...），但却迟迟未出操作系统。1991 年，芬兰大学生 Linus Torvalds 受到 Minix 启发，在贷款买来的 386 电脑上使用 GNU 的 BASH, GCC 等工具，成功实现了一个类似 Unix 的小型操作系统，于是发布到 BBS 上开放下载并征求意见。FTP 服务器管理员把下载目录命名为 Linux，从此这个系统就被称为 Linux 了。

虽然 Linux 的桌面操作系统在平时非常罕见，但 95% 以上的网站服务器都在使用 Linux[^3]。这一系列的笔记就从 Linux 开始吧～

[^3]: https://en.wikipedia.org/wiki/Linux

### 初体验

传统意义上的 Linux 指的是 Linux Kernel，使用起来并不方便。所幸目前已经有各种打包好的发行版（如 Ubuntu、Arch Linux 等），你可以通过虚拟机运行，也可以刻录到 U 盘直接启动，不想动手也可以使用云服务器。这里使用的是免费的 [Google Colaboratory](https://colab.research.google.com)，登录后可以看见：

```bash
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 5.4.144+ x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
...
~#
```

和平时使用的图形界面完全不一样，接下来可以试试执行简单的命令：

```bash
~# w
 13:55:23 up  1:13,  1 user,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    127.0.0.1        12:45    1.00s  0.00s  0.00s w
~# date
Sun Mar 27 13:55:30 UTC 2022
~# cal
     March 2022
Su Mo Tu We Th Fr Sa
       1  2  3  4  5
 6  7  8  9 10 11 12
13 14 15 16 17 18 19
20 21 22 23 24 25 26
27 28 29 30 31

~# pwd
/root
~# id
uid=0(root) gid=0(root) groups=0(root)
~# ls
```

### 参数与帮助

在执行一系列命令后，你未免会感觉有点失望：虽然这个操作系统勉强能用，但用起来很不顺手。为了让这些工具更顺手，往往需要传入额外的参数，比如：

```bash
~# date -u +"%Y-%m-%d %H:%M:%S"
2022-03-27 14:16:52
~# cal 01 2077
    January 2077
Su Mo Tu We Th Fr Sa
                1  2
 3  4  5  6  7  8  9
10 11 12 13 14 15 16
17 18 19 20 21 22 23
24 25 26 27 28 29 30
31
~# ls -alh
total 68K
drwx------ 1 root root 4.0K Mar 27 14:08 .
drwxr-xr-x 1 root root 4.0K Mar 27 12:42 ..
-r-xr-xr-x 1 root root 1.2K Jan  1  2000 .bashrc
drwxr-xr-x 3 root root 4.0K Mar 23 14:22 .gsutil
drwxr-xr-x 5 root root 4.0K Mar 23 20:22 .ipython
drwx------ 2 root root 4.0K Mar 23 20:22 .jupyter
drwxr-xr-x 2 root root 4.0K Mar 27 12:42 .keras
drwxr-xr-x 1 root root 4.0K Mar 23 20:22 .local
drwxr-xr-x 4 root root 4.0K Mar 23 20:22 .npm
-rw-r--r-- 1 root root  148 Aug 17  2015 .profile
-r-xr-xr-x 1 root root  254 Jan  1  2000 .tmux.conf
```

一般的 Linux 命令形式为 `command  [-options]  [parameter1...]`，要知道命令接受什么参数，只能查阅命令对应的帮助文档，如执行 `command --help` 或者 `man command`：

```bash
~# cal --help
cal: invalid option -- '-'
Usage: cal [general options] [-jy] [[month] year]
       cal [general options] [-j] [-m month] [year]
       ncal -C [general options] [-jy] [[month] year]
       ncal -C [general options] [-j] [-m month] [year]
       ncal [general options] [-bhJjpwySM] [-H yyyy-mm-dd] [-s country_code] [[month] year]
       ncal [general options] [-bhJeoSM] [year]
General options: [-31] [-A months] [-B months] [-d yyyy-mm]
```

然而查阅帮助文档很耗时，也难以记忆，故在开始之前，建议安装 `tldr`。

### 安装新软件

在 Ubuntu，安装软件只需要一行命令：

```bash
~# sudo apt install tldr
Reading package lists... Done
...
The following NEW packages will be installed:
  tldr
0 upgraded, 1 newly installed, 0 to remove and 41 not upgraded.
Need to get 860 kB of archives.
...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
```

那么怎么卸载呢？没关系，有 [tldr](https://tldr.sh/)！

```bash
~# tldr apt
...
 - Update the list of available packages and versions (it's recommended to run this before other apt commands):
   sudo apt update

 - Search for a given package:
   apt search {{package}}

 - Show information for a package:
   apt show {{package}}

 - Install a package, or update it to the latest available version:
   sudo apt install {{package}}

 - Remove a package (using purge instead also removes its configuration files):
   sudo apt remove {{package}}
...
```

可以看到卸载已安装程序的命令是 `sudo apt remove command`（如果需要同时移除程序的配置文件，把 `remove` 改成 `purge`）。遇到不懂的命令，只要 `tldr` 一下就好！

## 文件与目录管理

### 概述

和 Windows 的文件目录不太一样，Linux 的所有文件和目录都是从根目录开始的，被称为目录树。每一个目录都可以挂载各种文件系统，如硬盘的分区、光盘、ISO 文件、网络文件系统等等。

![Linux 根目录[3]](linux_dir.jpg)

从根目录开始，每个目录或文件都有唯一的路径，称为绝对路径。与绝对路径相对应的，自然是相对路径了，即使用 `.` 表示当前文件夹，使用 `..` 表示父文件夹，例如 `/var/log` 目录在 `/root` 下的相对路径为 `../var/log`。对于 Linux 运维而言，以下是需要定期备份的目录：

- `/etc` 存储了系统的重要配置文件；
- `/home` 或者 `/root` 是默认的用户目录；
- `/srv` 可能是服务器程序数据存放目录；
- `/var/log` 存放了大量的日志；
- `/var/spool` 存放队列文件，如 `/var/spool/mail` 存放收到的新邮件，`/var/spool/cron` 和 `/var/spool/at` 存放等待执行的任务等；
- `/var/www/html` 是常见的 Web 服务器页面默认目录；
- ……

### 基本操作

先来尝试最基本的目录和文件操作：

- `pwd` 打印当前工作目录；
- `cd path` 切换工作目录到指定路径；
- `ls path` 列出路径下的所有非隐藏文件（常常使用 `ls -alh path`）；
- `file filename` 确定文件类型；

现在可以像 Windows 文件管理器一样管理文件了！不懂可以使用 `tldr`：

- `mkdir dirname` 创建新文件夹；
- `touch filename` 创建新文件；
- `cp source destination` 复制文件；
- `mv source destination` 移动/重命名文件；
- `rm filename` 删除文件（直接删除，没有回收站和确认选项！）；
- `ln -s source destination` 创建文件的符号链接（快捷方式）； 
- `ln source destination` 创建文件的硬链接（替身）； 

### 查看文件内容

因为 Linux 命令行并不是图形界面，能打开查看或编辑的往往只有文本文件。常用的文件查看命令有：

- `cat filename` 打印文件所有内容，加上参数 `-n` 可显示行号，还可以连接文件；
- `head filename` 打印文件头十行内容，加上参数 `-n 20` 打印前 20 行；
- `tail filename` 打印文件最后十行内容，加上参数 `-n 20` 打印最后 20 行；
- `less filename` 打开文件阅读器，可以上下翻页，按 `q` 退出；
- `diff file1 file2` 比较两个文件的不同之处；
- `od filename` 打印二进制文件。

其中 `less` 有许多方便的快捷键，如 `G` 跳转到文件末尾，`F` 实时刷新，`/` 搜索关键词，是查看日志文件的好帮手。

### 管道与重定向（重点）

直接打印文本文件内容不太美观，我们往往需要对文本进行简单的处理、格式化再输出，有时还希望把结果保存到指定文件。



Linux 还有许多可以配合管道处理的文本小工具：

- echo
- sort
- uniq
- cut
- tr
- grep

现在你已经可以完成一些简单的任务了，比如

不过，最著名的 Unix 文本处理工具非 [AWK](https://github.com/mylxsw/growing-up/blob/master/doc/%E4%B8%89%E5%8D%81%E5%88%86%E9%92%9F%E5%AD%A6%E4%BC%9AAWK.md) 和 [SED](https://github.com/mylxsw/growing-up/blob/master/doc/%E4%B8%89%E5%8D%81%E5%88%86%E9%92%9F%E5%AD%A6%E4%BC%9ASED.md) 莫属。

### 查找文件

which whereis locate find

### 文件系统与存储管理

### Raid

### LVM

## 用户与权限管理

### 用户、群组和其他

su与sudo
visudo
passwd
adduser/useradd
groupadd/addgroup
usermod
getfacl/setfacl

chown chgrp chmod umask

### UID GID effective

### SUID SGID Sticky Bit

### sudo 的坑

### 文件系统

### ACL 

### 最小权限原则

## Vim 简介

### 活下来

### 常用模式

### 行间与行内跳转

### 文本编辑

### 多行模式

### 多文件切换

## Bash 基础

### 常用快捷键

- `TAB` 可以进行自动补全；
- 上下键可以按顺序翻查以前输入的命令；
- `Ctrl+R` 可以查找历史命令；


### 变量定义与环境变量

### 条件判断

### 循环

### Bash 编程基础

## 其他话题

### 开机流程与自启动项管理



### 进程管理与任务控制

ps top htop

fg bg jobs 

kill killall


### 系统服务（Systemd）

作用：开机自动启动服务器软件。

### 定时服务（Crontab）

作用：定期自动备份，每天自动安装安全更新等。

### 保持执行（nohup, screen/tmux）

作用：执行耗时长的任务，如文件同步。

### 遇到麻烦？



## 扩展阅读

[1] 黄玮的 [课件](https://c4pr1c3.github.io/cuc-wiki/linux/2022/index.html) 与 [课程录像](https://www.bilibili.com/video/BV1Hb4y1R7FE)；

[2] 鸟哥的 Linux 私房菜：[基础学习训练篇](https://linux.vbird.org/linux_basic_train/centos8/)；

[3] [快乐的 Linux 命令行](https://billie66.github.io/TLCL/book/)。