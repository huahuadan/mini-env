# mini-env

本地开发环境 Docker Compose 配置

## 包含的中间件

- MySQL 8.0 (支持 Mac M4 芯片，使用 linux/amd64 平台通过 Rosetta 2 运行)
- Redis 7 (Alpine，原生支持 ARM64，在 M4 芯片上使用 linux/arm64/v8 平台)

## 目录结构

```
mini-env/
├── docker-compose.yml
├── mysql/
│   ├── config/          # MySQL 配置文件
│   ├── data/            # MySQL 数据目录（已加入 .gitignore）
│   ├── logs/            # MySQL 日志目录（已加入 .gitignore）
│   └── init/            # MySQL 初始化 SQL 脚本目录
├── redis/
│   ├── config/          # Redis 配置文件
│   ├── data/            # Redis 数据目录（已加入 .gitignore）
│   └── logs/            # Redis 日志目录（已加入 .gitignore）
└── README.md
```

## 使用方法

### 启动服务

```bash
docker-compose up -d
```

### 停止服务

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
```

### 重启服务

```bash
docker-compose restart
```

## MySQL 初始化 SQL

MySQL 容器首次启动时（数据目录为空时），会自动执行 `mysql/init/` 目录下的 SQL 文件。

- 支持 `.sql`、`.sh`、`.sql.gz` 格式的文件
- 文件按文件名排序执行（建议使用数字前缀，如 `01-init.sql`）
- 只有在数据目录为空时才会执行
- 如需重新执行初始化，需要先删除数据目录：`rm -rf mysql/data/*`

示例文件：`mysql/init/01-init.sql`

## 连接信息

### MySQL

- **主机 (Host)**: `localhost` 或 `127.0.0.1`
- **端口 (Port)**: `3306`
- **用户名 (Username)**: `root`
- **密码 (Password)**: `root123`

#### 连接示例

**命令行连接：**
```bash
# 使用 mysql 客户端
mysql -h localhost -P 3306 -u root -p
# 输入密码: root123

# 或者直接指定密码（不推荐，密码会出现在命令历史中）
mysql -h localhost -P 3306 -u root -proot123
```

**连接字符串示例：**
```
# JDBC (Java)
jdbc:mysql://localhost:3306/?useSSL=false&serverTimezone=Asia/Shanghai&characterEncoding=utf8mb4

# Python (pymysql/mysql-connector)
mysql://root:root123@localhost:3306/

# Node.js (mysql2)
mysql://root:root123@localhost:3306/

# Go (go-sql-driver/mysql)
root:root123@tcp(localhost:3306)/
```

**常见数据库客户端：**
- **DBeaver**: Host: `localhost`, Port: `3306`, User: `root`, Password: `root123`
- **Navicat**: 主机: `localhost`, 端口: `3306`, 用户名: `root`, 密码: `root123`
- **MySQL Workbench**: 连接方式选择 "Standard TCP/IP"，主机: `localhost`, 端口: `3306`, 用户名: `root`, 密码: `root123`
- **TablePlus**: Host: `localhost`, Port: `3306`, User: `root`, Password: `root123`

### Redis

- **主机 (Host)**: `localhost` 或 `127.0.0.1`
- **端口 (Port)**: `6379`
- **密码**: 无（开发环境）

#### 连接示例

**命令行连接：**
```bash
# 使用 redis-cli
redis-cli -h localhost -p 6379

# 或者直接（默认连接 localhost:6379）
redis-cli
```

**连接字符串示例：**
```
# Redis URI
redis://localhost:6379

# Python (redis-py)
redis://localhost:6379/0

# Node.js (ioredis)
redis://localhost:6379
```

## 内存优化

- **MySQL**: 限制内存使用 **512MB**
  - InnoDB Buffer Pool: 128MB（优化后的配置）
  - 最大连接数: 50（从 200 降低）
  - 各种缓冲区已最小化配置
  - 关闭慢查询日志以节省内存
- **Redis**: 限制内存使用 **256MB**，最大内存 200MB

> **注意**: 
> - 如果遇到性能问题，可以适当增加 `innodb_buffer_pool_size` 到 192M 或更高
> - 如果连接数不足，可以增加 `max_connections` 到 100

## Mac M4 芯片兼容性

本配置已针对 Mac M4 (Apple Silicon) 芯片进行优化：

- **MySQL 8.0**: 使用 `platform: linux/amd64`，通过 Rosetta 2 转译运行，确保兼容性
- **Redis 7**: 使用 `platform: linux/arm64/v8`，原生 ARM64 支持，性能更优

所有镜像均已在 M4 芯片上测试通过。

## 添加新的中间件

当需要添加新的中间件时，请遵循以下目录结构：

```
{中间件名称}/
├── config/          # 配置文件
├── data/            # 数据目录
└── logs/            # 日志目录
```

并在 `docker-compose.yml` 中添加相应的服务配置。

## 重新初始化 MySQL

如果需要重新执行初始化 SQL（例如修改了初始化脚本），需要：

```bash
# 停止容器
docker-compose down

# 删除数据目录（注意：这会删除所有数据）
rm -rf mysql/data/*

# 重新启动，会自动执行初始化 SQL
docker-compose up -d
```
