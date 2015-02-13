---
layout: post
title: some ssh usages
date 2015-2-14 01:04:12
tags: linux sysops
---

*tips one*

曾经有点傻乎乎的上传公钥每次都是用scp,后来发现原来还有个更好用的ssh-copy-id,
可以直接 -i 指定公钥文件，这样省去了scp	过去后还要手动导入到authorized_keys这一步。

	ssh-copy-id -i ~/.ssh/id_rsa.pub user@my.server.org

*tips two*

开启本地端口监听，把此监听的端口的数据转发到远端服务器，可用做ssh代理：

	ssh -N -f -D 1080 user@remote.server.org

> *'-N' 表示不执行远端命令,对只做端口转发来说很有用。
> *'-f' 表示在后台运行。
> *'-D' 表示开启一个本地的socket端口用来监听，后跟端口号。

然后应用程序可以使用此命令开启的socket代理，地址为127.0.0.1,协议为socket，端口为上面命令指定的端口，本例中为1080，。
这样所有本地发往1080的数据都会加密传输到远端服务器走一遍，利用远端服务器中转了数据。



