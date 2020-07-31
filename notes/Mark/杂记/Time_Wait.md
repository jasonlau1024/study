


## 场景及影响

场景：线上高并发场景
- 一部分 TIME_WAIT 连接被回收，但新的 TIME_WAIT 连接产生；
- 一些极端情况下，会出现大量的 TIME_WAIT 连接。

影响：
- 每一个 time_wait 状态，都会占用一个 TCP 本地端口，当大量的连接处于 time_wait 时，新建立 TCP 连接会出错，address already in use : connect 异常。
*说明：*TCP 本地端口上限为 65535 ,这是因为 TCP 头部使用 16 bit，存储「端口号」，因此约束上限为 65535。

## 问题分析
大量的 TIME_WAIT 状态 TCP 连接存在，其本质原因是什么？
1.大量的短连接存在
2.特别是 HTTP 请求中，如果 connection 头部取值被设置为 close 时，基本都由「服务端」发起主动关闭连接
3.而，TCP 四次挥手关闭连接机制中，为了保证 ACK 重发和丢弃延迟数据，设置 time_wait 为 2 倍的 MSL（报文最大存活时间）
TIME_WAIT 状态：
1.TCP 连接中，主动关闭连接的一方出现的状态；（收到 FIN 命令，进入 TIME_WAIT 状态，并返回 ACK 命令）
2.保持 2 个 MSL 时间，即，4 分钟；（MSL 为 2 分钟）

## 解决问题
解决上述 time_wait 状态大量存在，导致新连接创建失败的问题，一般解决办法：
1.客户端，HTTP 请求的头部，connection 设置为 keep-alive，保持存活一段时间：现在的浏览器，一般都这么进行了
2.服务器端
允许 time_wait 状态的 socket 被重用
缩减 time_wait 时间，设置为 1 MSL（即，2 mins）

## 相关查询命令
统计各种连接的数量：
```shell
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
ESTABLISHED 34
TIME_WAIT 38734

或
ss -s
Total: 579 (kernel 1375)
TCP:   40818 (estab 426, closed 40385, orphaned 0, synrecv 0, timewait 40384/0), ports 0

Transport Total     IP        IPv6
*         1375      -         -
RAW       1         1         0
UDP       20        19        1
TCP       433       431       2
INET      454       451       3
FRAG      0         0         0


## 优化


### 调整系统内核参数

```shell

net.ipv4.tcp_syncookies = 1 表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；
net.ipv4.tcp_tw_reuse = 1 表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；
net.ipv4.tcp_tw_recycle = 1 表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。
net.ipv4.tcp_fin_timeout =  修改系统默认的 TIMEOUT 时间
net.ipv4.tcp_max_tw_buckets = 5000 表示系统同时保持TIME_WAIT套接字的最大数量，(默认是18000). 当TIME_WAIT连接数量达到给定的值时，所有的TIME_WAIT连接会被立刻清除，并打印警告信息。但这种粗暴的清理掉所有的连接，意味着有些连接并没有成功等待2MSL，就会造成通讯异常。一般不建议调整
net.ipv4.tcp_timestamps = 1(默认即为1)60s内同一源ip主机的socket connect请求中的timestamp必须是递增的。也就是说服务器打开了 tcp_tw_reccycle了，就会检查时间戳，如果对方发来的包的时间戳是乱跳的或者说时间戳是滞后的，那么服务器就会丢掉不回包，现在很多公司都用LVS做负载均衡，通常是前面一台LVS，后面多台后端服务器，这其实就是NAT，当请求到达LVS后，它修改地址数据后便转发给后端服务器，但不会修改时间戳数据，对于后端服务器来说，请求的源地址就是LVS的地址，加上端口会复用，所以从后端服务器的角度看，原本不同客户端的请求经过LVS的转发，就可能会被认为是同一个连接，加之不同客户端的时间可能不一致，所以就会出现时间戳错乱的现象，于是后面的数据包就被丢弃了，具体的表现通常是是客户端明明发送的SYN，但服务端就是不响应ACK，还可以通过下面命令来确认数据包不断被丢弃的现象，所以根据情况使用

其他优化：

net.ipv4.ip_local_port_range = 1024 65535 增加可用端口范围，让系统拥有的更多的端口来建立链接，这里有个问题需要注意，对于这个设置系统就会从1025~65535这个范围内随机分配端口来用于连接，如果我们服务的使用端口比如8080刚好在这个范围之内，在升级服务期间，可能会出现8080端口被其他随机分配的链接给占用掉，这个原因也是文章开头提到的端口被占用的另一个原因
net.ipv4.ip_local_reserved_ports = 7005,8001-8100 针对上面的问题，我们可以设置这个参数来告诉系统给我们预留哪些端口，不可以用于自动分配。
```




### 调整短链接为长链接
**短连接：**
连接->传输数据->关闭连接
HTTP是无状态的，浏览器和服务器每进行一次HTTP操作，就建立一次连接，但任务结束就中断连接。
**长连接：**
连接->传输数据->保持连接 -> 传输数据-> 。。。->关闭连接。
长连接指建立SOCKET连接后不管是否使用都保持连接，但安全性较差。
*总结：*从区别上可以看出，长连接比短连接从根本上减少了关闭连接的次数，减少了TIME_WAIT状态的产生数量，在高并发的系统中，这种方式的改动非常有效果，可以明显减少系统TIME_WAIT的数量。

在使用nginx作为反向代理场景，需要做到两点：
从client到nginx的连接是长连接
从nginx到server的连接是长连接

1、保持和client的长连接：

默认情况下，nginx已经自动开启了对client连接的keep alive支持（同时client发送的HTTP请求要求keep alive）。一般场景可以直接使用，但是对于一些比较特殊的场景，还是有必要调整个别参数（keepalive_timeout和keepalive_requests）。
```shell
http {
    keepalive_timeout  120s 120s; 
    # 第一个参数：设置keep-alive客户端连接在服务器端保持开启的超时值（默认75s）；值为0会禁用keep-alive客户端连接；
    # 第二个参数：（可选）在响应的header域中设置一个值“Keep-Alive: timeout=time”；通常可以不用设置；
    keepalive_requests 10000;
    # 设置一个keep-alive连接上可处理的客户端请求的数量，当达到该限制时，连接被关闭。默认是100。
}
```
2、保持和server的长连接
默认nginx访问后端都是用的短连接(HTTP1.0)，一个请求来了，Nginx 新开一个端口和后端建立连接，后端执行完毕后主动关闭该链接。
HTTP协议中对长连接的支持是从1.1版本之后才有的，因此最好通过proxy_http_version指令设置为"1.1"。
```shell
upstream http_backend {
 server 127.0.0.1:8080;

 keepalive 1000;//设置nginx到upstream服务器的空闲keepalive连接的最大数量
}

server {
 ...

location /http/ {
 proxy_pass http://http_backend;
 proxy_http_version 1.1;//开启长链接
 proxy_set_header Connection "";    # 清理"Connection" header,即使是client和nginx之间是短连接，nginx和upstream之间也是可以开启长连接。
 ...
 }
}
```



## 底层原理

### 三次握手与四次挥别

![img](https://ask.qcloudimg.com/http-save/yehe-1903727/9uwtloztcr.jpeg?imageView2/2/w/1620)


- TIME_WAIT 状态是在tcp断开链接时产生的（因为TCP连接是双向的，所以在关闭连接的时候，两个方向各自都需要关闭)。
- 先发FIN包的一方执行的是主动关闭；后发FIN包的一方执行的是被动关闭。
- 主动关闭的一方会进入TIME_WAIT状态，并且在此状态停留两倍的MSL时长。
  

### MSL 时间
- MSL，Maximum Segment Lifetime，报文最大生存时间。
  - 任何报文在网络上存在的最长时间，超过这个时间报文将被丢弃。（IP 报文）
  - TCP报文 （segment）是ip数据报（datagram）的数据部分。
  Tips：RFC 793中规定MSL为2分钟，实际应用中常用的是30秒，1分钟和2分钟等。

- 2MSL，TCP 的 TIME_WAIT 状态，也称为2MSL等待状态。
  - 当TCP的一端发起主动关闭（收到 FIN 请求），在发出最后一个ACK 响应后，即第3次握 手完成后，发送了第四次握手的ACK包后，就进入了TIME_WAIT状态。
  - 等待2MSL时间主要目的是怕最后一个 ACK包对方没收到，那么对方在超时后将重发第三次握手的FIN包，主动关闭端接到重发的FIN包后，可以再发一个ACK应答包。
  - 在 TIME_WAIT 状态时，两端的端口不能使用，要等到2MSL时间结束，才可继续使用。

### 扩展问题
001)time_wait 是服务器端的状态 or 客户端的状态？
time_wait 是「主动关闭 TCP 连接」一方的状态，可能是「客服端」的，也可能是「服务器端」的；
一般情况下，都是「客户端」所处的状态；「服务器端」一般设置「不主动关闭连接」。

002)服务器在对外服务时，是「客户端」发起的断开连接？还是「服务器」发起的断开连接？
正常情况下，都是「客户端」发起的断开连接；
「服务器」一般设置为「不主动关闭连接」，服务器通常执行「被动关闭」；
但 HTTP 请求中，http 头部 connection 参数，可能设置为 close，则，服务端处理完请求会主动关闭 TCP 连接，
在HTTP1.1协议中，有个 Connection 头，Connection有两个值，close和keep-alive，这个头就相当于客户端告诉服务端，服务端你执行完成请求之后，是关闭连接还是保持连接
HTTP默认的Connection值为close，但现在的浏览器发送请求的时候一般都会设置Connection为keep-alive。