

## 概述

### 简介

官方地址：<http://www.squid-cache.org/>
参考文档：<http://www.squid-cache.org/Doc/config/>

参考文章：<https://keenjin.github.io/2019/07/Squid_proxy/>

Squid是一个缓存Internet 数据的软件，其接收用户的下载申请，并自动处理所下载的数据。当一个用户想要下载一个主页时，可以向Squid 发出一个申请，要Squid 代替其进行下载，然后Squid 连接所申请网站并请求该主页，接着把该主页传给用户同时保留一个备份，当别的用户申请同样的页面时，Squid 把保存的备份立即传给用户，使用户觉得速度相当快。Squid 可以代理HTTP、FTP、GOPHER、SSL和WAIS等协议并且Squid 可以自动地进行处理，可以根据自己的需要设置Squid，使之过滤掉不想要的东西。

**工作流程：**
- 当代理服务器中有客户端需要的数据时：
  a. 客户端向代理服务器发送数据请求；
  b. 代理服务器检查自己的数据缓存；
  c. 代理服务器在缓存中找到了用户想要的数据，取出数据并返回给客户端。
- 当代理服务器中没有客户端需要的数据时：
  a. 客户端向代理服务器发送数据请求；
  b. 代理服务器检查自己的数据缓存，没有找到用户想要的数据；
  c. 代理服务器向Internet 上的远端服务器发送数据请求；
  d. 远端服务器响应，返回相应的数据；
  e. 代理服务器取得远端服务器的数据，返回给客户端，并保留一份到自己的数据缓存中。

**squid优点：**
- 开源软件，性能优秀。并仍在世界各地的squid开发者的共同努力下，不断发展。
- 快速响应，减少网络阻塞,Squid将远程Internet对象保存为本地拷贝。当本地用户再次访问这些对象时，Squid可以直接快速地提供对这些对象的访问，而不必再次占用带宽访问远程服务器上的对象。
- 增强访问控制，提高安全性。可以针对特定的的网站、用户、网络、数据类型实施访问控制。
- 可以工作在普通代理模式、透明代理模式各反向代理模式。

**代理分类：**
- 普通代理：需要客户机在浏览器中指定代理服务器的地址、端口；
- 透明代理：适用于企业的网关主机（共享接入Internet）中，客户机不需要指定代理服务器地址、端口等信息，代理服务器需要设置防火墙策略将客户机的Web访问数据转交给代理服务程序处理；
- 反向代理：是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。

### ACL访问控制
squid提供了强大的代理控制机制，通过合理设置ACL并进行限制，可以针对源地址、目标地址、访问的URL路径、访问的时间等各种条件进行过滤

ACL访问控制通过两个步骤来实现：
1. 使用ACL配置项定义需要控制的条件；
2. 通过http_access配置项对已定义的列表做“允许”或“拒绝”访问的控制。

```shell
acl 列表名称 列表类型 列表内容
... ...
http_access allow|deny 列表名
... ...
http_access deny all
```

## 部署
### 安装 squid

官方文档：https://wiki.squid-cache.org/SquidFaq/BinaryPackages#KnowledgeBase.2FCentOS.Stable_Repository_Package_.28like_epel-release.29


安装最新Squid(4.12)
```shell
yum install http://ngtech.co.il/repo/centos/7/squid-repo-1-1.el7.centos.noarch.rpm -y
yum info squid
yum install squid

```

### 用户认证
使用apache认证工具

```shell
## 认证文件
yum install httpd-tool -y
htpasswd -cb /data/proxyuser.pass testuser jS88qpsHlUPMI1pFyDOM
htpasswd -b /data/proxyuser.pass addusername password

## 访问控制，用户组及URL
mkdir /etc/squid/{group,url}
touch /etc/squid/group/accessusers
touch /etc/squid/url/accessurl
```

squid 启用认证
```shell
auth_param basic program /usr/lib/squid/ncsa_auth /etc/squid/squid_passwd   # 指定密码文件和用来验证密码的程序
auth_param basic children 20                             # 鉴权进程的数量
auth_param basic realm Squid proxy-caching web server   # 用户在web输入用户名和密码时，看到的提示信息
auth_param basic credentialsttl 5 hours                 # 认证有效期
auth_param basic casesensitive off     
```

### 访问控控制
**自定义访问策略：**
1. 使用ACL配置项定义需要控制的条件；
2. 通过http_access配置项对已定义的列表做“允许”或“拒绝”访问的控制。


```shell
# 定义URL列表
acl accessurl dstdom_regex "/etc/squid/url/accessurl"

# 定义用户组
acl accessusers proxy_auth "/etc/squid/group/accessusers"

# 定义访问控制策略
http_access allow accessusers accessurl

# 设置允许的 HTTP Method
acl get method GET POST

# 禁止访问除了上面允许的ACL的所有访问
http_access deny all

```



