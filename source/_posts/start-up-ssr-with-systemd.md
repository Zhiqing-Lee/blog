---
title: 通过Systemd让SSR开机自启动
date: 2017-12-25 16:27:57
tags:
- Linux
- SSR
---

果然，之前让项目支持部署到Docker这个决定是很正确的。这不，又把系统从Windows换成了Linux。而原因竟然是我的笔记本在Windows下玩游戏会无限崩溃，而在Linux下不会！！！用Linux是为了玩游戏我恐怕还是第一人吧!既然装Linux是为了玩游戏，那么也没有必要双系统了，毕竟个人感觉Linux下搞开发还是比Windows爽很多的。不过还是吐槽一下Linux上玩游戏的确还是没有Windows上流畅，可能是因为显卡驱动程序太旧了吧。

装上新系统第一件事当时然是解决科学上网。之前一直是用的SS，Linux下也有QT版的GUI客户端，用着还不错。可是在之前用Windows的时间里，SS被封了。于是换成了魔改版的SSR，Windows下也有GUI客户端，用着还不错。可是一换到Linux下，SSR好像没有GUI客户端。只好用CLI客户端，反正又这程序开着基本上看不着，开机时启动、关机时关闭。那么问题来了，不可能每次开机时手动开启吧，得让它开机时自启动啊。

之前试过在 `/etc/profile` 文件中添加命令以达到自启动的目的，但是会出现一些小问题。使用 `source` 命令时会再启动一次该程序。SSR的服务端是使用的Supervisor来管理的，但是又不想在电脑中多装软件。于是想到还有个Systemd。

好了，说了这么多废话开始正题吧。

首先在 `/etc/systemd/system` 目录中新建名为 `ssr.service` 的配置文件，并写入以下内容:

```
[Unit]
Description=Started SSR Service
After=network.target
Wants=network.target

[Service]
ExecStart=/usr/bin/python /home/zhiqing/App/shadowsocksr/shadowsocks/local.py -c /etc/ssr.json
```

> 自行修改其中ExecStart项的值，注意必须使用绝对目录写法。

这样就在Systemd中添加了一个服务。
接着在命令行中输入 `systemctl enable ssr` 就可以让该服务启用开机自启。
使用 `systemctl start ssr` 可立即启动服务。
使用 `systemctl stop ssr` 可停止该服务。（不过这里没有写停止服务的命令，应该是不能用的）

