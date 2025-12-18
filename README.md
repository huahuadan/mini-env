## mini-env 本地开发环境

`mini-env` 是一个基于 Docker Compose 的本地开发环境，内置常用中间件，开箱即用，已针对 **Mac M4 (Apple Silicon)** 进行优化。

### 已集成的中间件

- **MySQL 8.0**：支持 Mac M4，使用 `platform: linux/amd64` 通过 Rosetta 2 运行
- **Redis 7 (Alpine)**：原生 ARM64，`platform: linux/arm64/v8`
- **MongoDB 7.0 Community Server**：镜像 `mongodb/mongodb-community-server:7.0-ubuntu2204`，原生 ARM64
- **RocketMQ 5.3.2 & Dashboard**：镜像 `apache/rocketmq:5.3.2` 与 `apacherocketmq/rocketmq-dashboard:latest`
- **XXL-JOB 2.3.1**：分布式任务调度平台

---

## 项目结构

```bash
mini-env/
├── docker-compose.yml
├── mysql/
│   ├── config/          # MySQL 配置文件
│   ├── data/            # MySQL 数据目录（已加入 .gitignore）
│   ├── logs/            # MySQL 日志目录（已加入 .gitignore）
│   └── init/            # MySQL 初始化 SQL 脚本目录
├── mongodb/
│   ├── config/          # MongoDB 配置文件
│   ├── data/            # MongoDB 数据目录
│   └── logs/            # MongoDB 日志目录
├── rocketmq/
│   ├── conf/           # RocketMQ 配置文件（可选）
│   ├── store/          # RocketMQ 存储目录（已加入 .gitignore）
│   └── logs/           # RocketMQ 日志目录（已加入 .gitignore）
├── redis/
│   ├── config/          # Redis 配置文件
│   ├── data/            # Redis 数据目录（已加入 .gitignore）
│   └── logs/            # Redis 日志目录（已加入 .gitignore）
├── xxl-job/
│   ├── Dockerfile                  # XXL-JOB 镜像构建文件
│   ├── xxl-job-admin-2.3.1.jar     # XXL-JOB JAR 包
│   └── logs/                      # XXL-JOB 日志目录（已加入 .gitignore）
└── README.md
```

---

## 快速开始

### 启动全部服务

```bash
docker-compose up -d
```

### 停止全部服务

```bash
docker-compose down
```

### 查看日志

```bash
# 查看所有服务日志
docker-compose logs -f

# 查看特定服务日志
docker-compose logs -f mysql
docker-compose logs -f redis
docker-compose logs -f mongodb
docker-compose logs -f xxl-job
docker-compose logs -f rocketmq-namesrv
docker-compose logs -f rocketmq-broker
docker-compose logs -f rocketmq-dashboard
```

### 重启全部服务

```bash
docker-compose restart
```

---

## 服务详情与连接信息

### MySQL

- **镜像**: `mysql:8.0`
- **平台**: `linux/amd64`
- **端口**: `3306`
- **用户名**: `root`
- **密码**: `root123`

**命令行连接示例：**

```bash
# 使用 mysql 客户端
mysql -h localhost -P 3306 -u root -p
# 输入密码: root123

# 或者直接指定密码（不推荐，密码会出现在命令历史中）
mysql -h localhost -P 3306 -u root -proot123
```

**连接字符串示例：**

```text
# JDBC (Java)
jdbc:mysql://localhost:3306/?useSSL=false&serverTimezone=Asia/Shanghai&characterEncoding=utf8mb4

# Python (pymysql/mysql-connector)
mysql://root:root123@localhost:3306/

# Node.js (mysql2)
mysql://root:root123@localhost:3306/

# Go (go-sql-driver/mysql)
root:root123@tcp(localhost:3306)/
```

**常见客户端：**

- **DBeaver**: Host: `localhost`, Port: `3306`, User: `root`, Password: `root123`
- **Navicat**: 主机: `localhost`, 端口: `3306`, 用户名: `root`, 密码: `root123`
- **MySQL Workbench**: 连接方式 "Standard TCP/IP"，主机: `localhost`, 端口: `3306`, 用户名: `root`, 密码: `root123`
- **TablePlus**: Host: `localhost`, Port: `3306`, User: `root`, Password: `root123`

#### MySQL 初始化 SQL

MySQL 容器首次启动时（数据目录为空时），会自动执行 `mysql/init/` 目录下的 SQL 文件：

- 支持 `.sql`、`.sh`、`.sql.gz` 格式
- 按文件名排序执行（建议使用数字前缀，如 `01-init.sql`）
- 仅在数据目录为空时执行
- 如需重新执行初始化，需要先删除数据目录：`rm -rf mysql/data/*`

示例文件：

- `mysql/init/01-init.sql` - 通用初始化脚本示例
- `mysql/init/02-xxl-job.sql` - XXL-JOB 数据库初始化脚本

**XXL-JOB 数据库初始化脚本：**

`02-xxl-job.sql` 包含基本表结构。如需官方最新版脚本，可执行：

```bash
curl -o mysql/init/02-xxl-job.sql https://raw.githubusercontent.com/xuxueli/xxl-job/master/doc/db/tables_xxl_job.sql
```

或直接访问：  
`https://raw.githubusercontent.com/xuxueli/xxl-job/master/doc/db/tables_xxl_job.sql`

---

### Redis

- **镜像**: `redis:7-alpine`
- **平台**: `linux/arm64/v8`
- **端口**: `6379`
- **密码**: 无（开发环境）

**命令行连接示例：**

```bash
# 使用 redis-cli
redis-cli -h localhost -p 6379

# 或者直接（默认连接 localhost:6379）
redis-cli
```

**连接字符串示例：**

```text
# Redis URI
redis://localhost:6379

# Python (redis-py)
redis://localhost:6379/0

# Node.js (ioredis)
redis://localhost:6379
```

---

### MongoDB

- **镜像**: `mongodb/mongodb-community-server:7.0-ubuntu2204`
- **平台**: `linux/arm64/v8`
- **端口**: `27017`
- **认证**: 默认无认证（开发环境，可在配置中开启）

**命令行连接示例：**

```bash
# 使用 mongo shell / mongosh
mongosh "mongodb://localhost:27017"
```

**连接字符串示例：**

```text
# 通用 MongoDB URI
mongodb://localhost:27017

# Node.js (mongoose / 官方驱动)
mongodb://localhost:27017/test

# Java (MongoDB Java Driver)
mongodb://localhost:27017

# Python (pymongo)
mongodb://localhost:27017
```

---

### RocketMQ & Dashboard

- **RocketMQ 镜像**: `apache/rocketmq:5.3.2`
- **平台**: `linux/amd64`
- **端口**:
  - NameServer: `9876`
  - Broker: `10911`
- **Dashboard 镜像**: `apacherocketmq/rocketmq-dashboard:latest`
- **Dashboard 地址**: `http://localhost:8081`（容器内为 8080，对外映射为 8081，避免与 XXL-JOB 冲突）

在 `docker-compose.yml` 中，RocketMQ 以「单机模式」运行：容器内同时启动 NameServer 和 Broker，并允许自动创建 Topic（`autoCreateTopicEnable=true`），适合本地开发调试。

**客户端连接示例（Java）：**

```text
# NameServer 地址
namesrvAddr=localhost:9876
```

Dashboard 会通过环境变量 `ROCKETMQ_CONFIG_NAMESRV_ADDR=mini-env-rocketmq:9876` 自动连接到 RocketMQ。

---

### XXL-JOB

- **Web 管理界面**: `http://localhost:8080/xxl-job-admin`
- **默认用户名**: `admin`
- **默认密码**: `123456`
- **数据库**: `xxl_job`（存储在 MySQL 中）

**启动顺序建议：**

1. 先启动基础服务：

   ```bash
   docker-compose up -d mysql redis mongodb
   ```

2. 确认 MySQL 健康后启动 XXL-JOB：

   ```bash
   docker-compose up -d xxl-job
   ```

---

## 内存与性能配置

- **MySQL**：限制内存 **512MB**
  - InnoDB Buffer Pool: 128MB（优化配置）
  - 最大连接数: 50（从 200 降低）
  - 各类缓冲区尽量缩小
  - 关闭慢查询日志以节省资源
- **Redis**：限制内存 **256MB**，最大内存约 **200MB**
- **MongoDB**：容器内存限制 **512MB**
- **XXL-JOB**：容器内存限制 **512MB**

> **注意：**
> - 如果遇到性能瓶颈，可适当增大 `innodb_buffer_pool_size` 到 192M 或更高  
> - 如果连接数不足，可将 `max_connections` 提升到 100

---

## Mac M4 芯片兼容性说明

本项目已在 **Mac M4 (Apple Silicon)** 上测试通过，并针对平台做了优化：

- **MySQL 8.0**：`platform: linux/amd64`，通过 Rosetta 2 转译运行，保证兼容性
- **Redis 7**：`platform: linux/arm64/v8`，原生 ARM64，性能更优
- **MongoDB 7.0 Community Server**：`platform: linux/arm64/v8`，原生 ARM64
- **XXL-JOB**：基于 OpenJDK 8，`platform: linux/amd64`，通过 Rosetta 2 转译运行

---

## 自定义与扩展

### 添加新的中间件

建议遵循统一目录结构：

```bash
{中间件名称}/
├── config/          # 配置文件
├── data/            # 数据目录
└── logs/            # 日志目录
```

然后在 `docker-compose.yml` 中增加对应服务配置即可。

### 重新初始化 MySQL

如果需要重新执行初始化 SQL（例如修改了初始化脚本），操作步骤如下：

```bash
# 停止容器
docker-compose down

# 删除数据目录（注意：这会删除所有数据）
rm -rf mysql/data/*

# 重新启动，会自动执行初始化 SQL
docker-compose up -d
```
