# 🚀 Docker部署 n8n 流程指南

## 📁 项目结构预览

n8n 通过 Docker Compose 部署后的项目结构如下：

```bash
n8n-docker/
├── docker-compose.yml   # Docker Compose definition
├── .env                 # Environment variables
└── n8n-data/            # Persistent volume for n8n data and SQLite DB
````

包含内容：

* Docker Compose 的配置文件
* 环境变量配置文件

> ⚠️ `n8n-data` 文件夹用于本地持久化存储
> 默认通过 Docker 部署 n8n 时，如果不手动配置挂载目录，**所有数据（包括 `database.sqlite`）将保存在容器内部**。
> 一旦容器删除或重建，**数据将会全部丢失**。

---

## 🧰 第一步：准备本地目录并创建相关文件

在命令行中运行以下命令：

```bash
mkdir -p n8n-docker/n8n-data
cd n8n-docker
touch .env docker-compose.yml README.md
```

项目结构创建完毕。

---

## 📄 第二步：创建环境变量配置文件 `.env`

在 `.env` 文件中添加如下内容：

```env
# Timezone
GENERIC_TIMEZONE=Asia/Shanghai

# Keep user management enabled (do not disable)
N8N_USER_MANAGEMENT_DISABLED=false

# Basic Auth (optional, acts before the UI)
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=yourpassword

# n8n Port on host machine
N8N_PORT=5678

# Database type (SQLite for lightweight local usage)
DB_TYPE=sqlite
DB_SQLITE_DATABASE=/home/node/.n8n/database.sqlite
```

---

## 📝 第三步：配置 `docker-compose.yml`

在 `docker-compose.yml` 文件中添加以下配置：

```yaml
version: '3.8'

services:
  n8n:
    image: n8nio/n8n:latest
    restart: unless-stopped

    ports:
      - "${N8N_PORT}:5678"

    environment:
      - N8N_BASIC_AUTH_ACTIVE=${N8N_BASIC_AUTH_ACTIVE}
      - N8N_BASIC_AUTH_USER=${N8N_BASIC_AUTH_USER}
      - N8N_BASIC_AUTH_PASSWORD=${N8N_BASIC_AUTH_PASSWORD}
      - TZ=${GENERIC_TIMEZONE}
      - DB_TYPE=${DB_TYPE}
      - DB_SQLITE_DATABASE=${DB_SQLITE_DATABASE}
      - WEBHOOK_URL=http://localhost:${N8N_PORT}/
      - N8N_HOST=localhost
      - N8N_COMMUNITY_PACKAGE_ALLOW_TOOL_USAGE=true

    volumes:
      - ./n8n-data:/home/node/.n8n

    networks:
      - n8n_net

networks:
  n8n_net:
    driver: bridge
```

---

## ⚙️ 配置说明详解

* `version: '3.8'`：指定 Compose 文件版本
* `image: n8nio/n8n:latest`：使用官方 n8n 镜像
* `restart: unless-stopped`：自动重启容器
* `ports`：本地端口映射
* `environment`：通过环境变量配置认证、时区、数据库
* `volumes`：挂载本地 `n8n-data/` 以持久化 SQLite 数据库
* `networks`：定义 Docker 网络，方便未来扩展服务

---

## 🚀 第四步：启动服务

在 `n8n-docker/` 目录中执行：

```bash
docker compose up -d
```

首次运行将自动拉取镜像。

> 💡 建议使用代理加速镜像下载。

---

## 🌐 第五步：访问 n8n 界面

打开浏览器访问：

```
http://localhost:5678
```

首次访问会进入注册流程：

1. 输入邮箱、姓名与密码完成注册
2. 可跳过问卷信息收集
3. 系统将提示输入激活码

   * 点击获取激活码按钮
   * 检查你的邮箱
   * 输入激活码完成激活

---

## 🧪 第六步：验证 SQLite 是否工作

运行命令：

```bash
ls ./n8n-data/database.sqlite
```

如果能看到 `database.sqlite` 文件，说明 SQLite 数据库工作正常。
你也可以在 Cursor 编辑器中看到 `n8n-data/` 目录下已自动生成该文件。

---

> 📌 本文档适用于本地快速部署测试，如需生产部署请结合反向代理、安全加固等实践。

