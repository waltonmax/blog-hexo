---
title: mysql重置密码
comments: false
toc: true
date: 2020-08-12 20:31:41
categories:
  - 数据库
tags:
  - mysql
---

```mysql
# 进入容器
docker exec -it mysql bash
# 设置跳过权限表的加载 
# 警告：这就意味着任何用户都能登录进来，并进行任何操作，相当不安全。
echo "skip-grant-tables" >> /etc/mysql/conf.d/docker.cnf
# 退出容器
exit
# 重启容器
docker restart mysql
# 再次进入容器
docker exec -it mysql bash
# 登录 mysql（无需密码）
mysql -uroot 
# 更新权限
flush privileges;
# 修改密码
alter user 'root'@'localhost' identified by '123456';
# 退出mysql
exit
# 替换掉刚才加的跳过权限表的加载参数
sed -i "s/skip-grant-tables/ /" /etc/mysql/conf.d/docker.cnf
# 退出容器
exit
# 重启容器
docker restart mysql
```

