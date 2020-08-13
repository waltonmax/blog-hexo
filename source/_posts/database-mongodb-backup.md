---
title: mongodb备份与恢复
comments: false
toc: true
date: 2020-08-12 20:31:41
categories:
  - 数据库
tags:
  - mongodb
---

### mongodump备份/恢复

```shell
#进入容器
docker exec -itd mongodb bash

#备份全部数据库  单个数据库 -d database-name
mongodump -h 127.0.0.1:27017 -u mongoadmin -p pwd --authenticationDatabase "admin" -o /data/db/bak

#压缩为.tar.gz格式
tar -zcvf mongodb.tar.gz bak/

#复制
\cp -r /data/mongodb/mongo-data/bak /data/mongodb/mongo-data/bak

#恢复全部数据库
docker exec -it mongodb bash
mongorestore -u mongoadmin -p pwd --drop --authenticationDatabase "admin" --dir /data/db/bak

```

`mongorestore`恢复数据默认是追加，如打算先删除后导入，可以加上–drop参数，不过添加–drop参数后，会将数据库数据清空后再导入，如果数据库备份后又新加入了数据，也会将新加的数据删除，它不像mysql有一个存在的判断。

```shell
#备份单个表
mongodump -u  user_name -p password  --port 27010 --authenticationDatabase admin -d db_name -c table_name -o /backup/table_name.bak

#备份单个库
mongodump -u  user_name -p password  --port 27010 --authenticationDatabase admin -d db_name  -o  /backup/db_name/

#备份所有库
mongodump -u  user_name -p password  --port 27010 --authenticationDatabase admin -o /backup/all_db/

# 备份所有库推荐使用添加--oplog参数的命令，这样的备份是基于某一时间点的快照，只能用于备份全部库时才可用，单库和单表不适用：
mongodump -u  user_name -p password  --port 27010 --authenticationDatabase admin  --oplog -o  /backup/all_db/ 

# 同时，恢复时也要加上--oplogReplay参数，具体命令如下(下面是恢复单库的命令)：
mongorestore  -u  user_name -p password  --port 27010 --authenticationDatabase admin --oplogReplay  /backup/all_db/

#mongodump在mongo关闭时，也是可以备份的，不过需要指定数据目录，命令为：
mongodump  --dbpath  /data/db

#恢复单个库
mongorestore -u  user_name -p password  --port 27010 --authenticationDatabase admin db_name   /backup/db_name/

#恢复所有库
mongorestore   -u  user_name -p password  --port 27010 --authenticationDatabase admin  /backup/all_db/

#恢复单表
mongorestore  -u  user_name -p password  --port 27010 --authenticationDatabase admin -d db_name -c table_name /backup/table_name/table_name/table_name.bson
```





## mongoexport备份/还原

```shell
#导出文件格式支持csv和json，不同的是csv格式必须显示的指定要导出的字段,而json格式则不需要
mongoexport --port 21017 -u user -p'password' -d db_name -c table_name  -f col1,col2,col3 --type=csv -o ./table_name.csv --authenticationDatabase db_name

# 导出所有字段数据
mongoexport --port 21017 -u user -p'password' -d db_name -c table_name -o ./table_name.json --type=json --authenticationDatabase db_name

#`mongoimport`还原
#命令用来导入数据，语法和mongoexport差不多将table_name.csv里的数据导入到db_name数据库中的table_name数据表中,如果table_name数据表不存在则自动创建
# headerline 仅适用于导入csv,tsv格式的数据，表示文件中的第一行作为数据头
# upsert 以新增或者更新的方式来导入数据
mongoimport --port 21017 -u user -p'password' -d db_name -c table_name --type=csv --file ./table_name.csv --headerline --upsert

#备份脚本
#!/bin/bash
mongo_bin=/usr/local/mongodb/bin
back_path=/root/mongodbbackup
file_date=`date +"%Y-%m-%d"`
$mongo_bin}/mongodump -h 127.0.0.1:27010 -o ${back_path}/${file_date}
tar zcvPf ${back_path}/${file_date}.tar.gz ${back_path}/${file_date} --remove-files
find /root/mongodbbackup/* -type f -mtime +10 -exec rm -f {} \;
```



##### 部分参数说明

- --drop参数：恢复数据之前删除原来的数据，避免数据重复
- --noIndexRestore参数：恢复数据时不创建索引
- --dir参数：数据库备份目录
- -d参数：后面跟要恢复的数据库名称



##### 自动备份脚本

```shell
#!/bin/bash
#backup MongoDB

#mongodump命令路径
DUMP=/usr/local/mongodb/bin/mongodump
#临时备份目录
OUT_DIR=/data/mongodb_bak/mongodb_bak_now
#备份存放路径
TAR_DIR=/data/mongodb_bak/mongodb_bak_list
#获取当前系统时间
DATE=`date +%Y_%m_%d`
#数据库账号
DB_USER=user
#数据库密码
DB_PASS=123
#DAYS=15代表删除15天前的备份，即只保留近15天的备份
DAYS=15
#最终保存的数据库备份文件
TAR_BAK="mongodb_bak_$DATE.tar.gz"

cd $OUT_DIR
rm -rf $OUT_DIR/*
mkdir -p $OUT_DIR/$DATE
#备份全部数据库
$DUMP -h 15.62.32.112:27017 -u $DB_USER -p $DB_PASS --authenticationDatabase "admin" -o $OUT_DIR/$DATE
#压缩为.tar.gz格式
tar -zcvf $TAR_DIR/$TAR_BAK $OUT_DIR/$DATE
#删除15天前的备份文件
find $TAR_DIR/ -mtime +$DAYS -delete
exit
```


