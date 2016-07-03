---
layout: post
title: My kali-linux config note
date: 2015-03-01 19:30:46
tags: linux kali 
---

过期了，懒得改，现在都 kali 2016.1 roling 版本了。
===

## 0x00 Before installing OS

在其他linux电脑上使用`dd`命令刻u盘，不是说其他的不行。反正我用过那么多刻录U盘系统的工具，只有dd最好用，命令简单：
	dd if=/path/to/iso of=/dev/sdX #sdX为设备名，你的u盘可能是sdc/sdd等

## 0x01 System config after installing

### 改软件源

官方源外加一个中科大的源，够用了

	deb http://http.kali.org/kali kali main non-free contrib
	deb-src http://http.kali.org/kali kali main non-free contrib
	deb http://security.kali.org/kali-security kali/updates main contrib non-free
	deb http://mirrors.ustc.edu.cn/kali kali main non-free contrib
	deb-src http://mirrors.ustc.edu.cn/kali kali main non-free contrib
	deb http://mirrors.ustc.edu.cn/kali-security kali/updates main contrib non-free

把以上添加到/etc/apt/sources.list文件中，然后'apt-get update;apt-get dist-upgrade'把系统更新到最新

### 添加普通用户

毕竟一直用root操作也不太好，
右上角-->system settings-->user account 
添加用户设为管理员（sudo权限），并且设好密码。
重启使用新用户登录系统。

### 修改桌面为gnome3标准模式

Kali Linux的桌面环境为Gnome 3，但默认运行在fallback模式。想临时切换成gnome3的标准模式请在终端输入：

	gnome-shell –replace

gnome 3的标准模式支持一些桌面特效开启还有很多gnome-shell插件，如果您觉得比较好用请输入下面的命令使系统在启动时，自动进入gnome-shell的标准模式。

	gsettings set org.gnome.desktop.session session-name gnome

若想还原默认的桌面请输入：

	gsettings set org.gnome.desktop.session session-name gnome-fallback

注销或者重启之后进入桌面即可直接进入您要切换的模式。


### 修改grup分辨率

vim /etc/default/grub

取消"#GRUB\_GFXMODE=640×480" 这一行前面的注释符,并将后面的数字修改为一个合适的值，不需要太高，比如1024×768。这个值同时会影响grub启动菜单和控制台里文字的分辨率。

vim   /etc/grub.d/00\_header

在"set gfxmode=${GRUB\_GFXMODE}"这一行下添加新的一行:
set gfxpayload=keep

然后update-grub


## 0x02 Personal config 

其他软件的安装

输入法

	apt-get install fcitx fcitx-pinyin fcitx-module-cloudpinyin fcitx-googlepinyin #安装小企鹅输入法
	im-config #配置输入法

字体

	apt-get install ttf-wqy-zenhei  ttf-wqy-microhei xfonts-wqy gnome-tweak-tool

打开gnome-tweak-tool,在字体设置里面把Antialiasing（反锯齿）调整为Rgba,Hinting(字体微调)调整为Slight，这样看起来感觉稍微好些

安装monaco字体脚本如下，sudo执行

	#!/bin/bash
	echo "Start install"
	sudo mkdir -p /usr/share/fonts/truetype/custom
	echo "Downloading font"
	wget -c https://github.com/cstrap/monaco-font/raw/master/Monaco_Linux.ttf
	echo "Installing font"
	sudo mv Monaco_Linux.ttf /usr/share/fonts/truetype/custom/
	echo "Updating font cache"
	sudo fc-cache -f -v
	echo "Enjoy"

VPN完整安装

	apt-get install network-manager-openvpn-gnome network-manager-pptp network-manager-pptp-gnome network-manager-strongswan network-manager-vpnc network-manager-vpnc-gnome


如果开的是gnome的fallback模式，终端的透明没有开启混成特效，打开`gnome-session-properties`(这也是添加开机自启动的地方)。
在里面添加一个项目，命令为metacity --composite 注销重新进入即可。


其他软件

	apt-get install file-roller htop keepassx

chrome 浏览器

虽然可以用自带的源里面的chromium，但是chrome相比于它不用额外整flash的插件，所以去官网下载相应的deb安装包，使用dpkg安装可能会出现如下提示：
	
	fang@windows:~$ sudo dpkg -i google-chrome-stable_current_amd64.deb 
	[sudo] password for fang: 
	Selecting previously unselected package google-chrome-stable.
	(Reading database ... 322780 files and directories currently installed.)
	Unpacking google-chrome-stable (from google-chrome-stable_current_amd64.deb) ...
	dpkg: dependency problems prevent configuration of google-chrome-stable:
	 google-chrome-stable depends on libappindicator1; however:
	   Package libappindicator1 is not installed.
	
	dpkg: error processing google-chrome-stable (--install):
	dependency problems - leaving unconfigured
	Processing triggers for desktop-file-utils ...
	Processing triggers for gnome-menus ...
	Processing triggers for man-db ...
	Processing triggers for menu ...
	Errors were encountered while processing:
	 google-chrome-stable

而使用apt-get安装这个依赖的时候又会有如下提示：

	fang@windows:~$ sudo apt-get install libappindicator1
	Reading package lists... Done
	Building dependency tree       
	Reading state information... Done
	You might want to run 'apt-get -f install' to correct these:
	The following packages have unmet dependencies:
	 libappindicator1 : Depends: libdbusmenu-glib4 (>= 0.4.2) but it is not going to be installed
	                    Depends: libdbusmenu-gtk4 (>= 0.4.2) but it is not going to be installed
			    Depends: libindicator7 (>= 0.4.90) but it is not going to be installed
			    Recommends: indicator-application (>= 0.2.93) but it is not going to be installed
	E: Unmet dependencies. Try 'apt-get -f install' with no packages (or specify a solution).

我替换apt-get为aptitude安装就没有问题，不知道为啥在这个地方apt-get对依赖的包不自动关联上去，求解？

root用户的桌面无法打开chrome或者chromium的解决方法：

vim /etc/chromium/default

在Flag的引号里面追加"--user-data-dir"
