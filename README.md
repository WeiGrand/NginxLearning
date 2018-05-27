# Nginx 学习笔记 (基于 CentOS 7.x)

### Nginx 的优点

- IO多路复用 (`epoll` 模型)
- 轻量级(功能模块少，代码模块化)
- CPU亲和(把每个worker进程固定到一个cpu上执行)
- `sendfile` 机制(文件只通过内核空间传送给 `socket`)



### 安装Nginx

配置 `nginx.repo`

```ba
vim /etc/yum.repos.d/nginx.repo
```

```bash
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
```

安装

```bash
yum install nginx
```

检查是否安装成功

```bash
nginx -v
nginx version: nginx/1.14.0
```

安装常用的库

```bash
yum -y install gcc gcc-c++ autoconf pcre pcre-devel make automake
yum -y install wget httpd-tools vim
```

查看安装目录

```bash
rpm -ql nginx
```

查看编译配置参数

```bash
nginx -V
```

```bash
configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --
modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf
 --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/ngi
nx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.
lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-p
roxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/va
r/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsg
i_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --
group=nginx --with-compat --with-file-aio --with-threads --with-http_ad
dition_module --with-http_auth_request_module --with-http_dav_module --
with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_
module --with-http_mp4_module --with-http_random_index_module --with-ht
tp_realip_module --with-http_secure_link_module --with-http_slice_modul
e --with-http_ssl_module --with-http_stub_status_module --with-http_sub
_module --with-http_v2_module --with-mail --with-mail_ssl_module --with
-stream --with-stream_realip_module --with-stream_ssl_module --with-str
eam_ssl_preread_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY
_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size
=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC' --with-ld-opt='-Wl,
-z,relro -Wl,-z,now -pie'=
```



### 默认配置语法

```bash
cd /etc/nginx
vim nginx.conf
```

```bash
user  nginx; # 设置 nginx 服务的系统使用用户
worker_processes  1; # 工作进程数，一般和 CPU 数保持一直

error_log  /var/log/nginx/error.log warn; # 错误日志路径 其中 warn 是错误级别
pid        /var/run/nginx.pid; # 服务启动时候的 pid


events {
    worker_connections  1024;
}


http {
events {
    worker_connections  1024; # 每个进程允许的最大连接数
    use # 使用的内核模型
}

http { # 一个 http 允许多个 server 一个 server 允许多个 location
	...
	server {
        listen 80; # 监听的端口
        server_name locahost; # 域名
        location / { # 路由
            root /user/share/nginx/html; # 路径
            index index.html index.htm # 文件
        }
        error_page 500 502 503 504 /50x.html; # 错误页面
        location = /50x.html {
            root /usr/share/
        }
	},
	
	server {
        ...
	}

    include       /etc/nginx/mime.types; # 设置 Content-type
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for" "$request_uri"'
;

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65; # 超时时间，单位(s)

    #gzip  on;

    include /etc/nginx/conf.d/*.conf; # 引入 conf.d 下的所有配置文件，默认只有 default.conf
}
```

开启

```bas
nginx  -c /etc/nginx/nginx.conf;
```

重启 `nginx`

```bash
systemctl reload nginx.service
```

或者

```bash
systemctl restart nginx.service
```

或

```bash
nginx -s reload -c /etc/nginx/nginx.conf
```

关掉 `nginx`

```bas
nginx -s stop -c /etc/nginx/nginx.conf
```



### HTTP 请求

- `request` : `请求行`、`请求头`、`请求数据`
- `response`: `状态行`、`消息报头`、`响应正文`



### Nginx 变量

- HTTP 请求变量
- 内置变量
- 自定义变量

参考链接：http://nginx.org/en/docs/http/ngx_http_log_module.html#access_log



### Nginx 日志

- `error.log`
- `access.log`

```bash
http {
	# 日志格式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for" "$request_uri"'
}

# $remote_addr 客服端地址
# $remote_addr 用户名
# $time_local 请求时间
# $request request 头的请求行
# $status response 返回状态
# $body_bytes_sent response body 信息的大小
# $http_referer 上一级页面地址
# $http_user_agent ua
```

#### error.log

```bas
error_log  /var/log/nginx/error.log warn; # 错误日志路径 其中 warn 是错误级别
```

查看 `error.log`

```bash
tail -f /var/log/nginx/error.log

2018/05/19 11:25:07 [error] 43#43: *6 open() "/usr/share/nginx/h
tml/a" failed (2: No such file or directory), client: 172.16.67.
0, server: localhost, request: "GET /a HTTP/1.0", host: "weigran
d.com"
```

```bash
2018/05/19 11:25:07 # 日期
[error] # 错误级别
43#43: *6 
open() "/usr/share/nginx/h
tml/a" failed (2: No such file or directory), client: 172.16.67.
0, server: localhost, request: "GET /a HTTP/1.0", host: "weigran
d.com" # 错误信息
```

#### access.log

```bash
http {
    ...
    access_log  /var/log/nginx/access.log  main;
}
```

查看 `access.log`

```bash
tail -f /var/log/nginx/error.log

172.16.67.0 - - [19/May/2018:11:25:09 +0800] "GET / HTTP/1.0" 30
4 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWe
bKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181 Safari/537.
36" "119.129.129.222" "/"
```

#### 如果想要在 `access.log` 中记录访问的 `Cookie`

修改 `nginx.conf` 在 `log_format` 中加入 `User-Agent` 变量 (`$http_cookie`)

```bash
vim /etc/nginx/nginx.conf
```

```bash
http {
    log_format main '$http_cookie' '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for" "$request_uri"'
}
```

保存，然后检查配置是否正确

```bash
nginx -t -c /etc/nginx/nginx.conf
```

如果成功会看到

```bash
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

然后重启

```bash
nginx -s reload -c /etc/nginx/nginx.conf
```

然后请求一下 `nginx` 服务，再查看 `access.log`，可以看到已经有 `Cookie` 的信息了(`marking=d3n6cj`)

```bash
marking=d3n6cj 172.16.67.0 - - [19/May/2018:12:14:51 +0800] "GET / HTTP/1.0" 200 614 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181 Safari/537.36" "119.129.129.222" "/"
```



### Nginx 模块

#### Nginx 官方模块

- `http_stub_status_module` 

  | 编译选项                      | 作用                                                    |
  | ----------------------------- | ------------------------------------------------------- |
  | —with-http_stub_status_module | Nginx 的客户端状态，用于监控 Nginx 当前<br />的连接状态 |

  | Syntax      | Default | Context          |
  | ----------- | ------- | ---------------- |
  | stub_status | --      | server, location |

  修改 `/etc/nginx/conf.d/default.conf`

  ```bash
  server {
      ...
      location /mystatus {
          stub_status;
      }
  }
  ```

  重启 `nginx` 后访问 `你的域名/mystatus` 可以看到类似信息

  ```
  Active connections: 1  # 当前活跃的连接数
  server accepts handled requests # 接受的握手数 处理的连接数 总的请求数 一般 accepts == handled
   1 1 1 
  Reading: 0 Writing: 1 Waiting: 0 # 读 写 等待
  ```

- `http_random_index_module`

  | 编译选项                       | 作用                   |
  | ------------------------------ | ---------------------- |
  | —with-http_random_index_module | 目录中选择一个随机主页 |

  | Syntax               | Default          | Context  |
  | -------------------- | ---------------- | -------- |
  | random_index on\|off | random_index off | location |

  修改 `/etc/nginx/conf.d/default.conf`

  ```bash
  server {
      ...
      location /mystatus {
      	# root /usr/share/nginx/html;
          root /opt/app/code;
          random_index on;
          # index index.html index.htm
      }
  }
  ```

  > 不显示 `.` 开头的文件。eg: `.c.html`

- `http_sub_module`
  | 编译选项              | 作用         |
  | --------------------- | ------------ |
  | —with-http_sub_module | HTTP内容替换 |

  | Syntax                                            | Default                      | Context                |
  | ------------------------------------------------- | ---------------------------- | ---------------------- |
  | sub_filter string replacement                     | --                           | http, server, location |
  | sub_filter_last_modified on \| off (主要用于缓存) | sub_filter_last_modified off | http, server, location |
  | sub_filter_once on \| off (是否只替换第一个)      | sub_filter_once on           | http, server, location |

  修改 `/etc/nginx/conf.d/default.conf`

  ```bash
  http {
      ...
      sub_filter 'h1' 'div'; # 将 h1 替换为 div
      sub_filter_once off; # 将 once 关掉才能将全部的 h1 都替换掉
  }
  ```


### Nginx 的请求限制

- 连接频率限制 - `limit_conn_module`

    | Syntax                              | Default | Context                |
    | ----------------------------------- | ------- | ---------------------- |
    | limit_conn_zone key zone=name:size; | --      | http                   |
    | limit_conn zone number;             | --      | http, server, location |

    例子

    ```bash
    limit_conn_zone $binary_remote_addr zone=conn_zone:1m;
    
    server {
        ...
        
        location {
            ...
            limit_conn conn_zone 1;
        }
    }
    ```

    

- 请求频率限制 - `limit_req_module`

    | Syntax                                               | Default | Context                |
    | ---------------------------------------------------- | ------- | ---------------------- |
    | limit_req_zone key zone=name:size rate=rate;         | --      | http                   |
    | limit_conn zone=name number [burst=number][nodelay]; | --      | http, server, location |

    例子

    修改 `/etc/nginx/conf.d/default.conf`

    ```bash
    limit_req_zone $binary_remote_addr zone=req_zone:1m rate=1r/s; # 允许一秒发起一个
    
    server {
        ...
        
        locatino / {
            ...
            limit_req zone=req_zone
        }
    }
    ```

    然后同时发起两次请求，将会看到一个请求正常返回，而另一个请求则报错，同时查看 `error.log` 也有相应的记录

    ```bash
    tail -f /var/log/nginx/error.log
    
    ...
    2018/05/19 20:00:26 [error] 17962#17962: *39 limiting reques
    ts, excess: 0.196 by zone "req_zone", client: 172.16.67.0, s
    erver: localhost, request: "GET / HTTP/1.0", host: "weigrand
    .p.imooc.io"
    ```



### Nginx 访问控制

- 基于 `IP` (`remote_addr`)的访问控制 - `http_access_module`

    | Syntax                                 | Default | Context                              |
    | -------------------------------------- | ------- | ------------------------------------ |
    | allow address \| CIDR \| unix: \| all; | --      | http, server, location, limit_except |
    | deny address \| CIDR \| unix: \| all;  | --      | http, server, location, limit_except |

    例子

    ```bash
    server {
        ...
        
        location ~ ^/admin.html { # ~ 代表模式匹配 
        	...
            deny 0.0.0.0; # 限制的 ip
            allow all;
        }
    }
    ```

    然后访问 `你的域名/admin.html` 会返回 `403`

    

    这种方法的局限性是只能通过 `$remote_addr` 控制信任，客户端不一定直接访问服务端，可以使用别的 HTTP 头信息控制，如: `HTTP_X_FORWARDED_FOR`，或者通过HTTP自定义变量将客户端 `remote_addr` 一层一层地传递到服务端

    > http_x_forwarded_fir = Client IP, Proxy(1), Proxy(2)

- 基于 `用户的信任登录` - `http_auth_basic_module` 

    | Syntax                    | Default        | Context                              |
    | ------------------------- | -------------- | ------------------------------------ |
    | auth_basic string \| off  | auth_basic off | http, server, location, limit_except |
    | auth_basic_user_file file | --             | http, server, location, limit_except |

    例子

    首先生成密码配置文件

    ```bash
    cd /etc/nginx
    htpasswd -c ./auth_conf 用户名
    ```

    输入两次密码之后完成

    ```bash
    more auth_conf
    用户名:$apr1$tKVOaqxw$74NMfv6iua3xnqpkSGXp.0
    ```

    修改 `default.conf`

    ```bash
    server {
        ...
        location ~ ^/admin.html {
            root /opt/app/code;
            auth_basic "Auth required! input your password!";
            auth_basic_user_file /etc/nginx/auth_conf;
            index index.html index.htm;
        }
    }
    ```

    然后访问 `你的域名/admin.html`，将会出现一个弹窗让你填写账号和密码



### 常见的 Nginx 中间架构

- 静态资源WEB服务

- 代理服务

- 负载均衡调度器SLB

- 动态缓存



#### 静态资源WEB服务

##### **sendfile**

| Syntax             | Default      | Context                                |
| ------------------ | ------------ | -------------------------------------- |
| sendfile on \| off | sendfile off | http, server, location, if in location |

##### **tcp_nopush**

sendfile开启的情况下，提高网络包的传输效率

| Syntax               | Default        | Context          |
| -------------------- | -------------- | ---------------- |
| tcp_nopush on \| off | tcp_nopush off | server, location |

#####**tcp_nodelay**

keepalive 连接下，提高网络包的传输实时性

| Syntax                | Default         | Context          |
| --------------------- | --------------- | ---------------- |
| tcp_nodelay on \| off | tcp_nodelay  on | server, location |

**压缩**

服务端压缩，客户端解压

| Syntax                       | Default               | Context                          |
| ---------------------------- | --------------------- | -------------------------------- |
| gzip on \| off               | gzip  on              | server, location, if in location |
| gzip_comp_level level        | gzip_comp_level 1     | server, location                 |
| gzip_http_version 1.0 \| 1.1 | gzip_http_version 1.1 | server, location                 |

**例子**

修改 `default.conf`

```bash
server {
    ...
    sendfile on;
    
    location ~ .*\.(jpg|gif|png)$ {
    	# gzip 配置
    	gzip on;
    	gzip_http_version 1.1;
    	gzip_comp_level 2;
    	gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
    	
        root /opt/app/code/images;
    }
    
    location ~ .*\.(txt|xml)$ {
    	gzip on;
    	gzip_http_version 1.1;
    	gzip_comp_level 1;
    	gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
    
        root /opt/app/code/doc;
    }
    
    location ~ ^/download {
        gzip_static on; # 会预读 .gz 文件
        tcp_nopush on;
        root /opt/app/code;
    }
}
```

在响应头可以看到 `Content-Encoding: gzip` 代表设置成功



#### 浏览器缓存

**校验过期机制**

![](http://olkiij9c9.bkt.clouddn.com/%E6%B5%8F%E8%A7%88%E5%99%A8%E7%BC%93%E5%AD%98.jpg)

> **Etag 和 Last-Modified 实例**
>
> HTTP/1.1 200 OK Server: nginx Date: Sat, 26 May 2018 05:36:54 GMT Content-Type: text/html Content-Length: 174 Connection: keep-alive marking: d3n6cj Set-Cookie: marking=d3n6cj; path=/ month: 05 **Last-Modified: Sat, 26 May 2018 05:32:21 GMT ETag: "5b08f165-ae" Accept-Ranges: bytes**

**expires**

添加 `Cache-Control`、`Expires` 头

| Syntax                       | Default      | Context                                |
| ---------------------------- | ------------ | -------------------------------------- |
| expires [modified] time;     | expires off; | http, server, location, if in location |
| expires epoch \| max \| off; | expires off; | http, server, location, if in location |

**例子**

```bash
server {
    ...
    
    location ~ .*\.(htm|html)$ {
        expires 24h;
        ...
    }
}
```

然后请求 `htm | html` 查看 `响应头` 可以看到新增的 `Cache-Control` 和 `Expires`

```
HTTP/1.1 304 Not Modified
Server: nginx
Date: Sat, 26 May 2018 05:44:09 GMT
Content-Type: text/html
Connection: keep-alive
marking: d3n6cj
Set-Cookie: marking=d3n6cj; path=/
month: 05
Last-Modified: Sat, 26 May 2018 05:32:21 GMT
ETag: "5b08f165-ae"


Expires: Sun, 27 May 2018 05:44:09 GMT
Cache-Control: max-age=86400
```



#### 跨域访问

| Syntax                          | Default | Context                          |
| ------------------------------- | ------- | -------------------------------- |
| add_header name value [always]; | --      | server, location, if in location |

**例子**

```bash
server {
    ...
    
    location ~ .*\.(htm|html)$ {
        #add_header Access-Control-Allow-Origin http://www.taobao.com;
        #add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE,OPTIONS;
        ...
    } 
}
```



#### 防盗链

设置思路：区别哪些请求是非正常的用户请求

**基于 `http_refer` 配置防盗链**

如防止 `http:taobao.com` 的请求

```bash
server {
    ...
    
    location ~ .*\.(jpg|gif|png)$ {
        ...
 
        valid_referers none blocked http://taobao.com 你的域名/xxx.png;
        if ($invalid_referer) {
            return 403;
        }
    }
}
```

```bash
curl -I "你的域名/xxx.png"

HTTP/1.1 200 OK
Server: nginx
Date: Sat, 26 May 2018 06:17:55 GMT
Content-Type: image/png
Content-Length: 244044
Connection: keep-alive
marking: d3n6cj
Set-Cookie: marking=d3n6cj; path=/
month: 05
Last-Modified: Sat, 26 May 2018 06:08:58 GMT
ETag: "5b08f9fa-3b94c"
Accept-Ranges: bytes
```

如果 `refer` 是淘宝，则返回 403

```bash
curl -e "http://taobao.com" -I "http://weigrand.p.imooc.io/wei.png"

HTTP/1.1 403 Forbidden
Server: nginx
Date: Sat, 26 May 2018 06:25:17 GMT
Content-Type: text/html
Content-Length: 169
Connection: keep-alive
marking: d3n6cj
Set-Cookie: marking=d3n6cj; path=/
month: 05
```



#### 代理服务

> 正向代理的对象是客户端
>
> 反向代理的对象是服务端

| Syntax                                                       | Default                                                      | Context                                |
| ------------------------------------------------------------ | ------------------------------------------------------------ | -------------------------------------- |
| proxy_pass URL                                               | --                                                           | location, if in location, limit_except |
| proxy_buffering on \| off<br /> (把一个请求所有信息收集完再返回给客户端，减少 IO 损耗) | proxy_buffering on                                           | server, location                       |
| proxy_set_header field value;<br />(设置头信息)              | proxy_set header Host $proxy_host;<br />proxy_set_header Connection close | http, server, location                 |
| proxy_connect_timeout time;<br />(超时时间)                  | proxy_connect_timeout 60s;                                   | http, server, location                 |

**例子**

```bash
# in proxy_conf
proxy_redirect default;

proxy_set_header HOST $http_host;
proxy_set_header X_Real-IP $remore_addr; # 用户访问限制

proxy_connect_timeout 30;
proxy_send_timeout 60;
proxy_read_timeout 60;

proxy_buffer_size 32k;
proxy_buffering on;
proxy_buffers 4 128k;
proxy_busy_buffers_size 256k;
proxy_max_temp_file_size 256;
```



```bash
server {
    ...
    
    location / {
        proxy_pass http://127.0.0.1: 8080;
        
        include proxy_conf;
    }
}
```



**正向例子**

```bash
server {
    ...
    
    resolver 8.8.8.8; # DNS server
    location / {
        proxy_pass http://$http_host$request_uri;
    }
}
```

这样设置相当于别人请求你的服务器，然后你会代理他们的请求去请求他们想要请求的资源`http://$http_host$request_uri` 再返回给他们，相当于「科学上网」



**反向例子**

```bash
server {
    ...
    
    location ~ /xxx.html$ {
        proxy_pass http://127.0.0.1:8080;
    }
}
```

配置完成后，`你的域名/xxx.html` 将被代理到 `http://127.0.0.1:8080/xxx.html` 上



#### 负载均衡

> **负载平衡**（Load balancing）是一种计算机技术，用来在多个计算机（计算机集群）、网络连接、CPU、磁盘驱动器或其他资源中分配负载，以达到最优化资源使用、最大化吞吐率、最小化响应时间、同时避免过载的目的。

使用到了 `proxy_pass` 和 `upstream`

| Syntax              | Default | Context |
| ------------------- | ------- | ------- |
| upstream name { … } | --      | http    |

**简单的例子**

```bash
http {
    ...
    
    upstream weigrand {
        server 网站的IP：8001；
        server 网站的IP：8002；
        server 网站的IP：8003；
    }
    
    location / {
        proxy_pass http://weigrand;
        ...
    }
}
```

然后访问 `你的网站` nginx 会依次代理(轮询)到 `8001`、`8002`、`8003` 端口

**upstream server的参数**

| 参数         | 意义                                  |
| ------------ | ------------------------------------- |
| down         | 当前的 server 暂时不参与负载均衡      |
| backup       | 预留的备份服务器                      |
| max_fails    | 允许请求失败的次数                    |
| fail_timeout | 经过 max_fails 失败后，服务暂停的时间 |
| max_conns    | 限制最大的接收的连接数                |

**带参数的例子**

```bash
http {
    ...
    
    upstream weigrand {
        server 网站的IP：8001 down；
        server 网站的IP：8002 backup；
        server 网站的IP：8003 max_fails=1 fail_timeout=10s；
    }
}
```

然后可以看到只有 `8003` 端口可以被访问，而如果把 `8003` 端口的服务关掉的话，再次访问将访问 `8002` 端口



**调度算法**

| 术语          | 解释                                   |
| ------------- | -------------------------------------- |
| 轮询（默认）  | 按时间顺序逐一分配到不同的服务器       |
| 加权轮询      | weight值越大，分配到的访问几率越高     |
| ip_hash       | 根据访问 `IP` 的 `hash` 结果分配       |
| least_conn    | 最少连接数，哪个机器连接数少就分发到哪 |
| url_hash      | 根据访问的 `URL` 的 `hash` 结果分配    |
| hash 关键数值 | `hash` 自定义的 `key`                  |

**加权轮询例子**

```bash
http {
    ...
    
    upstream weigrand {
        server 网站的IP：8001；
        server 网站的IP：8002 weight=8；
        server 网站的IP：8003；
    }
}
```

这样每 `10个请求` 就会有 `8个` 命中 `8002` 端口



**ip_hash例子**

```bash
http {
    ...
    
    upstream weigrand {
    	ip_hash;
       	server 网站的IP：8001；
       	server 网站的IP：8002；
       	server 网站的IP：8003；
    }
}
```

这样来自同一个 `IP` 的固定访问一个服务器



**hash 关键数值例子**

```bash
http {
    ...
    
    upstream weigrand {
    	hash $request_uri;
       	server 网站的IP：8001；
       	server 网站的IP：8002；
       	server 网站的IP：8003；
    }
}
```

访问 `你的域名/xxx` 将被分配到同一个台服务器上



#### 缓存服务（代理缓存）

**proxy_cache**

| Syntax                         | Default                                           | Context                |
| ------------------------------ | ------------------------------------------------- | ---------------------- |
| proxy_cache zone \| off        | proxy_cache off;                                  | http, server, location |
| proxy_cache_valid [code…] time | --                                                | http, server, location |
| proxy_cache_key string         | proxy_cache_key $scheme\$proxy_host\$request_uri; | http, server, location |

**例子**

```bash
http {
    ...
    
    proxy_cache_path /opt/app/cache levels=1:2 keys_zone=weigrand_cache:10m max_size=10g inactive=60m use_temp_path=off;
    
    location / {
        proxy_cache weigrand_cache;
        proxy_pass http://weigrand;
        proxy_cache_valid 200 304 12h;
        proxy_cache_valid any 10m;
        proxy_cache_key $host$uri$is_args$args;
        add_header  Nginx-Cache "$upstream_cache_status";

        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        include proxy_params;
    }
}
```

然后访问一下 `你的网站` 之后 查看 `cache` 目录，可以看到缓存文件已经生成

```bash
cat /opt/app/cache/c/92/db6844138dd62ac4af0a5fe6c908592c

"5b092000-a9"	
KEY: 域名/url2.html
HTTP/1.1 200 OK
Server: nginx/1.12.1
Date: Sat, 26 May 2018 09:19:10 GMT
Content-Type: text/html
Content-Length: 169
Last-Modified: Sat, 26 May 2018 08:51:12 GMT
Connection: close
ETag: "5b092000-a9"
Accept-Ranges: bytes

<html>
<head>
    <meta charset="utf-8">
    <title>Cache</title>
</head>
<body style="background-color:yellow;">
    <h1>cache test<h1>
</body>
</html>
```



**让部分页面不缓存**

| Syntax                   | Default | Context                |
| ------------------------ | ------- | ---------------------- |
| proxy_no_cache string …; | --      | http, server, location |

**例子**

```bash
server {
    ...
    
    if ($request_uri ~ ^/(login|register|password\/reset)) {
		set $cookie_nocache 1;
    }
    
    location / {
    	...
    	proxy_no_cache $cookie_nocache;
    }
}
```

`login|register|password\/reset` 这些页面将不被缓存



**大文件分片请求**

| Syntax      | Default  | Context                |
| ----------- | -------- | ---------------------- |
| slice size; | slice 0; | http, server, location |

#### 动静分离

通过中间件将 `动态请求` 和 `静态请求` 分离

**例子**

```bash
http {
    ...
    upstream php_api {
        server 127.0.0.1:8000;
    }
    
    server {
        ...
        location ~ \.php$ {
            proxy_pass http://php_api;
            ...
        }
    }
}
```



#### Rewrite

实现 `url` 重写以及重定向

- URL访问跳转
- SEO优化
- 维护
- 安全（伪静态）

| Syntax                            | Default | Context              |
| --------------------------------- | ------- | -------------------- |
| rewrite regex replacement [flag]; | --      | server, location, if |

**flag**

| 取值      | 意义                                                         |
| --------- | ------------------------------------------------------------ |
| last      | 停止 `rewrite` 检测，会重新发起一次 对 `replacement` 的请求  |
| break     | 停止 `rewrite` 检测，查找服务器中是否有 `replacement` 的文件，不发起请求 |
| redirect  | 返回 `302` 临时重定向，地址栏会显示跳转后的地址              |
| permanent | 返回 `301` 永久重定向，地址栏会显示跳转后的地址              |

**优先级**

1. server
2. location



**例子**

```bash
server {
    ...
    root /opt/app/code;
    
    # break last redirect permanent 对比
    location ~ ^/break {
        rewrite ^/break /test/ break;
    }

    location ~ ^/last {
         rewrite ^/last /test/ last;
    }
    
    location ~ ^/redirect {
         rewrite ^/redirect /test/ redirect;
    }
    
    location ~ ^/permanent {
         rewrite ^/permanent /test/ permanent;
    }
    
    # 一般场景
    location / {
        rewrite ^/(\d+)-(\d+)-(\d+).html$ /$1/$2/$3.html break;
        
        if ($http_user_agent ~* Chrome) {
            rewrite ^/nginx url redirect;
        }
        
        if (!-f $request_filename) {
            rewrite ^/(.*)$ url/$1 redirect;
        }
    }
    
    location /test/ {
       default_type application/json;
       return 200 '{"status":"success"}';
    }
}
```

当访问 `/break/` 的时候会查找 `/opt/app/code/test/` 是否有对应的文件，如果没有则返回 `404`，而 `/last/` 会向 `/test/` 发起一个请求并返回响应内容

`redirect` 会有两次请求，第一次由 `/redirect/` 返回 `302 Moved Temporarily`，第二次由 `/test/` 返回 `200 OK`

`permanent` 和 `redirect` 的区别是，当 `nginx` 关掉的时候，`permanent` 仍会重定向



