# ML & DL 笔记

一个基于 **Hexo + NexT** 的个人博客，用来记录机器学习、深度学习与大语言模型的学习笔记、实战与思考。

- **机器学习**：经典算法原理与代码实现（线性回归、决策树、SVM、聚类、降维……）
- **深度学习**：CNN / RNN / Transformer / 训练技巧与工程实践
- **大模型 (LLM)**：预训练、微调、Prompt 工程、RAG、Agent

## 目录结构

```
blog/
├─ _config.yml               # 站点主配置
├─ themes/next/_config.yml   # NexT 主题配置
├─ source/
│  ├─ _posts/                # 博客文章（Markdown）
│  ├─ categories/index.md    # 分类聚合页
│  ├─ tags/index.md          # 标签聚合页
│  └─ about/index.md         # 关于页
├─ public/                   # hexo generate 输出（不入版本库）
└─ .deploy_git/              # hexo deploy 缓存（不入版本库）
```

## 环境要求

- Node.js ≥ 18
- npm ≥ 9
- git

## 常用命令

在 `blog/` 目录下执行：

| 命令 | 作用 |
| --- | --- |
| `npm install` | 安装依赖（首次或拉取新依赖时） |
| `hexo clean` | 清理 `public/` 与 `db.json` |
| `hexo generate` | 生成静态站点到 `public/` |
| `hexo server -p 4000` | 本地预览，访问 http://localhost:4000 |
| `hexo new post <title>` | 新建一篇博客文章 |
| `hexo deploy` | 一键部署到 GitHub Pages（需先在 `_config.yml` 配置 `deploy.repo`） |

推荐开发循环：`hexo clean && hexo generate && hexo server`。

## 写作约定

### Front-matter

```yaml
---
title: 文章标题
date: 2024-05-20 10:00:00
categories:
  - 机器学习     # 或 深度学习 / 大模型 / 随笔
tags:
  - 标签1
  - 标签2
mathjax: true             # 含公式时打开
description: 一句话简介   # 列表页与 RSS 用
---
```

### 数学公式

需要公式的文章，在 Front-matter 中加 `mathjax: true` 即可（NexT 已配置 MathJax，AMS 扩展）。

行内：$E = mc^2$

独立公式：

$$
\nabla \cdot \mathbf{E} = \frac{\rho}{\varepsilon_0}
$$

### 代码块

使用三反引号围栏代码块，语言标识（如 `python`、`bash`、`yaml`）会被 Next 自动识别并启用高亮 + 复制按钮 + 折叠：

```python
import torch
x = torch.randn(2, 3)
print(x.shape)  # torch.Size([2, 3])
```

### 图片与附件

`_config.yml` 已开启 `post_asset_folder: true`。在 `source/_posts/` 下建与文章同名的文件夹，把图片丢进去，Markdown 里直接 `![alt](image.png)` 即可。

## 主题与插件

- 主题：[NexT 8.x](https://theme-next.js.org/)（scheme = Gemini，深色模式开启）
- 渲染：Hexo 内置 `marked` + `highlight.js`
- 数学：`hexo-filter-mathjax`
- 永久链接：`hexo-abbrlink`
- 字数统计：`hexo-symbols-count-time`
- Feed / Sitemap：`hexo-generator-feed` / `hexo-generator-sitemap`
- 部署：`hexo-deployer-git`

主题主要开关集中在 `themes/next/_config.yml`：

- `scheme`（Muse / Mist / Pisces / Gemini）
- `menu`（导航菜单）
- `toc`、`codeblock`、`math`、`darkmode`
- `avatar`、`footer`

## 部署到 GitHub Pages

当前 `_config.yml` 的 `deploy.repo` 与 `url` 仍为占位符 `yourname.github.io`，部署前请改成自己的仓库地址：

```yaml
url: https://<your-github-username>.github.io
permalink_defaults:
deploy:
  type: git
  repo: https://github.com/<your-github-username>/<your-github-username>.github.io.git
  branch: main
```

然后执行 `hexo clean && hexo deploy`，首次部署需要 git 推送权限。

## 排错速查

| 现象 | 处理 |
| --- | --- |
| `Error: spawnSync pandoc ENOENT` | 不要安装 `hexo-renderer-pandoc`（本机无 pandoc）。如已装，运行 `npm uninstall hexo-renderer-pandoc`。 |
| `Error: ENOENT ... sitemap_template.xml` | `_config.yml` 中 sitemap 不要指定 `template:` 字段。 |
| 数学公式不渲染 | 确认文章 Front-matter 有 `mathjax: true`，且主题中 `math.mathjax.enable: true`。 |
| 本地 4000 端口占用 | `hexo server -p 4001` 换端口。 |

## 许可

博客正文采用 [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh)，代码片段采用 MIT。