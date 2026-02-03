# GitHub CI/CD 部署指南

## 部署流程

本项目使用 GitHub Actions 自动构建和部署。

### 触发条件

- 推送到 `main` 或 `master` 分支时自动触发部署
- Pull Request 到 `main` 或 `master` 分支时只构建，不部署

## 配置 GitHub Secrets

在 GitHub 仓库设置中添加以下 Secrets：

| Secret Name | 说明 | 示例 |
|------------|------|------|
| `SSH_PRIVATE_KEY` | 服务器的 SSH 私钥 | `-----BEGIN OPENSSH PRIVATE KEY-----...` |
| `REMOTE_HOST` | 服务器 IP 或域名 | `192.168.1.100` 或 `example.com` |
| `REMOTE_USER` | SSH 用户名 | `root` 或 `ubuntu` |
| `REMOTE_PATH` | 服务器上的部署目录 | `/var/www/html/` 或 `/usr/share/nginx/html/` |
| `REMOTE_PORT` | SSH 端口（可选，默认22） | `22` |

### 设置步骤

1. **生成 SSH 密钥对**（如果还没有）：
   ```bash
   ssh-keygen -t ed25519 -C "github-actions" -f ~/.ssh/github_actions
   ```

2. **将公钥添加到服务器**：
   ```bash
   ssh-copy-id -i ~/.ssh/github_actions.pub user@your-server
   # 或者手动添加到服务器的 ~/.ssh/authorized_keys
   ```

3. **在 GitHub 添加 Secrets**：
   - 进入仓库 -> Settings -> Secrets and variables -> Actions
   - 点击 "New repository secret"
   - 添加以上配置项
   - `SSH_PRIVATE_KEY` 的值是私钥文件内容：`cat ~/.ssh/github_actions`

## 服务器配置建议

### Nginx 配置示例

```nginx
server {
    listen 80;
    server_name your-domain.com;
    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # 静态资源缓存
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

## 手动测试部署

如需手动触发部署，可以：

1. 在 GitHub Actions 页面点击 "Run workflow"
2. 或者推送一个空提交：
   ```bash
   git commit --allow-empty -m "Trigger deployment"
   git push
   ```

## 常见问题

1. **权限问题**：确保 SSH 用户有写入 `REMOTE_PATH` 的权限
2. **端口问题**：如果 SSH 不是默认 22 端口，设置 `REMOTE_PORT`
3. **构建失败**：检查 `pnpm-lock.yaml` 是否已提交到仓库
