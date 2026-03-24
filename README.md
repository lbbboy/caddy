# Caddy with Cloudflare DNS Plugin

为 ARM64 / AMD64 平台预编译的 Caddy，内置 [caddy-dns/cloudflare](https://github.com/caddy-dns/cloudflare) 插件，支持 Cloudflare DNS-01 证书自动签发。

## 为什么需要自编译？

Caddy 官方下载页面打包的 cloudflare 插件版本存在滞后，旧版插件的 Token 格式校验不支持 `cfat_` 前缀的新式 API Token，导致启动时报错：
```
API token 'cfat_xxx' appears invalid; ensure it's correctly entered and not wrapped in braces nor quotes
```

本项目通过 GitHub Actions 使用 xcaddy 编译最新版插件，解决上述问题。

## 下载

前往 [Releases](../../releases/latest) 页面下载对应平台的二进制文件：

| 文件 | 平台 |
|------|------|
| `caddy_linux_amd64` | x86_64 |
| `caddy_linux_arm64` | ARM64 |

## 安装
```bash
# 以 arm64 为例
wget https://github.com/lbbboy/caddy/releases/latest/download/caddy_linux_arm64

chmod +x caddy_linux_arm64
sudo systemctl stop caddy
sudo mv /usr/bin/caddy /usr/bin/caddy.bak
sudo mv caddy_linux_arm64 /usr/bin/caddy
sudo systemctl start caddy
```

## Caddyfile 配置

### 全局配置（推荐）
```caddy
{
    acme_dns cloudflare {env.CF_API_TOKEN}
}

example.com {
    reverse_proxy localhost:8080
}
```

### 注入环境变量
```bash
systemctl edit caddy
```

填入：
```ini
[Service]
Environment="CF_API_TOKEN=你的 Cloudflare API Token"
```
```bash
systemctl daemon-reload
systemctl restart caddy
```

## Cloudflare API Token 权限

在 [Cloudflare Dashboard](https://dash.cloudflare.com/profile/api-tokens) 创建 Token，最少需要：

| 类型 | 权限 |
|------|------|
| Zone - DNS | 编辑 |
| Zone - Zone | 读取 |

区域资源选择包含你域名的账户即可。

## 构建

本项目通过 GitHub Actions 自动构建，使用 xcaddy 拉取最新插件：
```
github.com/caddy-dns/cloudflare@latest
```

如需手动触发构建，前往 Actions → Build Caddy → Run workflow。
