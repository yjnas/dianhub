# DianHub - Docker 镜像代理服务

<div align="center">

![DianHub Logo](static/logo.png)

**轻量级 Docker Hub 镜像信息查询工具**

[![Docker Image](https://img.shields.io/badge/docker-yjnas%2Fdianhub-blue)](https://hub.docker.com/r/yjnas/dianhub)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

[功能特性](#功能特性) • [快速开始](#快速开始) • [使用方法](#使用方法) • [配置说明](#配置说明) • [API 文档](#api-文档)

</div>

---

## 📖 项目介绍

DianHub 是一个基于 Go 语言开发的轻量级 Docker 镜像代理服务，专注于提供快速、准确的 Docker Hub 镜像 digest 查询功能。无需下载完整镜像，即可获取镜像的元数据信息。

### 为什么选择 DianHub？

- 🚀 **高性能**：基于 Go 语言开发，响应速度快
- 🔍 **精准查询**：快速获取镜像 digest 和 manifest 信息
- 🛡️ **安全可靠**：不存储镜像数据，直接转发到 Docker Hub
- 🎨 **友好界面**：提供简洁的 Web UI 和完整的 API
- 📦 **开箱即用**：Docker 一键部署，配置简单

---

## ✨ 功能特性

### 核心功能

- **Digest 查询**：快速查询任意 Docker Hub 镜像的 digest 信息
- **Manifest 详情**：获取完整的镜像 manifest 配置信息
- **多格式支持**：支持 Docker v2、OCI 等多种 manifest 格式
- **实时日志**：Web 界面实时查看系统运行日志
- **可选拉取**：支持配置是否允许镜像 blob 下载

### 支持的镜像类型

- ✅ 官方镜像：`nginx`、`redis`、`mysql` 等
- ✅ 用户镜像：`username/imagename`
- ✅ 完整路径：`library/nginx`

---

## 🚀 快速开始

### 使用 Docker

```bash
# 拉取镜像
docker pull yjnas/dianhub:latest

# 运行容器
docker run -d \
  -p 8080:8080 \
  -e HOST_KEY=yjnas \
  --name dianhub \
  yjnas/dianhub:latest
```

### 使用 Docker Compose（推荐）

创建 `docker-compose.yml` 文件：

```yaml
version: '3.8'

services:
  dianhub:
    image: yjnas/dianhub:latest
    container_name: dianhub
    ports:
      - "8080:8080"
    environment:
      - PORT=8080
      - HOST_KEY=yjnas
      - IMAGES_PULL=false
    restart: unless-stopped
```

启动服务：

```bash
docker-compose up -d
```

---

## 📝 使用方法

### Web 界面

访问 `http://localhost:8080` 打开 Web 界面，可以：

1. **镜像查询**：输入镜像名称和标签，查询 digest 信息
2. **查看日志**：实时查看系统运行日志
3. **项目介绍**：了解项目功能和使用说明

### API 调用

#### 查询镜像 Digest

```bash
curl http://localhost:8080/v2/nginx/manifests/latest
```

响应示例：

```json
{
  "image": "library/nginx",
  "reference": "latest",
  "digest": "sha256:...",
  "content_type": "application/vnd.docker.distribution.manifest.v2+json",
  "size": 1234,
  "manifest": { ... }
}
```

#### 检查服务状态

```bash
curl http://localhost:8080/v2/
```

#### 获取系统日志

```bash
curl http://localhost:8080/api/logs
```

---

## ⚙️ 配置说明

### 环境变量

| 变量名 | 必需 | 默认值 | 说明 |
|--------|------|--------|------|
| `PORT` | 否 | `8080` | 服务监听端口 |
| `HOST_KEY` | **是** | - | 启动密钥（必须设置为 `yjnas`） |
| `IMAGES_PULL` | 否 | `false` | 是否允许镜像 blob 下载（详见下方说明） |

#### IMAGES_PULL 变量说明

`IMAGES_PULL` 环境变量控制服务是否允许下载镜像 blob 数据：

- **`false`（默认，推荐）**：
  - ✅ 仅支持镜像元数据查询（digest、manifest）
  - ✅ 不允许下载镜像 blob 数据
  - ✅ 更安全，资源占用更少
  - ✅ 适合大多数使用场景
  - ⚠️ 无法使用 `docker pull` 通过此服务拉取镜像

- **`true`（按需启用）**：
  - ✅ 支持完整的镜像拉取功能
  - ✅ 可以使用 `docker pull` 拉取镜像
  - ⚠️ 会消耗更多带宽和存储资源
  - ⚠️ 需要确保服务器有足够的性能
  - ⚠️ 建议仅在需要镜像代理功能时启用

**使用建议：**
- 如果只需要查询镜像信息（digest、版本等），保持 `IMAGES_PULL=false`
- 如果需要作为 Docker 镜像代理使用，设置 `IMAGES_PULL=true`

### 配置示例

#### 仅查询模式（推荐）

```bash
docker run -d \
  -p 8080:8080 \
  -e HOST_KEY=yjnas \
  -e IMAGES_PULL=false \
  yjnas/dianhub:latest
```

#### 允许拉取模式

```bash
docker run -d \
  -p 8080:8080 \
  -e HOST_KEY=yjnas \
  -e IMAGES_PULL=true \
  yjnas/dianhub:latest
```

---

## 📚 API 文档

### 端点列表

| 方法 | 路径 | 说明 |
|------|------|------|
| `GET` | `/` | Web 界面首页 |
| `GET` | `/v2/` | Registry API 版本信息 |
| `GET` | `/v2/{name}/manifests/{tag}` | 查询镜像 digest |
| `GET` | `/v2/{name}/blobs/{digest}` | 下载 blob（需启用 IMAGES_PULL） |
| `GET` | `/api/logs` | 获取系统日志 |

### 使用示例

```bash
# 查询 nginx 最新版本
curl http://localhost:8080/v2/nginx/manifests/latest

# 查询 redis 7.0 版本
curl http://localhost:8080/v2/library/redis/manifests/7.0

# 查询用户镜像
curl http://localhost:8080/v2/username/imagename/manifests/v1.0
```

---

## 💡 使用场景

- ✅ 验证镜像是否存在于 Docker Hub
- ✅ 获取镜像的精确 digest 用于版本固定
- ✅ 查看镜像的层信息和配置
- ✅ CI/CD 流程中的镜像验证
- ✅ 镜像版本管理和追踪
- ✅ 开发环境中的镜像信息查询

---

## 🛡️ 安全说明

- 本服务**不存储**任何镜像数据
- 默认仅支持元数据查询，不支持镜像下载
- 所有请求转发到官方 Docker Hub
- 使用 Docker Hub 官方认证机制
- 需要设置 `HOST_KEY` 环境变量才能启动

---

## 👨‍💻 作者

**YJNAS DIAN**

- 微信：dongqc123
- 博客：[wiki.929722.xyz](https://wiki.929722.xyz)
- B站：[昱君NAS](https://space.bilibili.com/668023659)
- 闲鱼：[昱君NAS小店](https://m.tb.cn/h.7Tfpr15?tk=ambnfAdmiqA)

---

## 💖 支持项目

如果这个项目对你有帮助，欢迎：

- ⭐ Star 本项目
- 📢 分享给更多人

---

## 📮 联系方式

如有问题或建议，欢迎通过以下方式联系：

- 提交 [Issue](https://github.com/yourusername/dianhub/issues)
- B站私信：[昱君NAS](https://space.bilibili.com/668023659)
- 微信：dongqc123

---

<div align="center">

**⭐ 如果觉得项目不错，请给个 Star 支持一下！⭐**

Made with ❤️ by YJNAS DIAN

</div>
