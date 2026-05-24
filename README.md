# My Caddy

基于 [Caddy](https://github.com/caddyserver/caddy) 官方镜像，通过 [xcaddy](https://github.com/caddyserver/xcaddy) 集成常用第三方模块，每日自动构建并推送至 GitHub Container Registry (GHCR)。

## 已集成模块

| 模块 | 说明 |
|------|------|
| [caddy-dns/cloudflare](https://github.com/caddy-dns/cloudflare) | Cloudflare DNS-01 验证，用于自动化 HTTPS 证书签发 |
| [caddyserver/nginx-adapter](https://github.com/caddyserver/nginx-adapter) | 支持直接使用 Nginx 配置文件运行 Caddy |
| [caddyserver/replace-response](https://github.com/caddyserver/replace-response) | 对响应内容进行查找替换 |
| [hairyhenderson/caddy-teapot-module](https://github.com/hairyhenderson/caddy-teapot-module) | HTTP 418 "I'm a teapot" 趣味模块 |

> 完整构建配置见 [Dockerfile](./Dockerfile)。

## 快速开始

### 拉取预构建镜像

```bash
docker pull ghcr.io/sixiaolong1117/my-caddy:latest
```

### 运行

```bash
docker run -d \
  --name caddy \
  -p 80:80 \
  -p 443:443 \
  -v /path/to/Caddyfile:/etc/caddy/Caddyfile \
  -v caddy_data:/data \
  ghcr.io/sixiaolong1117/my-caddy:latest
```

## 本地构建

如果你需要自定义模块组合，可以克隆本项目并自行构建：

```bash
git clone https://github.com/SIXiaolong1117/my-caddy.git
cd my-caddy
docker build -t my-caddy .
```

构建过程分为两个阶段：
1. **构建阶段**：基于 `caddy:2-builder` 镜像，使用 `xcaddy` 编译带自定义模块的 Caddy 二进制文件
2. **运行阶段**：基于 `caddy:2` 官方镜像，将编译好的二进制文件复制到最终镜像中

## 自动构建

本项目使用 GitHub Actions 实现自动化构建（[`build.yml`](./.github/workflows/build.yml)）：

- **定时触发**：每天 UTC 2:00 自动构建，确保镜像始终包含 Caddy 和所有模块的最新版本
- **推送触发**：向 `main` 分支推送代码时自动构建
- **手动触发**：支持通过 `workflow_dispatch` 手动运行

构建完成后镜像自动推送至 `ghcr.io/${{ github.repository }}:latest`。

### Fork 后使用

Fork 本项目后，GitHub Actions 会自动将镜像推送至你的 GHCR 地址：

```bash
docker pull ghcr.io/<你的用户名>/my-caddy:latest
```

> [!NOTE]
> GitHub 用户名在 GHCR 镜像路径中必须使用**全小写**。如果 Fork 后镜像拉取失败，请检查 `ghcr.io/<用户名>/my-caddy` 中的用户名是否已转为小写。

## 自定义模块

编辑 [Dockerfile](./Dockerfile) 中的 `xcaddy build` 命令，增删 `--with` 参数即可：

```dockerfile
RUN xcaddy build \
    --with github.com/caddyserver/nginx-adapter \
    --with github.com/你的模块路径
```

可在 [Caddy 模块注册表](https://caddyserver.com/download) 中查找更多可用模块。

## 许可

本项目基于 MIT 协议开源，详见 [LICENSE](./LICENSE)。

Caddy 及其相关模块版权归各自作者所有。