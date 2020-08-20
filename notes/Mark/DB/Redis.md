

## 概述


## 安装
### Yum 安装最新版本
1）添加yum源
```shell
yum install -y http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
yum --enablerepo=remi info redis
yum --enablerepo=remi install redis
```
2）修改配置文件
```shell
sed -i 's/bind 127.0.0.1/# bind 127.0.0.1/g' /etc/redis.conf
sed -i "s/# requirepass foobared/requirepass UaHPDiEdhcNDcxth/g" /etc/redis.conf
systemctl restart redis
systemctl enable redis
```


## 维护

### Redis的持久化
**Redis的三种持久化方式：**
- RDB：指定时间间隔对数据进行快照存储。
- AOF：记录每次对服务的写操作（类似MySQL的binlog）。
- 混合(4.0+)：

#### RDB快照持久化

**持久化配置：**
```shell
save 900 1      # 表示900s内如果有1条写入命令，触发产生一次快照。
save 300 10     # 表示300s内有10条写入，触发快照。
save 60 10000   # 60s内有10000条写入，触发快照。
dbfilename "dump.rdb"   # RDB文件
dir "/data/redis/rdb"   # 文件存储路径

## 其它相关配置
top-writes-on-bgsave-error yes  # 持久化异常，主进程停止写入。默认开启。
rdbcompression no   # 数据压缩，建议关闭，Redis本身属于CPU密集型，相比硬盘成本，CPU更值钱。
rdbchecksum yes # 导入数据校验，如果最求性能可以关闭。

```


#### AOF追加持久化

**持久化配置：**
```shell
dir "/data/redis/aof"           # AOF文件存放目录
appendonly yes                       # 开启AOF持久化，默认关闭
appendfilename "appendonly.aof"      # AOF文件名称（默认）
appendfsync everysec                       # AOF持久化策略，这里是每秒 fsync 一次
auto-aof-rewrite-percentage 100      # 触发AOF文件重写的条件：当前AOF文件大小和最后一次重写后的大小之间的比率。如默认100代表当前AOF文件是上次重写的两倍时候才重写。
auto-aof-rewrite-min-size 64mb       # 触发AOF文件重写的条件：只有当AOF文件大小大于该值时候才可能重写，默认是64MB

```




