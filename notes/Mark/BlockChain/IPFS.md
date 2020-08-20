
# 概述
官网：https://ipfs.io/
文章：https://medium.com/chaindesk/ipfs-%E5%8C%BA%E5%9D%97%E9%93%BE-%E7%B3%BB%E5%88%97-%E5%85%A5%E9%97%A8%E7%AF%87-ipfs-ipns-%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA-3d18cce58e86
文章：www.chaindesk.cn
## 相关概念
### 什么是IPFS
IPFS:InterPlanetary File System，星际文件系统。是一个旨在创建持久且分布式存储和共享文件的网络传输协议。
- IPFS是开源的点对点的超媒体协议，面向全球，IPFS网络中的所有节点构成一个分布式文件系统。
- 完全去中心化，数据在IPFS上的存储都是以碎片的形式存储的，每个碎片的大小是256k，网络中的节点会对你的碎片进行存储。当你检索数据时，会将所有碎片收集起来，通过文件管理系统按序组合碎片得到你的文件。

### 什么是Filecoin
Filecoin是基于IPFS的分布式存储区块链项目。Filecoin在IPFS系统上加了一层激励机制，token是FIL。它旨在鼓励人们贡献他们的潜在存储空间和互联网带宽。用户可以在Filecoin上存储数据，也可以在Filecoin上检索数据。而矿工分为存储矿工和检索矿工，分别提供存储和数据检索。同时，经过选举成功的存储矿工，还有出块的功能，保证链的正常运行。

FIL代币总共有20亿枚。分配方案，总共有四个部分组成：
- 70%作为挖矿的回报：像比特币一样根据挖矿的进度逐步分发
- 15%预留Protocol Labs：作为研发费用, 6年逐步解禁
- 10%分配给ICO投资者： 根据挖矿进度, 逐步解禁
- 5%预留给Filecoin基金会： 作为长期社区建设, 网络管理等费用, 6年逐步解禁





# 部署

## go-ipfs
### 安装(Linux)
官方文档：https://docs.ipfs.io/how-to/command-line-quick-start/#install-ipfs
#### 安装IPFS
```shell
# 1. Download
wget https://github.com/ipfs/go-ipfs/releases/download/v0.6.0/go-ipfs_v0.6.0_linux-amd64.tar.gz
# 2. Unzip
tar -xvzf go-ipfs_v0.6.0_linux-amd64.tar.gz
# 3. Install
cd go-ipfs
sudo bash install.sh
ipfs --version
ipfs version 0.6.0
```

#### 初始化节点
1）**初始化存储库(Repository)**
`IPFS`将所有设置和内部数据存储在`Repository`，首次使用IPFS之前，需初始化该存储库。
```shell
$ ipfs init
initializing IPFS node at /home/centos/.ipfs
generating 2048-bit RSA keypair...done
peer identity: Qma657qVVWBXXcNZtJPGkfaWucycbx9DE7Np8covFHyzuL   # 节点ID，用于节点间通信
to get started, enter:

        ipfs cat /ipfs/QmQPeNsJPyVWPFDVHb77w8G42Fvo15z4bG2X8D2GhfbSXc/readme
$ ipfs cat /ipfs/QmQPeNsJPyVWPFDVHb77w8G42Fvo15z4bG2X8D2GhfbSXc/readme  # 查看相关说明
$ ipfs cat /ipfs/QmQPeNsJPyVWPFDVHb77w8G42Fvo15z4bG2X8D2GhfbSXc/quick-start   # 使用示例

```
2）运行加入IPFS
```shell
# 1. 运行节点
$ ipfs daemon
... ...
API server listening on /ip4/127.0.0.1/tcp/5001
WebUI: http://127.0.0.1:5001/webui
Gateway (readonly) server listening on /ip4/127.0.0.1/tcp/8080
# 2. 查看IPFS其它节点
## <transport address>/p2p/<hash-of-public-key>
$ ipfs swarm peers
/ip4/100.26.112.69/tcp/4001/p2p/QmSQxheQCKqggcRGEE1BG6nhAeyf2uiJjnhvRdBJnb6QgY
/ip4/111.229.116.215/tcp/4001/p2p/QmWGqaSFyiDYDxZgno4GL3zTzyBWWd4GeCayBEPzrENooC
... ...
# 3. 
```

3）**修改默认配置**
执行完ipfs init命令后，会在根目录生成一个.ipfs的文件夹存储节点数据。.ipfs节点默认存储空间为10个G。
*注：*
```shell
$ export EDITOR=/usr/bin/vim
$ ipfs config edit
# 保存退出重启ipfs
```

4）**跨域资源共享CORS配置**
```shell
ipfs config --json API.HTTPHeaders.Access-Control-Allow-Methods '["PUT", "GET", "POST", "OPTIONS"]'
ipfs config --json API.HTTPHeaders.Access-Control-Allow-Origin '["*"]'
ipfs config --json API.HTTPHeaders.Access-Control-Allow-Credentials '["true"]'
ipfs config --json API.HTTPHeaders.Access-Control-Allow-Headers '["Authorization"]'
ipfs config --json API.HTTPHeaders.Access-Control-Expose-Headers '["Location"]'
```

#### 文件上传下载

```shell
mdkir 0803
touch 0803/file.txt
# 1. 将文件添加到ipfs当前节点
$ ipfs add file.txt
added QmRXGk5jzDbpgvdRLuRpQcwmzdoTCozA7AiRXhu3iT9Pgj file.txt
$ cat file.txt
hello ipfs 0803...
$ ipfs cat QmRXGk5jzDbpgvdRLuRpQcwmzdoTCozA7AiRXhu3iT9Pgj
hello ipfs 0803...

# 2. 同步到IPFS网络
ipfs daemon

# 3. 浏览器访问https://ipfs.io/ipfs/QmRXGk5jzDbpgvdRLuRpQcwmzdoTCozA7AiRXhu3iT9Pgj

```
#### 简单站点搭建
```shell
$ ipfs add -r site/
added QmTVJ4XtUhqb6KMW8kxDwArVweACcy7VXAfinEks9Fd8cJ site/index.html
added QmfGLJ3mryLvicQqzdsghq4QRhptKJtBAPe7yDJxsBGSuy site/style.css
added QmPzuBfgH2ox4Eujj1THd6ykLiceyEqB9jecuAzPzd3nPV site

# 浏览器访问 根目录 hash
https://ipfs.io/ipfs/QmPzuBfgH2ox4Eujj1THd6ykLiceyEqB9jecuAzPzd3nPV/
或 https://ipfs.io/ipfs/QmPzuBfgH2ox4Eujj1THd6ykLiceyEqB9jecuAzPzd3nPV/index.html

```
*注：*当修改网站内容重新添加到ipfs时，hash会发生变化，我们可以将网站发布到IPNS，在IPNS中，允许我们节点的域名空间中引用一个IPFS hash。


#### 故障排除


