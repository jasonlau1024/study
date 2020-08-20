


# 概述

## 网络模型

### OSI七层模型
OSI（Open System Interconnection）开放系统互连参考模型是国际标准化组织（ISO）制定的一个用于计算机或通信系统间互联的标准体系。
- 应用层
- 表示层
- 会话层
- 运输层
- 网络层
- 链路层
- 物理层

### TCP/IP四层模型

- 应用层（HTTP/SMTP/FTP/DNS等）
  向用户提供的服务。
- 传输层（TCP/UDP）
  网络连接中的两台计算机之间的数据传输方式
- 网络层
  处理在网络上流动的数据包。
- 数据链路层
  用来处理连接网络的硬件部分。

## HTTP相关组件
### 浏览器
浏览器，Web Browser，检索、查看互联网上网页资源的应用程序。

内核的分类
浏览器的内核的种类很多，常见的浏览器内核可以分为四种：Trident、Gecko、Blink、Webkit。

（1）Trident (IE内核)
国内很多的双核浏览器的其中一核便是 Trident，美其名曰 "兼容模式"。
代表： IE、傲游、世界之窗浏览器、Avant、腾讯TT、猎豹安全浏览器、360极速浏览器、百度浏览器等。
Window10 发布后，IE 将其内置浏览器命名为 Edge，Edge 最显著的特点就是新内核 Edgehtml。

（2）Gecko(firefox)
 Mozilla FireFox(火狐浏览器) 采用该内核，Gecko 的特点是代码完全公开，因此，其可开发程度很高，全世界的程序员都可以为其编写代码，增加功能。 可惜这几年已经没落了， 比如 打开速度慢、升级频繁。

（3）webkit(Safari)
Safari 是苹果公司开发的浏览器，所用浏览器内核的名称是大名鼎鼎的 WebKit。
代表浏览器：傲游浏览器3、 Apple Safari (Win/Mac/iPhone/iPad)、Symbian手机浏览器、Android 默认浏览器。

（4）Chromium/Bink(chrome)
在 Chromium 项目中研发 Blink 渲染引擎（即浏览器核心），内置于 Chrome 浏览器之中。Blink 其实是 WebKit 的分支。
大部分国产浏览器最新版都采用Blink内核。

（5）Presto (Opera)
Presto 是挪威产浏览器 opera 的 "前任" 内核，最新的 opera 浏览器早已将之抛弃从而投入到了谷歌怀抱了


移动端的浏览器
移动端的浏览器内核主要说的是系统内置浏览器的内核。

目前移动设备浏览器上常用的内核有 Webkit，Blink，Trident，Gecko 等，其中 iPhone 和 iPad 等苹果 iOS 平台主要是 WebKit，Android 4.4 之前的 Android 系统浏览器内核是 WebKit，Android4.4 系统浏览器切换到了Chromium，内核是 Webkit 的分支 Blink，Windows Phone 8 系统浏览器内核是 Trident。

### CDN
CDN,Content Delivery Network，即内容分发网络，它应用了HTTP协议里的缓存和代理技术，代替源站响应客户端的请求。
CDN 是构建在现有网络基础之上的网络，它依靠部署在各地的边缘服务器，通过中心平台的负载均衡、内容分发、调度等功能模块，使用户就近获取所需内容，降低网络拥塞，提供用户访问响应速度和命中率。
CND 的关键技术主要有 内容存储和分发技术。

### WAF
WAF,Web Application Firewall，即 Web应用防火墙，通过检测HTTP流量和执行对应的安全策略，通过Web安全防护，如：阻止SQL注入、跨站脚本等攻击。
目前应用最多的开源项目是 ModSecurity，可集成到 Apache 或 Nginx。

### HTML
HTML 超文本标记语言，通过统一的标签格式，可将分散的 Internet 资源连接为一个逻辑的整体。

Web 页面的构成：
由一个个对象组成，通常是一个 HTML 基本文件 以及其它多个引用对象。

## HTTP 相关协议

### TCP/IP
TCP/IP 这里指的是 TCP/IP 协议簇，是一系列网络通信协议的统称。
TCP协议：Transmission Control Protocol，传输控制协议。
IP协议：Internet Protocol，主要解决通信双方寻址问题。

### DNS
DNS,Domain Name System，域名系统，是将域名和IP相互映射的分布式数据库。

### URI/URL
URI,Uniform Resource Identifier，统一资源标识符，唯一标识互联网上的资源。
URL,Uniform Resource Locator，统一资源定位符
URN,统一资源名称

### HTTPS
HTTP是明文传输的，容易被窃取信息，HTTPS是在HTTP的基础上增加了SSL层。

## HTTP请求
### HTTP请求响应过程
简单说明：
1. DNS服务器域名解析到服务器地址。
2. 客户端向服务器发起一个TCP连接。
3. 客户端向服务器发送HTTP请求报文。
4. 服务器封装请求资源到HTTP响应报文并发送通知TCP断开连接。
5. 客户端接收响应报文，关闭TCP连接。

### HTTP请求特征
- C/S模式
- 简单快速：客户端向服务器请求服务时，只需传送请求方法和路径。
- 灵活：HTTP允许传输任意类型的数据对象，由Content-Type标记。
- 无连接：每次连接只处理一个请求。节省传输时间。
- 无状态：HTTP协议是无状态协议。



## 详解HTTP报文
HTTP协议组成：
- Header(头部/请求头/响应头)
  - 起始行(Start Line)：描述请求或响应的基本信息。
    `GET /test.html HTTP/1.1  # 请求方法 URL 版本号`
  - 头部字段(Header)：使用 Key-Vaule 形式更详细地说明报文。


- body(实体)：实际传输的数据。
  - 消息正文(Entity)

### HTTP请求方法

最常用的是GET和POST方法，其它了解即可。

| 方法    | 说明                   | 指的的HTTP版本 |
| ------- | ---------------------- | -------------- |
| GET     | 获取资源               | 1.0/1.1        |
| POST    | 传输实体实体           | 1.0/1.1        |
| PUT     | 传输文件               | 1.0/1.1        |
| HEAD    | 获得报文首部           | 1.0/1.1        |
| DELETE  | 删除文件               | 1.0/1.1        |
| OPTIONS | 询问支持的方法         | 1.1            |
| TRACE   | 追踪路径               | 1.1            |
| CONNECT | 要求用隧道协议连接代理 | 1.1            |
| LINK    | 建立和资源之间的联系   | 1.0            |
| UNLINE  | 断开连接关系           | 1.0            |


### HTTP请求URL
示例：`http://www.example.com:80/path/to/myfile.html?key1=value1&key2=value2#SomewhereInTheDocument`
- `http://` 指定Protocol(协议) http
- `www.example.com` 主机
- `:80` 端口
- `/path/to/myfile.html` 路径（Path to file）
- `?key1=value1&key2=value2` 查询参数
- `#SomewhereInTheDocument` 片段标识，锚点，代表资源内的一种"标签"

### HTTP 版本说明
**HTTP 1.0:**（1996）

- 提供最基本的认证（用户名密码未经加密）。
- 短连接，每次发送数据都需TCP的三次握手和四次挥手。
- 只使用Header中的 `if-Modified-Since`和`Expires`作为缓存失效的标准。
- 不支持断点续传，每次都需传送全部的页面和数据。
- 默认每台计算机智能绑定一个IP，请求没有传递Hostname。

**HTTP 1.1:**（1999）
- 使用摘要算法来进行身份验证。
- 默认使用长连接，一次建立连接可以传输多次数据，连接时长可通过请求头的`keep-alive`设置。
- 新增加了`E-tag,If-Unmodified-Since,If-Match,If-None-Match`等缓存控制标头来控制缓存失效。
- 支持断点续传，通过使用请求头中的`Range`来实现。
- 使用了虚拟网络，一个IP可对应多个虚拟主机(Multi-homed Web Servers)。

**HTTP 2.0:**（2015）
- 头部压缩，HTTP1.x的header带有大量信息，而且每次都要重复发送，HTTP2.0使用encoder来减少需要传输的header大小，通讯双方各自cache一份header fields表，既避免了重复header的传输，又减小了需要传输的大小。
- 二进制格式，使用更加靠近 TCP/IP 的二进制格式，抛弃ASCII码，提高解析效率。
- 强化安全，HTTP2 一般运行在HTTPS上
- 多路复用，连接共享，一个连接可并发处理多个请求。

## HTTP 标头
HTTP 1.1 的标头主要分为四种，**通用标头**，**实体标头**，**请求标头**，**响应标头**
### 通用标头

- Cache-Control(Cache-Control: vaule)
  管理 HTTP 请求/响应 如何使用缓存。
  - `no-cache`：每个请求都经缓存服务器到达服务器，如果有新资源从服务器返回，没有则从缓存服务器返回。
  - `no-store`：真正意义上的不缓存，返回服务器最新的资源。

### 请求标头

### 响应标头


### 实体标头