
# Linux Base

## 系统常用基本配置
### 修改时区
```shell
# 查看当前时区
ls -l /etc/localtime
# 修改为上海时区
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
### 时间同步
```shell
yum install ntpdate -y
ntpdate time.windows.com
echo '*/5 * * * * ntpdate time.windows.com' >> 
```

### 测试机免密配置
```shell
#!/bin/bash

set -e

PubKey='xx'

yum install ntpdate -y
echo '*/5 * * * * ntpdate time.windows.com' /var/spool/cron/root

useradd centos
usermod centos -G wheel
sed -i 's/^\(%wheel\s\+ALL=(ALL)\s\+ALL\)/# \1/' /etc/sudoers
sed -i 's/^#\s*\(%wheel\s\+ALL=(ALL)\s\+NOPASSWD:\s\+ALL\)/\1/' /etc/sudoers

mkdir /home/centos/.ssh
chmod 700 /home/centos/.ssh
echo ${PubKey} >> /home/centos/.ssh/authorized_keys
chmod 600 /home/centos/.ssh/authorized_keys
chown -R centos:centos /home/centos/.ssh

# ssh 远程执行本地脚本
ssh -o StrictHostKeyChecking=no root@192.168.36.202 bash -s < /home/centos/scripts/remote_init.sh

# 命令补全
yum -y install bash-completion
```


## Centos7
### 文件系统结构

![img](https://images.cnblogs.com/cnblogs_com/jcsu/linux/linux_dir_structure.PNG)


| 目录   | 简介               | 说明                                                         |
| ------ | ------------------ | ------------------------------------------------------------ |
| /      | 根目录             | 只有root用户具有该目录的写权限                               |
| /bin   | 用户二进制文件     | 包含二进制可执行文件，用户执行命令                           |
| /sbin  | 系统二进制文件     | 系统维护命令，系统管理员                                     |
| /etc   | 配置文件           | 包含所有程序配置文件启动脚本                                 |
| /dev   | 设备文件           | 终端、USB等连接到系统的任何设备                              |
| /proc  | 进程信息           | 包含系统进程的相关信息，虚拟文件系统                         |
| /var   | 变量文件           | 增长文件，log,lib(库文件),mail,spool(队列),lock(锁文件),tmp(重启临时文件) |
| /tmp   | 临时文件           | 系统和用户创建的临时文件，重启会删除该文件                   |
| /usr   | 用户程序           | 二进制文件(bin、sbin)、库文件(lib)、文档、程序源码(local)    |
| /home  | HOME目录           | 用户存储个人档案                                             |
| /boot  | 引导加载程序文件   | 包含引导加载程序相关的文件,如内核的initrd、vmlinux、grub文件等 |
| /lib   | 系统库             | 包含支持位于/bin和/sbin下的二进制文件的库文件，库文件名为 ld*或lib*.so.* |
| /opt   | 可选的附件应用程序 | 包含从个别厂商的附加应用程序                                 |
| /mnt   | 挂载目录           | 临时安装目录，系统管理员可以挂载文件系统                     |
| /media | 可移动媒体设备     | 用于挂载可移动设备的临时目录                                 |
| /srv   | 服务数据           | 包含服务器特定服务相关的数据                                 |



# Tools
## Network Manage
### Firewalld

#### 使用实例
**端口控制：**
可以通过两种方式控制端口的开放，一种是指定端口号另一种是指定服务名。
```shell
## 指定服务名
firewall-cmd --add-service=mysql 	# 开放mysql端口
firewall-cmd --remove-service=http  # 阻止http端口
firewall-cmd --list-services 		# 查看开放的服务
## 指定端口号
firewall-cmd --add-port=3306/tcp 	# 开放通过tcp访问3306
firewall-cmd --remove-port=80tcp 	# 阻止通过tcp访问3306
firewall-cmd --list-ports 			# 查看开放的端口
```
*示例：*
- 允许指定IP访问指定端口
  `firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="x.x.x.x" port protocol="tcp" port="8080" accept"`


**端口转发与 IP 伪装：**
端口转发可以将指定地址访问指定的端口时，将流量转发至指定地址的指定端口。转发的目的如果不指定 ip 的话就默认为本机，如果指定了 ip 却没指定端口，则默认使用来源端口。
*注：*端口转发需开启 IP 伪装。
```shell
## IP 伪装
firewall-cmd --query-masquerade # 检查是否允许伪装IP
firewall-cmd --add-masquerade # 允许防火墙伪装IP
firewall-cmd --remove-masquerade # 禁止防火墙伪装IP
```
*示例：*
- 将80端口的流量转发至8080
  `firewall-cmd --add-forward-port=port=80:proto=tcp:toport=8080`
- 将80端口的流量转发至192.168.0.1
  `firewall-cmd --add-forward-port=proto=80:proto=tcp:toaddr=192.168.1.0`
- 将80端口的流量转发至192.168.0.1的8080端口
  `firewall-cmd --add-forward-port=proto=80:proto=tcp:toaddr=192.168.0.1:toport=8080`

## System Manage
### Logrotate
#### 概述
logrotate软件是一个日志管理工具，用于非分隔日志，删除旧的日志文件，并创建新的日志文件，起到“转储作用”，可以为系统节省磁盘空间。一般centos系统已经自带安装好了。

​    logrotate是基于crontab运行的，其脚本是/etc/cron.daily/logtotate，日志轮转是系统自发完成的，实际运行时，logrotate会调用配置文件/etc/logrotate.conf。可以在/etc/logrotate.d目录里放置自定义好的配置文件，用来覆盖logrotate.conf的缺省值。

#### logrotate 切割日志的两种机制
1）默认方式：重命名原日志文件，创建新的日志文件。

说明：

1. 重命名只会修改目录文件的内容，而进程操作文件靠的是inode编号，所以并不影响程序继续输出日志。
2. 创建新的日志文件，文件名和原来日志文件一样，但是inode编号不一样，所以程序输出的日志还是往原日志文件输出。
3. 通过某些方式通知程序，重新打开日志文件。程序重新打开日志文件，靠的是文件路径而不是inode编号，所以打开的是新的日志文件。

优点：不会丢失数据。

缺点：需要程序支持重新打开日志的功能，如：可以通过信号通知nginx。还有一种方法重开进程，这种方式会影响在线的服务。

2）copytruncate：把正在输出的日志拷(copy)一份出来，再清空(trucate)原来的日志。

优点：对程序无影响

缺点：日志在拷贝完到清空文件这段时间的写入数据将丢失。

*总结：*能用create的方案就别用copytruncate，很多程序提供了重开日志文件功能来支持create方案，或者自己做了日志滚动，不依赖logrotate。

#### logrotate 配置参数说明
| 配置参数               | 参数说明                                               |
| ---------------------- | ------------------------------------------------------ |
| daily\|weekly\|monthly | 转储周期 每天\|每周\|每月                              |
| dateext                | 日志切割文件后缀以当前日期结尾，如：messages-20181125  |
| rotate N               | 日志保留N份                                            |
| compress\|nocompress   | 通过gzip压缩转储日志\|不压缩                           |
| copytruncate           | 拷贝日志文件后截断                                     |
| olddir Dir             | 转储日志存放目录，必须和当前日志文件在同一个文件系统。 |

*示例：*

```shell
## 钱包服务日志切割
cat /etc/logrotate.d/applog
/home/logs/app01/app01.log
/home/logs/app02/app02.log
/home/logs/app03/app03.log
/home/logs/app04/app04.log
{
daily
copytruncate
dateext
olddir /backup
compress
notifempty
rotate 30
}

## 测试
/usr/sbin/logrotate --debug --verbose --force /etc/logrotate.d/applog
/usr/sbin/logrotate --force /etc/logrotate.d/applog

## 查看日志切割状态
cat /var/lib/logrotate/logrotate.status
```

# 应用场景
## System

### Yum 源
#### EPEL源
EPEL 即Extra Packages for Enterprise Linux的简称，是为企业级Linux提供的一组高质量的额外软件包，包括但不限于Red Hat Enterprise Linux (RHEL), CentOS and Scientific Linux (SL), Oracle Enterprise Linux (OEL)。
EPEL官网:http://fedoraproject.org/wiki/EPEL/zh-cn

**Centos7安装EPEL：**`yum -y install epel-release`
**重新创建 本地仓库缓存：**`yum clean all && yum makecache`


### LVM
#### 概念
LVM主要在磁盘与文件系统之间建立一个层,主要用来管理多磁盘多分区，及与多文件系统的映射。
PV * N --> VG --> LV * N
- 物理卷(Physical Volume,PV)
  - `pvcreate`：创建物理卷
  - `pvremove`：移除物理卷
  - `pvdisplay`：列出物理卷
- 卷组(Volume Group,VG)
  - `vgcreate`：创建卷组
  - `vgreduce`：
  - `vgremove`：移除卷组
  - `vgdisplay`：列出卷组
  - `vgextend`：
- 逻辑卷(Logic Volumn)
  - `lvcreate`：创建逻辑卷
  - `lvremove`：移除逻辑卷
  - `lvdisplay`：列出逻辑卷
  - `lvextend`：


#### 多个硬盘整合
1）列出硬盘信息
`fdisk -l`
2）创建物理卷
`pvcreate /dev/nvme0n1 /dev/nvme1n1`
查看物理卷信息：
```shell
pvdisplay
  "/dev/nvme0n1" is a new physical volume of "931.51 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/nvme0n1
  VG Name
  PV Size               931.51 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               dIxETA-cF70-dxY6-duvj-8yeq-Z7qw-EcTLKP

  "/dev/nvme1n1" is a new physical volume of "931.51 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/nvme1n1
  VG Name
  PV Size               931.51 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               Wkpm3X-cmEO-TxEn-iZli-przw-P2Ud-tXRk7y
```
3）创建卷组
`vgcreate vg_ssd /dev/nvme0n1 /dev/nvme1n1`
扩展卷组：`vgextend vg_ssd NEW_PV`
查看卷组：
```shell
vgdisplay
  --- Volume group ---
  VG Name               vg_ssd
  System ID
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               <1.82 TiB
  PE Size               4.00 MiB
  Total PE              476934
  Alloc PE / Size       0 / 0
  Free  PE / Size       476934 / <1.82 TiB
  VG UUID               0uvaKB-kOCU-VWnZ-ZC1B-2Z0k-cqTL-h15k4z
  ```
4）创建逻辑卷
`lvcreate -l 476934 -n lvdisk01 vg_ssd`
显示逻辑卷：
```shell
lvdisplay
  --- Logical volume ---
  LV Path                /dev/vg_ssd/lvdisk01
  LV Name                lvdisk01
  VG Name                vg_ssd
  LV UUID                busfKu-z3KC-d3fi-0dO0-YOpl-S8iu-jYAgqP
  LV Write Access        read/write
  LV Creation host, time fc, 2020-07-10 13:56:17 +0800
  LV Status              available
  # open                 0
  LV Size                <1.82 TiB
  Current LE             476934
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
  ```
5）挂载逻辑卷
格式化：`mkfs.ext4 /dev/vg_ssd/lvdisk01`
挂载逻辑卷：`mount /dev/vg_ssd/lvdisk01 /data`
重启自动挂载：`/dev/vg_ssd/lvdisk01 /data ext4  defaults 0 0`
补充：
- 扩容逻辑卷：`lvextend -l +n /dev/vg_ssd/lvdisk01`
- 收缩逻辑卷：`lvreduce -l -n /dev/vg_ssd/lvdisk01` (注：数据可能会丢失)
- 扩容生效：`resize2fs /dev/vg1/lvdisk1 (ext*) 或 xfs_growfs /dev/vg1/lvdisk1 (xfs)`



## Centos8 VS Centos7 (参照redhat)
> 红帽RHEL8与RHEL7的区别
镜像下载地址：
- 官网默认下载：https://wiki.centos.org/Download
- 官网之前版本：https://www.centos.org/download/
- 阿里云镜像：http://mirrors.aliyun.com/centos




### 红帽RHEL8和RHEL7功能区别对比
1.1 默认的文件系统
RHEL8与RHEL7都是采用XFS
1.2 


### Centos8 网络配置
1）RHEL8与RHEL7的区别：
RHEL7：同时支持`network.service`和`NetworkManager.service（NM）`。默认情况下，这2个服务都有开启，但通常会将NM禁用掉。
RHEL8：废弃`network.service`，因此只能通过NM进行网络配置，包括动态ip和静态ip，即必须开启NM，否则无法使用网络。
*注：*rhel8依然支持`network.service`，只是默认没安装。可以通过`yum install network-script`s来安装传统的network.service，但下一个rhel的大版本里将彻底废除，因此不建议使用network.service。

#### 网络配置的3种方式
`ifcfg`和`NM connection`的关联：虽然`network.service`被废弃了，但是redhat 8 为了兼容传统的ifcfg，通过NM进行网络配置时候，会自动将connection同步到ifcfg配置文件中。也可以通过`nmcli c reload`或者`nmcli c load /etc/sysconfig/network-scripts/ifcfg-ethX`的方式来让NM读取ifcfg配置文件到connection中。因此ifcfg和connection是一对一的关系，connection和device是多对一的关系。
第一种.手工配置ifcfg，通过NM来生效（推荐）
```shell
cat > /etc/sysconfig/network-scripts/ifcfg-eth0 <<EOF
NAME=eth0
DEVICE=eth0
ONBOOT=yes
BOOTPROTO=none
TYPE=Ethernet
IPADDR=192.168.1.10
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
EOF
nmcli c reload
或 nmcli c up eth0
```

第二种.通过NM自带工具配ip，比如nmcli。

第三种.手工配置ifcfg，通过传统network.service来生效。


#### nmcli
**基本概念：**
- nmcli connection 可理解为配置文件，相当于ifcfg-ethX
  connection有2种状态：
  - 活跃：表示当前该connection生效。
  - 非活跃：表示当前该connection不生效。
- nmcli device 可理解为实际存在的网卡（包括物理网卡和虚拟网卡），可以简写为nmcli d
  device有4种常见状态：
  - connected：已被NM纳管，并且当前有活跃的connection。
  - disconnected：已被NM纳管，但是当前没有活跃的connection。
  - unmanaged：未被NM纳管。
  - unavailable：不可用，NM无法纳管，通常出现于网卡link为down的时候（比如ip link set ethX down）。

**常用命令：**
IP配置：
- 配置静态IP：`nmcli c add type ethernet con-name ethX ifname ethX ipv4.addr 192.168.1.100/24 ipv4.gateway 192.168.1.1 ipv4.method manual`
- 配置动态IP：`nmcli c add type ethernet con-name ethX ifname ethX ipv4.method auto`
- 修改IP(非交互式)：`nmcli c modify ethX ipv4.addr '192.168.1.200/24'`
- 修改IP(交互式)：`nmcli c edit ethX` --> `goto ipv4.addresses` --> `change`

nmcli connection 指令：
```shell
nmcli c up ethX   # 启用connection（相当于ifup）
nmcli c down      # 停止connection（相当于ifdown）
nmcli c delete ethX   # 删除connection（类似于ifdown并删除ifcfg）
nmcli c show      # 查看connection列表
nmcli c show ethX # 查看connection详细信息

## 重载ifcfg或route到connection（不会立即生效）
nmcli c reload
nmcli c load /etc/sysconfig/network-scripts/ifcfg-ethX
nmcli c load /etc/sysconfig/network-scripts/route-ethX
## 立即生效connection，有3种方法
nmcli c up ethX
nmcli d reapply ethX
nmcli d connect ethX
```
nmcli device 指令：
```shell
nmcli d           # 查看device列表
nmcli d show      # 查看所有device详细信息
nmcli d show ethX # 查看指定device的详细信息
nmcli d connect ethX  # 激活网卡
nmcli d disconnect ethX  # 让 NM 暂时不管理指定网卡
```

#### arp-scan
arp-scan 用于arp批量扫描. 运维最常见的需求就是分配新ip,确认这个ip有没有被使用.即使有ip表和系统查询,你也不能保证数据的准确性.大部分人会用ping,但是禁ping的服务器再正常不过了. 这个时候就要用到nmap和arp-scan了. 这两个工具安装在同一个子网的服务器内.nmap 用于端口扫描, 可以批量扫描.  arp-scan用于mac地址扫描,可以批量扫描.

centos7安装：
```shell
yum -y install epel-release
yum install arp-scan -y
arp-scan -I ens33 -l
```

#### Linux使用技巧
##### 限制用户使用chmod命令

1）修改用户对应HOME目录下的`~/.bashrc`文件
```shell
# 添加
alias chmod='echo "do not use chmod command"'
alias chown='echo "do not use chmod command"'
```
2）禁止用户修改文件
`chattr +i /home/username/.bashrc`