---
layout: post
title: My kali-linux config note
date: 2015-03-01 19:30:46
tags: linux kali 
---


## 0x00 Before installing OS

在其他linux电脑上使用`dd`命令刻u盘，不是说其他的不行。反正我用过那么多刻录U盘系统的工具，只有dd最好用，命令简单：
	dd if=/path/to/iso of=/dev/sdX #sdX为设备名，你的u盘可能是sdc/sdd等

## 0x01 System config after installing

**改软件源**

官方源外加一个中科大的源，够用了

	deb http://http.kali.org/kali kali main non-free contrib
	deb-src http://http.kali.org/kali kali main non-free contrib
	deb http://security.kali.org/kali-security kali/updates main contrib non-free
	deb http://mirrors.ustc.edu.cn/kali kali main non-free contrib
	deb-src http://mirrors.ustc.edu.cn/kali kali main non-free contrib
	deb http://mirrors.ustc.edu.cn/kali-security kali/updates main contrib non-free

把以上添加到/etc/apt/sources.list文件中，然后'apt-get update;apt-get dist-upgrade'把系统更新到最新

**添加普通用户**

毕竟一直用root操作也不太好，
右上角-->system settings-->user account 
添加用户设为管理员（sudo权限），并且设好密码。
重启使用新用户登录系统。

**修改桌面为gnome3标准模式**

Kali Linux的桌面环境为Gnome 3，但默认运行在fallback模式。想临时切换成gnome3的标准模式请在终端输入：

	gnome-shell –replace

gnome 3的标准模式支持一些桌面特效开启还有很多gnome-shell插件，如果您觉得比较好用请输入下面的命令使系统在启动时，自动进入gnome-shell的标准模式。

	gsettings set org.gnome.desktop.session session-name gnome

若想还原默认的桌面请输入：

	gsettings set org.gnome.desktop.session session-name gnome-fallback

注销或者重启之后进入桌面即可直接进入您要切换的模式。


**修改grup分辨率**

vim /etc/default/grub

取消"#GRUB_GFXMODE=640×480" 这一行前面的注释符,并将后面的数字修改为一个合适的值，不需要太高，比如1024×768。这个值同时会影响grub启动菜单和控制台里文字的分辨率。
vim   /etc/grub.d/00_header
在"set gfxmode=${GRUB_GFXMODE}"这一行下添加新的一行:
set gfxpayload=keep

然后update-grub




