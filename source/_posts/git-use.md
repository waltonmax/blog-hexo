---
title: git 使用
comments: false
toc: true
date: 2020-08-12 20:31:41
categories:
  - 开发工具使用
tags:
  - git
  - bash 
---

####  命令:

``` bash
git init                     #初始化本地git库
git add .                    #添加当前所有文件到git库
git commit -m "first commit" #提交到本地库
git remote add origin git@github.com:waltonmax/test.git #添加远程仓库地址  
git push -u origin master    #推送到远程仓库

git branch <name>            #创建分支
git checkout <name>          #切换分支
git checkout -b <name>       #创建+切换分支
git merge <name>             #合并某分支到当前分支
git branch -d <name>         #删除分支

curl -u '用户名' https://api.github.com/user/repos -d '{"name":"仓库名"}' #使用命令创建远程仓库
```

#### 常见问题解决

```
#版本控制无效
git rm -r --cached .
git add .
git commit -m 'update .gitignore'
```

