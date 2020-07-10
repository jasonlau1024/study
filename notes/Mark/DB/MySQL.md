
## 概述


## 安装


### Yum 安装

```shell
## 安装 Yum 源
wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
rpm -ivh mysql-community-release-el7-5.noarch.rpm
yum repolist enabled | grep "mysql.*-community.*"
yum info mysql-community-server
####  说明  ####
默认5.6
如果需要安装5.7 需要修改/etc/yum.repos.d/mysql-community.repo文件
将5.6 enabled=0 5.7 enabled=1
###############
yum install mysql-community-server -y
systemctl start mysqld
netstat -lntp |grep mysqld
```

### 初始化数据库

- 设置 root 密码
  ```shell
  ## 5.7 获取root初始密码，
  cat /var/log/mysqld.log |grep pass
  ## mysqladmin 重置root 密码
  mysqladmin -u root -p[oldpasswd] password New@pass123
  ## mysql_secure_installation 重置密码
  mysql_secure_installation
  ## SQL 重置密码 (mysql5.6)
  update mysql.user set password=PASSWORD('passwd') where User='root';
  ## SQL 重置密码 (mysql5.7)
  update mysql.user set authentication_string=password('新密码') where user='root' and Host='localhost';
  ```
- 创建数据库(UTF8)
  `CREATE DATABASE `database` CHARACTER SET utf8 COLLATE utf8_general_ci;`
- 创建用户并授权
  `grant all privileges on `DatabaseName`.* to UserName@'%' identified by 'Password';`
- 创建一个超级管理员用户
  ```shell
  grant all on . to testadmin@'%' identified by '123456';
  grant all on . to testadmin@'%' with grant option;
  ```

## 维护
### 数据库管理
#### 修改密码策略
查看密码策略规则：

```shell
mysql> show VARIABLES like "%password%";
+----------------------------------------+-----------------+
| Variable_name                          | Value           |
+----------------------------------------+-----------------+
| default_password_lifetime              | 0               |
| disconnect_on_expired_password         | ON              |
| log_builtin_as_identified_by_password  | OFF             |
| mysql_native_password_proxy_users      | OFF             |
| old_passwords                          | 0               |
| report_password                        |                 |
| sha256_password_auto_generate_rsa_keys | ON              |
| sha256_password_private_key_path       | private_key.pem |
| sha256_password_proxy_users            | OFF             |
| sha256_password_public_key_path        | public_key.pem  |
| validate_password_check_user_name      | OFF             |
| validate_password_dictionary_file      |                 |
| validate_password_length               | 8               |
| validate_password_mixed_case_count     | 1               |
| validate_password_number_count         | 1               |
| validate_password_policy               | MEDIUM          |
| validate_password_special_char_count   | 1               |
+----------------------------------------+-----------------+
```

密码策略说明：(默认为 MEDIUM)

| Policy      | Tests Performed                                              |
| ----------- | ------------------------------------------------------------ |
| 0 or LOW    | Length                                                       |
| 1 or MEDIUM | Length; numeric, lowercase/uppercase, and special characters |
| 2 or STRONG | medium + dictionary file                                     |

更改密码策略为LOW

`set global validate_password_policy=0;`


### ERROR
#### 001 Need SUPER privilege
相关错误信息：`ERROR 1227 (42000) at line 1350: Access denied; you need (at least one of) the SUPER privilege(s) for this operation`

说明：
DEFINER指令中定义的用户没有SUPER特权。
CREATE PROCEDURE和CREATE FUNCTION需要CREATE ROUTINE特权。它们可能还需要SUPER特权，具体取决于DEFINER值

参考：<https://support.plesk.com/hc/en-us/articles/360005033014-Unable-to-import-MySQL-dump-using-Import-Dump-feature-in-Plesk-ERROR-1227-42000-at-line-1421-Access-denied-you-need-at-least-one-of-the-SUPER-privilege-s-for-this-operation>


#### 锁库只读模式
```shell
MariaDB [(none)]> show global variables like "%read_only%";
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| read_only     | OFF   |
+---------------+-------+
MariaDB [(none)]> set global read_only=1; 
```

