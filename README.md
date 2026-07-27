# Mukun Jin's Blog

基于 Hugo 与 PaperMod 主题构建的个人博客，部署于 Cloudflare。

## 技术栈

- Hugo
- PaperMod 主题
- Markdown

## 目录结构

```
.
├── content/
│   └── posts/          # 文章源文件（Markdown）
├── static/
│   └── images/         # 图片资源
├── themes/
│   └── PaperMod/       # 主题（git submodule）
├── hugo.toml           # Hugo 配置
├── .gitignore
└── README.md
```

## 本地运行

```bash
hugo server
```

默认在 `http://localhost:1313` 启动。

## 写作

新建文章：

```bash
hugo new content posts/article-title.md
```

在 `content/posts/` 下编辑对应的 Markdown 文件。Front-matter 示例：

```yaml
---
title: "文章标题"
date: 2026-06-16
categories: ["分类名"]
---
```

## 部署

推送到 `main` 分支后，会自动触发构建和部署。

```bash
git add .
git commit -m "update blog"
git push
```

## 主要配置

| 文件 | 说明 |
|------|------|
| `hugo.toml` | 网站标题、描述、菜单、社交链接、主题配置 |
