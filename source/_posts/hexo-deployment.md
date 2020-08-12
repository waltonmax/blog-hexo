---
title: hexo 部署
comments: false
toc: true
date: 2020-08-12 17:33:20
tags:
---
### 安装 

``` bash
$ npm install -g hexo-cli
```


### 初始化

``` bash
$ hexo init
```

### 生成静态文件

``` bash
$ hexo g
```

### 启动本地服务

``` bash
$ hexo s
```
### 发布

``` bash
$ hexo d
```
### 常用命令

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