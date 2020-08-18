---
title: 配置SSH免密登录
comments: false
toc: true
date: 2020-08-01 20:31:41
categories:
  - liux
tags:
  - liux
---

`id_rsa.pub`为公钥，

`id_rsa`为私钥

**命令**

```shell
ssh-keygen -t rsa #生成密钥,连续按回车键三次
```

# 1 配置github ssh

将生成的`id_rsa.pub`内容拷贝添加至github ssh即可.

# 2 配置SSH免密登录

> 说明: 这里演示所用的服务器操作系统是Cent OS 7. 我们的目标是:
>
> **A服务器(172.16.22.131) 能免密登录 B服务器 (172.16.22.132).**
>
> 注意: ssh连接是单向的, A能免密登录B, 并不能同时实现B能免密登录A.

## 2.1 安装必需的软件

在操作之前, 先确保所需要的软件已经正常安装.

这里我们需要安装`ssh-keygen`和`ssh-copy-id`, 安装方式如下:

```bash
# 安装ssh-keygen, 需要确保服务器可以联网. 博主这里已经安装完成, 所以没有做任何事.
[root@localhost ~]# yum install -y ssh-keygen
Loaded plugins: fastestmirror, langpacks
base                                                         | 3.6 kB  00:00:00     
epel                                                         | 3.6 kB  00:00:00     
extras                                                       | 2.9 kB  00:00:00     
updates                                                      | 2.9 kB  00:00:00     
Loading mirror speeds from cached hostfile
No package ssh-keygen available.
Error: Nothing to do

# 安装ssh-copy-id
[root@localhost ~]# yum install -y ssh-copy-id 
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
No package ssh-copy-id available.
Error: Nothing to do
```

## 2.2 ssh-keygen创建公钥-私钥对

(1) 在指定目录下生成rsa密钥, 并指定注释为“shoufeng”, 实现示例:

```bash
[root@localhost ~]# ssh-keygen  -t rsa    -f ~/.ssh/id_rsa   -C "shoufeng"
#                               ~密钥类型  ~密钥文件路径及名称  ~ 备注信息
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):  # 输入密码, 若不输入则直接回车
Enter same passphrase again: # 再次确认密码, 若不输入则直接回车
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
9a:e3:94:b9:69:c8:e9:68:4b:dc:fa:43:25:7f:53:f1 shoufeng
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|          .      |
|           o     |
|    . .   . E    |
|     +  S.       |
| . .. .=o        |
|  oo.oB. .       |
| ..o=o.+         |
| .++oo+          |
+-----------------+
```

**注意: 密钥的文件名称必须是id_xxx, 这里的xxx就是-t参数指定的密钥类型.** 比如密钥类型是rsa, 那么密钥文件名就必须是id_rsa.

(2) `ssh-keygen`常用参数说明:

> -t: 密钥类型, 可以选择 dsa | ecdsa | ed25519 | rsa;
>
> -f: 密钥目录位置, 默认为当前用户home路径下的.ssh隐藏目录, 也就是`~/.ssh/`, 同时默认密钥文件名以`id_rsa`开头. 如果是root用户, 则在`/root/.ssh/id_rsa`, 若为其他用户, 则在`/home/username/.ssh/id_rsa`;
>
> -C: 指定此密钥的备注信息, 需要配置多个免密登录时, 建议携带;
>
> -N: 指定此密钥对的密码, 如果指定此参数, 则命令执行过程中就不会出现交互确认密码的信息了.

举例说明: 同时指定目录位置、密码、注释信息, 就不需要输入回车键即可完成创建:

```
ssh-keygen -t rsa -f ~/.ssh/id_rsa -N shoufeng -C shoufeng
```

(3) 前往`~/.ssh/`目录下查看生成的文件:

```bash
# 生成的文件以test_rsa开头, test_rsa是私钥, test_rsa.pub是公钥:
[root@localhost .ssh]# ls
test_rsa  test_rsa.pub

# 通过cat命令查看公钥文件: 
[root@localhost .ssh]# cat id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC2JpLMqgeg9jB9ZztOCw0WMS8hdVpFxthqG1vOQTOji/cp0+8RUZl3P6NtzqfHbs0iTcY0ypIJGgx4eXyipfLvilV2bSxRINCVV73VnydVYl5gLHsrgOx+372Wovlanq7Mxq06qAONjuRD0c64xqdJFKb1OvS/nyKaOr9D8yq/FxfwKqK7TzJM0cVBAG7+YR8lc9tJTCypmNXNngiSlipzjBcnfT+5VtcFSENfuJd60dmZDzrQTxGFSS2J34CuczTQSsItmYF3DyhqmrXL+cJ2vjZWVZRU6IY7BpqJFWwfYY9m8KaL0PZ+JJuaU7ESVBXf6HJcQhYPp2bTUyff+vdV shoufeng
# 可以看到最后有一个注释内容shoufeng
```

## 2.3 ssh-copy-id把A的公钥发送给B

默认用法是: `ssh-copy-id root@172.16.22.132`, ssh-copy-id命令连接远程服务器时的默认端口是22, 当然可以指定文件、远程主机的IP、用户和端口:

```bash
# 指定要拷贝的本地文件、远程主机的IP+用户名+端口号:
[root@localhost .ssh]# ssh-copy-id -i ~/.ssh/id_rsa.pub -p 22 root@172.16.22.132
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@172.16.22.132's password:  # 输入密码后, 将拷贝公钥

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh -p '22' 'root@172.16.22.132'"
and check to make sure that only the key(s) you wanted were added.
```

## 2.4 在A服务器上免密登录B服务器

```bash
[root@localhost .ssh]# ssh root@172.16.22.132
Last login: Fri Jun 14 08:46:04 2019 from 192.168.34.16    # 登录成功
```

# 3 扩展说明

## 3.1 其他方式发送公钥文件

上述2.3步骤是通过`ssh-copy-id`工具发送公钥文件的, 当然我们也可以通过其他方式实现:

(1) 将A的公钥文件发给B:

通过scp命令将A服务器的 **公钥文件** 发送到B服务器的用户目录下, 因为还没有配置成功免密登录, 所以期间需要输入B服务器对应用户的密码:

```bash
[root@localhost .ssh]# scp id_rsa.pub root@172.16.22.132:/root/.ssh 
root@172.16.22.132's password: 
id_rsa.pub                                           100%  390     0.4KB/s   00:00 
```

(2) 在B上创建authorized_keys文件:

```bash
[root@localhost .ssh]# cd /root/.ssh/
[root@localhost .ssh]# ls
id_rsa.pub
# 通过A服务器的公钥生成"authorized_keys"文件:
[root@localhost .ssh]# cat id_rsa.pub >> authorized_keys
[root@localhost .ssh]# cat authorized_keys 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC2JpLMqgeg9jB9ZztOCw0WMS8hdVpFxthqG1vOQTOji/cp0+8RUZl3P6NtzqfHbs0iTcY0ypIJGgx4eXyipfLvilV2bSxRINCVV73VnydVYl5gLHsrgOx+372Wovlanq7Mxq06qAONjuRD0c64xqdJFKb1OvS/nyKaOr9D8yq/FxfwKqK7TzJM0cVBAG7+YR8lc9tJTCypmNXNngiSlipzjBcnfT+5VtcFSENfuJd60dmZDzrQTxGFSS2J34CuczTQSsItmYF3DyhqmrXL+cJ2vjZWVZRU6IY7BpqJFWwfYY9m8KaL0PZ+JJuaU7ESVBXf6HJcQhYPp2bTUyff+vdV shoufeng
```

注意: 上述重定向时使用`>>`进行追加, 不要用`>`, 那会清空原有内容.

## 3.2 文件权限

为了让私钥文件和公钥文件能够在认证中起作用, 需要确保权限的正确性:

> ① 对于`.ssh`目录以及其内部的公钥、私钥文件, 当前用户至少要有执行权限, 其他用户最多只能有执行权限.
>
> ② 不要图省事设置成777权限: 太大的权限不安全, 而且数字签名也不支持这种权限策略.
>
> ③ 对普通用户, 建议设置成600权限: `chmod 600 authorized_keys id_rsa id_rsa.pub`;
>
> ④ 对root用户, 建议设置成644权限: `chmod 644 authorized_keys id_rsa id_rsa.pub`.

## 3.3 文件的编辑和查看

在Liunx环境下, 如果要查看、复制私钥、公钥, 以及authorized_keys等文件, 不要使用vim等编辑器打开, 因为它会产生不必要的回车;

应该通过cat、more、less等查看命令把内容打印到终端上, 再作查看、复制等操作.

# 4 常见问题及解决

(1) 问题描述:

通过`ssh-copy-id`命令拷贝公钥文件之后, 尝试免密登录另一台服务器时发生错误:

```bash
[root@localhost ~]# ssh root@172.16.22.131
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for '/root/.ssh/id_rsa' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
bad permissions: ignore key: /root/.ssh/id_rsa
root@172.16.22.131's password: 
```

(2) 问题原因:

提示信息说明密钥文件不受保护, 具体是说这里的密钥文件权限是0644, 而数字签名机制要求密钥文件不能被其他用户访问(读取), 所以该密钥文件被强制忽略处理了.

(3) 问题解决:

只要修改该密钥文件的权限即可:

```bash
chmod 600 /root/.ssh/id_rsa
```

这里`/root/.ssh/id_rsa`就是warning里给出的密钥文件名.