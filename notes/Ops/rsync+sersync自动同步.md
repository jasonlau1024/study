# 简介说明

为什么选择rsync+sersync架构：

**1、sersync是基于inotify开发的，类似于inotify-tools的工具**

**2、sersync可以记录下被监听目录中发生变化的（包括增加、删除、修改）具体某一个文件或者某一个目录的名字，然后使用rsync同步的时候，只同步发生变化的文件或者目录** 



**rsync+inotify-tools与rsync+sersync架构的区别？**

1、rsync+inotify-tools

 a、inotify只能记录下被监听的目录发生了变化（增，删，改）并没有把具体是哪个文件或者哪个目录发生了变化记录下来；

 b、rsync在同步的时候，并不知道具体是哪个文件或目录发生了变化，每次都是对整个目录进行同步，当数据量很大时，整个目录同步非常耗时（rsync要对整个目录遍历查找对比文件），因此效率很低    

2、rsync+sersync

 a、sersync可以记录被监听目录中发生变化的（增，删，改）具体某个文件或目录的名字；

 b、rsync在同步时，只同步发生变化的文件或目录（每次发生变化的数据相对整个同步目录数据来说很小，rsync在遍历查找对比文件时，速度很快），因此效率很高。 


同步过程：

1.  在同步服务器上开启sersync服务，sersync负责监控配置路径中的文件系统事件变化；

2.  调用rsync命令把更新的文件同步到目标服务器；

3.  需要在主服务器配置sersync，在同步目标服务器配置rsync server（注意：是rsync服务）

 同步过程和原理：

1.  用户实时的往sersync服务器上写入更新文件数据；

2.  此时需要在同步主服务器上配置sersync服务；

3.  在另一台服务器开启rsync守护进程服务，以同步拉取来自sersync服务器上的数据；

**通过rsync的守护进程服务后可以发现，实际上sersync就是监控本地的数据写入或更新事件；然后，在调用rsync客户端的命令，将写入或更新事件对应的文件通过rsync推送到目标服务器**



# 实时同步实践

**Sersync服务器（数据源【部署的项目，代码等】,源机器 ）：192.168.1.151**

**Rsync服务器（备份端磁盘大专门用来存储数据 ,目标机器）：192.168.1.145**



## Sersync服务端

```shell
https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/sersync/sersync2.5.4_64bit_binary_stable_final.tar.gz
mv GNU-Linux-x86 sersync
cd sersync
touch /etc/rsync.pas  ## 创建密码文件，里面需要密码，给600权限
## 在开启实时监控的之前对主服务器目录与远程目标机目录进行一次整体同步（除xml指定过滤的文件或目录）
./sersync2 -r
## 后台实时同步
./sersync2 -d
```

***其它用法补充：***

1. **指定配置文件**：`./sersync -o XXXX.xml`

2. **指定线程池的线程总数**（默认10）：`./sersync2 -n num`

3. **不进行同步，只运行插件**：`./sersync2 -m pluginName`

4. **多个参数可以配合使用**：`./sersync -n 8 -o abc.xml -r -d`

   表示，设置线程池工作线程为8个，指定abc.xml作为配置文件，在实时监控前作一次整体同步，以守护进程方式在后台运行。

***配置文件详解：***

```shell
<?xml version="1.0" encoding="ISO-8859-1"?>
<head version="2.5">
    <host hostip="localhost" port="8008"></host>
    ## 插件的保留字段，没有任何作用，默认即可。
    <debug start="false"/>
    ## 设置为true，开启debug模式，会在sersync正在运行的控制台，打印inotify事件与rsync同步命令。
    <fileSystem xfs="false"/>
    ## 对于xfs文件系统的用户，需要将这个选项开启，才能使sersync正常工作
    <filter start="false">
    ## 文件过滤功能（默认关闭）
    ## sersync监控的文件，会默认过滤系统的临时文件(以“.”开头，以“～”结尾)
    ## 其它的可自定义过滤
        <exclude expression="(.*)\.svn"></exclude>
        <exclude expression="(.*)\.gz"></exclude>
        ## 过滤以”.gz”结尾的文件
        <exclude expression="^info/*"></exclude>
        ## 过滤监控目录下的info目录
        <exclude expression="^static/*"></exclude>
    </filter>
    <inotify>
    ## inotify监控参数设定（优化）
        <delete start="true"/>
        ## 默认情况下对创建文件（目录）事件与删除文件（目录）事件都进行监控，如果项目中不需要删除远程目标服务器的文件（目录），则可以将delete 参数设置为false，则不对删除事件进行监控。
        <createFolder start="true"/>
        ## createFolder保持为true，如果将createFolder设为false，则不会对产生的目录进行监控，该目录下的子文件与子目录也不会被监控。
        <createFile start="false"/>
        ## 可以尝试把createFile（监控文件事件选项）设置为false来提高性能，减少 rsync通讯。因为拷贝文件到监控目录会产生create事件与close_write事件，所以如果关闭create事件，只监控文件拷贝结束时的事 件close_write，同样可以实现文件完整同步。
        <closeWrite start="true"/>
        <moveFrom start="true"/>
        <moveTo start="true"/>
        <attrib start="false"/>
        <modify start="false"/>
    </inotify>

    <sersync>
        <localpath watch="/opt/tongbu">
        ## 文件监控与远程同步设置
        ## 本地监听目录
            <remote ip="127.0.0.1" name="tongbu1"/>
            ## 远程服务器ip地址和模块
            <!--<remote ip="192.168.8.39" name="tongbu"/>-->
            <!--<remote ip="192.168.8.40" name="tongbu"/>-->
        </localpath>
        <rsync>
        ## Rsync参数配置
            <commonParams params="-artuz"/>
            ## 用户自定义rsync参数，默认是-artuz
            <auth start="false" users="root" passwordfile="/etc/rsync.pas"/>
            ## 设置为true的时候，使用rsync的认证模式传送，需要配置user与passwrodfile(–password-file=/etc/rsync.pas)
            <userDefinedPort start="false" port="874"/><!-- port=874 -->
            ## 远程同步目标服务器的rsync端口，默认874
            <timeout start="false" time="100"/><!-- timeout=100 -->
            ## 设置rsync的timeout时间
            <ssh start="false"/>
            ## 使用rsync -e ssh的方式进行传输。
        </rsync>
        <failLog path="/tmp/rsync_fail_log.sh" timeToExecute="60"/><!--default every 60mins execute once-->
        ## 失败日志脚步配置
        ## 对于失败的传输，会进行重新传送，再次失败就会写入rsync_fail_log，然后每隔一段时间（timeToExecute进行设置）执行该脚本再次重新传送，然后清空该脚本。可以通过path来设置日志路径。
        <crontab start="false" schedule="600"><!--600mins-->
        ## Crontab定期整体同步功能
        ## 可以对监控路径与远程目标主机每隔一段时间进行一次整体同步，可能由于一些原因两次失败重传都失败了，这个时候如果开启了crontab功 能，还可以进一步保证各个服务器文件一致，如果文件量比较大，crontab的时间间隔要设的大一些，否则可能增加通讯开销。
            <crontabfilter start="false">
                <exclude expression="*.php"></exclude>
                <exclude expression="info/*"></exclude>
            </crontabfilter>
        </crontab>
        <plugin start="false" name="command"/>
        ## 插件设置
        ## 当设置为true的时候，将文件同步到远程服务器后会调用name参数指定的插件
        ## 目前支持的有command refreshCDN socket http四种插件。（http插件目前由于兼容性原因暂时去除）
        ## 单独运行插件：./sersync -d -m refreshCDN
    </sersync>

    <plugin name="command">
    ## 当文件同步完成后，会调用command插件执行自定义命令
        <param prefix="/bin/sh" suffix="" ignoreError="true"/>  <!--prefix /opt/tongbu/mmm.sh suffix-->
        <filter start="false">
            <include expression="(.*)\.php"/>
            <include expression="(.*)\.sh"/>
        </filter>
    </plugin>

    <plugin name="socket">
    ## 向指定ip与端口发送inotify所产生的文件路径信息。
        <localpath watch="/opt/tongbu">
            <deshost ip="192.168.138.20" port="8009"/>
        </localpath>
    </plugin>
    <plugin name="refreshCDN">
    ## 在同步过程中将文件发送到目的服务器后刷新cdn接口。
        <localpath watch="/data0/htdocs/cms.xoyo.com/site/">
            <cdninfo domainname="ccms.chinacache.com" port="80" username="xxxx" passwd="xxxx"/>
            <sendurl base="http://pic.xoyo.com/cms"/>
            <regexurl regex="false" match="cms.xoyo.com/site([/a-zA-Z0-9]*).xoyo.com/images"/>
        </localpath>
    </plugin>
</head>

```

***开启自动运行sersync：***

```shell
vi /etc/rc.d/rc.local  #编辑，在最后添加一行
/usr/local/sersync/sersync2 -d -r -o  /usr/local/sersync/confxml.xml 
```

***监控sersync进程：***

```shell
# vi  /usr/local/sersync/check_sersync.sh  #编辑，添加以下代码
#!/bin/sh
sersync="/usr/local/sersync/sersync2"
confxml="/usr/local/sersync/confxml.xml"
status=$(ps aux |grep 'sersync2'|grep -v 'grep'|wc -l)
if [ $status -eq 0 ];
then
$sersync -d -r -o $confxml &
else
exit 0;
fi
# chmod +x /usr/local/sersync/check_sersync.sh  #添加脚本执行权限  
```







## Rsync同步端

```shell
yum -y install rsync
## 创建rsync运行用户
useradd -s /no/login -M rsync
## 创建远程同步验证用户
vi /etc/rsyncd.password
rsyncbak:password
chmod 600 /etc/rsyncd.password  ## 一定要600权限，不然认证失败

systemctl start rsyncd
systemctl enable rsyncd



systemctl restart rsyncd

```



rsync配置详解：

```shell
vi /etc/rsyncd.conf

uid=git
gid=git
## 设置rsync运行权限为rsync
max connections=600
## 最大连接数
timeout = 300
## 设置超时时间
use chroot=no
## 默认为true，修改为no，增加对目录文件软连接的备份
log file=/var/log/rsyncd.log
pid file=/var/run/rsyncd.pid
lock file=/var/run/rsyncd.lock
## 相关文件存放路径
path=/opt/tongbu1
## 指定的同步目标目录
comment  = xoyo video files
read only = false
## 设置rsync服务端文件为读写权限
list = false 
## 不显示rsync服务端资源列表
hosts allow =  192.168.0.100/24
hosts deny = *
auth users = rsyncbak
## 执行数据同步的用户名，可以设置多个，用英文状态下逗号隔开
secrets file = /etc/rsync.password
## 用户认证配置文件，里面保存用户名称和密码，后面会创建这个文件

[Website]
## 同步模块名
path = /backup  
## rsync服务端数据目录路径
```







补充：rsync常用参数

```shell
常用的命令 –avz
-a, --archive archive mode 权限保存模式,相当于 -rlptgoD 参数，存档，递归，保持属性等   
-z, --compress 压缩模式, 当资料在传送到目的端进行档案压缩
--exclude=filname，需要过滤的文件
--delete， 删除那些目标位置有的文件而备份源没有的文件
--password-file=FILE ，从 FILE 中得到密码
```



补充：rsync定时备份

`rsync -aqzrtopg rsync://rsyncuser@IP/backup /home/gougou/backup --password-file=/etc/rsync.passwd`