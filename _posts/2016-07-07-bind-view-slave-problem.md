---
layout: post
title: "Bind9-slave配置view的问题"
date: 2016-07-07
tags: [ linux, bind , dns ]
---

困扰很久，排查结果
===


### master in 192.168.1.0/24

```shell
acl office    {192.168.1.0/24;};
acl idc1    {192.168.2.0/24;};
acl idc2    {192.168.3.0/24;};
view office-zone{
        match-clients {office;};
            zone xxx {};

};
view idc1-zone{
        match-clients {idc1;};
            zone xxx {};

};
view idc2-zone{
        match-clients {idc2;};
            zone xxx {};

};
```

### slave1 in 192.168.2.0/24;

```shell
acl office    {192.168.1.0/24;};
acl idc1    {192.168.2.0/24;};
acl idc2    {192.168.3.0/24;};
view office-zone{
        match-clients {office;};
            zone xxx {};

}; 
view idc1-zone{
        match-clients {idc1;};
            zone xxx {};

}; 
view idc2-zone{
        match-clients {idc2;};
            zone xxx {};

};
```

### slave2 in 192.168.3.0/24;

```shell
acl office    {192.168.1.0/24;};
acl idc1    {192.168.2.0/24;};
acl idc2    {192.168.3.0/24;};
view office-zone{
        match-clients {office;};
            zone xxx {};

}; 
view idc1-zone{
        match-clients {idc1;};
            zone xxx {};

}; 
view idc2-zone{
        match-clients {idc2;};
            zone xxx {};

};
```


每次master reload的时候， slave收到notify，应该会去transfer整个zone，但是发现只transfer了master机器所做view的zone-file。也就是说，每次更新master并reload之后，除了和master一个view内的机器，能够查询到更新后的域名指向，其他view内的机器不管查询哪个slave都不能查询到。

因为slave自己也做了acl+view的配置，所以slave机器在向master同步域文件的时候，先在本地匹配了master机器所在的view，然后向master同步了这个view中的域文件。反正我是这么理解，不知道是不是还有其他配置可以调整这种情况。

本来slave可以不管view而只做同步，但是因为idc1和idc2中的服务器resolv.conf是混写的。所以怕会有交叉请求，才在slave里面做了view。
旧的：
idc1:resolv.conf
```shell
nameserver 192.168.2.1
nameserver 192.168.3.1
```

idc2:resolv.conf
```shell
nameserver 192.168.3.1
nameserver 192.168.2.1
```
为了解决这个问题，考虑每个idc的机器中nameserver不混写，把master写在第二行作为备用。
新的：
idc1:resolv.conf
```shell
nameserver 192.168.2.1
nameserver 192.168.1.1
```

idc2:resolv.conf
```shell
nameserver 192.168.3.1
nameserver 192.168.1.1
```

