
# hermes-webui (Docker Mirror)

`difyz9/hermes-webui` 是对上游 `nesquena/hermes-webui` 的自动同步镜像仓库。  
项目会定期检查上游最新版本并自动构建多架构镜像发布到 Docker Hub。

## 项目亮点

- 自动跟踪上游版本 tag
- 自动构建并推送多架构镜像
- 固定发布仓库：`difyz9/hermes-webui`
- 提供稳定的 `latest` 与版本标签

## 支持架构

- `linux/amd64`
- `linux/arm64`

## 标签策略

每次发布会同步推送以下标签：

- `vX.Y.Z`（例如 `v1.2.3`）
- `X.Y.Z`（例如 `1.2.3`）
- `latest`

## 更新策略

镜像发布触发方式：

- 每周六 UTC 01:00 定时同步
- 推送 `v*` tag 时触发构建
- 支持手动触发发布流程

## 快速开始

拉取最新版：

```bash
docker pull difyz9/hermes-webui:latest
```

拉取指定版本：

```bash
docker pull difyz9/hermes-webui:v1.2.3
```

如果你希望直接从本项目仓库一键启动，推荐使用仓库内默认的 `docker-compose.yml`：

```bash
git clone https://github.com/difyz9/hermes-webui.git
cd hermes-webui
mkdir -p ./workspace
cp .env.example .env
docker compose pull
docker compose up -d
docker compose exec hermes-agent hermes setup
```

启动后访问：

```text
http://127.0.0.1:8787
```

如需监控面板，可改用仓库内的 `docker-compose.three-container.yml`。

## Compose 使用建议

推荐两种部署方式：见本仓库源码中的 `docker-compose.two-container.yml` 和 `docker-compose.three-container.yml`。

- Two-container：Hermes Agent + Hermes WebUI（推荐默认）
- Three-container：Hermes Agent + Hermes WebUI + Dashboard

如需生产部署，请优先使用仅暴露必要目录和端口的方式，并保持 `127.0.0.1` 本地绑定。

## 说明

- 本仓库为自动构建镜像分发仓库，便于 Docker Hub 直接拉取与部署。
- 上游项目地址：<https://github.com/nesquena/hermes-webui>
- 镜像仓库地址：<https://hub.docker.com/r/difyz9/hermes-webui>
- 源码仓库地址：<https://github.com/difyz9/hermes-webui>