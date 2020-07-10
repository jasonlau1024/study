

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
