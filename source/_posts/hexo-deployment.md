---
title: hexo 部署
comments: false
toc: true
date: 2020-08-12 17:33:20
categories:
  - 前端
tags:
  - hexo
---

##### 部署

``` bash
$ npm install -g hexo-cli #安装
$ hexo init               #初始化
$ hexo g                  #生成
$ hexo s                  #启动本地服务
$ hexo d                  #发布
```
#### 常用命令

``` bash
$ hexo new "postName" #新建文章
$ hexo new page "pageName" #新建页面
$ hexo generate #生成静态页面至public目录
$ hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
$ hexo deploy #将.deploy目录部署到GitHub
$ hexo help  # 查看帮助
$ hexo version  #查看Hexo的版本
$ hexo clean && hexo g && hexo d #一键发布(liux)
$ hexo clean ; hexo g ; hexo d #一键发布(windows)
```