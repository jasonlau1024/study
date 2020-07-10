

## 部署Geth节点
### 编译安装
- 安装编译依赖环境：
  `yum install git wget bzip2 vim gcc-c++ ntp epel-release nodejs cmake -y`
- 安装 Go 环境：
  ```shell
cd /usr/local/src
wget https://dl.google.com/go/go1.11.linux-amd64.tar.gz
tar xf go1.11.linux-amd64.tar.gz
mv go /usr/local/
echo "export GOROOT=/usr/local/go" >> /etc/profile
echo "export PATH=$PATH:/usr/local/go/bin" >> /etc/profile
source /etc/profile
go version
  ```
- 编译安装 Geth:
  
  ```shell
wget https://codeload.github.com/ethereum/go-ethereum/tar.gz/v1.8.15
  # https://github.com/ethereum/go-ethereum/archive/v1.8.20.tar.gz
  mv v1.8.15 v1.8.15.tar.gz
  tar xf v1.8.15.tar.gz
  cd go-ethereum-1.8.15/
  make all
  cd ..
  mv go-ethereum-1.8.1 go-ethereum
  echo 'export PATH=$PATH:/usr/local/go-ethereum/build/bin' >> /etc/profile
  source /etc/profile
  geth version
  ```
- 运行 Geth 节点

  `nohup geth --rpc --rpcport 8645 --rpcaddr=172.16.0.246 --datadir /application/coin/eth/data/ --maxpeers 100 &`

### 二进制运行节点

下载地址：https://geth.ethereum.org/downloads/

`wget https://gethstore.blob.core.windows.net/builds/geth-linux-amd64-1.9.15-0f77f34b.tar.gz`

运行节点：

`./geth --datadir /data/gethdata --syncmode=fast --cache=10240 --maxpeers 900 --rpc --rpcaddr 0.0.0.0 --rpcport 8565 --rpcapi admin,debug,eth,miner,net,personal,shh,txpool,web3`

### Geth 配置说明
- 同步模式：--syncmode
  - fast Enable fast syncing through state downloads
  - light Enable light client mode
  - syncmode full
  
  ```shell
  ## --syncmode=full
  获取区块的header
  获取区块的body
  从创始块开始校验每一个元素(下载所有区块数据信息)
  ## --syncmode=fast
  获取区块的header
  获取区块的body
  在同步到当前块之前不处理任何事务，然后获得一个快照，像full节点一样进行后面的同步操作。
  注：沿着区块下载最近数据库中的交易，有可能丢失历史数据。比如，你的账户地址A上面有10个ETH，但转入的的交易存在于较老的历史交易中，此同步模式无法获取到交易的详细情况。
  补充：使用此模式时注意需要设置–cache，默认16M，建议设置为1G（1024）到2G（2048）。
  ## --syncmode=ligth
  仅获取当前状态。验证元素需要向full节点发起相应的请求。
  ```
  
  

### 查看同步信息
```shell
> eth.syncing
{
  currentBlock: 6713522,
  highestBlock: 6713605,
  knownStates: 169824244,
  pulledStates: 169809794,
  startingBlock: 0
}
## 说明
startingBlock：开始同步的起始区块编号；
currentBlock：当前正在导入的区块编号；
highestBlock：通过所链接的节点获得的当前最高的区块高度；
pulledStates：当前已经拉取的状态条目数；
knownStates：当前已知的待拉取的总状态条目数。
```

### 使用 Supervisor 管理 Geth 进程
```shell
mkdir -p /var/log/supervisor/geth
cat /etc/supervisord.d/geth.ini

[program:geth]
command=/data/geth-linux-amd64-1.9.9-01744997/geth --datadir /data/gethdata --syncmode=fast --cache=10240 --maxpeers 900 --rpc --rpcaddr 0.0.0.0 --rpcport 8565 --rpcapi admin,debug,eth,miner,net,personal,shh,txpool,web3
directory=/data/geth-linux-amd64-1.9.9-01744997
user=root
autostart=true
autorestart=true
startretries=9999
exitcodes=0
stopsignal=KILL
stopwaitsecs=10
redirect_stderr=true
logfile_backups=10
stdout_logfile_maxbytes=8MB
stdout_logfile=/var/log/supervisor/geth/geth.log

supervisorctl reload
supervisorctl restart geth
```

### Geth 区块同步监控
Shell脚本
```shell
#!/bin/bash
# check.geth.sh
# 区块高度监控

#定时任务
#check blockchain
#*/4 * * * * bash /opt/shell/check.geth.sh

#var
eth_api=127.0.0.1:8565
Stime=180

H1=$((`curl -s -X POST -H "Content-Type":application/json --data '{"jsonrpc":"2.0", "method":"eth_blockNumber","params":[],"id":67}'  $eth_api |awk -F'"' '{print $(NF-1)}'`))

sleep $Stime

H2=$((`curl -s -X POST -H "Content-Type":application/json --data '{"jsonrpc":"2.0", "method":"eth_blockNumber","params":[],"id":67}'  $eth_api |awk -F'"' '{print $(NF-1)}'`))

# zabbix
Status_file=/tmp/geth_status.log
echo 0 > ${Status_file}
[[ $H1 -eq $H2 ]] && echo 1 > ${Status_file}


# restart geth
if [[ $H1 -eq $H2 ]];then
    echo "geth restart at  $(date +%F" "%T)  block $H1" >>/tmp/geth.restart.log
    supervisorctl restart geth &>/dev/null
fi
```

结合 zabbix：

```shell
修改zabbix_agentd.conf配置文件
UnsafeUserParameters=1  （默认为0，即不可以自定义）
UserParameter=geth.rsyncing[*],cat /tmp/geth_status.log

systemctl restart zabbix-agent

测试：
zabbix_get -s AgentIP -p 10050 -k "geth.rsyncing"
1

在对应的主机下
创建监控项（键值geth.rsyncing,更新间隔5m）
创建触发器（监控项选择上面创建的，last()=1 触发）
```


