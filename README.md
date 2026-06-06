# hermes-webui Docker Mirror

每日自动跟踪 [nesquena/hermes-webui](https://github.com/nesquena/hermes-webui) 上游仓库的最新 tag，构建多架构 Docker 镜像并推送到 Docker Hub。

## 功能

- **定时同步**：每周六 UTC 01:00（北京时间 09:00）自动运行，检测上游最新 tag
- **多架构构建**：同时构建 `linux/amd64` 和 `linux/arm64` 镜像，适配 x86 服务器及 ARM 设备（如树莓派、Apple Silicon）
- **自动打标签**：每次构建同时推送三个 tag：
  - `vX.Y.Z`（完整版本号，如 `v1.2.3`）
  - `X.Y.Z`（不带 `v` 前缀）
  - `latest`
- **构建缓存**：使用 GitHub Actions Cache 加速重复构建

## 使用镜像

```bash
docker pull difyz9/hermes-webui:latest
```

或指定版本：

```bash
docker pull difyz9/hermes-webui:v1.2.3
```

## 配置

在 GitHub 仓库中配置以下 Secrets 和 Variables：

| 类型 | 名称 | 说明 |
|------|------|------|
| Secret | `DOCKERHUB_USERNAME` | Docker Hub 用户名 |
| Secret | `DOCKERHUB_TOKEN` | Docker Hub Access Token |
| Variable（可选）| `DOCKERHUB_IMAGE` | 自定义镜像名，默认为 `<username>/hermes-webui` |

## 触发方式

| 触发条件 | 说明 |
|----------|------|
| 定时任务 | 每周六 UTC 01:00 自动执行 |
| 推送 tag | 推送 `v*` 格式的 tag 时触发 |
| 手动触发 | 在 Actions 页面手动运行 `workflow_dispatch` |

## 工作流程

```
检测上游最新 tag
       ↓
克隆对应版本源码
       ↓
设置 QEMU + Buildx（多架构支持）
       ↓
登录 Docker Hub
       ↓
构建 amd64 + arm64 镜像
       ↓
推送镜像（vX.Y.Z / X.Y.Z / latest）
```

## 上游仓库

[https://github.com/nesquena/hermes-webui](https://github.com/nesquena/hermes-webui)
