# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

Hexo 静态博客，部署在 GitHub Pages，使用自定义域名 `foampray.com`。仓库为 `foamshadow/foamshadow.github.io`。

## 常用命令

```bash
# 本地开发服务器（http://localhost:4000）
npm run server

# 生成静态文件
npm run build

# 清理生成文件
npm run clean

# 新建文章
npx hexo new "文章标题"

# 新建草稿
npx hexo new draft "文章标题"

# 发布草稿
npx hexo publish "文章标题"
```

## 部署

- 推送到 `main` 分支自动触发 GitHub Actions 部署（`.github/workflows/pages.yml`）
- Pages Source 必须设为 **GitHub Actions**（不是 Deploy from branch）
- 自定义域名通过 `source/CNAME` 文件和 GitHub Pages Settings 共同控制
- Node.js 版本：22

## 配置

- `_config.yml` — 主配置（站点信息、URL、permalink 格式等）
- 主题：`landscape`（Hexo 官方默认主题，通过 npm 安装在 node_modules）
- 语言：zh-CN，时区：Asia/Shanghai
- Permalink 格式：`:year/:month/:day/:title/`

## 目录结构

- `source/_posts/` — 博客文章（Markdown）
- `source/CNAME` — 自定义域名声明
- `scaffolds/` — 文章模板
- `public/` — 生成的静态文件（已 gitignore，由 CI 生成）
