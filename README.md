# M365-Copilot2API-CE ARM64 Build

![Build](https://github.com/MIGHTYBLANK001/M365-Copilot2API-CE-Build/actions/workflows/build.yml/badge.svg)

基于上游项目：

- https://github.com/s12ryt/M365-Copilot2API-CE

自动构建的 **ARM64 Docker 镜像仓库**。

本仓库不维护源码，仅通过 GitHub Actions：

```
上游源码更新
        ↓
GitHub Actions 自动检测
        ↓
Docker Buildx ARM64 编译
        ↓
推送 GHCR
        ↓
生成最新镜像
```

---

## 镜像地址

Docker Registry:

```
ghcr.io
```

镜像：

```bash
ghcr.io/mightyblank001/m365-copilot2api-ce:latest
```

拉取：

```bash
docker pull ghcr.io/mightyblank001/m365-copilot2api-ce:latest
```

支持：

```
linux/arm64
```

适用于：

- ARM64 VPS
- Raspberry Pi
- RK3588
- Armbian
- Debian ARM64
- Ubuntu ARM64
- NAS ARM设备


---

# 快速部署


## 1. 创建目录

```bash
mkdir -p m365-copilot2api/{data,secrets}

cd m365-copilot2api
```


目录结构：

```
m365-copilot2api

├── docker-compose.yml

├── data

│   ├── accounts.json

│   ├── token-cache.json

│   ├── sessions.json

│   └── api-keys.json

└── secrets

    └── m365_admin_password
```


---

## 2. 创建管理员密码


```bash
echo "your_password" > secrets/m365_admin_password

chmod 600 secrets/m365_admin_password
```


---

## 3. docker-compose.yml


创建：

```yaml
services:

  m365-copilot2api:

    image: ghcr.io/mightyblank001/m365-copilot2api-ce:latest

    container_name: m365-copilot2api

    restart: unless-stopped


    ports:

      - "127.0.0.1:4141:4141"


    environment:

      M365_LISTEN: 0.0.0.0:4141

      M365_DATA_DIR: /data

      M365_CONFIG: /data/accounts.json

      M365_TOKEN_CACHE: /data/token-cache.json

      M365_SESSION_CACHE: /data/sessions.json

      M365_API_KEYS: /data/api-keys.json


      M365_ADMIN_PASSWORD_FILE: /data/admin-password

      M365_ADMIN_PASSWORD_BOOTSTRAP_FILE: /run/secrets/m365_admin_password


      M365_ACCOUNT_CONCURRENCY_LIMIT: "256"

      M365_ACCOUNT_DEFAULT_CONCURRENCY: "256"


      M365_CHAT_TIMEOUT_SECONDS: "120"

      M365_IMAGE_TIMEOUT_SECONDS: "150"


      M365_TOKEN_PRE_REFRESH: "true"

      M365_TOKEN_PRE_REFRESH_MINUTES: "5"

      M365_TOKEN_PRE_REFRESH_INTERVAL_SECONDS: "60"

      M365_TOKEN_PRE_REFRESH_CONCURRENCY: "4"


    volumes:

      - ./data:/data

      - ./secrets/m365_admin_password:/run/secrets/m365_admin_password:ro


    secrets:

      - m365_admin_password



secrets:

  m365_admin_password:

    file: ./secrets/m365_admin_password
```


---

## 4. 启动


```bash
docker compose up -d
```


查看状态：

```bash
docker ps
```


查看日志：

```bash
docker logs -f m365-copilot2api
```


---

# 更新镜像


本项目保持：

```
latest
```

标签。


更新：

```bash
docker compose pull

docker compose up -d
```


检查版本：

```bash
docker images | grep m365
```


---

# 配置说明


## 数据目录


容器：

```
/data
```


映射：

```
./data
```


包含：

|文件|用途|
|-|-|
|accounts.json|M365账号配置|
|token-cache.json|Token缓存|
|sessions.json|会话缓存|
|api-keys.json|API Key管理|


建议：

定期备份：

```bash
tar czf m365-data-backup.tar.gz data/
```


---

# 性能配置


默认：

```yaml
M365_ACCOUNT_CONCURRENCY_LIMIT=256
```

适合：

- 多账号
- 高并发调用


低配置设备建议：

例如：

2核 / 2GB：

```yaml
M365_ACCOUNT_CONCURRENCY_LIMIT=32
M365_ACCOUNT_DEFAULT_CONCURRENCY=32
```


高性能服务器：

```yaml
M365_ACCOUNT_CONCURRENCY_LIMIT=256
M365_ACCOUNT_DEFAULT_CONCURRENCY=256
```


---

# 网络访问


默认绑定：

```yaml
127.0.0.1:4141
```


表示：

仅本机访问。


如果需要局域网访问：

修改：

```yaml
ports:

  - "4141:4141"
```


然后：

```
http://设备IP:4141
```


---

# 反向代理推荐


生产环境建议：

```
Client

 |

Nginx / Caddy

 |

M365-Copilot2API

 |

127.0.0.1:4141
```


可以增加：

- HTTPS
- 域名访问
- 访问控制


---

# 自动构建机制


本仓库 GitHub Actions：

```
每6小时检查一次上游

        ↓

检测 commit SHA

        ↓

无变化:

        不构建


有变化:

        ↓

Docker Buildx

        ↓

linux/arm64

        ↓

GHCR latest

        ↓

自动清理旧镜像
```


因此：

用户无需关注源码更新。


---

# 注意事项


## ARM64限制


当前镜像：

```
linux/arm64
```


不是：

```
linux/amd64
```


x86服务器需要自行构建。


---

## 数据持久化


删除容器不会影响：

```
./data
```


但是删除数据目录会导致：

- Token丢失
- 账号配置丢失
- API Key丢失


请提前备份。


---

# License


本仓库仅用于自动化构建和发布镜像。


源码版权及许可证遵循：

https://github.com/s12ryt/M365-Copilot2API-CE
