

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



