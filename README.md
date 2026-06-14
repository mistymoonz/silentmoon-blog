# Silentmoon-静月

基于 [Hugo](https://gohugo.io/) + [Congo](https://github.com/jpanther/congo) 主题的个人博客。

在线地址：[silentmoon.top](https://silentmoon.top)

## 技术栈

- **框架**：Hugo Extended（Go 静态站点生成器）
- **主题**：Congo v2 + 自定义覆盖
- **样式**：Tailwind CSS 3.0 + 紫蓝品牌色系
- **部署**：纯静态 HTML，Caddy/Nginx 服务

## 目录结构

```
├── hugo.toml                  # Hugo 主配置
├── config/_default/           # 站点配置（语言、菜单、主题参数）
├── content/                   # Markdown 内容
│   ├── _index.md              # 首页 Hero 文字
│   ├── posts/                 # 博客文章
│   ├── about/                 # 关于页
│   └── friends/               # 友链页
├── assets/
│   ├── css/custom.css         # 全部自定义样式
│   ├── css/schemes/custom.css # 紫蓝配色方案
│   └── img/bgpic.png          # Hero 背景图
├── layouts/                   # 自定义模板（覆盖 Congo）
│   ├── _default/baseof.html   # 全站基础结构
│   ├── partials/              # 组件（hero、卡片、页脚、导航）
│   └── posts/list.html        # 博客列表（标签筛选）
└── static/                    # 静态资源
```

## 本地运行

```bash
# 安装 Hugo Extended（v0.163+）
# macOS: brew install hugo
# Windows: winget install Hugo.Hugo.Extended

# 启动开发服务器
hugo server -D

# 浏览器打开 http://localhost:1313
```

## 写新文章

```bash
hugo new content posts/2026-06-14-文章标题.md
```

编辑生成的 Markdown 文件：

```yaml
---
title: "文章标题"
date: 2026-06-14
tags: ["标签1", "标签2"]
category: "分类名"
description: "一句话摘要"
draft: false
---

正文内容...
```

## 构建部署

```bash
# 本地构建
hugo --minify

# 上传到服务器
scp -r public/* user@你的服务器:/var/www/silentmoon/
```

服务器 Caddy 配置示例：

```caddyfile
silentmoon.top {
    root * /var/www/silentmoon
    file_server
    encode gzip zstd
}
```

## 自定义

- **Hero 文字**：编辑 `content/_index.md` 的 `title` 和 `description`
- **配色**：编辑 `assets/css/schemes/custom.css`
- **样式**：编辑 `assets/css/custom.css`
- **友链**：编辑 `content/friends/_index.md`
- **联系方式**：编辑 `content/about/_index.md`
- **备案号**：编辑 `layouts/partials/footer.html`

## License

MIT
