OVERWALL
===

使用 docker-compose 部署服务，现在支持：

- v2ray：vmess+tls+ws 模式
- shadowsocks：obfs + tls

## 前置条件

将域名 A 记录指向对应的服务器 IP。

## 服务器端

### V2ray 配置方法

1. 填写 `.v2ray.env` 中的配置，比如

```conf
# 域名
VIRTUAL_HOST=v2ray.domain.com
LETSENCRYPT_HOST=v2ray.domain.com
# 邮箱
LETSENCRYPT_EMAIL=v2ray@domain.com
```

2. 配置 `v2ray/config.json` 的配置中 `clients -> id` UUID。

### Shadowsocks 配置

设置 `shadowsocks/config.json` 中的 `password`。

### 运行

运行 `sudo ./start.sh` 脚本即可。

## 客户端

### v2ray 配置

```json
{
    "log": {
    "access": "/dev/stdout",
    "error": "/dev/stderr",
        "loglevel": "warning"
    },
    "inbounds": [
        {
            "port": 1234,
            "listen": "0.0.0.0",
            "tag": "socks-in",
            "protocol": "socks",
            "settings": {
                "auth": "noauth"
            }
        },
        {
            "port": 2234,
            "listen": "0.0.0.0",
            "tag": "http-in",
            "protocol": "http",
            "settings": {}
        }
    ],
    "outbounds": [
        {
            "mux": {
                "enabled": true
            },
            "protocol": "vmess",
            "settings": {
                "vnext": [
                    {
                        "users": [
                            {
                                "id": "{你的uuid}",
                                "alterId": 32,
                                "security": "auto"
                            }
                        ],
                        "address": "{你的v2ray域名}",
                        "port": 443
                    }
                ]
            },
            "streamSettings": {
                "tlsSettings": {
                    "allowInsecure": false
                },
                "wsSettings": {
                    "headers": {
                        "User-Agent": "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.4489.62 Safari/537.36",
                        "Host": "{你的v2ray域名}",
                        "Accept-Encoding": "gzip",
                        "Pragma": "no-cache"
                    },
                    "path": "/"
                },
                "network": "ws",
                "security": "tls"
            },
            "tag": "proxy"
        },
        {
            "protocol": "blackhole",
            "settings": {},
            "tag": "blocked"
        },
        {
            "protocol": "freedom",
            "settings": {},
            "tag": "dicert"
        }
    ],
    "routing": {
        "domainStrategy": "AsIs",
        "rules": [
            {
                "type": "field",
                "ip": [
                    "geoip:private",
                    "geoip:cn"
                ],
                "outboundTag": "dicert"
            },
            {
                "type": "field",
                "inboundTag": [
                    "socks-in",
                    "http-in"
                ],
                "outboundTag": "proxy"
            }
        ]
    },
    "other": {}
}
```

### shadowsocks 客户端

```json
{
    "server":"{你的shadowsocs域名}",
    "server_port": 443,
    "local_address": "127.0.0.1",
    "local_port":5146,
    "password":"{你的密码}",
    "timeout":300,
    "fast_open": false,
    "workers": 1,
    "method":"chacha20-ietf-poly1305",
    "plugin": "obfs-local",
    "plugin_opts": "obfs=tls;obfs-host=www.bing.com"
}
```
