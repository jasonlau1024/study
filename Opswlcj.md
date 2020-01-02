# 钱包机

## 运行环境

java+sqlite3+node

sqlite3 使用系统预安装的即可。



### 部署

```shell
## 创建相关目录
mkdir /home/{app,db,log,sign}
mkdir /home/conf/{eth,eos,btc,ltc,bch}
mkdir /home/db/{eth,eos,btc,ltc,bch,sign}
## 拷贝文件到/home/sign
cd /home/sign/coinSign/database/
rm -f bchaddress.txt btcaddress.txt ltcaddress.txt
sqlite3 coinsign.db
## 创建以太坊签名数据库
cd /home/db/sign
sqlite3 ethsign.db
## 创建 eth bch btc eos ltc coin数据库
cd /home
sqlite3 bch/bchabc.db
sqlite3 btc/bitcoin.db
sqlite3 eos/eoscoin.db
sqlite3 ltc/litecoin.db
sqlite3 eth/eth.db
## 复制相关配置文件到/home/app/conf对应的目录下
cat /home/app/conf/bch/application.yml
## 复制相关jar包到/home/app/jar对应的目录下
ll /home/app/app/bch/

## 生产以太坊eth地址
java -jar eth-sign-0.0.1-SNAPSHOT.jar 100000
## 其它地址
java -jar bchabc-0.0.1-SNAPSHOT.jar 100000
java -jar bchabc-0.0.1-SNAPSHOT.jar 100000
java -jar btc-0.0.1-SNAPSHOT.jar 100000
java -jar ltc-0.0.1-SNAPSHOT.jar 100000
# 生成完地址后，kill 掉对应的java进程
vi bchaddress.txt
vi btcaddress.txt
vi ltcaddress.txt
vi ethaddress.txt
## 拷贝地址到相应目录
ls /home/db/sign/ethaddress.txt
cp ethaddress.txt /home/db/eth/
## 导入地址
cd /home/db/eth/
python importAddress.py ethaddress.txt
sqlite3 eth.db
## 修改配置文件启动服务




## 安装 node npm pm2
yum install node
yum install npm
npm install pm2 # npm install -g pm2

sqlite3 /home/sign/coinSign/database/coinsign.db

## 运行 pm2 
pm2 start /home/sign/coinSign/bin/www

mkdir -p /home/{app/{jar,conf/{eth,eos,btc,ltc,bch}},db,log/{eth,eos,ltc,bch,btc},sign}

```



#### 目录结构说明

```shell



# tree db/
db/
├── bch
│   └── bchabc.db
├── btc
│   └── bitcoin.db
├── eos
│   └── eoscoin.db
├── eth
│   ├── ethaddress.txt
│   ├── eth.db
│   └── importAddress.py
├── ltc
│   └── litecoin.db
└── sign
    ├── ethaddress.txt
    └── ethsign.db
```





#### 配置文件说明：

```shell
spring:
  datasource:
    password:
    driver-class-name: org.sqlite.JDBC
    url: jdbc:sqlite:/Users/lch/Workspace/RECWorkspace/eth-wallet/eth-wallet/db/eth.db
    username:
server:
  port: 8888

wallet:
  appid: at
  appsecret: atXHs0yB
  mainAddress: de8237889bce734bb20821fdeff2be2a3df4b5ff
  feeAccountAddress: de8237889bce734bb20821fdeff2be2a3df4b5ff
  maxBalance:
  minBalance:
  maxTokenBalance:
  minTokenBalance:
  transferValue:
  tokentransferValue:
  coldAddress:
  usdtCollectionValue: 100

sign:
  rpc:
    url: http://127.0.0.1:8000/

financeapi:
  rpc:
    url: http://43.240.204.157:8158/

eth:
  rpc:
    url: http://47.56.115.230:8565/
```





## 测试环境部署USDT钱包

说明：USDT是ETH上的代币

### 签名程序部署

1）创建签名数据库

`sqlite3 /home/db/sign/ethsign.db`

创建`account` 表：

```shell
CREATE TABLE "account"
(
        address varchar(150) not null,
        privatekey varchar(250) not null,
        createDate Date
);
## 查看表结构
sqlite> .schema account
```

2）生成地址：

```shell
java -jar eth-sign-0.0.1-SNAPSHOT.jar 200  ## 地址数根据需要制定，一般测试200，生产100000
```

3）导出地址：`address.txt`

```shell
## 导出生成的地址到
sqlite3 ethsign.db
sqlite> .output address.txt
sqlite> select address from account;
sqlite> .output stdout
或
sqlite3 ethsign.db "select address from account;" > address.txt
```

【注】：

1. 该地址需要导入钱包数据库的address表里。可使用脚本`importAddress.py`
2. 取里面的一个地址作为热钱包和手续费地址，将type改为0。
3. 在发给PHP的地址文件中一定要删除掉系统使用地址，否则可能作为用户地址被分配给用户【慎记】

4）运行签名程序

`nohup java -jar /home/apps/jar/eth-sign-0.0.1-SNAPSHOT.jar >> /home/apps/jar/eth-sign.log 2>&1 &`





### 钱包程序部署

1）创建USDT钱包数据库：

```shell
## 创建数据库，并创建5张表，
sqlite3 /home/db/eth/eth.db
## 创建 Address 表
sqlite> CREATE TABLE Address
(
address varchar(150)
primary key,
type int default 1,
amount DECIMAL(32,18)
, nonce INTEGER default 0);
## 创建 DepositRecord 表
sqlite> CREATE TABLE "DepositRecord"
(
        txid varchar(250),
        address varchar(150),
        amount DECIMAL(32,18),
        blockHeight INTEGER,
        collectionTxid varchar(250),
        collectionFee INTEGER,
        status int default 0,
        collectionStatus int default 0,
        token int default 0,
        sendFeeStatus int default 0
, contractAddress varchar(150), sendFeeTxid varchar(250));
## 创建 Tx 表
sqlite> CREATE TABLE "Tx"
(
        txid varchar(250)
                primary key,
        address varchar(150),
        amount DECIMAL(32,18),
        Id Integer default 0
);
## 创建 ContractAddress 表
sqlite> CREATE TABLE ContractAddress
(
        address varchar(150)
                constraint ContractAddress_pk
                        primary key,
        symbol varchar(50),
        decimal int
);
sqlite> INSERT INTO "ContractAddress" VALUES('0xdac17f958d2ee523a2206206994597c13d831ec7','usdt',6);
sqlite> CREATE UNIQUE INDEX ContractAddress_address_uindex
        on ContractAddress (address);
## 创建 MainChain 表
sqlite> CREATE TABLE MainChain
(
hash String,
id int,
height Integer
);
sqlite> INSERT INTO "MainChain" VALUES(NULL,1,8460503); ## 8460503 这个是区块的高度，上线时改成eth最新的。
```

补充： USDT Wallet 数据库初始化脚本

```shell
# cat usdt_wallet_int.sql
PRAGMA foreign_keys=OFF;
BEGIN TRANSACTION;
CREATE TABLE Address
(
address varchar(150)
primary key,
type int default 1,
amount DECIMAL(32,18)
, nonce INTEGER default 0);
CREATE TABLE MainChain
(
hash String,
id int,
height Integer
);
INSERT INTO "MainChain" VALUES(NULL,1,8460503);
CREATE TABLE "Tx"
(
        txid varchar(250)
                primary key,
        address varchar(150),
        amount DECIMAL(32,18),
        Id Integer default 0
);
CREATE TABLE "DepositRecord"
(
        txid varchar(250),
        address varchar(150),
        amount DECIMAL(32,18),
        blockHeight INTEGER,
        collectionTxid varchar(250),
        collectionFee INTEGER,
        status int default 0,
        collectionStatus int default 0,
        token int default 0,
        sendFeeStatus int default 0
, contractAddress varchar(150), sendFeeTxid varchar(250));
CREATE TABLE ContractAddress
(
        address varchar(150)
                constraint ContractAddress_pk
                        primary key,
        symbol varchar(50),
        decimal int
);
INSERT INTO "ContractAddress" VALUES('0xdac17f958d2ee523a2206206994597c13d831ec7','usdt',6);
CREATE UNIQUE INDEX ContractAddress_address_uindex
        on ContractAddress (address);
COMMIT;
```

2）导入签名生成的地址到钱包数据库

```shell
$ ls
ethaddress.txt  eth.db  importAddress.py
$ python importAddress.py ethaddress.txt
sqlite> select count() from address;
200
## 选择一个地址设置为系统使用地址,将其type设置为0
sqlite> select * from Address where address='0x37a94c21c48e897eeb4eb96697be5971e1eb7a6a';
0x37a94c21c48e897eeb4eb96697be5971e1eb7a6a|1|0|0
sqlite> UPDATE Address SET type=0 WHERE address='0x37a94c21c48e897eeb4eb96697be5971e1eb7a6a';
sqlite> select * from Address where address='0x37a94c21c48e897eeb4eb96697be5971e1eb7a6a';
0x37a94c21c48e897eeb4eb96697be5971e1eb7a6a|0|0|0
sqlite> select * from Address where type=0;
0x37a94c21c48e897eeb4eb96697be5971e1eb7a6a|0|0|0
## 拷贝一份地址，将里面的系统地址删除【一定要删除】，发给PHP导入MySQL数据库
cp ethaddress.txt test_eth_address.txt
sed -i '/0x37a94c21c48e897eeb4eb96697be5971e1eb7a6a/d' test_eth_address.txt
grep "0x37a94c21c48e897eeb4eb96697be5971e1eb7a6a" test_eth_address.txt
wc -l test_eth_address.txt
199 test_eth_address.txt
```

补充：导入地址python脚本

```shell
$ cat 
#coding=utf-8
import sqlite3            #导入sqlite3
import sys
cx = sqlite3.connect('eth.db')   #创建数据库，如果数据库已经存在，则链接数据库；如果数据库不存在，则先创建数据库，再链接该数据库。
cu = cx.cursor()                     #定义一个游标，以便获得查询对象。
address = sys.argv[1]
fr = open(address)        #打开要读取的txt文件
i = 0
for line in fr.readlines():        #将数据按行插入数据库的表train4中。
    line="".join(line.split())
    cu.execute('insert into address (address,type,amount) values(?,?,?)',(line,1,0))
    i +=1
cu.close()      #关闭游标
cx.commit()     #事务提交
cx.close()      #关闭数据库
```

3）运行USDT钱包程序

```shell
## 配置文件
cat /home/apps/conf/eth/application.yml
spring:
  datasource:
    password:
    driver-class-name: org.sqlite.JDBC
    url: jdbc:sqlite:/home/db/eth/eth.db  ## 钱包数据库
    username:
server:
  port: 8087  ## 运行服务端口

wallet:
  appid: rmcusdtpay    ## php接口appid，可在后管系统管理查看
  appsecret: 20191114  ## php接口appsecret
  mainAddress: 0x37a94c21c48e897eeb4eb96697be5971e1eb7a6a  ## 归集地址
  feeAccountAddress: 0x37a94c21c48e897eeb4eb96697be5971e1eb7a6a  ## 手续费地址，以太坊
  # 归集地址和手续费地址是钱包数据库上type的值0的地址
  phone:
  maxBalance: 120  ## ETH热钱包持有ETH币最大值
  minBalance: 0    ## ETH热钱包持有ETH币最小值
  maxTokenBalance: 30000  ## ETH热钱包持有代币（usdt等）最大值
  minTokenBalance: 0      ## ETH热钱包持有代币（usdt等）最小值
  transferValue: 60       ## ETH热钱包ETH热转冷数量
  tokentransferValue: 10000000000  ## ETH热钱包代币（usdt等）热转冷数量
  coldAddress: 0x2516a850714429cd3086896772BfeEBC88ad39CF  ## ETH冷钱包地址
  usdtCollectionValue: 0  ## ETH归集usdt的最小值，小于这个值不归集

sign:
  rpc:
    url: http://127.0.0.1:8000/  ## 签名服务
financeapi:
  rpc:
    url: http://127.0.0.1:8084/api/blockwallet/  ## php finance 接口
eth:
  rpc:
    url: http://47.56.115.230:8565/  ## eth 区块节点
## 运行
nohup java -jar /home/apps/jar/eth-wallet-0.0.1-SNAPSHOT.jar --spring.config.location=/home/apps/conf/eth/application.yml > /home/logs/eth/eth.log 2>&1 &
```

### 导入用户地址到MySQL数据库

**注意：**该地址文件一定要去掉系统地址，避免系统地址成为用户地址，切记！

这里以测试环境的`phoenixs`导入用户地址到表`fa_receive_code`，其它项目或币种的表可能不同。

```shell
load data local infile '/home/db/eth/test_eth_address.txt' into table fa_receive_code(code);
## 如果需清空表旧数据，可使用 truncate table 表名;删除数据以及让自动增长id归0
```









示例：6个Java程序和1个node程序。

服务启动：

```shell
# node 程序
pm2 start /home/sign/coinSign/bin/www

$ nohup java -jar /home/app/jar/eth-wallet-0.0.1-SNAPSHOT.jar --spring.config.location=/home/app/conf/eth/application.yml > /home/logs/eth/eth.log 2>&1 &
$ nohup java -jar /home/app/jar/btc-0.0.1-SNAPSHOT.jar --spring.config.location=/home/app/conf/btc/application.yml > /home/logs/btc/btc.log 2>&1 &
$ nohup java -jar /home/app/jar/eoswallet-0.0.1-SNAPSHOT.jar --spring.config.location=/home/app/conf/eos/application.yml > /home/logs/eos/eos.log 2>&1 &
$ nohup java -jar /home/app/jar/bchabc-0.0.1-SNAPSHOT.jar --spring.config.location=/home/app/conf/bch/application.yml > /home/logs/bch/bch.log 2>&1 &
$ nohup java -jar /home/app/jar/ltc-0.0.1-SNAPSHOT.jar --spring.config.location=/home/app/conf/ltc/application.yml > /home/logs/ltc/ltc.log 2>&1 &

nohup java -jar /home/app/jar/eth-sign-0.0.1-SNAPSHOT.jar /home/app/jar/eth-sign.log 2>&1 &
```



服务查看：

```shell
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:6088            0.0.0.0:*               LISTEN      3008/java
tcp        0      0 0.0.0.0:6188            0.0.0.0:*               LISTEN      2824/java
tcp        0      0 0.0.0.0:6286            0.0.0.0:*               LISTEN      2871/java
tcp        0      0 0.0.0.0:6288            0.0.0.0:*               LISTEN      2970/java
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1590/sshd
tcp        0      0 0.0.0.0:8888            0.0.0.0:*               LISTEN      2915/java
tcp        0      0 0.0.0.0:8000            0.0.0.0:*               LISTEN      2706/java
tcp        0      0 0.0.0.0:10050           0.0.0.0:*               LISTEN      3714/zabbix_agentd
tcp6       0      0 :::3000                 :::*                    LISTEN      2295/node /home/sig
tcp6       0      0 :::3038                 :::*                    LISTEN      2295/node /home/sig
tcp6       0      0 :::10050                :::*                    LISTEN      3714/zabbix_agentd
```



## 交易排查

### 日志查看

1. 拿到交易hashtx

   可通过交易单号查询`fa_order_info` 表获得`pay_addr`    （order_no订单号字段)

   如：`select * from fa_order_info where order_no=<交易单号>`

   `pay_addr:即系统分配给用户地址`

   `return_address 即回款地址`

   也可查询php日志。`app../log` 和`runtime/log`

2.  查询钱包日志

   查询钱包日志获取记录段到临时文件，再通过vim打开匹配`pay_addr`

3. 查询PHP日志

   `app../log` 和`runtime/log`





# Web 机

## 运行环境

### 安装 nginx

```shell
## Yum 安装
cat > /etc/yum.repos.d/nginx.repo <<"EOF"
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
EOF
yum install nginx -y
## 配置 nginx

```

### 安装 php

```shell
## 添加 PHP Yum 源
# CentOs 7.X：
rpm -Uvh https://mirror.webtatic.com/yum/el7/epel-release.rpm
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
## 安装PHP以及扩展
yum install php71w-cli php71w-mysql php71w-bcmath php71w-gd php71w-mcrypt php71w-mbstring php71w-odbc php71w-xml php71w-fpm
## 配置 php
```

### 部署代码

1. git clone

2. 修改部分文件名

   ```shell
   cp ./public/assets/js/{addons.example.js,addons.js}
   cp ./application/{config.example.php,config.php}
   cp ./application/extra/{addons.example.php,addons.php}
   cp ./application/extra/{site.example.php,site.php}
   cp ./application/{database.example.php,database.php}
   ```

3. 修改数据库配置文件`application/database.php`

4. 修改`application/php.config`文件

   ```shell
   # 将下面对应的值改为false:
   app.debug
   app.trace
   
   # 将下面对应的值改为true:
   login_captcha
   login_failure_retry
   login_unique
   ```

### 数据导出导入

导出测试环境数据库：

`mysqldump -uroot -P63956 -hgz-cdb-qmrb10se.sql.tencentcdb.com -p btf > btf-$(date +%F).sql`

导入生产环境数据库：

`mysql -hrm-3ns3mqsl8ey45392y.mysql.rds.aliyuncs.com -P3306 -uroot -p btf < ./btf-xx.sql`

## 维护记录

### 修改文件上传大小限制

```shell
## nginx
client_max_body_size 50M;
## php.ini:
max_execution_time = 600  ## 每个PHP页面运行的最大时间值(秒)，默认30秒
max_input_time = 600  ## 每个PHP页面接收数据所需的最大时间，默认60秒
memory_limit = 128M    ## 每个PHP页面所吃掉的最大内存，默认8M
file_uploads = On  ## 是否允许通过HTTP上传文。默认为ON
upload_max_filesize = 50M  ## 允许上传文件大小的最大值.
post_max_size = 50M  ## 通过表单POST给PHP的所能接收的最大值.
```



# 区块节点

## BTC 节点

### 部署

```shell
wget https://github.com/OmniLayer/omnicore/releases/download/v0.5.0/omnicore-0.5.0-x86_64-linux-gnu.tar.gz
tar -zxvf omnicore-0.5.0-x86_64-linux-gnu.tar.gz
cd omnicore-0.5.0


## 启动
nohup /data/omnicore/bin/omnicored –conf=/data/omnicore/conf/bitcoin.conf 2>/root/omni/data/btclog.log &

cat /root/.bitcoin/bitcoin.conf
daemon=1 # 后台运行
txindex=1  ## 所有交易进行索引；否则只保留钱包地址交易索引记录
listen=1  ## 监听模式，默认启动
server=1  ## 允许bitcoin接收JSON-RPC
rpcuser=rpcuser
rpcpassword=rpcpasswd
rpcport=8368
rpcallowip=0.0.0.0/0  ## 允许IP
datadir=/data/omnicore/data
```









## ETH 节点

### 部署

```shell
## 下载最新版本的 geth 二进制包
wget https://gethstore.blob.core.windows.net/builds/geth-linux-amd64-1.9.9-01744997.tar.gz
tar xf 
## 启动 eth 节点服务
nohup ./geth --datadir /root/eth/mainnet/ --syncmode=fast --cache=10240 --maxpeers 900 --rpc --rpcaddr 0.0.0.0 --rpcport 8565 --rpcapi admin,debug,eth,miner,net,personal,shh,txpool,web3 &
```











# Godaddy

## 域名使用注意

1. 去掉godaddy上一级域名默认跳到goddady页面的设定记录。
2. 所有已在使用对外的域名需要做隐私保护。