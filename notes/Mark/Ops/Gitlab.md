
# Gitlab介绍
## 概述
    GitLab是一个开源版本管理系统，是集代码托管，测试，部署于一体的开源git仓库管理软件，可通过web界面来进行访问公开的或私人项目。与Github类似，GitLab能够浏览代码，管理缺陷和注释。可以管理团队对仓库的访问，它非常易于浏览提交过的版本，并提供一个文件历史库。是目前非常流行好用的研发版本控制系统。
- Git的三个类别
  - Git：是本地版本控制系统工具
  - GitHub：官方在线代码托管仓库
  - GitLab：开源私有版本仓库

# Gitlab部署
## 环境准备
### 硬件需求
> 参考：https://docs.gitlab.com/ce/install/requirements.html

最低配置：2*CPU+8Mem
### 更换阿里Yum源
```shell
yum install wget -y
mv /etc/yum.repos.d /etc/yum.repos.d.backup
mkdir /etc/yum.repos.d
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# 重建缓存
yum clean all
yum makecache
yum update -y
```
### OS 环境配置
关闭Selinux和firewalld
```shell
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config 
setenforce 0
systemctl stop NetworkManager
systemctl disable NetworkManager
systemctl disable iptables
systemctl stop iptables
```
## 安装Gitlab

```shell
# 安装GitLab依赖项
yum install -y curl openssh-server openssh-clients postfix cronie policycoreutils-python
# 获取GitLab的rpm包并安装
wget --content-disposition https://packages.gitlab.com/gitlab/gitlab-ce/packages/el/7/gitlab-ce-11.1.4-ce.0.el7.x86_64.rpm/download.rpm
rpm -ivh gitlab-ce-11.1.4-ce.0.el7.x86_64.rpm
# 汉化
git clone https://gitlab.com/xhang/gitlab.git -b v11.1.4-zh
cat gitlab/VERSION
\cp -rf gitlab/* /opt/gitlab/embedded/service/gitlab-rails/
sudo gitlab-ctl reconfigure
```


# 维护

## 配置HTTPS
修改 /etc/gitlab/gitlab.rb
```shell
external_url 'https://gitlab.example.com'
letsencrypt['enable'] = false
nginx['redirect_http_to_https'] = true
nginx['ssl_certificate'] = "/etc/gitlab/ssl/full_chain.pem"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/private.key"
# 重新加载配置
gitlab-ctl reconfigure
```

## 备份
- 手动备份GitLab
   - 使用gitlab指令
`gitlab-rake gitlab:backup:create`
   - 备份目录说明
    默认备份在/var/opt/gitlab/backups目录下，可通过修改/etc/gitlab/gitlab.rb的配置参数"backup_path"，自定备份目录。
- 定时自动备份
```Shell
# 使用命令crontab -e 或 修改/etc/crontab文件
crontab -l
0 0 * * * /opt/gitlab/bin/gitlab-rake gitlab:backup:create
# 重启crontab
systemctl restart crond
```
- 自动清理
```Shell
# 修改配置文件，保留7天备份（7*3600*24=604800）
gitlab_rails['backup_keep_time'] = 604800
# 重新加载配置
gitlab-ctl reconfigure
```

## 恢复
```shell
# 停止相关数据连接服务
gitlab-ctl stop unicorn 
gitlab-ctl stop sidekiq
# 恢复仓库数据
gitlab-rake gitlab:backup:restore BACKUP=XXX
# 启动 Gitlab
gitlab-ctl start
```
*补充：*如需异机恢复，需运行相同版本的Gitlab服务，并将GitLab配置文件/etc/gitlab/gitlab.rb以及备份文件拷贝到对应的目录。


## 升级

- 下载更新的RPM包
- 关闭部分GitLab服务
`gitlab-ctl stop unicorn`
`gitlab-ctl stop sidekiq`
`gitlab-ctl stop nginx`
- 开始升级
`rpm -Uvh xx.rpm`
- 重载配置
`gitlab-ctl reconfigure`
- 重启服务
`gitlab-ctl restart`
- 登录web查看版本
- 更新汉化补丁
`cd ~/gitlab`
`git diff v10.0.4 v10.0.4-zh > ../10.0.4-zh.diff`
`yum install patch -y`
`patch -d /opt/gitlab/embedded/service/gitlab-rails -p1 < 10.0.4-zh.diff`

## 卸载
- 完全卸载参考
    参考：https://yq.aliyun.com/articles/114619
    思路：gitlab-ctl stop --> rpm -e 卸载gitlab-ce包 --> 查看进程ps -aux |grep gitlab kill掉 --> find / -name gitlab | xargs rm -rf
- 补充：卸载报错①
   - 故障现象：
      在卸载gitlab然后再次安装执行sudo gitlab-ctl reconfigure的时候往往会出现：ruby_block[supervise_redis_sleep] action run，会一直卡无法往下进行！
   - 解决方案：
    1、按住CTRL+C强制结束；
    2、运行：sudo systemctl restart gitlab-runsvdir；
    3、再次执行：sudo gitlab-ctl reconfigure
   - 解决方案来源：
    https://gitlab.com/gitlab-org/omnibus-gitlab/issues/160
   - 详细日志文件：
    /var/log/gitlab/reconfigure 目录下的日志文件






## 重置Gitlab root密码
```shell
gitlab-rails console production

irb(main):001:0> user = User.where(id: 1).first
    => #<User id: 1, email: "admin@example.com", ...
irb(main):002:0> user.password='xxxx'
    => 12345678
irb(main):003:0> user.password_confirmation='xxxx'
    => 12345678
irb(main):004:0> user.save!
    => true
irb(main):005:0> quit
```

# Git Usage
## 异常处理
### 文件冲突
根本原因：本地修改的文件和目标远程库的统一文件都有修改。
- 不同分支下的冲突
  不同分支下的同一文件都发生了修改。
- 同一分支下的冲突
  本地文件和目标远程库下同一文件都发生了修改。
**解决方式：**
1. 忽略本地修改，强制拉取远程到本地