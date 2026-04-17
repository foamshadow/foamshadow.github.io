---
title: Hexo 博客部署到 GitHub Pages 操作流程
data: 2026-04-17 14:27:57
---
## 0. 前置准备

### 0.1 创建 GitHub Pages 仓库

1. 登录 [GitHub](https://github.com)，点击右上角 `+` → `New repository`
2. Repository name 填写：`你的用户名.github.io`（如 `foamshadow.github.io`）
   - 必须严格按此格式命名，GitHub 才会识别为 Pages 仓库
3. 选择 Public（Pages 需要 public 仓库）
4. 可勾选 Add a README file
5. 点击 Create repository

### 0.2 安装 Hexo 前置依赖

Hexo 依赖 Node.js 和 Git，确保已安装：

```bash
node -v
npm -v
git --version
```

如未安装：
- **Node.js**：推荐使用 [fnm](https://github.com/Schniz/fnm) 管理（`fnm install 22`）
- **Git**：`winget install Git.Git`

安装 Hexo CLI（可选，方便全局使用 hexo 命令）：

```bash
npm install -g hexo-cli
```

### 0.3 安装 GitHub CLI（可选，用于命令行管理仓库）

```bash
winget install GitHub.cli
```

安装后登录认证：

```bash
gh auth login
```

选择 `GitHub.com` → `HTTPS` → `Login with a web browser`。

## 1. 初始化 Hexo 项目

```bash
npx hexo init .
```

在当前目录初始化 Hexo 博客框架，生成基础文件结构。

```bash
npm install
```

安装所有依赖包。

```bash
npx hexo server
```

启动本地开发服务器（默认 `http://localhost:4000`），确认博客能正常运行。按 `Ctrl+C` 停止。

## 2. 配置 _config.yml

编辑 `_config.yml`，修改以下关键项：

```yaml
# Site
title: Foam's Blog
subtitle: '记录技术与生活'
description: '个人技术博客'
author: foamshadow
language: zh-CN
timezone: 'Asia/Shanghai'

# URL
url: https://foampray.com
```

## 3. 创建自定义域名文件

```bash
echo "foampray.com" > source/CNAME
```

GitHub Pages 读取此文件来绑定自定义域名。

## 4. 创建 GitHub Actions 工作流

创建文件 `.github/workflows/pages.yml`：

```yaml
name: Deploy Hexo to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Build Hexo
        run: npx hexo generate

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: public

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

## 5. 初始化 Git 并推送

```bash
git init
git branch -M main
git remote add origin https://github.com/foamshadow/foamshadow.github.io.git
git add .
git commit -m "Initial Hexo blog setup"
git push -u origin main
```

如果远程仓库已有内容导致推送被拒绝，使用强制推送：

```bash
git push -u origin main --force
```

如果需要撤销上一个 commit 并重新提交：

```bash
git reset --soft HEAD~1
git add .
git commit -m "新的提交信息"
git push --force
```

## 6. 配置 GitHub Pages

### 6.1 设置 Source

进入仓库 **Settings → Pages → Source**，选择 **GitHub Actions**（不是 Deploy from branch）。

### 6.2 设置自定义域名

通过 GitHub CLI 设置：

```bash
gh api repos/foamshadow/foamshadow.github.io/pages --method PUT -f cname=foampray.com
```

或在仓库 **Settings → Pages → Custom domain** 中填入 `foampray.com`。

## 7. 配置 DNS 记录

到域名注册商后台添加以下 DNS 记录：

| 主机记录 | 记录类型 | 记录值 |
|---------|---------|--------|
| @ | A | 185.199.108.153 |
| @ | A | 185.199.109.153 |
| @ | A | 185.199.110.153 |
| @ | A | 185.199.111.153 |
| www | CNAME | foamshadow.github.io |

## 8. 验证部署

```bash
# 检查工作流运行状态
gh run list --limit 5

# 检查 Pages 配置
gh api repos/foamshadow/foamshadow.github.io/pages
```

确认以下项目：
- `cname` 为 `foampray.com`
- `status` 为 `built`
- `https_enforced` 为 `true`
- `https_certificate.state` 为 `approved`

等待 DNS 生效（通常几分钟到几小时）后访问 `https://foampray.com`。
