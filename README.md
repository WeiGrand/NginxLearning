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

重启 `nginx`

```bash
systemctl reload nginx.service
```

或者

```bash
systemctl restart nginx.service
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

**如果想要在 `access.log` 中记录访问的 `Cookie`**

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

