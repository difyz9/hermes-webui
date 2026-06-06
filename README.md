# hermes-webui Docker Mirror

本仓库做两件事：

1. 每周自动跟踪 [nesquena/hermes-webui](https://github.com/nesquena/hermes-webui) 上游最新 tag，构建多架构镜像并发布到 Docker Hub：`difyz9/hermes-webui`。
2. 提供可直接运行的 Docker Compose 部署方式：
  - `docker-compose.yml`（推荐默认，一键启动）
  - `docker-compose.two-container.yml`（与默认方案等价的显式命名版本）
  - `docker-compose.three-container.yml`（带 Dashboard）

## 快速开始

这是默认推荐方式，适合绝大多数用户。

### 1. 准备工作目录

```bash
mkdir -p ./workspace
```

### 2. 生成 `.env`

```bash
cp .env.example .env
```

按需修改 `.env` 中的以下配置：

- `UID` 和 `GID`：建议改成当前系统用户的 `id -u` / `id -g`
- `HERMES_WORKSPACE`：默认是仓库内的 `./workspace`，也可以改成你自己的专用目录
- `HERMES_WEBUI_PASSWORD`：如果不是纯本机访问，建议设置强密码

### 3. 一键启动

```bash
docker compose pull
docker compose up -d
```

### 4. 初始化 Hermes

```bash
docker compose exec hermes-agent hermes setup
docker compose exec hermes-agent hermes model
docker compose exec hermes-agent hermes tools
```

### 5. 访问

- WebUI: `http://127.0.0.1:8787`

### 6. 常用命令

```bash
docker compose ps
docker compose logs -f hermes-agent
docker compose logs -f hermes-webui
docker compose down
```

### 7. 常见问题

- 如果 `docker compose exec hermes-agent hermes setup` 报错，先执行 `docker compose ps`，确认容器已经启动
- 如果 WebUI 打不开，先看 `docker compose logs -f hermes-webui`
- 如果工作目录权限异常，检查 `.env` 中的 `UID`、`GID` 是否与宿主机用户一致

## 一、镜像自动构建与发布

### 功能

- 每周六 UTC 01:00（北京时间 09:00）自动检测上游最新 tag
- 支持多架构：`linux/amd64`、`linux/arm64`
- 固定发布仓库：`difyz9/hermes-webui`
### 触发方式

- 定时任务：每周六 UTC 01:00
- 推送 tag：推送 `v*` 格式 tag 时触发
- 手动触发：Actions 页面 `workflow_dispatch`

### 必需 Secrets

在 GitHub 仓库中配置：

- `DOCKERHUB_USERNAME`
- `DOCKERHUB_TOKEN`

### 拉取镜像

```bash
docker pull difyz9/hermes-webui:latest
```

## 二、Compose 使用教程（默认 two-container，另提供 three-container）

### 文件说明

- `docker-compose.yml`：Hermes Agent + Hermes WebUI（推荐默认，一键启动）
- `docker-compose.two-container.yml`：Hermes Agent + Hermes WebUI（与默认方案等价）
- `docker-compose.three-container.yml`：Hermes Agent + Hermes WebUI + Hermes Dashboard

### 推荐选择

- 日常使用：优先 `docker-compose.yml`
- 需要监控面板（Dashboard 9119）：使用 `three-container`

### 启动前准备

创建专用工作目录（不要挂载整个 home 目录）：

```bash
mkdir -p ./workspace
```

创建 `.env`（macOS/Linux 都建议显式设置 UID/GID）：

```bash
cp .env.example .env
```

然后按需修改 `.env` 的 `UID`、`GID`、`HERMES_WORKSPACE`、`HERMES_WEBUI_PASSWORD`。

### 启动方式

启动默认方案（推荐，一键启动）：

```bash
docker compose pull
docker compose up -d
```

启动 two-container（显式文件名方式）：

```bash
docker compose --env-file .env -f docker-compose.two-container.yml pull
docker compose --env-file .env -f docker-compose.two-container.yml up -d
```

启动 three-container：

```bash
docker compose --env-file .env -f docker-compose.three-container.yml pull
docker compose --env-file .env -f docker-compose.three-container.yml up -d
```

### 访问地址

- 默认方案 / two-container：
  - WebUI: `http://127.0.0.1:8787`
- three-container：
  - WebUI: `http://127.0.0.1:8787`
  - Dashboard: `http://127.0.0.1:9119`

### 初始化 Hermes

默认方案：

```bash
docker compose exec hermes-agent hermes setup
docker compose exec hermes-agent hermes model
docker compose exec hermes-agent hermes tools
```

two-container（显式文件名方式）：

```bash
docker compose --env-file .env -f docker-compose.two-container.yml exec hermes-agent hermes setup
docker compose --env-file .env -f docker-compose.two-container.yml exec hermes-agent hermes model
docker compose --env-file .env -f docker-compose.two-container.yml exec hermes-agent hermes tools
```

three-container：

```bash
docker compose --env-file .env -f docker-compose.three-container.yml exec hermes-agent hermes setup
docker compose --env-file .env -f docker-compose.three-container.yml exec hermes-agent hermes model
docker compose --env-file .env -f docker-compose.three-container.yml exec hermes-agent hermes tools
```

### 停止与清理

停止（保留数据）：

```bash
docker compose down
docker compose --env-file .env -f docker-compose.two-container.yml down
docker compose --env-file .env -f docker-compose.three-container.yml down
```

删除数据卷（谨慎）：

```bash
docker volume rm hermes-home hermes-agent-src
```

### 安全建议

- 只挂载 `HERMES_WORKSPACE`，不要挂载 `/`、`/home`、`/Users`、`~/.ssh`、`~/.hermes`
- 不要挂载 Docker socket（`/var/run/docker.sock`）
- 保持端口绑定到 `127.0.0.1`，远程访问优先走 SSH 隧道

## 上游仓库

- [https://github.com/nesquena/hermes-webui](https://github.com/nesquena/hermes-webui)
