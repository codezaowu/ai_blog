# AI Blog — Hugo 使用指南

> 地址: https://codezaowu.github.io/ai_blog/
>
> Hugo v0.163.1 | Theme: PaperMod

---

## 快速发布

```bash
# 1. 写新文章
hugo new content posts/你的文章名.md

# 2. 编辑文章，把 draft 改为 false
vim content/posts/你的文章名.md

# 3. 本地预览
hugo server -D

# 4. 发布（自动触发 GitHub Actions 构建）
git add -A
git commit -m "新文章: 你的文章名"
git push
```

等待约 1-2 分钟，Actions 构建完成后即可访问。

---

## 日常操作

### 写文章

```bash
hugo new content posts/my-post.md
```

在 `content/posts/` 下生成 `.md` 文件，格式：

```markdown
+++
title = '文章标题'
date = 2026-06-15T22:00:00+08:00
draft = false          # true=草稿（不发布）, false=发布
description = '摘要'
tags = ['tag1', 'tag2']
+++

正文内容...
```

### 本地预览

```bash
hugo server -D
# -D 显示草稿，去掉则只显示已发布文章
```

访问 http://localhost:1313

### 构建静态文件

```bash
hugo --gc --minify
# 输出到 public/ 目录
```

---

## 文章规范

### 注意事项

1. **日期不能是未来时间** — Hugo 默认不渲染未来日期的页面。如要在未来定时发布，需要在构建时加 `--buildFuture` 参数（CI 已配置）
2. **draft 改为 false** 才会发布
3. **图片**放在 `static/images/` 下，引用路径 `/images/xxx.png`
4. **标签不能有空格**，用引号包裹

### Front Matter 模板

```markdown
+++
title = ''
date = {{ now.Format "2006-01-02T15:04:05+08:00" }}
draft = true
description = ''
tags = []
+++
```

---

## 站点配置

主配置文件 `hugo.toml`：

| 配置项 | 说明 |
|--------|------|
| `baseURL` | 站点根 URL |
| `title` | 站点名称 |
| `theme` | PaperMod |
| `paginate` | 每页文章数 |

---

## CI/CD

GitHub Actions 配置在 `.github/workflows/hugo.yml`。

触发方式：推送到 `master` 分支后自动构建并部署到 GitHub Pages。

手动触发：在 Actions 页面点击 "Run workflow"。

---

## 目录结构

```
ai_blog/
├── content/          # 文章源码 (.md)
│   └── posts/        # 博客文章
├── static/           # 静态资源（图片等）
├── themes/
│   └── PaperMod/     # 主题（git submodule）
├── hugo.toml         # 站点配置
├── .github/
│   └── workflows/
│       └── hugo.yml  # CI 配置
└── public/           # 构建产物（gitignore）
```

---

## 主题参考

- PaperMod 文档: https://github.com/adityatelange/hugo-PaperMod
- Hugo 文档: https://gohugo.io/documentation/
