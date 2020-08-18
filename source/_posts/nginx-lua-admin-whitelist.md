---
title: nginx-Lua对后台白名单限制访问
comments: false
toc: true
categories:
  - nginx
tags:
  - nginx 
  - openresty 
  - lua
date: 2020-08-11 21:14:24
---

1. #### 使用lua实现白名单(lua-resty-iputils模块)

   - 依赖模块lrucache.lua,iputils.lua (注意:[lua-resty-iputils](https://github.com/hamishforbes/lua-resty-iputils)不支持`ipv6`.[lua-resty-iputils](https://github.com/hamishforbes/lua-resty-iputils)的作者推荐[lua-libcidr-ffi](https://github.com/GUI/lua-libcidr-ffi),以支持`ipv6`.`lua-libcidr-ffi`需要`libcidr`)
   - 模块存放位置 /usr/local/nginx/conf/lua/lib/resty/,必须包含resty文件夹
   - 新建whitelist_init.lua,whitelist_access.lua,存放至/usr/local/nginx/conf/lua/
   - http  导入模块
   - server 块导入模块

whitelist_init.lua 内容:

```lua
local iputils = require("resty.iputils")
iputils.enable_lrucache()
local whitelist_ips = {
	"127.0.0.1",
	"172.19.0.0/16",
	"8.8.8.8/24" --vpn IP
}

whitelist = iputils.parse_cidrs(whitelist_ips)
```
whitelist_access.lua 内容:

```lua
local iputils = require("resty.iputils")
local headers = ngx.req.get_headers()
local ip = headers["X-REAL-IP"] or headers["X_FORWARDED_FOR"] or ngx.var.remote_addr or "0.0.0.0"
--ngx.say("IP: ", ip)
if not iputils.ip_in_cidrs(ip, whitelist) then
  return ngx.exit(ngx.HTTP_FORBIDDEN)
end
```
http 内容:

```nginx
   http {
   		.................
        #lua模块导入
        lua_package_path "/usr/local/nginx/conf/lua/lib/?.lua;;";
        #访问白名单初始化
        init_by_lua_file /usr/local/nginx/conf/lua/whitelist_init.lua;
        .................
    }
```
server 内容:

```nginx
   server {
          ..............
          location / {
            access_by_lua_file /usr/local/nginx/conf/lua/whitelist_access.lua;
          ..............
          }
	}
```
2. #### 使用lua实现白名单(lua-libcidr-ffi模块 支持ipv6)

   `libcidr`编译

   ```nginx
    ...
       && curl -fSL http://www.over-yonder.net/~fullermd/projects/libcidr/libcidr-1.2.3.tar.xz -o /tmp/libcidr-1.2.3.tar.xz \
       && xz -d /tmp/libcidr-1.2.3.tar.xz && tar -xvf /tmp/libcidr-1.2.3.tar \
       && cd libcidr-1.2.3/src && make && mv libcidr.so.0 /usr/local/openresty/lualib/libcidr.so \
       ...
   ```

   - `alpine`需要安装`xz`,`coreutils`.其中`xz`用来解压,`libcidr`编译的时候会用到`coreutils`里面的`tsort`

   - `make`后不用`make DESTDIR=/your_path install`.执行`make DESTDIR=/your_path install`的话会报错.其实`make`执行后，就已经生成so文件了,只不过叫`libcidr.so.0`.把这个文件移动到`openresty`可以解析的目录就可以了

     ## 修改libcidr-ffi.lua

     只是编译了`libcidr`的话，运行会报无法找到模块.所以还需要修改`libcidr-ffi.lua`文件,找到里面的

     ```lua
     local cidr = ffi.load("cidr")
     ```

     改为

     ```lua
     local cidr = ffi.load("/usr/local/openresty/lualib/libcidr.so")
     ```

     路径就是上面编译后移动到的路径
     构建镜像的时候，记得把修改后的`libcidr-ffi.lua`加到镜像里

     ## 使用lua-libcidr-ffi

     whitelist_init.lua

     ```lua
     whitelist_ips = {
         "103.21.244.0/22",
     }
     ```

     whitelist_access.lua

     ```lua
     local cidr = require("resty.libcidr-ffi")
     local flag=false
     for i, v in ipairs(whitelist_ips) do
       --proxy_protocol_addr在白名单里
       if cidr.contains(cidr.from_str(v), cidr.from_str(ngx.var.proxy_protocol_addr)) then
         ngx.log(ngx.ERR,ngx.var.proxy_protocol_addr,' match ',v)
         flag=true
         break
       end
     end
      
     if not flag then
       return ngx.exit(ngx.HTTP_FORBIDDEN)
     end
     ```

     nginx.conf

     ```nginx
     http{
         init_by_lua_file whitelist_init.lua;
      
         server{
             listen 4430;
             location / {
                 access_by_lua_file whitelist_access.lua;
             }
         }
     }
     ```

3. #### 使用nginx的allow/deny指令实现白名单

   使用nginx的[allow/deny指令](https://nginx.org/en/docs/stream/ngx_stream_access_module.html)也可以实现，不过要注意的是，allow/deny指令默认对`remote_addr`过滤，如果nginx前面有反向代理，`remote_addr`将会是127.0.0.1,使指令不起作用.所以需要使用`ngx_http_realip_module`,修改allow/deny指令的过滤对象
   `ngx_http_realip_module`需要编译的时候添加

   ```nginx
   --with-http_realip_module
   ```

   ##### allow/deny指令的语法:

   ```nginx
   allow/deny address | CIDR | unix: | all
   ```

   允许/拒绝某个ip或者一个ip段访问.

   如果指定unix:,那将允许socket的访问。
   注意：unix:是在1.5.1中新加入的功能。


   allowed-ip.conf

   ```nginx
   allow 1.1.1.0/22;
   ...
   ```

   nginx.conf

```
server {
  ......
  location / {
    real_ip_header proxy_protocol; #X-Forwarded-For
	set_real_ip_from 127.0.0.1;
  ......
	include /root/allowed-ip.conf;
	deny all;
  }
}
```

set_real_ip_from必须要设置，否则real_ip_header不起作用

```nginx
deny 192.168.1.1    #屏蔽单个IP的命令是
deny 123.0.0.0/8    #封整个段即从123.0.0.1到123.255.255.254的命令
deny 124.45.0.0/16  #封IP段即从123.45.0.1到123.45.255.254的命令
deny 123.45.6.0/24  #封IP段即从123.45.6.1到123.45.6.254的命令是
```

斜杠后的数值(子网掩码)：
8：  匹配后三位最大值的
16：匹配后两位最大值的
24：匹配后一位最大值的

