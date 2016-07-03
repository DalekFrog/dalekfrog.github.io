---
layout: post
title: inotify配合rsync 实时同步
date: 2015-02-14 00:32:30
tags: linux ops
---



### 0x00 先编译安装inotify

假设需要从A-server同步文件到B-server，则需要在A上安装好inotify

	wget http://nchc.dl.sourceforge.net/project/inotify-tools/inotify-tools/3.13/inotify-tools-3.13.tar.gz
	tar xzvf inotify-tools-3.14.tar.gz
	cd inotify-tools-3.13
	./configure  --prefix=/data/program/inotify
	make && make install && echo ok



### 0x01 安装配置rsync

yum install -y rsync

rsync的传输可以走ssh的通道也可以走rsync自己的方式。如果用rsync自带的验证，则必须在服务端和客户端都安装rsync。

使用ssh验证时，只需要客户端即A-server上安装rsync并且不用做额外的配置，B-server可以不用安装rsync服务。但需要将A-server的ssh公钥导入到B-server当中。

使用rsync自带的验证方式时，需要在服务端，即B-server上做如下配置：
	
	mkdir /etc/rsyncd
	cat > /etc/rsyncd/rsyncd.conf << EOF
	#Rsync server
	##rsyncd.conf start##
	port = 873
	address = 192.168.X.Y   #本地服务端的ip
	uid = root
	gid = root
	use chroot = no
	max connections = 2000
	timeout = 600
	pid file = /var/run/rsyncd.pid
	lock file = /var/run/rsync.lock
	log file = /var/log/rsyncd.log
	ignore errors
	read only = false
	list = false
	hosts allow = 192.168.0.0/16
	hosts deny = *
	auth users = rsync_user
	secrets file = /etc/rsyncd/rsyncd.secrets
	#####################################
	[data1]
	comment = data1  files 
	path = /path/to/the/data1_file
	secrets file=/etc/rsyncd/rsyncd.secrets
	read only = false
	####################################
	[data2]
	comment = data2 site  files
	path = /path/to/the/data2_file
	secrets file=/etc/rsyncd/rsyncd.secrets
	read only = false
	#####################################
	EOF

其中data1和data2是针对要同步过来的文件的配置模块，客户端在使用rsync时要指定哪一个模块，data1或者data2或者其他自定义。

然后建立密码文件并以后台模式启动rsync服务端

	echo 'rsync_user:YourOwnPasswd' > /etc/rsyncd/rsyncd.secrets
	chmod 600 /etc/rsyncd/rsyncd.secrets
	rsync --daemon --config=/etc/rsyncd/rsyncd.conf

在客户端，即A-server上需要配置好rsync在连接时使用的密码

	mkdir /etc/rsync
	echo "YourOwnPasswd">/etc/rsync/rsync.secrets
	chmod 600 /etc/rsync/rsync.secrets




### 0x02 编写脚本让inotify监控文件夹然后使用rsync同步

#### 在需要被同步文件的客户端，即A-server运行的脚本

##### 使用rsync自带的验证时
	
	#!/bin/bash
	log=/var/log/inotify-rsync.log
	src="/data_path/need_to_be/sync" #需要被同步的文件路径
	rhost="192.168.X.Y" #B-server 即远程服务端的地址
	rpath="data1" #服务端定义的模块，data1或者data2或者其他自定义
	user=rsync_user
	password='--password-file=/etc/rsync/rsync.secrets'
	/data/program/inotify/bin/inotifywait -mr --timefmt '%d/%m/%y %H:%M' --format '%T %w %f' -e modify,delete,create,attrib $src |  while read files; do
	FILECHANGE=${DIR}${FILE}
	/usr/bin/rsync -rpgovzt --delete --progress $password $src $user@$rhost::$rpath  
	echo " ${files}  was backed up via rsync" >> $log
	done
	

##### 使用ssh通道时
	
	#!/bin/bash
	log=/var/log/inotify-rsync.log
	src="/data_path/need_to_be/sync" #需要被同步的文件路径
	rhost="192.168.X.Y" #B-server 即远程服务端的地址
	rpath="/data/remote/backup/dir"   #远程主机上的存放此备份文件的目录
	/data/program/inotify/bin/inotifywait -mr --timefmt '%d/%m/%y %H:%M' --format '%T %w %f' -e modify,delete,create,attrib $src |  while read files; do
	/usr/bin/rsync -vzrtopg --delete --progress $src root@$rhost:$rpath  #请先将root（或其他自定义用户）的公钥copy到远端服务器
	echo " ${files}  was backed up via rsync" >> $log
	done


**以上两个脚本实测在rsync这一步，如果行尾加入'&'符号，可能导致rsync不断占用系统资源，拖垮整个系统。原因应该是在while循环当中没有被执行完成就被丢到后台，导致inotifywait程序发现块变更后就不断去调用rsync**


