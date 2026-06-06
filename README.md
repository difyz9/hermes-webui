# hermes-webui Docker Mirror

本仓库做两件事：

1. 每周自动跟踪 [nesquena/hermes-webui](https://github.com/nesquena/hermes-webui) 上游最新 tag，构建多架构镜像并发布到 Docker Hub：`difyz9/hermes-webui`。
2. 提供两套可直接运行的 Docker Compose 模板：
   - `docker-compose.two-container.yml`（推荐默认）
   - `docker-compose.three-container.yml`（带 Dashboard）

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

## 二、Compose 使用教程（只保留 two/three）

### 文件说明

- `docker-compose.two-container.yml`：Hermes Agent + Hermes WebUI（推荐）
- `docker-compose.three-container.yml`：Hermes Agent + Hermes WebUI + Hermes Dashboard

### 推荐选择

- 日常使用：优先 `two-container`
- 需要监控面板（Dashboard 9119）：使用 `three-container`

### 启动前准备

创建专用工作目录（不要挂载整个 home 目录）：

```bash
mkdir -p ~/hermes-workspace
```

创建 `.env`（macOS/Linux 都建议显式设置 UID/GID）：

```bash
cat > .env <<EOF
UID=$(id -u)
GID=$(id -g)
HERMES_WORKSPACE=$HOME/hermes-workspace
HERMES_WEBUI_PASSWORD=change-this-strong-password
EOF
```

### 启动方式

启动 two-container（推荐）：

```bash
docker compose --env-file .env -f docker-compose.two-container.yml pull
docker compose --env-file .env -f docker-compose.two-container.yml up -d
```

如果你把 `docker-compose.two-container.yml` 重命名为 `docker-compose.yml`，并且环境文件仍使用默认 `.env`，可以简化为：

```bash
docker compose pull
docker compose up -d
```

启动 three-container：

```bash
docker compose --env-file .env -f docker-compose.three-container.yml pull
docker compose --env-file .env -f docker-compose.three-container.yml up -d
```

### 访问地址

- two-container：
  - WebUI: `http://127.0.0.1:8787`
- three-container：
  - WebUI: `http://127.0.0.1:8787`
  - Dashboard: `http://127.0.0.1:9119`

### 初始化 Hermes

two-container：

```bash
docker compose --env-file .env -f docker-compose.two-container.yml exec hermes-agent hermes setup
docker compose --env-file .env -f docker-compose.two-container.yml exec hermes-agent hermes model
docker compose --env-file .env -f docker-compose.two-container.yml exec hermes-agent hermes tools
```

如果 two-container 已重命名为 `docker-compose.yml`，对应命令可以简化为：

```bash
docker compose exec hermes-agent hermes setup
docker compose exec hermes-agent hermes model
docker compose exec hermes-agent hermes tools
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
