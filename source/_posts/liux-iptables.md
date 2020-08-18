---
title: liux iptables的使用
comments: false
toc: true
date: 2020-08-01 20:31:41
categories:
  - liux
tags:
  - liux
  - iptables
---

文件位置: `/etc/sysconfig/iptables`

**命令**

```shell
systemctl stop iptables    #停止
systemctl start iptables   #启动
systemctl restart iptables #重启
systemctl reload iptables  #重新加载
service iptables status    #查看状态
service iptables save      #保存
```

 **清除已有iptables规则**

```shell
iptables -F
iptables -X
iptables -Z
```

**开放指定的端口**

```shell
iptables -A INPUT -s 127.0.0.1 -d 127.0.0.1 -j ACCEPT               #允许本地回环接口(即运行本机访问本机)
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT    #允许已建立的或相关连的通行
iptables -A OUTPUT -j ACCEPT         #允许所有本机向外的访问
iptables -A INPUT -p tcp --dport 22 -j ACCEPT    #允许访问22端口
iptables -A INPUT -j reject       #禁止其他未允许的规则访问
iptables -A FORWARD -j REJECT     #禁止其他未允许的规则访问
```

```shell
*filter
:INPUT DROP [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT # 允许本地环回接口在INPUT表的所有数据通信，-i 参数是指定接口，接口是lo，lo就是Loopback（本地环回接口）
-A INPUT -s 172.19.1.8/32 -p tcp -m tcp -m state --state NEW -m multiport --dports 22,2288 -m comment --comment "VPN_SSH_PORT" -j ACCEPT  #对IP:172.19.1.8 开放22,2288端口
-A INPUT -s 172.19.0.0/16   -p tcp -m tcp -m state --state NEW -m multiport --dports 22,2288 -m comment --comment "LOCAL_SSH_PORT" -j ACCEPT  #对IP段即从172.19.0.1到172.19.255.254 开放
-A INPUT -p tcp -m tcp -m state --state NEW -m multiport --dports 80,443 -m comment --comment "WEB_PORT" -j ACCEPT  #对外开放80,443端口
#这两条的意思是在INPUT表和FORWARD表中拒绝所有其他不符合上述任何一条规则的数据包。并且发送一条host prohibited的消息给被拒绝的主机。
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
```

**屏蔽IP**

```shell
iptables -I INPUT -s 123.45.6.7 -j DROP       #屏蔽单个IP的命令
iptables -I INPUT -s 123.0.0.0/8 -j DROP      #封整个段即从123.0.0.1到123.255.255.254的命令
iptables -I INPUT -s 124.45.0.0/16 -j DROP    #封IP段即从123.45.0.1到123.45.255.254的命令
iptables -I INPUT -s 123.45.6.0/24 -j DROP    #封IP段即从123.45.6.1到123.45.6.254的命令是
```

子网掩码：
8：  匹配后三位最大值的
16：匹配后两位最大值的
24：匹配后一位最大值的
32：匹配整个IP