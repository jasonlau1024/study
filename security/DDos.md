参考文档：

[DDoS 攻击与防御：从原理到实践（上）](https://sq.163yun.com/blog/article/155831083719946240)

[DDoS 攻击与防御：从原理到实践（下）](<https://sq.163yun.com/blog/article/155829126267629568>)

[反射型 DDoS 攻击的原理和防范措施](<https://sq.163yun.com/blog/article/155835434018500608>)

[DDoS反射放大之SSDP攻击](<https://sq.163yun.com/blog/article/185561041311997952>)









# DDoS

出于打击报复、敲诈勒索、政治需要等各种原因，加上攻击成本越来越低、效果特别明显等趋势，DDoS 攻击已经演变成全球性的网络安全威胁。

### 趋势

1. **国际化**

   现在的 DDoS 攻击越来越国际化，而我国已经成为仅次于美国的第二大 DDoS 攻击受害国，而国内的 DDoS 攻击源海外占比也越来越高。

2. **超大规模化**

   由于跨网调度流量越来越方便、流量购买价格越来越低廉，现在 DDoS 攻击的流量规模越来越大。

3. **市场化**

   市场化势必带来成本优势，现在各种在线 DDoS 平台、肉鸡交易渠道层出不穷，使得攻击者可以以很低的成本发起规模化攻击。









SSDP 实践：





```shell
# 下面是Nmap运行页面，表面该IP返回了SSDP响应包，服务正常，可以被利用，UDP本就是无连接的协议，有时候需要多试几次
# nmap 94.73.193.111 -p 1900 -sU --script=upnp-info

Starting Nmap 6.40 ( http://nmap.org ) at 2020-05-09 11:25 CST
Nmap scan report for 111.193.73.94.ip.orionnet.ru (94.73.193.111)
Host is up (0.085s latency).
PORT     STATE SERVICE
1900/udp open  upnp
| upnp-info:
| 94.73.193.111
|     Server: Ubuntu/7.10 UPnP/1.0 miniupnpd/1.0
|_    Location: http://192.168.0.1:1900/rootDesc.xml

Nmap done: 1 IP address (1 host up) scanned in 1.20 seconds
```









