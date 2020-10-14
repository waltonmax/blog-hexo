---
title: Linux进程、日志筛选命令
comments: false
toc: true
date: 2020-08-25 20:31:41
categories:
  - liux
tags:
  - liux
  - bash 
---

# Linux进程、日志筛选命令


### Linux目录操作命令

> cd ..退出当前目录，返回上一级目录；cd / 退出当前目录，返回根目录；

> mkdir命令用于创建一个新的目录；rmdir命令功能删除指定的空目录。

### Linux筛选日志

#### tail用法

> tail -n 10 test.log 查询日志尾部最后10行的日志;

> tail -n +10 test.log 查询10行之后的所有日志;

> tail -n +92表示查询92行之后的日志

#### head用法

> head -n 10 test.log 查询日志文件中的头10行日志;

> head -n -10 test.log 查询日志文件除了最后10行的其他所有日志;

> head -n 20 则表示在前面的查询结果里再查前20条记录

#### cat用法

> 此时如果我想查看这个关键字前10行和后10行的日志:
> cat -n test.log |tail -n +92|head -n 20
> cat -n test.log |grep “地形” 得到关键日志的行号
> 得到”地形”关键字所在的行号是102行

#### sed用法

> 那么按日期怎么查呢? 通常我们非常需要查找指定时间端的日志

> sed -n ‘/2014-12-17 16:17:20/,/2014-12-17 16:17:36/p’ test.log

> 第二种方式，两个行号之间的命令：

> sed -n ‘5,10p’ filename 这样你就可以只查看文件的第5行到第10行。

> 特别说明:上面的两个日期必须是日志中打印出来的日志,否则无效.

> 关于日期打印,可以先 grep ‘2014-12-17 16:17:20’ test.log 来确定日志中是否有该时间点,以确保第4步可以拿到日志

> 这个根据时间段查询日志是非常有用的命令
> 如果我们查找的日志很多,打印在屏幕上不方便查看, 有两个方法:

#### more和less

> (1)使用more和less命令, 如: cat -n test.log |grep “地形” |more 这样就分页打印了,通过点击空格键翻页

> a.More命令

> more命令，功能类似 cat ，cat命令是整个文件的内容从上到下显示在屏幕上。 more会以一页一页的显示方便使用者逐页阅读，而最基本的指令就是按空白键（space）就往下一页显示，按 b 键就会往回（back）一页显示，而且还有搜寻字串的功能 。more命令从前向后读取文件，因此在启动时就加载整个文件。

> b.Less

> less 工具也是对文件或其它输出进行分页显示的工具，应该说是linux正统查看文件内容的工具，功能极其强大。less 的用法比起 more 更加的有弹性。在 more 的时候，我们并没有办法向前面翻， 只能往后面看，但若使用了 less 时，就可以使用 [pageup] [pagedown] 等按键的功能来往前往后翻看文件，更容易用来查看一个文件的内容！除此之外，在 less 里头可以拥有更多的搜索功能，不止可以向下搜，也可以向上搜。

> (2)使用 >xxx.txt 将其保存到文件中,到时可以拉下这个文件分析.如:

> cat -n test.log |grep “地形” >xxx.txt

> 这几个日志查看方法应该可以满足日常需求了.

#### grep命令多条件查询

> 1、或操作

```shell
grep -E ’123|abc’ filename  // 找出文件（filename）中包含123或者包含abc的行
egrep ’123|abc’ filename    // 用egrep同样可以实现
awk ’/123|abc/’ filename   // awk 的实现方式
```

> 2、与操作

```shell
grep pattern1 files | grep pattern2 
//显示既匹配 pattern1 又匹配 pattern2 的行。
```

> 3、其他操作

```shell
grep -i pattern files ：不区分大小写地搜索。默认情况区分大小写，
grep -l pattern files ：只列出匹配的文件名，
grep -L pattern files ：列出不匹配的文件名，
grep -w pattern files ：只匹配整个单词，而不是字符串的一部分（如匹配‘magic’，而不是‘magical’），
grep -C number pattern files ：匹配的上下文分别显示[number]行

grep的-A, -B, -C选项分别可以显示匹配行的后,前,后前多少行内容: 

grep -A 100 'TooManyResultsException' catalina.log.2017-09-25  后

grep -B 100 'TooManyResultsException' catalina.log.2017-09-25  前

grep -C 100 'TooManyResultsException' catalina.log.2017-09-25  前后

grep -C 100 --color 'TooManyResultsException' catalina.log.2017-09-25  带颜色输出
```

> 查询日志特殊场景：

> 如果日志非常的多，在短短的一个小时的时间中就有上千条或者上万条数据，仅仅根据条件筛选的话非常麻烦，即使筛选出来也会有很多条数据。如果我们知道该条调用发生的时间，就可以根据日志最前面打印的时间判断出哪些记录符合查询条件。所以，我们可以根据日志的日期作为搜索条件，并且配合grep使用，如下所示：

```shell
sed -n '/2017-03-08 15:42:03/,/2017-03-08 15:42:05/p' dubbo-access-consumer.2017-03-08.log | grep countOrgOrder
```

> 我们可以使用sed命令的查询模式：

```shell
使用模式进行查询
     [root@localhost ruby] # sed -n '/ruby/p' ab  #查询包括关键字ruby所在所有行
     [root@localhost ruby] # sed -n '/\$/p' ab    #查询包括关键字$所在所有行，使用反斜线\屏蔽特殊含义

查询.bz2类型的日志文件，如下所示：bzgrep 60000000005137 dubbo-access-provider.2017-05-17.log.bz2
```

> 下面介绍一下如何查询筛选服务器上运行的进程：

```shell
1.使用ps命令执行相应操作，如果想查询服务器上所有运行的进程的话，可以使用命令ps aux即可查出；

2.如果有具体的筛选条件的话，就可以使用ps aux | grep xxx即可；

3.或者使用命令 ps -ef | grep xxx 也可以完成相应的筛选工作；

4.终止某个进程的命令 kill -9 XXXXX     XXXXX为上述查出的序号  如： 19979线程终止为： kill -9 19979
```