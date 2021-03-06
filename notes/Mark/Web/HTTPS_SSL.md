

## 加密与解密

### ChaCha20-Poly1305

AEAD加密模式：
- AES-GCM(块加密算法)
- ChaCha20-Poly1305(流加密算法)
AES块加密算法在有AES-NI加速指令的服务器和台式机等运行的非常快。而大部分移动设备（比如手机）由于没有AES-NI支持，运行 AES 加密是很缓慢的，在没有AES-NI加速指令的设备上，ChaCha20-Poly1305流加密效率高得多。

nginx使用的密码库是OpenSSL,OpenSSL从 1.1.0+ 开始支持ChaCha20-Poly1305算法

## SSL 证书申请
### Certbot
Certbot 是一个简单易用的 SSL 证书部署工具，由 EFF 开发，前身即 Let’s Encrypt 官方（Python）客户端。简单来说，certbot 就是一个简化 Let’s Encrypt 部署，和管理 Let’s Encrypt 证书的工具。certbot的开源项目在GitHub上，https://github.com/certbot/certbot

**指定二级域名申请：**
```shell
wget https://dl.eff.org/certbot-auto
chmod a+x certbot-auto
./certbot-auto certonly --standalone --email 邮箱地址 -d 域名1 -d 域名2 ...
## 查看生成的证书
tree /etc/letsencrypt/live/
## 注：Let’s Encrypt 生成的免费证书有效期为3个月
## 证书续签
./certbot-auto renew
```
**泛域名证书申请：**
```shell
./certbot-auto certonly --server https://acme-v02.api.letsencrypt.org/directory --manual --preferred-challenges dns -d *.example.com
## 在域名管理商设置DNS TXT 记录 ， 再dig查看是否生效
dig txt _acme-challenge.example.com
```


## HTTPS

### HTTPS实现原理
HTTPS的整体过程分为证书验证和数据传输阶段
- 证书验证（非对称加密）
- 1. 浏览器发起 HTTPS 请求
- 2. 服务端返回 HTTPS 证书
- 3. 客户端验证证书是否合法，如果不合法则提示告警
- 数据传输（对称加密）
- 1. 当证书验证合法后，在本地生成随机数
- 2. 通过公钥加密随机数，并把加密后的随机数传输到服务端
- 3. 服务端通过私钥对随机数进行解密
- 4. 服务端通过客户端传入的随机数构造对称加密算法，对返回结果内容进行加密后传输
![img](https://static.blog.leapmie.com/2019/11/1378987910.png)


### HTTPS解答
#### 为什么数据传输是用对称加密？
- 首先，非对称加密的加解密效率是非常低的，而 http 的应用场景中通常端与端之间存在大量的交互，非对称加密的效率是无法接受的；
- 另外，在 HTTPS 的场景中只有服务端保存了私钥，一对公私钥只能实现单向的加解密，所以 HTTPS 中内容传输加密采取的是对称加密，而不是非对称加密。

#### 为什么需要 CA 认证机构颁发证书？
如果不存在认证机构，任何人都可以制作证书，这带来的安全风险便是经典的 “中间人攻击” 问题。 由于缺少对证书的验证，所以客户端虽然发起的是 HTTPS 请求，但客户端完全不知道自己的网络已被拦截，传输内容被中间人全部窃取。
**“中间人攻击”原理：**
1. 本地请求被劫持（如DNS劫持等），所有请求均发送到中间人的服务器
2. 中间人服务器返回中间人自己的证书
3. 客户端创建随机数，通过中间人证书的公钥对随机数加密后传送给中间人，然后凭随机数构造对称加密对传输内容进行加密传输
4. 中间人因为拥有客户端的随机数，可以通过对称加密算法进行内容解密
5. 中间人以客户端的请求内容再向正规网站发起请求
6. 因为中间人与服务器的通信过程是合法的，正规网站通过建立的安全通道返回加密后的数据
7. 中间人凭借与正规网站建立的对称加密算法对内容进行解密
8. 中间人通过与客户端建立的对称加密算法对正规内容返回的数据进行加密传输
9. 客户端通过与中间人建立的对称加密算法对返回结果数据进行解密

*注：*HTTPS保证的只是传输过程安全，而随机数存储于本地，本地的安全属于另一安全范畴，应对的措施有安装杀毒软件、反木马、浏览器升级修复漏洞等。

#### 浏览器是如何确保 CA 证书的合法性？
证书包含信息：
颁发机构信息、公钥、公司信息、域名、有效期、指纹... ...
证书合法性依据：只要是权威机构生成的证书，我们就认为是合法的。

**浏览器验证证书合法性**
浏览器发起 HTTPS 请求时，服务器会返回网站的 SSL 证书，浏览器需要对证书做以下验证：
1. 验证域名、有效期等信息是否正确。
2. 判断证书来源是否合法。
   每份签发证书都可以根据验证链查找到对应的根证书，操作系统、浏览器会在本地存储权威机构的根证书，利用本地根证书可以对对应机构签发证书完成来源验证；
3. 判断证书是否被篡改。（需要与 CA 服务器进行校验）
4. 判断证书是否已吊销。
   判断证书是否已吊销。通过CRL（Certificate Revocation List 证书注销列表）和 OCSP（Online Certificate Status Protocol 在线证书状态协议）实现，其中 OCSP 可用于第3步中以减少与 CA 服务器的交互，提高验证效率。
