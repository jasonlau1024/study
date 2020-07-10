

## 安装
### Yum 安装
php7.1 php7.2
```shell
## Centos 7.x
rpm -Uvh https://mirror.webtatic.com/yum/el7/epel-release.rpm
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm

## 安装PHP以及扩展
## php 7.1
yum install php71w-cli php71w-mysqlnd php71w-bcmath php71w-gd php71w-mcrypt php71w-mbstring php71w-soap php71w-xml php71w-fpm php71w-devel php71w-xmlrpc php71w-process php71w-pecl-redis
## php 7.2
yum install php72w-fpm php72w-xml php72w-soap php72w-mbstring php72w-bcmath php72w-mysqlnd php72w-pdo php72w-pecl-redis php72w-gd -y
```
PHP7.3
```SHELL
rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum install -y php73-php-fpm php73-php-cli php73-php-bcmath php73-php-gd php73-php-json php73-php-mbstring php73-php-mcrypt php73-php-mysqlnd php73-php-opcache php73-php-pdo php73-php-pecl-crypto php73-php-pecl-mcrypt php73-php-pecl-geoip php73-php-recode php73-php-snmp php73-php-soap php73-php-xml
```

### PHP 通用配置
php.ini
```shell
[PHP]
engine = On
short_open_tag = Off
precision = 14
output_buffering = 4096
zlib.output_compression = Off
implicit_flush = Off
unserialize_callback_func =
serialize_precision = 17
disable_functions = passthru,exec,system,putenv,chroot,chgrp,chown,shell_exec,popen,proc_open,pcntl_exec,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru,pcntl_alarm,pcntl_fork,pcntl_waitpid,pcntl_wait,pcntl_wifexited,pcntl_wifstopped,pcntl_wifsignaled,pcntl_wifcontinued,pcntl_wexitstatus,pcntl_wtermsig,pcntl_wstopsig,pcntl_signal,pcntl_signal_dispatch,pcntl_get_last_error,pcntl_strerror,pcntl_sigprocmask,pcntl_sigwaitinfo,pcntl_sigtimedwait,pcntl_exec,pcntl_getpriority,pcntl_setpriority,imap_open,apache_setenv
disable_classes =
zend.enable_gc = On
expose_php = Off
max_execution_time = 30
max_input_time = 60
memory_limit = 256M
error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT
display_errors = Off
display_startup_errors = Off
log_errors = On
log_errors_max_len = 1024
ignore_repeated_errors = Off
ignore_repeated_source = Off
report_memleaks = On
track_errors = Off
html_errors = On
error_log = /var/log/php-fpm/php_errors.log
variables_order = "GPCS"
request_order = "GP"
register_argc_argv = Off
auto_globals_jit = On
post_max_size = 200K
auto_prepend_file =
auto_append_file =
default_mimetype = "text/html"
default_charset = "UTF-8"
doc_root =
user_dir =
enable_dl = Off
file_uploads = On
upload_max_filesize = 2M
max_file_uploads = 20
allow_url_fopen = Off
allow_url_include = Off
default_socket_timeout = 60
[CLI Server]
cli_server.color = On
[Date]
[filter]
[iconv]
[intl]
[sqlite]
[sqlite3]
[Pcre]
[Pdo]
[Pdo_mysql]
pdo_mysql.cache_size = 2000
pdo_mysql.default_socket=
[Phar]
[mail function]
sendmail_path = /usr/sbin/sendmail -t -i
mail.add_x_header = On
[SQL]
sql.safe_mode = Off
[ODBC]
odbc.allow_persistent = On
odbc.check_persistent = On
odbc.max_persistent = -1
odbc.max_links = -1
odbc.defaultlrl = 4096
odbc.defaultbinmode = 1
[Interbase]
ibase.allow_persistent = 1
ibase.max_persistent = -1
ibase.max_links = -1
ibase.timestampformat = "%Y-%m-%d %H:%M:%S"
ibase.dateformat = "%Y-%m-%d"
ibase.timeformat = "%H:%M:%S"
[MySQLi]
mysqli.max_persistent = -1
mysqli.allow_persistent = On
mysqli.max_links = -1
mysqli.cache_size = 2000
mysqli.default_port = 3306
mysqli.default_socket =
mysqli.default_host =
mysqli.default_user =
mysqli.default_pw =
mysqli.reconnect = Off
[mysqlnd]
mysqlnd.collect_statistics = On
mysqlnd.collect_memory_statistics = Off
[OCI8]
[PostgreSQL]
pgsql.allow_persistent = On
pgsql.auto_reset_persistent = Off
pgsql.max_persistent = -1
pgsql.max_links = -1
pgsql.ignore_notice = 0
pgsql.log_notice = 0
[bcmath]
bcmath.scale = 0
[browscap]
[Session]
session.save_handler = files
session.use_strict_mode = 0
session.use_cookies = 1
session.use_only_cookies = 1
session.name = PHPSESSID
session.auto_start = 0
session.cookie_lifetime = 0
session.cookie_path = /
session.cookie_domain =
session.cookie_httponly =
session.serialize_handler = php
session.gc_probability = 1
session.gc_divisor = 1000
session.gc_maxlifetime = 1440
session.referer_check =
session.cache_limiter = nocache
session.cache_expire = 180
session.use_trans_sid = 0
session.hash_function = 0
session.hash_bits_per_character = 5
url_rewriter.tags = "a=href,area=href,frame=src,input=src,form=fakeentry"
[Assertion]
zend.assertions = -1
[mbstring]
[gd]
[exif]
[Tidy]
tidy.clean_output = Off
[soap]
soap.wsdl_cache_enabled=1
soap.wsdl_cache_dir="/tmp"
soap.wsdl_cache_ttl=86400
soap.wsdl_cache_limit = 5
[sysvshm]
[ldap]
ldap.max_links = -1
[mcrypt]
[dba]
[curl]
[openssl]
```
php-fpm.d/www.conf
```shell
[www]
user = nginx
group = nginx
listen = 127.0.0.1:9000
listen.allowed_clients = 127.0.0.1
;pm = dynamic
pm = static
pm.max_children = 100
pm.start_servers = 20
pm.min_spare_servers = 20
pm.max_spare_servers = 60
pm.status_path = /phpfpm_status
slowlog = /var/log/php-fpm/www-slow.log
request_slowlog_timeout = 5
php_admin_value[error_log] = /var/log/php-fpm/www-error.log
php_admin_flag[log_errors] = on
php_value[session.save_handler] = files
php_value[session.save_path]    = /var/lib/php/session
php_value[soap.wsdl_cache_dir]  = /var/lib/php/wsdlcache
```

## 维护
### 添加扩展
查看当前包仓库支持的php扩展版本，如：`yum search php |grep -i soap`
#### swoole
Swoole是一个面向生产环境的 [PHP](https://baike.baidu.com/item/PHP/9337) 异步网络通信引擎，使 PHP 开发人员可以编写高性能的异步并发 TCP、UDP、Unix Socket、HTTP，WebSocket 服务。
```shell
## 在 yum 安装中，通过php-peal来安装swoole，本质是编译swoole，所以需要gcc-c++环境
yum install php71w-pear gcc-c++ glibc-headers
pecl install swoole
## php.ini 添加 swoole.so
extension=swoole.so
```
### 修改文件上传大小限制
```shell
## nginx
client_max_body_size 50M;
## php.ini:
max_execution_time = 600  ## 每个PHP页面运行的最大时间值(秒)，默认30秒
max_input_time = 600  ## 每个PHP页面接收数据所需的最大时间，默认60秒
memory_limit = 128M    ## 每个PHP页面所吃掉的最大内存，默认8M
file_uploads = On  ## 是否允许通过HTTP上传文。默认为ON
upload_max_filesize = 50M  ## 允许上传文件大小的最大值.
post_max_size = 50M  ## 通过表单POST给PHP的所能接收的最大值.
```
### php-fpm 的超时设置
默认配置：
php-fpm: `request_terminate_timeout = 0`
php.ini: `max_execution_time = 30`
说明：
`request_terminate_timeout` ：由于某种原因当`max_execution_time`无法终止脚本的时，超过这个时间会把这个php-fpm请求干掉。
`max_execution_time`：设置脚本被解析器中止之前允许的最大执行时间。
注：`max_execution_time`指的是脚本的执行时间；
`request_terminate_timeout`指的是从http请求开始计算，包括等待响应等。
### 文件权限管理
- php-fpm配置用户跟nignx保持一致。
- 所有文件属主属组跟php-fpm和nginx定义的用户保持一致。
- 目录权限755，文件权限644
  目录：`find /var/www/html/ -type d -exec chmod 755 {} \;`
  文件：`find /var/www/html/ -type f -exec chmod 644 {} \;`

### 安全优化
#### 控制脚本访问权限
PHP 默认配置允许 PHP 脚本程序访问服务器上的任意文件，为避免 PHP 脚本访问不该访问的文件，从一定程度上限制了 PHP 木马的危害，需设置 PHP 只能访问网站目录或者其他必须可访问的目录。
配置文件：`php.ini` （多个目录通过冒号隔开）
`open_basedir=/var/www/:/var/lib/php/:/tmp/`

#### 隐藏 PHP 版本信息
攻击者在信息收集时候无法判断程序版本，增加防御系数。
配置文件：`php.ini` 
`expose_php =off`

#### 禁用 PHP 危险函数
Web 木马程序通常利用 PHP 的特殊函数执行系统命令，查询任意目录文件，增加修改删除文件等。PHP 木马程序常使用的函数为： dl, eval, assert, exec, popen, system, passthru, shell_exec 等。
配置文件：`php.ini` 
`disable_functions= dl,eval,assert,exec,passthru,popen,proc_open,shell_exec,system,phpinfo,assert`

