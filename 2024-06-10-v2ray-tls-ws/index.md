# 使用 v2ray 部署 TLS Web Service


经过2年的试用, 可以获得比较稳定的科学上网体验.

<!--more-->
总的思路其实很简单, 就是把代理服务器隐藏在域名后面, 客户端访问代理服务器通过 https 域名, 和访问一个网站并无二致.

## 安装 v2ray

[fhs-install-v2ray](https://github.com/v2fly/fhs-install-v2ray)

## 配置 v2ray
默认配置路径: `/usr/local/etc/v2ray/config.json`

```json
{
    "inbounds": [
        {
            "port": 12345,
            "listen": "127.0.0.1",
            "protocol": "vmess",
            "settings": {
                "clients": [
                    {
                        "id": "guid",
                        "alterId": 0
                    }
                ]
            },
            "streamSettings": {
                "network": "ws",
                "wsSettings": {
                    "path": "/ray" // 路径需要和 web server 中的一致
                }
            }
        }
    ],
    "outbounds": [
        {
            "protocol": "freedom",
            "settings": {}
        }
    ]
}
```
该配置可以让 v2ray 监听本地 12345 端口, 然后需要 nginx 反向代理将 https 请求转发给 12345 端口.

这里 `alterId` 被设置为 0, 如果能够保证服务端和客户端时间同步, 也是可以设置成其他值的.

最后, 启动 v2ray, 这部分的工作就完成了.
```bash
# start
sudo systemctl start v2ray
# enable auto start upon startup
sudo systemctl enable v2ray
# debug
sudo netstat -anp | grep v2ray
```

## 申请合法域名 + SSL证书

### 申请域名
申请一个域名, 添加一个 A Record 指向部署的 VPS 的 IP 地址. (我用了 Namecheap 购买域名, 添加 A Record 为 @ 即可), 同时可以使用 [dnschecker](https://dnschecker.org/) 检查 A record 是否生效.

注意: 如果发现遭到运营商的 DNS 污染, 请使用第三方 DNS, 比如[阿里云 DNS](https://alidns.com/).

### 申请证书

```bash
# 安装 acme.sh
sudo apt install -y openssl cron socat curl
curl https://get.acme.sh | sh

# 注册邮箱
sudo ~/.acme.sh/acme.sh --register-account -m my_email --force

# 申请证书
sudo ~/.acme.sh/acme.sh --issue -d your.domain --standalone -k ec-256 --force

# 安装证书和密钥到 v2ray 目录下
sudo ~/.acme.sh/acme.sh --installcert -d your.domain --fullchainpath /etc/v2ray/v2ray.crt --keypath /etc/v2ray/v2ray.key --ecc --force
```

注意事项: 由于 acme 申请或更新证书需要访问 80 端口
1. 虚拟机需要开放 80 端口的访问权限.
2. nginx 默认 http 服务会占用 80 端口, 需要在 `/etc/nginx/sites-enabled/default` 下修改默认端口为其他端口
3. 重启 nginx 服务: `sudo systemctl reload nginx`, `sudo systemctl restart nginx`
4. 如果同一个域名更换了虚拟机 ip 地址, 需要先刷新虚拟机的 dns 缓存, 不然证书下载时会出问题. 刷新方法: `sudo resolvectl flush-caches`

### 更新证书
由于每过三个月证书会自动失效, 所以需要及时更新
```bash
# 更新证书
sudo ~/.acme.sh/acme.sh --renew -d your.domain --ecc --force
# 安装证书到 v2ray 目录下
sudo ~/.acme.sh/acme.sh --installcert -d your.domain --fullchainpath /etc/v2ray/v2ray.crt --keypath /etc/v2ray/v2ray.key --ecc --force
# 检查证书到期时间
openssl x509 -in /path/to/your.crt -noout -enddate
```
完成后可以通过 [SSL Server Test](https://www.ssllabs.com/ssltest/) 来检查证书

最后, 注册 crontab 来自动更新:
```bash
# 注册 crontab 自动更新
sudo ~/.acme.sh/acme.sh --cron --force
# 检查 crontab
crontab -l
```
也可以手动更新:
```bash
sudo ~/.acme.sh/acme.sh --renew -d your.domain --force
```

## 配置 nginx web server

在 `/etc/nginx/conf.d` 目录下创建一个文件 `Default.conf` 作为默认配置.

```conf
server {
  listen 443 ssl;
  listen [::]:443 ssl;
  
  ssl_certificate       /etc/v2ray/v2ray.crt;
  ssl_certificate_key   /etc/v2ray/v2ray.key;
  ssl_session_timeout 1d;
  ssl_session_cache shared:MozSSL:10m;
  ssl_session_tickets off;
  
  ssl_protocols         TLSv1.1 TLSv1.2 TLSv1.3;
  ssl_ciphers           ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
  ssl_prefer_server_ciphers off;
  
  server_name           your.domain;
    location /ray { # 与 V2Ray 配置中的 path 保持一致
      if ($http_upgrade != "websocket") { # WebSocket协商失败时返回404
          return 404;
      }
      proxy_redirect off;
      proxy_pass http://127.0.0.1:12345; # 反向代理转发给 localhost: 12345 端口
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header Host $host;
      # Show real IP in v2ray access.log
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

启动 nginx
```bash
sudo systemctl start nginx
```

## 配置本地客户端

### v2ray 客户端
```json
{
    "inbounds": [
        {
            "port": 1080,
            "protocol": "socks",
            "sniffing": {
                "enabled": true,
                "destOverride": [
                    "http",
                    "tls"
                ]
            },
            "settings": {
                "auth": "noauth",
                "udp": false,
                "ip": "127.0.0.1"
            }
        }
    ],
    "outbounds": [
        {
            "protocol": "vmess",
            "settings": {
                "vnext": [
                    {
                        "address": "your.domain",
                        "port": 443,
                        "users": [
                            {
                                "id": "guid",
                                "alterId": 0
                            }
                        ]
                    }
                ]
            },
            "streamSettings": {
                "network": "ws",
                "security": "tls",
                "wsSettings": {
                    "path": "/ray"
                },
                "tlsSettings": {
                   "serverName": "",
                   "allowInsecure": false
                }
            }
        }
    ]
}
```
直接配置 outbounds 发送给你注册好的域名即可, 本地就和其他方法一样暴露 socks5 1080 端口给其他应用, 例如 chrome.

### ClashX 客户端
不过 ClashX 删库跑路了, 目前还没找到替代品, 凑合用呗.
```yaml
- name: "v2ray-tls-ws"
  type: vmess
  server: your.domain
  port: 443
  uuid: guid
  alterId: 0
  cipher: auto
  # udp: true
  tls: true
  # skip-cert-verify: true
  # servername: example.com # priority over wss host
  network: ws
  ws-opts:
    path: /ray
    headers:
      Host: your.domain
  #   max-early-data: 2048
  #   early-data-header-name: Sec-WebSocket-Protocol
```

# Ref
1. [V2Ray进阶指南：WSS组合配置(WebSocket + TLS + Nginx + CDN)](https://cyfeng.science/2020/03/22/advanced-v2ray-with-wss/)
2. https://toutyrater.github.io/advanced/tls.html
3. [V2Ray高级技巧：流量伪装](https://v2xtls.org/v2ray高级技巧：流量伪装/)
4. [使用TLS+WS方式的V2Ray简单搭建说明：一个目前较为稳定的科学上网方式](https://www.wenjinyu.me/a-simple-instructions-for-how-to-deploy-v2ray-with-tls-and-ws-mode/)

