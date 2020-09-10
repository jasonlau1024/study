# 概述


# 部署

## 配置阿里云repo
```shell
# 备份
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
# Centos7
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# 重建缓存
yum makecache
```

## Yum安装docker-ce
```shell
## 更新yum包
$ sudo yum update -y
## 卸载旧版本
$ sudo yum remove docker docker-common docker-selinux docker-engine
## 安装需要的软件包 （yum -util 提供 yum-config-manager 功能；另外两个是 devicemapper 驱动的依赖）
$ sudo yum install yum-utils device-mapper-persistent-data lvm2 -y
## 设置 Yum 源
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
## 查看仓库中的所有 docker 版本
$ sudo yum list docker-ce --showduplicates | sort -r
## 安装指定版本  yum install <FQPN>
$ sudo yum install docker-ce-19.03.1
$ sudo yum install docker-ce -y
## 启动并加入开机自启动
$ sudo systemctl start docker
$ sudo systemctl enable docker
## 验证是否安装成功
$ sudo docker version
```



国内镜像加速：
```shell
# 加速站点：
	https://registry.docker-cn.com
	http://hub-mirror.c.163.com
	https://3laho3y3.mirror.aliyuncs.com
	http://f1361db2.m.daocloud.io
	https://mirror.ccs.tencentyun.com

cat /etc/docker/daemon.json
{
	"registry-mirrors":["https://3laho3y3.mirror.aliyuncs.com"]
}

systemctl restart docker
docker info
```

