---
title: liux中profile、bash_profile、bashrc作用与区别
comments: false
toc: true
date: 2020-08-12 20:31:41
categories:
  - liux
tags:
  - liux
---

###  profile 文件

1. #### `profile`文件的作用

   `profile`（`/etc/profile`），用于设置系统级的环境变量和启动程序，在这个文件下配置会对**所有用户**生效。当用户登录（`login`）时，文件会被执行，并从`/etc/profile.d`目录的配置文件中查找`shell`设置。

2. #### 在`profile`中添加环境变量

   一般不建议在`/etc/profile`文件中添加环境变量，因为在这个文件中添加的设置会对所有用户起作用。当需要添加时，我们可以按以方式添加：

   如，添加一个`HOST`值为`itbilu.com`的环境变量：

   ```
   export HOST=itbilu.com
   ```

   添加时，可以在行尾使用`;`号，也可以不使用。一个变量名可以对应多个变量值，多个变量值使用`:`分隔。

   添加环境变量后，需要重新登录才能生效，也可以使用`source`命令强制立即生效：

   ```
   source /etc/profile
   ```

   查看是否生效可以使用`echo`命令：

   ```
   $ echo $HOST
   itbilu.com
   ```



### `bashrc`文件

这个文件用于配置函数或别名。`bashrc`文件有两种级别：系统级的位于`/etc/bashrc`、用户级的`~/.bashrc`，两者分别会对所有用户和当前用户生效。

`bashrc`文件只会对指定的`shell`类型起作用，`bashrc`只会被`bash shell`调用。

### `bash_profile`文件

`bash_profile`只有单一用户有效，文件存储位于`~/.bash_profile`，该文件是一个用户级的设置，可以理解为某一个用户的`profile`目录下。这个文件同样也可以用于配置环境变量和启动程序，但只针对单个用户有效。

和`profile`文件类似，`bash_profile`也会在用户登录（`login`）时生效，也可以用于设置环境变理。但与`profile`不同，`bash_profile`只会对当前用户生效。


