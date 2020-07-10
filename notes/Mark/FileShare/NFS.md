

## 概述
### 简介
NFS（Network File System）即网络文件系统，它允许网络中的计算机之间通过网络共享资源。将NFS主机分享的目录，挂载到本地客户端当中，本地NFS的客户端应用可以透明地读写位于远端NFS服务器上的文件，在客户端端看起来，就像访问本地文件一样。

RPC，基于C/S模型。程序可以使用这个协议请求网络中另一台计算机上某程序的服务而不需知道网络细节，甚至可以请求对方的系统调用。

对于Linux而言，文件系统是在内核空间实现的，即文件系统比如ext3、ext4等是在Kernel启动时，以内核模块的身份加载运行的。

### NFS 优缺点
**优点：**
1. 简单易上手。
2. NFS文件系统内数据是在文件系统之上的，即数据是能看得见的。
3. 部署快速，维护简单方便，且可控，满足需求得就是最好的。
4. 可靠，从软件层面上看。数据可靠性高，经久耐用。数据是文件系统之上的。
5. 服务非常稳定。

**缺点：**
1. 存在单点故障，如NFS server宕机，所有客户端都不能访问共享目录。（可考虑HA方案）
2. 在大数据高并发的场合，NFS效率低，性能有限。
3. 客户端认证是基于IP和主机名的，权限要根据ID识别，安全性一般。
4. NFS数据是明文的，NFS本身不对数据完整性作验证。
5. 多台客户机器挂载一个NFS服务器，维护比较麻烦。

### NFS的工作原理
NFS本身的服务并没有提供数据传递的协议，而是通过使用RPC（远程过程调用 Remote Procedure Call）来实现。当NFS启动后，会随机的使用一些端口，NFS就会向RPC去注册这些端口。RPC就会记录下这些端口，RPC会开启111端口。通过client端和sever端端口的连接来进行数据的传输。因此，在启动nfs之前，要确保rpc服务正在运行。

当访问程序通过NFS客户端向NFS服务端存取文件时，其请求数据流程大致如下：
1）首先用户访问网站程序，由程序在`NFS Clinet`发出存取NFS文件的请求，这时`NFS Client`的`rpcbind`服务就会通过网络向`NFS Server`的`rpcbind`的111端口发出NFS文件存取功能的询问请求；
2）`NFS Server`的`rpcbind`找到对应的已注册的NFS端口后，通知`NFS Clinet`的`rpcbind`。此时`NFS Client`获取到正确的端口，并与`NFS daemon`联机存取数据。
3）`NFS Client`把数据存取成功后，返回给前端访问程序，告知给用户存取结果，作为网站用户，就完成了一次存取操作。

*补充说明：*NFS 与 RPC 之间的关系
NFS的各项功能都需要向RPC服务注册，所以只有RPC服务才能获取到NFS服务的各项功能对应的端口号、PID、NFS在主机所监听的IP等信息，而NFS客户端也只能通过向RPC服务询问才能找到正确的端口。

### 相关组件
- `rpc`：远程过程调用协议，是实现本地调用远程主机实现系统调用的协议。
- `portmapper`：负责分配rpc server的端口，并在client端请求时，负责响应目的rpc server端口返回给client端，工作在tcp与udp的111端口上。
- `rpc.mountd`：是nfs服务的认证服务的守护进程，client在收到返回的真正端口时，就会去连接mountd，认证取得令牌。
- `nfsd`：nfs的守护进程，负责接收到用户的调用请求后与内核发出请求并得到调用结果响应给用户，工作在tcp和udp的2049端口。
- `rpc.lockd`：文件锁定，避免多个用户同时修改一个文件。
- `rpc.statd`：文件一致性校验，检查修复多个用户同时修改一个文件造成的文件损坏。同lockd需要服务端和客户端同时运行才能生效。
- `rpc.idmapd`：是NFS的一个程序，用来负责远程client端创建文件后的权限问题。
- `rpc.rquotad`：用用于实现磁盘配额，当client端挂载nfs后可以限制磁盘空间的大小。

## 安装
### Yum 安装
nfs 程序包：`nfs-utils`
rpcbind 程序包：`rpcbind`
1）安装 `nfs-util` 和 `rpcbind`
```SHELL
## 客户端服务端都需安装
## 启动服务需要先启动rpcbind然后再启动NFS
yum install nfs-utils rpcbind -y
systemctl start rpcbind
systemctl start nfs
systemctl enable rpcbind
systemctl enable nfs
```
2）创建共享目录
```SHELL
## 服务端
mkdir -p /mnt/backup
echo '/mnt/backup *(rw,async,no_root_squash,no_subtree_check)' >> /etc/exports
exportfs -a
## 客户端
mkdir /backup
mount -t nfs server_ip:/mnt/backup /backup
echo "echo "server_ip:/mnt/backup /backup nfs rw,tcp,intr 0 1" >> /etc/fstab"
```
### 配置说明
`/etc/exports`，如：`/export 172.16.1.34(rw,sync,no_root_squash) *(ro)`
- rw|ro：读写|只读 权限
- sync|async 同步|异步
  - 同步：资料同步写入存储器中。
  - 异步：资料会先暂时存放在内存中，不会直接写入硬盘。 提升写入效率，宕机丢失风险。
- no_root_squash|root_squash|all_squash
  - `no_root_squash`：root登录NFS，则具有改共享目录的root权限。（不安全）
  - `root_squash`：root登录NFS，压缩为匿名用户（默认nfsnobody）。
  - `all_squash`：对所有访问NFS共享目录的用户重设为指定匿名用户。
- anonuid|anongid：指定匿名用户的UID|UID,此ID必须存在于/etc/passwd中。
- insecure：允许从这台机器过来的非授权访问。
  注：使用匿名用户，客户端和服务端都必须有该用户且UID和GID保持一致。

`/etc/fstab`，如：`server_ip:/export  /mnt/export  nfs  rw,tcp,intr  0 1`
第1列.需要挂载的文件系统或存储设备
第2列.挂载点
第3列.指定文件系统或分区的类型
第4列.挂载选项(可参考man mount)
   auto: 系统自动挂载，fstab默认就是这个选项
   ro: read-only
   rw: read-write
   defaults: rw, suid, dev, exec, auto, nouser, and async. 
第5列.dump选项，设置是否让备份程序dump备份文件系统，0为忽略，1为备份。
第6列.fsck选项，告诉fsck程序以什么顺序检查文件系统，0为忽略。


## 维护
### 故障排查
#### 客户端与服务端连接异常
- 服务端自检：`showmount -e server_ip`
- 客户端检查：`showmount -e server_ip`
- 查看挂载信息：`cat /proc/mounts`
- 平滑重启NFS：`systemctl reload nfs`
- 重载所有挂载点：`exportfs -a`
#### 卸载异常
`umount:/mnt:device is busy` 需要退出挂载目录才能执行卸载。
`NFS server` 宕机，需要强制卸载，`umount -lf /mnt`
### 优化
#### 客户端挂载参数优化
兼顾安全与性能：`mount -t nfs -o nosuid,noexec,nodev,noatime,nodiratime,rsize=131071,wsize=131072 10.0.0.7:/data/ /mnt`

#### 系统内核优化
```shell
cat >> /etc/sysctl.conf << EOF
net.core.wmem_default = 8388608
net.core.rmem_default = 8388608
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
EOF
sysctl -p
```




