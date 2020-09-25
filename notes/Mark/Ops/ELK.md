
# 概述

官网地址：https://www.elastic.co/guide/cn/index.html
中文文档：https://elkguide.elasticsearch.cn/

    ELK 是一款开源实时日志分析平台。“ELK”是三个开源项目的首字母缩写，这三个项目分别是：Elasticsearch、Logstash 和 Kibana。
    Elastic Stack，是ELK Stack 在 5.0 版本加入 Beats 套件后的新称呼（新增FileBeat）。
    Elastic Stack 目前已成为机器数据分析，或者说实时日志处理领域，开源界的第一选择。

**ELK Stack 组件：**
- Elasticsearch 
  分布式搜索和分析引擎，具有高可伸缩、高可靠和易管理等特点。基于 Apache Lucene 构建，能对大容量的数据进行接近实时的存储、搜索和分析操作。通常被用作某些应用的基础搜索引擎，使其具有复杂的搜索功能；
- Logstash
  数据收集引擎。它支持动态的从各种数据源搜集数据，并对数据进行过滤、分析、丰富、统一格式等操作，然后存储到用户指定的位置，如：elasticsearch。
- Kibana
  数据分析和可视化平台。通常与 Elasticsearch 配合使用，对其中数据进行搜索、分析和以统计图表的方式展示；
- FileBeat
  ELK 协议栈的新成员，一个轻量级开源日志文件数据搜集器，基于 Logstash-Forwarder 源代码开发，是对它的替代。在需要采集日志数据的 server 上安装 Filebeat，并指定日志目录或日志文件后，Filebeat 就能读取数据，迅速发送到 Logstash 进行解析，亦或直接发送到 Elasticsearch 进行集中式存储和分析。

**ELK Stack架构：**
![img](https://img-blog.csdnimg.cn/20190627104608534.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zODA4ODQ4NQ==,size_16,color_FFFFFF,t_70)

- `Input`
  输入，数据源可以是Stdin、File、TCP、Redis、Syslog等。（https://www.elastic.co/guide/en/logstash/current/input-plugins.html）
- `Filter`
  过滤，将日志格式化。插件有Grok正则捕获、Date时间处理、Json编解码、Mutate数据修改等。（https://www.elastic.co/guide/en/logstash/current/filter-plugins.html）
- `Output`
  输出，输出目标可以是Stdout、File、TCP、Redis、ES等。（https://www.elastic.co/guide/en/logstash/current/output-plugins.html）


**ELK Stack 优点：**
1）处理方式灵活。Elasticsearch 是实时全文索引，不需要像 storm 那样预先编程才能使用；
2）配置简易上手。Elasticsearch 全部采用 JSON 接口，Logstash 是 Ruby DSL 设计，都是目前业界最通用的配置语法设计；
3）检索性能高效。虽然每次查询都是实时计算，但是优秀的设计和实现基本可以达到全天数据查询的秒级响应；
4）集群线性扩展。不管是 Elasticsearch 集群还是 Logstash 集群都是可以线性扩展的；
5）前端操作炫丽。Kibana 界面上，只需要点击鼠标，就可以完成搜索、聚合功能，生成炫丽的仪表板。


**ELK常用架构及使用场景：**
架构1.
各个节点日志由Logstash Agent收集、分析、过滤后发送给Elasticsearch进行存储。
Elasticsearch将数据以分片的形式压缩存储并提供多种API供用户查询，操作。
用户通过配置Kibana Web 对日志查询，并根据数据生成报表。
架构2.引入消息队列机制
各个节点上的Logstash Agent先将数据/日志传递给Kafka（或者Redis），并将队列中消息或数据间接传递给Logstash，Logstash过滤、分析后将数据传递给Elasticsearch存储。
说明：这种架构适合于日志规模比较庞大的情况。但由于 Logstash 日志解析节点和 Elasticsearch 的负荷比较重，可将他们配置为集群模式，以分担负荷。引入消息队列，均衡了网络传输，从而降低了网络闭塞，尤其是丢失数据的可能性，但依然存在 Logstash 占用系统资源过多的问题。

架构3.将收集端logstash Agent替换为beats
更灵活，消耗资源更少，扩展性更强。Beats 将搜集到的数据发送到 Logstash，经 Logstash 解析、过滤后，将其发送到 Elasticsearch 存储，并由 Kibana 呈现给用户。
说明：这种架构解决了 Logstash 在各服务器节点上占用系统资源高的问题。相比 Logstash，Beats 所占系统的 CPU 和内存几乎可以忽略不计。另外，Beats 和 Logstash 之间支持 SSL/TLS 加密传输，客户端和服务器双向认证，保证了通信安全。
因此这种架构适合对数据安全性要求较高，同时各服务器性能比较敏感的场景。

目前 Beats 包括四种：
- Packetbeat（搜集网络流量数据）；
- Topbeat（搜集系统、进程和文件系统级别的 CPU 和内存使用情况等数据）；
- Filebeat（搜集文件数据）；
- Winlogbeat（搜集 Windows 事件日志数据）。

## ElaticSearch
**基本概念：**
- Node：运行单个ES实例的服务器.
- Cluster：一个或多个节点构成集群
- Index：索引是多个文档的集合
- Document：Index里每条记录称为Document，若干文档构建一个Index
- Type：一个Index可以定义一种或多种类型，将Document逻辑分组
- Field：ES存储的最小单元
- Shards：ES将Index分为若干份，每一份就是一个分片
- Replicas：Index的一份或多份副本。
**相关概念类比：**
| ElasticSearch | 关系型数据库（比如MySQL） |
| ------------- | ------------------------- |
| Index         | Database                  |
| Type          | Table                     |
| Document      | Row                       |
| Field         | Column                    |







# 部署

## 二进制部署ELK Stack集群
参考文档：https://blog.csdn.net/weixin_38088485/article/details/93854306
### 实验环境



### ES 集群
相关机器：`elk-es01 elk-es02 elk-es03`
```shell
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
cat > /etc/yum.repos.d/elastic.repo << EOF
[elastic-7.x]
name=Elastic repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
EOF

yum install -y elasticsearch
# 也可直接下载RPM包安装

# 集群状态检查，green表示集群健康
# curl -X GET "192.168.36.203:9200/_cat/health?v"
epoch      timestamp cluster     status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1599615721 01:42:01  elk-cluster green           3         3      0   0    0    0        0             0                  -                100.0%

# 查看集群节点信息
# curl -X GET "192.168.36.203:9200/_cat/nodes?v"
ip             heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
192.168.36.203           25          96   0    0.00    0.01     0.05 dilmrt    *      node-01
192.168.36.205           12          97   9    0.07    0.27     0.18 dilmrt    -      node-03
192.168.36.204           18          96   0    0.00    0.01     0.05 dilmrt    -      node-02

```
### ES数据操作
#### RestFul API格式
**RestFul API格式：**`curl -X<verb> '<protocol>://<host>:<port>/<path>?<query_string>' -d '<body>'`
-X 指定动作（GET POST PUT等）
| 参数         | 描述                                                    |
| ------------ | ------------------------------------------------------- |
| verb         | HTTP方法，比如GET、POST、PUT、HEAD、DELETE              |
| host         | ES集群中的任意节点主机名                                |
| port         | ES HTTP服务端口，默认9200                               |
| path         | 索引路径                                                |
| query_string | 可选的查询请求参数。例如?pretty参数将格式化输出JSON数据 |
| -d           | 里面放一个GET的JSON格式请求主体                         |
| body         | 自己写的 JSON格式的请求主体                             |

#### 索引
**创建索引：**
`curl -X PUT "192.168.36.203:9200/logs-test-2019.06.20?pretty"`
`?pretty`:表示是采用json格式化显示结果

**列出索引：**
`curl -X GET "192.168.36.203:9200/_cat/indices?v"`

**向索引添加内容：**
```SHELL
curl -X PUT "192.168.36.203:9200/logs-test-2019.06.20/_doc/1?pretty" -H 'Content-Type: application/json' -d '
{
  "name": "LAO WANG"
}'
```

**获取索引内容：**
`curl -X GET "192.168.36.203:9200/logs-test-2019.06.20/_doc/1?pretty"`

**删除索引：**
`curl -X DELETE "192.168.36.203:9200/logs-test-2019.06.20?pretty"`

**修改数据：**
```shell
# 可通过PUT或POST修改
curl -X PUT "192.168.36.203:9200/logs-test-2019.06.20/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "Lisi"
}'
```
**增加字段：**
```shell
curl -X POST "192.168.36.203:9200/logs-test-2019.06.20/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "Lisi","age": 20
}'
```

#### 多类型查询
官方文档：https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-search.html

**导入测试数据**
```shell
# 测试数据
wget https://raw.githubusercontent.com/elastic/elasticsearch/master/docs/src/test/resources/accounts.json

# 导入测试数据
curl -H "Content-Type: application/json" -XPOST "192.168.36.203:9200/bank/_bulk?pretty&refresh" --data-binary "@accounts.json"

# 查看测试数据
curl -X GET "192.168.36.203:9200/_cat/indices?v"
```
1) **match_all查询**
```shell
# 将查询结果返回，通过search接口查询，q=*表示所有文档 asc升序排序。
curl -X GET "192.168.36.203:9200/bank/_search?q=*&sort=account_number:asc&pretty"
curl -X GET "192.168.36.203:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}
'
```
2) **from&size查询**
```shell
# 1. 规定记录条数
curl -X GET "192.168.36.203:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "size": 1
}
'
# 2. 规定记录范围
curl -X GET "192.168.36.203:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "from": 10,
  "size": 10
}
'
# 3. 指定字段牌序
curl -X GET "192.168.36.203:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "_source": ["account_number", "balance"]
}
'
```
3) **match根据字段输出**
```shell
curl -X GET "192.168.36.203:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
# match 模糊查询，不分区大小写
curl -X GET "192.168.36.203:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match": { "address": "mill" } }
}
'
# 匹配或关系查询
curl -X GET "192.168.36.203:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match": { "address": "mill lane" } }
}
'
```

4) **bool值查询**
```shell
# 1. 与查询，查询包含mill和lane的记录
curl -X GET "192.168.36.203:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
'
# 2. 或查询，查询包含mill或lane的记录
curl -X GET "192.168.36.203:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "should": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
'
# 3. '非'查询
curl -X GET "192.168.36.203:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must_not": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
'
# 4. 混合查询，精确查询
curl -X GET "192.168.36.203:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}
'
```
5) filter范围查询
```shell
# 1. 数值范围查询(20000<=balance<=30000),也可以采用时间之类的范围查询
curl -X GET "192.168.36.203:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}
'
```

### 插件管理

#### Head插件图形管理ElasticSearch
只在es-01操作
```shell
# 安装node环境
wget https://npm.taobao.org/mirrors/node/latest/node-v12.4.0-linux-x64.tar.gz
tar zxvf node-v12.4.0-linux-x64.tar.gz
mv node-v12.4.0-linux-x64 /usr/local/node-v12.4

cat >> /etc/profile <<EOF
NODE_NAME=/usr/local/node-v12.4
PATH=$NODE_NAME/bin:$PATH
export NODE_HOME PATH
EOF

source /etc/profile
npm -v
node -v

# 安装Head插件
git clone https://github.com/mobz/elasticsearch-head.git
cd elasticsearch-head/ && npm install
## 修改配置监听地址
vim Gruntfile.js
......
 port: 9100,
 base: '.',
 keepalive: true,
 hostname: '*'
## 启动npm
npm run start
## 修改elasticsearch配置并重启
## 在elasticsearch.yml配置文件最后加入
http.cors.enabled: true
http.cors.allow-origin: "*"
systemctl restart elasticsearch
```
浏览器打开http://192.168.36.203:9100
![1599717816953.png](https://github.com/jasonlau1024/study/blob/master/src/images/1599717816953.png?raw=true)

**根据字段来查询：**
数据浏览-->选择对应的索引及字段

**复合查询：**
![1599718427944.png](https://github.com/jasonlau1024/study/blob/master/src/images/1599718427944.png?raw=true)


### Logstash
#### 安装
机器：`elk-lk`
```shell
# 安装OpenJDK
yum install java-1.8.0-openjdk.x86_64 -y
# 安装logstash
yum install logstash -y  # 或 rpm -ivh logstash-7.9.1.rpm

rpm -ql logstash | more

cat >> /etc/profile <<EOF
PATH=$PATH:/usr/share/logstash/bin
export PATH
EOF
source /etc/profile
logstash -V
```
#### logstash条件判断
**比较操作符：**
- 相等：==, !=, <, >, <=, >=
- 正则：=~(匹配正则), !~(不匹配正则)
- 包含：in(包含), not in(不包含)
**布尔操作符：**
and(与), or(或), nand(非与), xor(非或)
**一元运算符：**
!(取反)
()(复合表达式), !()(对复合表达式结果取反)

#### Input插件
1) **Stdin**
https://www.elastic.co/guide/en/logstash/current/plugins-inputs-stdin.html
```shell
cat > /etc/logstash/conf.d/test.conf << EOF
input {
  stdin {

  }
}
filter {

}
output {
  stdout {
    codec => rubydebug
  }
}
EOF
# 测试配置文件
logstash -t -f /etc/logstash/conf.d/test.conf
... ...
Configuration OK
# 指定配置文件直接运行，键入123 345,形成timestamp日志，系统自动加入时间戳
logstash -r -f /etc/logstash/conf.d/test.conf
... ...
[INFO ] 2020-09-10 02:48:20.405 [Api Webserver] agent - Successfully started Logstash API endpoint {:port=>9600}
123
{
    "@timestamp" => 2020-09-10T06:48:29.664Z,
          "host" => "elk-lk",
      "@version" => "1",
       "message" => "123"
}
456
{
    "@timestamp" => 2020-09-10T06:48:33.710Z,
          "host" => "elk-lk",
      "@version" => "1",
       "message" => "456"
}
```
2) **File**
https://www.elastic.co/guide/en/logstash/current/plugins-inputs-file.html
```shell
cat > /etc/logstash/conf.d/test.conf <<EOF
input {
file {
path =>"/var/log/messages"
tags =>"123"
type =>"syslog"
}
}
filter {
}
output {
stdout {
codec => rubydebug
}
}
EOF

logstash -t -f /etc/logstash/conf.d/test.conf
logstash -r -f /etc/logstash/conf.d/test.conf
... ...
{
       "message" => "Sep 10 03:00:36 localhost yum[13094]: Installed: httpd-tools-2.4.6-93.el7.centos.x86_64",
          "host" => "elk-lk",
    "@timestamp" => 2020-09-10T07:00:37.809Z,
          "tags" => [
        [0] "123"
    ],
          "type" => "syslog",
          "path" => "/var/log/messages",
      "@version" => "1"
}
```
3) **TCP**
https://www.elastic.co/guide/en/logstash/current/plugins-inputs-tcp.html
```shell
cat > /etc/logstash/conf.d/test.conf <<EOF
input {
tcp {
port =>12345
type =>"nc"
}
}
filter {
}
output {
stdout {
codec => rubydebug
}
}
EOF
# 运行
logstash -r -f /etc/logstash/conf.d/test.conf

# 在其它机器，通过nc发送TCP数据
nc 192.168.36.202 12345
1
hello
... ...
```
#### Codec常用插件
https://www.elastic.co/guide/en/logstash/current/codec-plugins.html
https://www.elastic.co/guide/en/logstash/current/plugins-codecs-multiline.html
1) **Json/Json_lines示例**
```shell
cat > /etc/logstash/conf.d/test.conf << EOF
input {
stdin {
codec =>json {
charset => ["UTF-8"]
}
}
}
filter {
}
output {
stdout {
codec => rubydebug
}
}
EOF
logstash -r -f /etc/logstash/conf.d/test.conf
... ...
[INFO ] 2020-09-10 03:22:12.711 [Api Webserver] agent - Successfully started Logstash API endpoint {:port=>9600}
# 粘贴如下内容，json必须为一行。回车就能识别出来json数据
{"ip":"192.168.36.203","hostname":"elk-es01","method":"GET"}
{
      "hostname" => "elk-es01",
        "method" => "GET",
          "host" => "elk-lk",
      "@version" => "1",
    "@timestamp" => 2020-09-10T07:22:50.762Z,
            "ip" => "192.168.36.203"
}
```
2) **Multiline示例**
日志可以按照特征进行聚合，避免报重复错误。
```shell
# pattern => "^\s"表示匹配以字符开头的行，what => "previous" 表示将不是以字符串开头的行聚合到上一行。
cat > /etc/logstash/conf.d/test.conf << EOF
input {
  stdin {
    codec => multiline {
      pattern => "^\s"
      what => "previous"
    }
  }
}
filter {
}
output {
   stdout {
     codec => rubydebug
   }
}
EOF
logstash -r -f /etc/logstash/conf.d/test.conf
```
#### Filter插件
主要对日志进行结构化处理。
https://www.elastic.co/guide/en/logstash/current/filter-plugins.html
1) **json**
https://www.elastic.co/guide/en/logstash/current/plugins-filters-json.html
```shell
cat > /etc/logstash/conf.d/test.conf <<EOF
input {
  stdin {
  }
}
filter {
  json {
    source => "message"
    target => "content"
  }
}
output {
  stdout {
    codec => rubydebug
  }
}
EOF
logstash -r -f /etc/logstash/conf.d/test.conf
... ...
{"ip":"192.168.36.203","hostname":"elk-es01","method":"GET"}
# 把message中的数据结构化到content中，test.conf明确指定了message结构化指定的字段名称。
{
      "@version" => "1",
    "@timestamp" => 2020-09-10T09:15:00.542Z,
       "content" => {
              "ip" => "192.168.36.203",
          "method" => "GET",
        "hostname" => "elk-es01"
    },
          "host" => "elk-lk",
       "message" => "{\"ip\":\"192.168.36.203\",\"hostname\":\"elk-es01\",\"method\":\"GET\"}"
}
```
2) **KV**
主要用于拆分一个字符串。
https://www.elastic.co/guide/en/logstash/current/plugins-filters-kv.html
```shell
cat > /etc/logstash/conf.d/test.conf << EOF
input {
  stdin {
  }
}
filter {
  kv {
    field_split => "&?"
  }
}
output {
  stdout {
    codec => rubydebug
  }
}
EOF
pin=12345~0&d=123&e=foo@bar.com&oq=bobo&ss=12345?a=123
# 配置文件指定了“&”和“?”作为分隔符，采用的是key=value进行解析
{
            "oq" => "bobo",
       "message" => "pin=12345~0&d=123&e=foo@bar.com&oq=bobo&ss=12345?a=123",
            "ss" => "12345",
    "@timestamp" => 2020-09-10T09:18:35.303Z,
           "pin" => "12345~0",
             "a" => "123",
      "@version" => "1",
          "host" => "elk-lk",
             "d" => "123",
             "e" => "foo@bar.com"
}
# 修改配置文件field_split_pattern => ":+"，使用正则匹配，匹配1个或多个":"符号。
k1=v1:k2=v2::k3=v3:::k4=v4
{
          "host" => "elk-lk",
       "message" => "k1=v1:k2=v2::k3=v3:::k4=v4",
            "k4" => "v4",
            "k2" => "v2",
            "k3" => "v3",
      "@version" => "1",
    "@timestamp" => 2020-09-10T09:26:50.800Z,
            "k1" => "v1"
}

### GeoIP插件
获取IP地理位置信息的插件。
下载地址（免费版）：https://dev.maxmind.com/geoip/geoip2/geolite2/ 
maxmind db binary 压缩包


```shell
cat > /etc/logstash/conf.d/test.conf << EOF
input {
  stdin {
  }
}
filter {
  grok {
    match => {
      "message" => "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}"
    }
  }
  geoip {
    source => "client"
    database => "/opt/geoip/databases/GeoLite2-City.mmdb"
  }
}
output {
  stdout {
    codec => rubydebug
  }
}
EOF

```




参考文档：https://blog.csdn.net/m0_37385780/article/details/105858473
## Docker 各组件分别部署
```shell
docker pull elasticsearch:7.5.2
docker pull logstash:7.5.2
docker pull kibana:7.5.2
```

### elasticsearch
```shell
docker run -d --name elasticsearch -p 9200:9200 -e "discovery.type=single-node" elasticsearch:7.5.2

# 查看日志是否成功,可通过访问IP:9200查看实时日志
docker logs -f elasticsearch
# 复制配置文件夹
docker cp elasticsearch:/usr/share/elasticsearch/config /data/elk/elasticsearch/config
# 停止并删除容器
docker stop elasticsearch && docker rm elasticsearch
# 创建数据卷，用于挂载ES容器
docker volume create elasticsearch-data
# 修改配置文件后，通过-v挂载并重新启动容器
docker run -d --name elasticsearch -v /data/elk/elasticsearch/config:/usr/share/elasticsearch/config -v elasticsearch-data:/usr/share/elasticsearch/data -p 9200:9200 -e "discovery.type=single-node" -d elasticsearch:7.5.2

```

### kibana
```shell
docker run -d --name kibana --link elasticsearch:elasticsearch -p 5601:5601 -d kibana:7.5.2
# 查看日志是否成功，可通过访问IP:5601查看实时日志
docker logs -f kibana
# 复制配置文件夹
docker cp kibana:/usr/share/kibana/config /data/elk/kibana/config
# 停止并删除容器
 docker stop kibana && docker rm kibana
# 创建kibana数据卷
docker volume create kibana-data
# 修改配置文件后，通过-v挂载并重新启动容器
docker run -d --name kibana --link elasticsearch:elasticsearch -v /data/elk/kibana/config:/usr/share/kibana/config -v kibana-data:/usr/share/kibana/data -p 5601:5601 -d kibana:7.5.2
```
### logstash
这里的职责只做数据的转换和过滤，再传送给elasticsearch
```shell
docker run -d --name logstash --link elasticsearch:elasticsearch -p 5044:5044 -d logstash:7.5.2
# 查看日志
docker logs -f logstash
# 复制配置文件夹
docker cp logstash:/usr/share/logstash/config /data/elk/logstash/config
# 停止并删除容器
docker stop logstash && docker rm logstash
# 修改配置文件
cat config/logstash-sample.conf
# Sample Logstash configuration for creating a simple
# Beats -> Logstash -> Elasticsearch pipeline.

input {
  beats {
    port => 5044
  }
}

filter{
  if "java-logs" in [tags] {
    grok {
      match => {
        "message" => "(?<date>\d{4}-\d{2}-\d{2}\s\d{2}:\d{2}:\d{2}.\d{3})\s%{DATA:thread}\s%{DATA:level}\s%{DATA:class}\-(?<msg>.*)"
      }
      remove_field => ["message"]
    }
    if [level] !~ "(DEBUG|ERROR|WARN|INFO)" {
      drop {}
    }
  }

}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    #index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
    #user => "elastic"
    #password => "changeme"
  }
  # 控制台输出
  stdout{ }
}

cat config/pipelines.yml
# This file is where you define your pipelines. You can define multiple.
# For more information on multiple pipelines, see the documentation:
#   https://www.elastic.co/guide/en/logstash/current/multiple-pipelines.html

- pipeline.id: main
  path.config: "/usr/share/logstash/config/*.conf"
  pipeline.workers: 1

# 创建数据卷
docker volume create logstash-data
# 通过-v挂载数据卷重启容器

```

### filebeat
`filebeat`这里负责采集数据传输给logstash,demo放的是log文件
```shell
docker run -d --name filebeat --link logstash:logstash -d elastic/filebeat:7.5.2
# 复制配置文件
docker cp filebeat:/usr/share/filebeat/filebeat.yml /data/elk/filebeat/filebeat.yml
# 停止并删除容器
docker stop filebeat && docker rm filebeat
# 修改配置文件filebeat.yml
cat filebeat.yml
filebeat.inputs:
- type: log
  #开启监视，不开不采集
  enable: true
  # 采集日志的路径这里是容器内的path
  paths:
    - /var/log/demo/*.log
  # 为每个项目标识,或者分组，可区分不同格式的日志
  tags: ["java-logs"]
  # 这个文件记录日志读取的位置，如果容器重启，可以从记录的位置开始取日志
  registry_file: /usr/share/filebeat/data/registry

output.logstash:
  enable: true
  # 输出到logstash中,logstash更换为自己的ip，这里是使用容器名称
  hosts: "logstash:5044"
  max_retries: 3
  worker: 1

# 创建数据卷
docker volume create filebeat-data

# 运行容器
docker run -d --name filebeat --link logstash:logstash -v /data/elk/filebeat/log/demo:/var/log/demo -v /data/elk/filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml -v filebeat-data:/usr/share/filebeat/data -d elastic/filebeat:7.5.2 

```
### Linux通过RPM安装Filebeat
```shell
# 下载Filebeat RPM包
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.5.2-x86_64.rpm
rpm -ivh filebeat-7.5.2-x86_64.rpm

####  相关目录说明  ####
/etc/filebeat  #配置文件的目录
/usr/share/filebeat  #安装的目录，bin等
/var/lib/filebeat  #数据缓存的目录，registry等


```





## Docker部署ELK
镜像仓库地址：https://hub.docker.com/r/sebp/elk
镜像说明文档：https://elk-docker.readthedocs.io/

### 运行ELK容器
`docker run -p  5601:5601 -p 9200:9200 -p 5044:5044  -v /home/nya/dockerFile:/data -it -d --name elk sebp/elk`
**端口说明：**
- `5601` Kibana Web UI 服务端口
- `9200` ES Json 接口
- `5044` Logstash 服务端口

**docker-compose部署：**
```yaml
elk:
  image: sebp/elk
  ports:
    - "5601:5601"
    - "9200:9200"
    - "5044:5044"
# docker-compose up elk
```








