---
title: nginx-Lua限流模块lua-resty-limit-traffic使用
comments: false
toc: true
categories:
  - nginx
tags:
  - nginx
date: 2020-08-14 21:14:24
---

###### 限制接口总并发数

按照 ip 限制其并发连接数

```js
lua_shared_dict my_limit_conn_store 100m;
...
location /hello {
    access_by_lua_block {
        local limit_conn = require "resty.limit.conn"
        -- 限制一个 ip 客户端最大 1 个并发请求
        -- burst 设置为 0，如果超过最大的并发请求数，则直接返回503，
        -- 如果此处要允许突增的并发数，可以修改 burst 的值（漏桶的桶容量）
        -- 最后一个参数其实是你要预估这些并发（或者说单个请求）要处理多久，以便于对桶里面的请求应用漏桶算法
        
        local lim, err = limit_conn.new("my_limit_conn_store", 1, 0, 0.5)              
        if not lim then
            ngx.log(ngx.ERR, "failed to instantiate a resty.limit.conn object: ", err)
            return ngx.exit(500)
        end

        local key = ngx.var.binary_remote_addr
        -- commit 为true 代表要更新shared dict中key的值，
        -- false 代表只是查看当前请求要处理的延时情况和前面还未被处理的请求数
        local delay, err = lim:incoming(key, true)
        if not delay then
            if err == "rejected" then
                return ngx.exit(503)
            end
            ngx.log(ngx.ERR, "failed to limit req: ", err)
            return ngx.exit(500)
        end

        -- 如果请求连接计数等信息被加到shared dict中，则在ctx中记录下，
        -- 因为后面要告知连接断开，以处理其他连接
        if lim:is_committed() then
            local ctx = ngx.ctx
            ctx.limit_conn = lim
            ctx.limit_conn_key = key
            ctx.limit_conn_delay = delay
        end

        local conn = err
        -- 其实这里的 delay 肯定是上面说的并发处理时间的整数倍，
        -- 举个例子，每秒处理100并发，桶容量200个，当时同时来500个并发，则200个拒掉
        -- 100个在被处理，然后200个进入桶中暂存，被暂存的这200个连接中，0-100个连接其实应该延后0.5秒处理，
        -- 101-200个则应该延后0.5*2=1秒处理（0.5是上面预估的并发处理时间）
        if delay >= 0.001 then
            ngx.sleep(delay)
        end
    }

    log_by_lua_block {
        local ctx = ngx.ctx
        local lim = ctx.limit_conn
        if lim then
            local key = ctx.limit_conn_key
            -- 这个连接处理完后应该告知一下，更新shared dict中的值，让后续连接可以接入进来处理
            -- 此处可以动态更新你之前的预估时间，但是别忘了把limit_conn.new这个方法抽出去写，
            -- 要不每次请求进来又会重置
            local conn, err = lim:leaving(key, 0.5)
            if not conn then
                ngx.log(ngx.ERR,
                        "failed to record the connection leaving ",
                        "request: ", err)
                return
            end
        end
    }
    proxy_pass http://10.100.157.198:6112;
    proxy_set_header Host $host;
    proxy_redirect off;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_connect_timeout 60;
    proxy_read_timeout 600;
    proxy_send_timeout 600;
}
```

说明：

其实此处没有设置 `burst` 的值，就是单纯的限制最大并发数，如果设置了 `burst` 的值，并且做了延时处理，其实就是对并发数使用了漏桶算法，但是如果不做延时处理，其实就是使用的令牌桶算法。参考下面对请求数使用漏桶令牌桶的部分，并发数的漏桶令牌桶实现与之相似

###### 限制接口时间窗请求数

场景：

限制 `ip` 每分钟只能调用 120 次 `/hello` 接口（允许在时间段开始的时候一次性放过120个请求）

```js
lua_shared_dict my_limit_count_store 100m;
...

init_by_lua_block {
    require "resty.core"
}
....

location /hello {
    access_by_lua_block {
        local limit_count = require "resty.limit.count"

        -- rate: 10/min 
        local lim, err = limit_count.new("my_limit_count_store", 120, 60)
        if not lim then
            ngx.log(ngx.ERR, "failed to instantiate a resty.limit.count object: ", err)
            return ngx.exit(500)
        end

        local key = ngx.var.binary_remote_addr
        local delay, err = lim:incoming(key, true)
        -- 如果请求数在限制范围内，则当前请求被处理的延迟（这种场景下始终为0，因为要么被处理要么被拒绝）和将被处理的请求的剩余数
        if not delay then
            if err == "rejected" then
                return ngx.exit(503)
            end

            ngx.log(ngx.ERR, "failed to limit count: ", err)
            return ngx.exit(500)
        end
    }

    proxy_pass http://10.100.157.198:6112;
    proxy_set_header Host $host;
    proxy_redirect off;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_connect_timeout 60;
    proxy_read_timeout 600;
    proxy_send_timeout 600;
}
```

###### 平滑限制接口请求数

场景：

限制 ip 每分钟只能调用 120 次 /hello 接口（平滑处理请求，即每秒放过2个请求）

```js
lua_shared_dict my_limit_req_store 100m;
....

location /hello {
    access_by_lua_block {
        local limit_req = require "resty.limit.req"
        -- 这里设置rate=2/s，漏桶桶容量设置为0，（也就是来多少水就留多少水） 
        -- 因为resty.limit.req代码中控制粒度为毫秒级别，所以可以做到毫秒级别的平滑处理
        local lim, err = limit_req.new("my_limit_req_store", 2, 0)
        if not lim then
            ngx.log(ngx.ERR, "failed to instantiate a resty.limit.req object: ", err)
            return ngx.exit(500)
        end

        local key = ngx.var.binary_remote_addr
        local delay, err = lim:incoming(key, true)
        if not delay then
            if err == "rejected" then
                return ngx.exit(503)
            end
            ngx.log(ngx.ERR, "failed to limit req: ", err)
            return ngx.exit(500)
        end
    }

    proxy_pass http://10.100.157.198:6112;
    proxy_set_header Host $host;
    proxy_redirect off;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_connect_timeout 60;
    proxy_read_timeout 600;
    proxy_send_timeout 600;
}
```

###### 漏桶算法限流

场景：

限制 ip 每分钟只能调用 120 次 /hello 接口（平滑处理请求，即每秒放过2个请求），超过部分进入桶中等待，（桶容量为60），如果桶也满了，则进行限流

```js
lua_shared_dict my_limit_req_store 100m;
....

location /hello {
    access_by_lua_block {
        local limit_req = require "resty.limit.req"
        -- 这里设置rate=2/s，漏桶桶容量设置为0，（也就是来多少水就留多少水） 
        -- 因为resty.limit.req代码中控制粒度为毫秒级别，所以可以做到毫秒级别的平滑处理
        local lim, err = limit_req.new("my_limit_req_store", 2, 60)
        if not lim then
            ngx.log(ngx.ERR, "failed to instantiate a resty.limit.req object: ", err)
            return ngx.exit(500)
        end

        local key = ngx.var.binary_remote_addr
        local delay, err = lim:incoming(key, true)
        if not delay then
            if err == "rejected" then
                return ngx.exit(503)
            end
            ngx.log(ngx.ERR, "failed to limit req: ", err)
            return ngx.exit(500)
        end
        
        -- 此方法返回，当前请求需要delay秒后才会被处理，和他前面对请求数
        -- 所以此处对桶中请求进行延时处理，让其排队等待，就是应用了漏桶算法
        -- 此处也是与令牌桶的主要区别既
        if delay >= 0.001 then
            ngx.sleep(delay)
        end
    }

    proxy_pass http://10.100.157.198:6112;
    proxy_set_header Host $host;
    proxy_redirect off;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_connect_timeout 60;
    proxy_read_timeout 600;
    proxy_send_timeout 600;
}
```

###### 令牌桶算法限流

令牌桶其实可以看着是漏桶的逆操作，看我们对把超过请求速率而进入桶中的请求如何处理，如果是我们把这部分请求放入到等待队列中去，那么其实就是用了漏桶算法，但是如果我们允许直接处理这部分的突发请求，其实就是使用了令牌桶算法。

场景：

限制 ip 每分钟只能调用 120 次 /hello 接口（平滑处理请求，即每秒放过2个请求），但是允许一定的突发流量（突发的流量，就是桶的容量（桶容量为60），超过桶容量直接拒绝

这边只要将上面漏桶算法关于桶中请求的延时处理的代码修改成直接送到后端服务就可以了，这样便是使用了令牌桶

```js
lua_shared_dict my_limit_req_store 100m;
....

location /hello {
    access_by_lua_block {
        local limit_req = require "resty.limit.req"

        local lim, err = limit_req.new("my_limit_req_store", 2, 0)
        if not lim then
            ngx.log(ngx.ERR, "failed to instantiate a resty.limit.req object: ", err)
            return ngx.exit(500)
        end

        local key = ngx.var.binary_remote_addr
        local delay, err = lim:incoming(key, true)
        if not delay then
            if err == "rejected" then
                return ngx.exit(503)
            end
            ngx.log(ngx.ERR, "failed to limit req: ", err)
            return ngx.exit(500)
        end
        
        -- 此方法返回，当前请求需要delay秒后才会被处理，和他前面对请求数
        -- 此处忽略桶中请求所需要的延时处理，让其直接返送到后端服务器，
        -- 其实这就是允许桶中请求作为突发流量 也就是令牌桶桶的原理所在
        if delay >= 0.001 then
        --    ngx.sleep(delay)
        end
    }

    proxy_pass http://10.100.157.198:6112;
    proxy_set_header Host $host;
    proxy_redirect off;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_connect_timeout 60;
    proxy_read_timeout 600;
    proxy_send_timeout 600;
}
```

说明：

其实nginx的ngx_http_limit_req_module 这个模块中的delay和nodelay也就是类似此处对桶中请求是否做延迟处理的两种方案，也就是分别对应的漏桶和令牌桶两种算法