## 概述

## 安装
### Yum 安装
官方文档：http://nginx.org/en/linux_packages.html#stable
```shell
cat > /etc/yum.repos.d/nginx.repo <<"EOF"
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
EOF
yum install nginx -y
```

## 配置
### 通用配置
```shell
cat /etc/nginx/nginx.conf
user  nginx nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;
worker_rlimit_nofile 51200;


events {
    use epoll;
    worker_connections 51200;
    multi_accept on;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    server_names_hash_bucket_size 512;
        client_header_buffer_size 32k;
        large_client_header_buffers 4 32k;
        client_max_body_size 50m;

        sendfile   on;
        tcp_nopush on;
        keepalive_timeout 60;
        tcp_nodelay on;

        fastcgi_connect_timeout 300;
        fastcgi_send_timeout 300;
        fastcgi_read_timeout 300;
        fastcgi_buffer_size 64k;
        fastcgi_buffers 4 64k;
        fastcgi_busy_buffers_size 128k;
        fastcgi_temp_file_write_size 256k;
        fastcgi_intercept_errors on;

        gzip on;
        gzip_min_length  1k;
        gzip_buffers     4 16k;
        gzip_http_version 1.1;
        gzip_comp_level 2;
        gzip_types     text/plain application/javascript application/x-javascript text/javascript text/css application/xml;
        gzip_vary on;
        gzip_proxied   expired no-cache no-store private auth;
        gzip_disable   "MSIE [1-6]\.";

        limit_conn_zone $binary_remote_addr zone=perip:10m;
        limit_conn_zone $server_name zone=perserver:10m;

        server_tokens off;
        access_log off;

    include /etc/nginx/conf.d/*.conf;
}
```
### 设置指定域名
设置指定域名访问，其它访问返回403
```shell
cat /etc/nginx/conf.d/default.conf
server {
  listen 80 default;
  server_name _;
#  access_log   /var/log/nginx/default-access.log;
#    error_log    /var/log/nginx/default-error.log;
  return 403;
}
```

### HTTPS配置


## 扩展

### Nginx日志

**日志分类：**
- 访问日志(access_log)
- 错误日志(error_log)

#### 设置access_log
记录客户端向Nginx服务器发起的每一次请求，包括客户端IP，浏览器信息，referer，请求处理时间，请求URL等。

1）配置启用access_log
**语法：**
```shell
access_log path [format [buffer=size] [gzip[=level]] [flush=time] [if=condition]]; # 设置访问日志
access_log off; # 关闭访问日志
####  说明  ####
path    指定日志的存放位置。
format  指定日志的格式。默认使用预定义的combined。
buffer  用来指定日志写入时的缓存大小。默认是64k。
gzip    日志写入前先进行压缩。压缩率可以指定，从1到9数值越大压缩比越高，同时压缩的速度也越慢。默认是1。
flush   设置缓存的有效时间。如果超过flush指定的时间，缓存中的内容将被清空。
if      条件判断。如果指定的条件计算为0或空字符串，那么该请求不会写入日志。
```
**作用于：**
`http` `server` `location` `limit_except`

**示例：**
基本用法：
`access_log /var/logs/nginx-access.log`

2）log_format自定义日志格式
**语法：**
```shell
log_format name [escape=default|json] string ...;

####  说明  ####
name 格式名称。在access_log指令中引用。
escape 设置变量中的字符编码方式是json还是default，默认是default。
string 要定义的日志格式内容。该参数可以有多个。参数中可以使用Nginx变量。

####  string  ####
$bytes_sent
发送给客户端的总字节数

$body_bytes_sent
发送给客户端的字节数，不包括响应头的大小

$connection
连接序列号

$connection_requests
当前通过连接发出的请求数量

$msec
日志写入时间，单位为秒，精度是毫秒

$pipe
如果请求是通过http流水线发送，则其值为"p"，否则为“."

$request_length
请求长度（包括请求行，请求头和请求体）

$request_time
请求处理时长，单位为秒，精度为毫秒，从读入客户端的第一个字节开始，直到把最后一个字符发送张客户端进行日志写入为止

$status
响应状态码

$time_iso8601
标准格式的本地时间,形如“2017-05-24T18:31:27+08:00”

$time_local
通用日志格式下的本地时间，如"24/May/2017:18:31:27 +0800"

$http_referer
请求的referer地址。

$http_user_agent
客户端浏览器信息。

$remote_addr
客户端IP

$http_x_forwarded_for
当前端有代理服务器时，设置web节点记录客户端地址的配置，此参数生效的前提是代理服务器也要进行相关的x_forwarded_for设置。

$request
完整的原始请求行，如 "GET / HTTP/1.1"

$remote_user
客户端用户名称，针对启用了用户认证的请求

$request_uri
完整的请求地址，如 "https://daojia.com/"

```
**示例：**
```shell
access_log /var/logs/nginx-access.log main
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';
####  日志记录  ####
112.195.209.90 - - [20/Feb/2018:12:12:14 +0800] 
"GET / HTTP/1.1" 200 190 "-" "Mozilla/5.0 (Linux; Android 6.0; Nexus 5 Build/MRA58N) 
AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Mobile Safari/537.36" "-"
```

#### 设置error_log
**语法：**
```shell
## 配置错误日志文件的路径和日志级别。
error_log file [level];
## Default:    
error_log logs/error.log error;
####  说明  ####
file  指定日志的写入位置。
level 指定日志的级别，默认error。级别：debug, info, notice, warn, error, crit, alert,emerg。
```
**作用域：**
`main` `http` `mail` `stream` `server` `location`

#### 日志缓存open_log_file_cache
每一条日志记录的写入都是先打开文件再写入记录，然后关闭日志文件。如果你的日志文件路径中使用了变量，如`access_log /var/logs/$host/nginx-access.log`，为提高性能，可以使用`open_log_file_cache`指令设置日志文件描述符的缓存。
**语法：**
```shell
open_log_file_cache max=N [inactive=time] [min_uses=N] [valid=time];

####  说明  ####
max       设置缓存中最多容纳的文件描述符数量，如果被占满，采用LRU算法将描述符关闭。
inactive  设置缓存存活时间，默认是10s。
min_uses  在inactive时间段内，日志文件最少使用几次，该日志文件描述符记入缓存，默认是1次。
valid     设置多久对日志文件名进行检查，看是否发生变化，默认是60s。
off       不使用缓存。默认为off。
```
**示例：**
`open_log_file_cache max=1000 inactive=20s valid=1m min_uses=2;`
设置缓存最多缓存1000个日志文件描述符，20s内如果缓存中的日志文件描述符至少被被访问2次，才不会被缓存关闭。每隔1分钟检查缓存中的文件描述符的文件名是否还存在。

**作用域：**
`http` `server` `location`

*总结：*Nginx中通过access_log和error_log指令配置访问日志和错误日志，通过log_format我们可以自定义日志格式。如果日志文件路径中使用了变量，我们可以通过open_log_file_cache指令来设置缓存，提升性能。
*补充：*更多变量查询 http://nginx.org/en/docs/varindex.html

### Nginx 负载均衡器
#### 负载均衡配置
**涉及模块：**
- `ngx_http_proxy_module`
- `ngx_http_upstream_module`

```shell
http{
  ... ...
  upstream testweb {
    server 192.168.10.24:8080 weight=1 max_fails=3 fail_timeout=20s;
    server 192.168.10.24:8081 weight=2 max_fails=3 fail_timeout=20s;
    server 192.168.10.24:8082 weight=3 max_fails=3 fail_timeout=20s;
    # 定义了一个负载均衡池，池中有三台服务器，权重分别是 1、2、3 ( 越大越高 )
    # 最大失败次数 3 次，超过 3 次失败后，20 秒内不检测
  }
  server {
    ... ...
    location / {
      proxy_pass http://testweb;
    }
  }
}
```
#### 方向代理配置
```shell
# 故障转移策略，当后端服务器返回如下错误时，自动负载到后端其余机器
proxy_next_upstream http_500 http_502 http_503 error timeout invalid_header;

# 设置后端服务器获取用户真实IP、代理者真实IP等
proxy_redirect off;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

# 用于指定客户端请求主体缓存区大小，可以理解成先保存到本地再传给用户
client_body_buffer_size 128k;

# 表示与后端服务器连接的超时时间，即发起握手等侯响应的超时时间
proxy_connect_timeout 90;

# 表示后端服务器的数据回传时间，即在规定时间之后端服务器必须传完所有的数据，否则 Nginx 将断开这个连接
proxy_send_timeout 90;

# 设置 Nginx 从代理的后端服务器获取信息的时间，表示连接建立成功后，Nginx 等待后端服务器的响应时间，其实是 Nginx 已经进入后端的排队中等候处理的时间
proxy_read_timeout 90;

# 设置缓冲区大小，默认该缓冲区大小等于指令 proxy_buffers 设置的大小
proxy_buffer_size 4k;

# 设置缓冲区的数量和大小。Nginx 从代理的后端服务器获取的响应信息，会放置到缓冲区
proxy_buffers 4 32k;

# 用于设置系统很忙时可以使用的 proxy_buffers 大小，官方推荐大小为 proxu_buffers 的两倍
proxy_busy_buffers_size 64k;

# 指定 proxy 缓存临时文件的大小
proxy_temp_file_write_size 64k;
```

#### 后端服务健康检查
**涉及模块：**
- `nginx_upstream_check_module`
模块地址：https://github.com/yaoweibin/nginx_upstream_check_module

nginx自带后端健康检查机制：
当后端某一服务器无法提供服务时，该链接先被转发到这台机器，然后发现该机故障，而后才转发到其它机器，导致资源浪费。

引入第三方模块，用于提供负载均衡器内节点的健康检查，通过它可以用来检测后端服务的健康状态。如果后端服务不可用，则后面的请求就不会转发到该节点上。

**添加扩展模块：**
```shell
git clone https://github.com/yaoweibin/nginx_upstream_check_module.git
yum -y install patch
cd /usr/local/src/nginx-1.12.2; patch -p1 < /usr/local/src/nginx_upstream_check_module/check_1.12.1+.patch
# 切换到 Nginx 源码目录（注意版本匹配）,加上原来的编译参数
./configure --prefix=/usr/local/nginx-1.12.2 --add-module=/usr/local/src/nginx_upstream_check_module ... ...
make && make install
```

**check配置：**
```shell
upstream testweb {
        server 192.168.10.24:8080;
        server 192.168.10.24:8081;
        server 192.168.10.24:8082;

        check interval=3000 rise=2 fall=5 timeout=1000 type=http;
        # interval    检测间隔时间
        # rise        连续成功2次标记为UP
        # fall        连续失败5次标记为DOWN
        # timeout     超时时间1000毫秒
        # type        检测类型HTTP
}

server {
  ... ...
  location /status {
    check_status;
    access_log off;
    allow 127.0.0.1;
    deny all;
  }
}

## 获取readserver状态信息
curl localhost/status?format=jason

```







