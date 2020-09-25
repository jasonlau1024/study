
# Openresty


## 安装
1）安装环境依赖
`yum install openssl-devel pcre-devel gcc -y`
2）编译安装Openresty（Orange依赖项）
```shell
# 下载源码包：https://openresty.org/en/download.html
wget https://openresty.org/download/openresty-1.17.8.1.tar.gz
cd openresty-1.17.8.1
onfigure -j2 --with-http_stub_status_module --with-http_v2_module --with-http_ssl_module
gmake && gmake install
## 默认安装目录
/usr/local/openresty

## 配置环境变量
PATH=$PATH:$HOME/bin:/usr/local/openresty/bin:/usr/local/openresty/nginx/sbin
export PATH
source /etc/profile
```

3）编译安装Lor框架（Orange依赖项）
```shell
git clone https://github.com/sumory/lor.git
cd lor && make install
##默认安装目录为：/usr/local/lor
##可执行文件：/usr/local/bin/lord﻿​
```

4）编译安装luarocks（Orange依赖项）
```shell
git clone git://github.com/luarocks/luarocks.git

cd luarocks
./configure --prefix=/usr/local/openresty/luajit \
--with-lua=/usr/local/openresty/luajit/ \
--lua-suffix=jit \
--with-lua-include=/usr/local/openresty/luajit/include/luajit-2.1


```