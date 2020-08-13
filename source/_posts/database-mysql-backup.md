---
title: mysql备份与恢复
comments: false
toc: true
date: 2020-08-12 20:31:41
categories:
  - 数据库
tags:
  - mysql
---

```mysql
#导出所有数据
mysqldump -uroot -proot --all-databases >/tmp/all.sql

#导出单个
mysqldump -uroot -proot --databases db1 db2 >/tmp/user.sql

#导出db1中的a1、a2表
mysqldump -uroot -proot --databases db1 --tables a1 a2  >/tmp/db1.sql

#条件导出，导出db1表a1中id=1的数据
mysqldump -uroot -proot --databases db1 --tables a1 --where='id=1'  >/tmp/a1.sql

#只导出表结构不导出数据，--no-data
mysqldump -uroot -proot --no-data --databases db1 >/tmp/db1.sql

#跨服务器导出导入数据
mysqldump --host=h1 -uroot -proot --databases db1 |mysql --host=h2 -uroot -proot db2

#将h1服务器中的db1数据库的所有数据导入到h2中的db2数据库中，db2的数据库必须存在否则会报错
mysqldump --host=192.168.80.137 -uroot -proot -C --databases test |mysql --host=192.168.80.133 -uroot -proot test 

#恢复导入数据库数据
mysql -uroot -p'123456' mytest < /mnt/mytest_bak.sql  

```