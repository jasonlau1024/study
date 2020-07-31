
## 概述
 计算机数据（Data）的存储一般以硬盘为数据存储空间资源，从而保证计算机内的数据能够持续保存。对于数据的处理，一般会采用数据库相关的技术进行处理，从而保证数据处理的高效性。
采用数据库的管理模式不仅提高了数据的存储效率，而且在存储的层面上提高了数据的安全性。通过分类的存储模式让数据管理更加安全便捷，更能实现对数据的调用和对比，并且方便查询等操作的使用。

### MySQL数据库特性
- 使用 C 和 C++ 开发，开源，支持跨平台。
- 为多种编程语言提供 API。如：C，C++，Python，Java，PHP等等
- 支持多线程，充分利用 CPU 资源。
- 优化的 SQL 查询算法，有效地提高查询速度。
- 可作为单独的应用程序，也可作为一个库嵌套到其它软件中。
- 提供多种语言支持。
- 提供 TCP/IP、ODBC 和 JDBC 等多种数据库连接途径。
- 提供用于管理、检查、优化数据库操作的管理工具。
- 支持大型的数据库。可以处理拥有上千万条记录的大型数据库。
- 支持多种存储引擎。

### 5.7 的新特性

1. 自动生成root随机密码。（5.6默认root密码为空）
2. 默认不生成test数据库。
3. 连接默认使用SSL加密方式。
4. 支持用户设置密码过期策略。
5. 支持用户锁。（被锁定用户无法访问和使用数据库）
6. 全面支持JSON。
7. 支持两类生成列。
8. 引入系统库。

### 实用工具集

**服务端实用工具：**

| 服务端工具       | 说明                                  |
| ---------------- | ------------------------------------- |
| mysqld           | MySQL 服务器进程。                    |
| mysqld_safe      | 服务器启动脚本。                      |
| mysqld_multi     | 启动或停止多个MySQL服务。             |
| mysql_install_db | 数据库初始化，即创建 MySQL 授予权表。 |

**客户端使用工具：**

| 客户端工具   | 说明                                          |
| ------------ | --------------------------------------------- |
| mysql        | 交互式输入 SQL 语句或从文件经批处理模式执行。 |
| mysqladmin   | 执行数据库管理操作。                          |
| mysqlbinlog  | 从二进制日志读取语句。                        |
| mysqlcheck   | 检查、修复、分析以及优化表。                  |
| mysqldump    | 将 MySQL 数据库转储到一个文件。               |
| mysqlshow    | 显示数据库、表、列以及索引相关信息。          |
| perror       | 显示系统或 MySQL 错误代码含义。               |
| mysql import | 使用 load data infile 导入数据。              |





## 安装


### Yum 安装

官方文档：<https://dev.mysql.com/doc/mysql-yum-repo-quick-guide/en/

1）添加 MySQL Repo
下载地址：<https://dev.mysql.com/downloads/repo/yum/>
`yum install -y https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm`

2）选择版本安装

```shell
## 列出所有版本
$ yum repolist all | grep mysql
## 使用 yum-config-manager 指定版本
## 或修改 /etc/yum.repos.d/mysql-community.repo 文件中对应版本enabled的值
$ sudo yum-config-manager --disable mysql80-community
$ sudo yum-config-manager --enable mysql57-community
## 安装MySQL
$ sudo yum install mysql-community-server -y
## 启动服务
$ sudo systemctl start mysqld.service
## 随机生成的密码保存在/var/log/mysqld.log文件中
$ sudo cat /var/log/mysqld.log |grep password
## 登录并修改密码
$ mysql -uroot -p
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'Aa@123456'; 
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |  ## 是一个信息数据库，保存其它数据库的信息。
| mysql              |  ## 核心数据库，类似于sql server中的master表，主要负责存储数据库的用户、权限设置、关键字等
| performance_schema |  ## 主要用于收集数据库服务器性能参数。
| sys                |  ## Sys库所有的数据源来自：performance_schema。降低复杂度，方便DBA了解DB的运行情况
+--------------------+
```

*注：*在 EL8-based 系统中默认情况下启用了MySQL模块，它将屏蔽`MySQL repo`提供的软件包。如果需要使用`MySQL repo`安装，则需禁止该模块：`yum module disable mysql` 。



其它：http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm


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

#### Delele后释放磁盘空间
*说明：*InnoDB 数据库在使用 delete 进行删除操作的时候，只会将已经删除的数据标记为删除，并没有把数据文件删除，因此并不会彻底的释放空间。这些被删除的数据会被保存在一个链接清单中，当有新数据写入的时候，MySQL 会重新利用这些已删除的空间进行再写入。
*解决：*官方推荐可以使用 OPTIMIZE TABLE 命令来优化表，该命令会重新利用未使用的空间，并整理数据文件的碎片。

`OPTIMIZE [NO_WRITE_TO_BINLOG | LOCAL] TABLE tbl_name [, tbl_name] ...`
OPTIMIZE TABLE 将重新组织表数据和相关索引数据的物理存储空间，减少存储空间并提高I/O访问效率。对每个表所做的影响取决于该表所使用的存储引擎。该命令对视图无效。

**示例：**
```shell
## 查看表大小
mysql> SELECT TABLE_NAME,(DATA_LENGTH+INDEX_LENGTH)/1048576,TABLE_ROWS FROM TABLES WHERE TABLE_SCHEMA='test_if' AND TABLE_NAME='fa_mt_block_trade';
+-------------------+------------------------------------+------------+
| TABLE_NAME        | (DATA_LENGTH+INDEX_LENGTH)/1048576 | TABLE_ROWS |
+-------------------+------------------------------------+------------+
| fa_mt_block_trade |                           431.3906 |          0 |
+-------------------+------------------------------------+------------+
## OPTIMIZE优化表
mysql> optimize table test_if.fa_mt_block_trade;
+---------------------------+----------+----------+-------------------------------------------------------------------+
| Table                     | Op       | Msg_type | Msg_text                                                          |
+---------------------------+----------+----------+-------------------------------------------------------------------+
| test_if.fa_mt_block_trade | optimize | note     | Table does not support optimize, doing recreate + analyze instead |
| test_if.fa_mt_block_trade | optimize | status   | OK                                                                |
+---------------------------+----------+----------+-------------------------------------------------------------------+
## 查看优化后表大小
mysql> SELECT TABLE_NAME,(DATA_LENGTH+INDEX_LENGTH)/1048576,TABLE_ROWS FROM TABLES WHERE TABLE_SCHEMA='test_if' AND TABLE_NAME='fa_mt_block_trade';
+-------------------+------------------------------------+------------+
| TABLE_NAME        | (DATA_LENGTH+INDEX_LENGTH)/1048576 | TABLE_ROWS |
+-------------------+------------------------------------+------------+
| fa_mt_block_trade |                             0.0156 |          0 |
+-------------------+------------------------------------+------------+
```
*注：*
1. 在OPTIMIZE TABLE运行过程中，MySQL会锁定表。
2. OPTIMIZE实际上是alter操作，将表拷贝一份到临时表，所以需要注意预留足够的磁盘空间


### ERROR
#### 001 Need SUPER privilege
相关错误信息：`ERROR 1227 (42000) at line 1350: Access denied; you need (at least one of) the SUPER privilege(s) for this operation`

说明：
DEFINER指令中定义的用户没有SUPER特权。
CREATE PROCEDURE和CREATE FUNCTION需要CREATE ROUTINE特权。它们可能还需要SUPER特权，具体取决于DEFINER值

参考：<https://support.plesk.com/hc/en-us/articles/360005033014-Unable-to-import-MySQL-dump-using-Import-Dump-feature-in-Plesk-ERROR-1227-42000-at-line-1421-Access-denied-you-need-at-least-one-of-the-SUPER-privilege-s-for-this-operation>




## 基本SQL
http://c.biancheng.net/view/7782.html

### 库操作

#### 创建数据库

**创建语句：**`CREATE DATABASE`

`CREATE DATABASE [IF NOT EXISTS] <数据库名>
[[DEFAULT] CHARACTER SET <字符集名>] [[DEFAULT] COLLATE <校对规则名>];`

- `IF NOT EXISTS` 在创建数据库之前判断诗句哭是否已经存在。
- `数据库名` MySQL 的数据存储区将以目录方式表示 MySQL 数据库，因此数据库名称必须符合操作系统的文件夹命名规则。（Linux下默认区分大小写，Windows默认不区分）
- `[DEFAULT] CHARACTER SET` 指定数据库的默认字符集。
- `[DEFAULT] COLLATE` 指定字符集的默认校对规则。

**查看数据库的定义声明：**`SHOW CREATE DATABASE <数据库名>`

实例①.简单创建一个数据库

`CREATE DATABASE test_db;`

实例②.指定字符集和校对规则创建数据库

`mysql> CREATE DATABASE IF NOT EXISTS test_db_char
    -> DEFAULT CHARACTER SET utf8
    -> DEFAULT COLLATE utf8_chinese_ci;`

补充①.：中文常用字符集

- `UTF8`：能够存储全球的所有字符，在任何国家都可以使用。

  默认校验规则`utf8_general_ci`

- `FBK`：只能存储中文涉及到的字符。

  默认的校对规则为 gbk_chinese_ci。

#### 显示数据库

**语句：**`SHOW DATABASES [LIKE '数据库名'];`

`LIKE` 匹配指定数据库名称，这里的数据库需使用单引号`'数据库名'`

实例①.列出所有数据库

`mysql> SHOW DATABASES;`

实例②.显示数据库名包含test的数据库

`SHOW DATABASES LIKE '%test%';`

#### 修改数据库

`使用语句`：`ALTER DATABASE` 或 `ALTER SCHEMA`

`ALTER DATABASE [数据库名] { [ DEFAULT ] CHARACTER SET <字符集名> | [ DEFAULT ] COLLATE <校对规则名>}`

使用语法同`CREATE DTABASE`

- `ALTER DATABASE` 用于更改数据库的全局特性。这些特性存储在数据库目录的 db.opt 文件中。
- `[数据库名` 数据库名忽略时，表示更改默认数据库。
- `CHARACTER SET` 用于更改默认的数据库字符集。

实例①.修改数据库的字符集以及默认校对规则

`mysql> ALTER DATABASE test_db
    -> DEFAULT CHARACTER SET gb2312
    -> DEFAULT COLLATE gb2312_chinese_ci;`

#### 删除数据库

**使用语句：**`DROP DATABASE [ IF EXISTS ] <数据库名>`

#### 选择数据库

**使用语句：** `USE <数据库名>`

注：只有使用 USE 语句来指定某个数据库作为当前默认数据库之后，才能对该数据库及其存储的数据对象执行操作。

### 表操作

#### 创建数据表

**使用语句：**`CREATE TABLE <表名> ([表定义选项])[表选项][分区选项];`

- `表定义选项` 格式为`<列名1> <类型1> [,…] <列名n> <类型n>`

**查看表定义：** `SHOW CREATE TABLE <表名>\G；`

**查看表结构：**`DESC <表名>;`

实例①.指定数据库创建表

```shell
mysql> CREATE TABLE test_db.tb_test1
    -> (
    -> id INT(11),
    -> name VARCHAR(25),
    -> deptid INT(11),
    -> salary FLOAT
    -> );
mysql> SHOW CREATE TABLE test_db.tb_test1\G;
*************************** 1. row ***************************
       Table: tb_test1
Create Table: CREATE TABLE `tb_test1` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(25) DEFAULT NULL,
  `deptid` int(11) DEFAULT NULL,
  `salary` float DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1
mysql> DESC test_db.tb_test1;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| id     | int(11)     | YES  |     | NULL    |       |
| name   | varchar(25) | YES  |     | NULL    |       |
| deptid | int(11)     | YES  |     | NULL    |       |
| salary | float       | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
4 rows in set (0.00 sec)

```

#### 修改数据表

**使用语句：**`ALTER TABLE <表名> [修改选项]`

`修改选项`：

```shell
{ ADD COLUMN <列名> <类型>
| CHANGE COLUMN <旧列名> <新列名> <新列类型>
| ALTER COLUMN <列名> { SET DEFAULT <默认值> | DROP DEFAULT }
| MODIFY COLUMN <列名> <类型>
| DROP COLUMN <列名>
| RENAME TO <新表名> }
```

示例①.**添加字段**

`ALTER TABLE <表名> ADD <新字段名> <数据类型> [约束条件] [FIRST|AFTER 已存在的字段名]；`

- `FIRST` 表示添加为第一个字段
- `AFTER 已存在的字段名` 表示添加到该字段名后面

示例②.**修改字段**

修改字段数据类型：

`ALTER TABLE <表名> MODIFY <字段名> <数据类型>`

修改字段命令：

`ALTER TABLE <表名> CHANGE <旧字段名> <新字段名> <新数据类型>；`

注：这一的`新数据类型`不能为空，即使需要保持跟原来的一样

示例③.**删除字段**

`ALTER TABLE <表名> DROP <字段名>；`

示例④.**修改表名**

`ALTER TABLE <旧表名> RENAME [TO] <新表名>；  ## 这里的TO可以省略，无影响`

#### 删除数据表

**使用语句：** `DROP TABLE [IF EXISTS] <表名> [ , <表名1> , <表名2>] …`

### 数据操作

#### 插入数据

两种语法形式：

1）**INSERT INTO…VALUES语句**（常用方式）

`INSERT INTO <表名> [ (<列名1> [ , … <列名n>]) ]
VALUES ([值1,…,值n]);`

2）**INSERT INTO…SET语句**（了解即可）

注：在MySQL中，用单条 INSERT 语句处理多个插入要比使用多条 INSERT 语句更快。当使用单条 INSERT 语句插入多行数据的时候，只需要将每行数据用圆括号括起来即可。

实例①.指定表添加两条数据

`mysql> INSERT INTO test_db.tb_test1 (id,name,deptid,salary) VALUES (01,"zhangsan",01,15000),(02,"lisi",01,16000);`

**使用 INSERT INTO…FROM 语句复制表数据：**

可以快速地从一个或多个表中取出数据，并将这些数据作为行数据插入另一个表中。

`INSERT INTO <表名> (字段1,字段2,...,字段n) <SELECT语句>`

注：结果集中的每行数据的字段数、字段的数据类型都必须与被操作的表完全一致。

实例②.将一个表查询到的所有记录，插入到另一个表中

`mysql> INSERT INTO test_db.tb_test2 (id,name,deptid,gongzi) SELECT id,name,deptid,salary FROM test_db.tb_test1;`

#### 修改数据

**使用语句：** `UPDATE`

`UPDATE <表名> SET 字段1=值1 [,字段2=值2… ] [WHERE 子句 ]
[ORDER BY 子句] [LIMIT 子句]`

- `WHERE 子句` 指定表中要修改的行，morning所有行。
- `ORDER BY 子句` 指定表中的行被修改的次序。
- `LIMIT 子句` 限定被修改的行数。

实例①.根据条件修改表中的数据

`UPDATE test_db.tb_test2 SET deptid=02,gongzi=22000 WHERE id=02;`

#### 删除数据

**使用语句：**`DELETE FROM`

`DELETE FROM <表名> [WHERE 子句] [ORDER BY 子句] [LIMIT 子句]`

实例①.清空表中所有数据

`DELETE FROM test_db.test_lb01`

实例②.根据条件删除表中数据

`INSERT INTO test_db.tb_test2 (id,name,deptid,gongzi) SELECT id,name,deptid,salary FROM test_db.tb_test1;`

#### 查询数据

**使用语句：**`SELECT ... FROM ...`

`SELECT
{* | <字段列名>}
[
FROM <表1>, <表2>…
[WHERE <表达式>
[GROUP BY <字段>
[HAVING <expression> [{<operator> <expression>}…]]
[ORDER BY <字段> [ASC|DESC]]
[LIMIT[<offset>,] <row count>]
]`

- `* | <字段列名>` 显示所有列，或指定列数据。
- `WHERE <表达式>` 指定查询条件。
  1. `<字段x> {=|<|>|<=|>=|!=} <value>` 
  2. `<字段x> [NOT] LIKE [%|_]value[%|_] ` %通配符，_匹配1个字符。
  3. `<字段x> [NOT] BETWEEN <value1> AND <value2>` 常指定时间区间。
  4. 多条件匹配：`WHERE <表达式1> AND <表达式2>`
- `GROUP BY <字段>` 按指定字段分组，显示查询出来的数据。
- `ORDER BY <字段> [ASC|DESC]` 按指定字段排序，默认升序(ASC)或降序(DESC)，显示查询出来的数据。
- `[LIMIT[<offset>,] <条数>]` 指定每次查询显示的数据条数。

示例①.查询表中的所有内容

`SELECT * FROM 表名;`

示例②.查询表中的指定字段

`SELECT <列名> FROM <表名>;`

示例③.显示表中的前4条数据

`SELECT * FROM <表名> LIMIT 4`

示例④.显示表中的第4条记录开始的5条数据

`SELECT * FROM <表名> LIMIT 3,5`

实例①.查询所有学生的身高并按身高从大到小排序

```shell
mysql> SELECT name,height FROM tb_student_info ORDER BY height,name ASC;
+--------+--------+
| name   | height |
+--------+--------+
| Henry  |    185 |
| Thomas |    178 |
| Jim    |    175 |
| John   |    172 |
| Susan  |    170 |
| Lily   |    165 |
```

实例②.查询年龄大于21且身高大于175的学生

```shell
mysql> SELECT * FROM tb_students_info
    -> WHERE age>21 AND height>=175;
+----+--------+---------+------+------+--------+------------+
| id | name   | dept_id | age  | sex  | height | login_date |
+----+--------+---------+------+------+--------+------------+
|  3 | Henry  |       2 |   23 | M    |    185 | 2015-05-31 |
|  5 | Jim    |       1 |   24 | M    |    175 | 2016-01-15 |
|  9 | Thomas |       3 |   22 | M    |    178 | 2016-06-07 |
+----+--------+---------+------+------+--------+------------+
```

实例③.查询注册日期在2015-10-01 和 2016-05-01之间的学生

```shell
mysql> SELECT * FROM tb_students_info
    -> WHERE login_date BETWEEN "2015-10-01" AND "2016-05-01";
```



补充①.**SELECT语句去重：** `SELECT DISTINCT <字段名> FROM <表名>;`

### 用户管理

#### 创建用户

**使用语句：**`CREATE USER`

`CREATE USER <用户名> [IDENTIFIED BY] [PASSWORD] <'口令'>`

- `IDENTIFIED BY` 表示指定口令创建。
- `PASSWORD` 表示口令是已经经过pasword()函数加密过的字串。

#### 修改用户

**修改用户账号：**`RENAME USER <旧用户> TO <新用户>`

**修改用户密码：**`SET PASSWORD [FOR <用户>] = PASSWORD('新明文口令')`

#### 删除用户

**使用语句：**`DROP USER <用户名1>[,<用户名2>]…`

注①.删除用户会自动撤销其原来权限。

注②.在没有明确给出账户的主机名时，默认人为`%`。

#### 用户授权

**使用语句：**`GRANT`


### 数据备份与恢复
#### mysqldump

**数据库备份：**
- 备份一个数据库：`mysqldump -u username -p dbname [tbname ...]> filename.sql`
- 备份多个数据库：`mysqldump -u username -P --databases dbname1 dbname2 ... > filename.sql`
- 备份所有数据库：`mysqldump -u username -P --all-databases>filename.sql`

**数据库恢复：**
`mysql -u username -p [dbname] < filename.sql`

#### infile 和 outfile
**文件导出：**
`select ... from 表名 into outfile "文件名" fields terminated by "分隔符" lines terminated by "\n"`
- `fields terminated`：指定字段分隔符，默认Tab。
- `lines terminated`：指定行分隔符，默认 \n。

**文件导入：**
`load data infile "文件名" into table 表名 fields terminated by "分隔符" lines terminated by "\n";`


### MySQL 日志

#### 日志分类
- 二进制日志：该日志文件会以二进制的形式记录数据库的各种操作（DDL 和 DML 语句），但不记录查询语句。
- 错误日志：该日志文件会记录 MySQL 服务器的启动、关闭和运行错误等信息。
- 通用查询日志：该日志记录 MySQL 服务器的启动和关闭信息、客户端的连接信息、更新、查询数据记录的 SQL 语句等。
- 慢查询日志：记录执行事件超过指定时间的操作，通过工具分析慢查询日志可以定位 MySQL 服务器性能瓶颈所在。
*注：*默认MySQL 只会启动错误日志。

#### 错误日志

SHOW 查看错误日志文件路径
```shell
mysql> SHOW VARIABLES LIKE 'log_error';
+---------------+---------------------+
| Variable_name | Value               |
+---------------+---------------------+
| log_error     | /var/log/mysqld.log |
+---------------+---------------------+
```

**错误日志常见错误信息：**
001）`Packets out of order`
简述：出现这个问题是因为 `packets` 超过了MySQL配置的值。
解决：
```shell
## 查看当前 max_allowed_packet 的值
mysql> show variables like 'max_allowed_packet';
+--------------------+---------+
| Variable_name      | Value   |
+--------------------+---------+
| max_allowed_packet | 4194304 |
+--------------------+---------+
## 修改 max_allowed_packet 为 20M
mysql> set global max_allowed_packet = 2*1024*1024*10;
#### 注：说明 ####
SET GLOBAL 设置全局变量只对新的会话连接生效，所以需要重新连接数据库才能看到是否设置成功。
SET GOLBAL 服务重启后失效，如果需要永久生效需要修改[mysqld]添加：max_allowed_packet = 20M
```

#### 二进制日志

**启动和设置二进制日志:**
```shell
mysql> SHOW VARIABLES LIKE 'log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | OFF   |
+---------------+-------+
## 启用二进制日志，需修改配置文件并重启服务
[mysqld]
log-bin=mysql-bin
server-id=1
server-id参数用于在复制中，为主库和备库提供一个独立的ID，以区分主库和备库；开启二进制文件的时候，需要设置这个参数。

## 查看二进制日志文件列表
mysql> SHOW binary logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |       154 |
+------------------+-----------+
或
cat mysql-bin.index
每次重启 MySQL 服务后，都会生成一个新的二进制日志文件

## 查看当前正在写入的二进制日志文件
mysql> SHOW master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      154 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+

## mysqld 配置扩展
expire_logs_days = 10 # 二进制日志自动删除的天数，默认值为 0，表示不删除
max_binlog_size = 1​00M # 定义了单个文件的大小限制，默认1GB

```
*注：*实际工作中，二进制日志文件与数据库的数据文件不放在同一块硬盘上，避免硬盘损坏导致数据丢失。

**删除二进制日志：**
1. 删除所有二进制日志
   `mysql> RESET MASTER;`
   删除所有二进制日志后，MySQL 将会重新创建新的二进制日志，新二进制日志的编号从 000001 开始。
2. 根据编号删除
   `PURGE MASTER LOGS TO 'filename.number';`
   该语句将删除编号小于 filename.number 的所有二进制日志。
3. 根据创建时间删除
   `PURGE MASTER LOGS TO 'yyyy-mm-dd hh:MM:ss';`
   该语句将删除在指定时间之前创建的所有二进制日志。

**暂停二进制日志：**
`SET SQL_LOG_BIN=0/1;` 暂停/开启二进制日志功能

**使用二进制日志还原数据库：**
备份：由于二进制日志占用大量的磁盘空间，因此，在备份 MySQL 数据库之后，应该删除备份之前的二进制日志。
还原：应该先使用最近的备份文件来还原数据库，再使用二进制日志来还原。还原操作时，必须是编号（number）小的先还原。

```shell
mysqlbinlog mylog.000001 | mysql -u root -p
mysqlbinlog mylog.000002 | mysql -u root -p
mysqlbinlog mylog.000003 | mysql -u root -p
```

#### 通用查询日志
通用查询日志（General Query Log）用来记录用户的所有操作，包括启动和关闭 MySQL 服务、更新语句和查询语句等。

```shell
## 查看 通用查询日志 信息
mysql>  SHOW VARIABLES LIKE '%general%';
+------------------+----------------------------------+
| Variable_name    | Value                            |
+------------------+----------------------------------+
| general_log      | OFF                              |
| general_log_file | /var/lib/mysql/test_server01.log |
+------------------+----------------------------------+

## 启用 通用查询日志
[mysqld]
log=general_log

```


#### 慢查询日志


```shell
## 查看慢查询日志信息
mysql> SHOW VARIABLES LIKE 'slow_query%';
+---------------------+---------------------------------------+
| Variable_name       | Value                                 |
+---------------------+---------------------------------------+
| slow_query_log      | OFF                                   |
| slow_query_log_file | /var/lib/mysql/test_server01-slow.log |
+---------------------+---------------------------------------+

mysql> SHOW VARIABLES LIKE 'long_query_time';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+

## 启用慢查询日志
[mysqld]
log-slow-queries=slow_log
long_query_time=10
和
SET GLOBAL slow_query_log=ON/OFF;
SET GLOBAL long_query_time=10;

```

