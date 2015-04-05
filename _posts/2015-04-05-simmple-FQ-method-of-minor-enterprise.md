---
layout: post
title: simmple-FQ-method-of-minor-enterprise
date: 2015-04-05 20:45:33
tags: openwrt FQ 
---


### 0x001 vps 搭建ss-server服务端

vps是必须的，选用shadowsocks是因为混淆协议，不像现有的vpn或者ssh协议特征明显，很容易被发现和监控。
不多扯，我一般使用shadowsocks-libve自己编译,感觉c写的在linux上应该性能会更好。只是感觉上，具体没测试过，不清楚。

我的服务端配置如下`cat /etc/shadowsocks/config.json`

	{
		"server"        :       "x.y.z.w",
		"server_port"	:	8908,
		"password"	:	"fuckFBX",
		"timeout"	:	300,
		"method"	:	"aes-256-cfb",
		"fast_open"	:	false
	}

**配置简单说明**，fast\_open只有在linux内核3.7以上才可以使用，
打开前需要`echo 3 > /proc/sys/net/ipv4/tcp_fastopen` && `echo 'net.ipv4.tcp_fastopen = 3' >>/etc/sysctl.conf`。
然后在以上ss的json配置文件中把fast\_open一项设为true，方可开启。
ss-server使用-c参数指定配置文件，若要打开UDP，则使用-u参数。这样可以方便DNS或者其他的查询等连接。

有些厂商的vps，防火墙策略需要在厂商的主机管理面板上配置。
开放你所需的的TCP/UDP端口，如果仅供企业内用的话也可以只开放给企业的公网IP，其他来源一律不允许。
如果防火墙策略不在厂商的面板上配置的话记得改ssh等一些常用端口并做些简单的安全策略。

### 0x002 nginx反代google

鉴于开始时只是想google可以畅通，就考虑内网用个虚拟机nginx反代一下谷歌了事。
因为对于内网来说希望这层FQ是透明的，所以最好不要让每个人都需配个代理或者说PAC文件。
先是试了nginx的http代理，如果只代80端口那就只能单纯的用来搜索，代了443的话又有证书问题。
然后发现nginx有个TCP的代理模块，首先安装依赖

	yum -y install git pcre-devel  zlib-devel     

nginx默认不支持tcp的负载均衡，需要打补丁！下载tcp补丁模块：

	git clone https://github.com/yaoweibin/nginx_tcp_proxy_module     

下载nginx，解压并安装nginx

	tar xzvf nginx.tar.gz     
	cd nginx     
	patch -p1 < ../nginx_tcp_proxy_module/tcp.patch    #打补丁     
	./configure --prefix=/opt/nginx  --add-module=../nginx_tcp_proxy_module     
	make && make install && echo OK     

安装完成，tcp负载均衡的配置示例如下/etc/nginx/nginx.conf：

	tcp {     
		upstream cluster80 {     
			server www.google.com:80;     
	
			check interval=3000 rise=2 fall=5 timeout=3000;     
			#默认使用轮询算法，如果要固定ip地址，使用ip hash的负载均衡算法就可以,把下面这行取消注释     
			# ip_hash;     
			#check interval=3000 rise=2 fall=5 timeout=1000 type=ssl_hello;     
			
			#check interval=3000 rise=2 fall=5 timeout=1000 type=http;     
			#check_http_send "GET / HTTP/1.0\r\n\r\n";     
			#check_http_expect_alive http_2xx http_3xx;     
		}     
		upstream cluster443 {
			server www.google.com:443;
			check interval=3000 rise=2 fall=5 timeout=3000;     
		}
	
		server {     
			listen 80;     
			proxy_pass cluster80;     
		}     
		server {     
			listen 443;     
			proxy_pass cluster443;     
		}     
	}     
	
当然以上只是nginx的部分，然后在这台内网机子上也编译好shadowsocks，利用ss-redir程序配合iptables做转发，ss-redir指定好本机端口，比如1080,其他配置和server端一样。

iptables配置如下`cat ss_iptables.sh`：

	# Create new chain
	iptables -t nat -N SHADOWSOCKS
	
	# Ignore your shadowsocks server's addresses,replace 'x.y.z.w' with your own vps ip
	# It's very IMPORTANT, just be careful.
	iptables -t nat -A SHADOWSOCKS -d x.y.z.w  -j RETURN
	
	# Ignore LANs and any other addresses you'd like to bypass the proxy
	# See Wikipedia and RFC5735 for full list of reserved networks.
	# See ashi009/bestroutetb for a highly optimized CHN route list.
	iptables -t nat -A SHADOWSOCKS -d 0.0.0.0/8 -j RETURN
	iptables -t nat -A SHADOWSOCKS -d 10.0.0.0/8 -j RETURN
	iptables -t nat -A SHADOWSOCKS -d 127.0.0.0/8 -j RETURN
	iptables -t nat -A SHADOWSOCKS -d 169.254.0.0/16 -j RETURN
	iptables -t nat -A SHADOWSOCKS -d 172.16.0.0/12 -j RETURN
	iptables -t nat -A SHADOWSOCKS -d 192.168.0.0/16 -j RETURN
	iptables -t nat -A SHADOWSOCKS -d 224.0.0.0/4 -j RETURN
	iptables -t nat -A SHADOWSOCKS -d 240.0.0.0/4 -j RETURN
	
	# Anything else should be redirected to shadowsocks's local port
	iptables -t nat -A SHADOWSOCKS -p tcp -j REDIRECT --to-ports 1080
	
	# Apply the rules
	iptables -t nat -A OUTPUT -p tcp -j SHADOWSOCKS
	
	# Start the shadowsocks-redir

如此配置后开启ss-redir和iptables应该就可以直接在内网访问此虚拟机地址，即可反代到google。
然后配合内网的DNS，把www.google.com指到这台机子来，体验上就没有差别了。

但是这样的问题在于，一台机子，或者说一个内网ip只能反代一个网站，如果又其他的FQ需求，就需要另开机子。
因为nginx的tcp代理模块只能针对端口，不能在同个端口上监听不同的servername，不是http协议。
想过用docker封装实现，但是这样也太浪费内网的ip。




4.kvm上安装openwrt系统，kvm相关配置以及网卡配置（。。。）
5.openwrt网络配置，wan口和lan口
6.openwrt安装shadowsocks，配置文件，iptables设置，设置静态路由，把vps的地址指定到wan口出去
7.找到google或者其他相关网站的相关ip段，在出口主路由上做设置，把下一跳地址指向openwrt的内网ip1.178，
8.因为内网的dns上层还是去114查询的，所以还是会有dns污染的问题，在dns上开个ss-tunnel，打开7913udp端口，用过shadowsocks把7913端口的流量映射到8.8.8.8的udp53
9.在named.conf里面把google、android等的zone配置forwarder到127的7913端口查询
10.基本搞定。
