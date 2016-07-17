---
layout: post
title: "Apply Free StartSSL certificate"
date: 2016-05-27
tags: [ssl, cert , web, linux]
---

证书需求越来越多了，免费的其实也挺好用的，就是每个域名要申请会麻烦一点。
===

## 注册

注册用邮箱，填写验证码后 会生成一个证书文件让你导入系统，这样每次登陆他网站直接读取你系统的证书即可。

2、免费证书只有一年并且不能通配，申请www.example.com时，他会去拉example.com域名的whois信息。

验证域名是否是你的有好几种方法。一是给你个文件放到你pan.caimi-inc.com一个目录下，他能访问到即可通过.
二是通过whois中域名注册人的邮箱收验证码或者example.com域名下的postmaster@example.com等邮箱收取验证码。
还有第三种没看，我用了第二种，直接给邮件系统配了个example.com的邮件服务，指了个MX记录收个邮件即可。

3、certificates wizard 可以继续申请example.com域名下其他证书。
生成证书命令
openssl req -newkey rsa:2048 -keyout yourname.key -out yourname.csr 
时必须输入pass phrase，可以在生成后使用如下命令去除
openssl rsa -in ssl.key -out ssl.key

或者直接命令
openssl req -nodes -newkey rsa:2048 -keyout  ${DOMAIN_NAME}.key -out  ${DOMAIN_NAME}.csr  -subj "/C=CN/ST=ZJ/L=HZ/O=WC/OU=wacaiops/CN=${DOMAIN_NAME}"

-nodes参数指定不用密码

关于p12(pfx) 文件，深信服sslvpn需要pfx证书，还一定要密码的。。。。。
在startssl的web页面可以操作，点击tool box 然后"Create PKCS#12 (PFX) File"，需要填入crt和key以及密码的文本。
我尝试了用这条命令"openssl req -newkey rsa:2048 -keyout yourname.key -out yourname.csr "生成的key和证书文本以及密码，不行。然后把上传到startssl网站后生成的crt文件(OtherServer.zip)，配合自己生成的key+密码
上传后可生成。
导入深信服设备可用。
网上说"openssl pkcs12 -export -inkey server.key -in server.crt -out server.pfx "命令可用
还未测试。

