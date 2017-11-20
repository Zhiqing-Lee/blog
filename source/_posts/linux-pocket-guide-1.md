---
title: 《Linux pocket guide》读书笔记之一：文件系统
date: 2016-7-19
categories:
- 读书笔记
- Linux
tags:
- Linux
- Code
- 《Linux pocket guide》
- 读书笔记
---

linux中文件系统为树形结构，并用/分隔，例：`/home/zhiqing`表示根目录下的home目录中的zhiqing文件夹
- 根目录：`/`（绝对路径以/开头）
- 家目录：`～`，例：`～/Documents`代表`/home/username/Documents`
- 父目录：`..`，例：`../file`代表上一级目录下的file文件

## 家目录

普通用户的家目录为`/home/username`，超级用户（root）的家目录为`/root`

常用操作：

```bash
echo $HOME  #显示当前用户的家目录
echo ~  #同上
cd ~/Documents  #进入当前用户家目录中的Documents文件夹
cd ~zhiqing  #进入用户zhiqing的家目录
```

## 系统目录

系统目录大致分为三部分：`/Scope(域)/Category(分类)/Application(应用)`。例如：`/usr/local/share/emacs`，其中`usr/local`为Scope，`share`为Category，`emacs`为Application。

### Category（分类）

分类通常指明了目录的类型，比如目录名为`bin`则代表该目录下存放的是可执行文件。常见分类如下：

- 程序类
  - `bin` 程序的二进制文件（可执行文件）
  - `sbin` 程序的二进制文件（通常需要root权限）
  - `lib` 程序的库文件
- 文档类
  - `doc` 文档
  - `info` emacs的内置文档
  - `man` 提供给`man`命令显示的文档
  - `share` 程序的特殊文件，如安装说明
- 配置类
  - `etc` 系统的配置文件和和各种其他的配置文件
  - `init.d` linux启动相关的配置文件
  - `rc.d` linux启动相关配置文件，按级别分类，如：rc1.d,rc2.d,...
- 编程类
  - `include` 头文件
  - `src` 程序源文件
- 网站类
  - `cgi-bin` 网站运行时的脚本或程序
  - `html` 网页文件
  - `public_html` 网页文件，通常存在与用户家目录
  - `www` 网页文件
- 硬件类
  - `dev` 磁盘或其他硬件的接口设备文件
  - `media` 挂载点：连接到外部磁盘
  - `mnt` 挂载点：链接到外部磁盘
- 运行时类
  - `lock` 锁文件
  - `log` 日志文件，包括错误、警告和提示信息
  - `run` PID文件
  - `tmp` 零时文件
  - `proc` 操作系统的状态

### Scope（域）

域通常包含了目录的描述

- `/` linux的系统文件
- `/usr` 更多的linux系统文件
- `/usr/local` 为组织或个人开发的本地化的系统文件
- `/usr/games` 游戏

一个分类会出现在多个域中，例如：`/lib`、`/usr/lib`、`/usr/local/lib`和`/usr/games/lib`可能会同时存在。

### Application（应用）

应用部分通常为程序名。系统目录中，在Scope和Category之后，每个应用都会有一个自己的子文件夹（如：`/usr/local/doc/myprogram`）用于存放自己需要的文件。

## 操作系统目录

操作系统目录是保证系统内核运行的目录，最少要包含以下几个目录：

目录 | 作用
----|-------
`/boot` | 存放系统启动相关的文件，Linux内核(kernel)一般存放在`/boot/vmlinuz`或类似的目录中。
`/lost+found` | 用于磁盘修复工具恢复已损坏文件
`/proc` | 系统中正在运行的进程的描述

`/proc` 目录中的文件显示的是正在运行的内核和一些特殊的配置信息。它们的大小总是为0、修改时间为现在，并且权限为只读(-r--r--r--)。常见文件如下：

文件 | 含义
----|--------
`/proc/ioports` | 计算机中IO设备的列表
`/proc/cpuinfo` | CPU的信息
`/proc/version` | 操作系统的版本，同`uname`命令显示的内容
`/proc/uptime` | 系统运行时间

## 文件保护

linux中的权限包括：读(read)、写(write)、执行(execute)

- 读：读取文件内容、显示目录中的文件
- 写：修改或删除文件、创建或删除目录
- 执行：运行脚本或二进制文件、进入目录

执行`ls -l file`命令，每条记录前10个字符即为该文件的权限。表现形式类似：`-rwxr-x---`。解释如下：

位数 | 含义
----|------
1 | 文件类型（-为文件，d为目录，l为链接，p为管道，c为字符设备，b为块设备）
2-4 | 文件所有者的读、写、执行权限（为-则表示没有该权限）
5-7 | 文件所在组的读、写、执行权限
8-10 | 其他用户的读、写、执行权限

所以`-rwxr-x---`表示`file`是一个文件，该文件的所有者可以读、写、执行，该文件所所在组的用户可以读和执行，其他用户不能读、写和执行。
