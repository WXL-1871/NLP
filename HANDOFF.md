# HANDOFF · 博客接续说明

> 把这份文档贴给下一个 AI 模型（或别的开发者），对方就能 0 上下文接续此博客项目。

---

## 一句话现状

基于 **Hexo 8 + NexT 8.29（Gemini scheme）** 的个人博客，用于发布机器学习 / 深度学习 / 大模型相关文章。
**源码、主题、3 篇示例文章均已就绪并推送到 GitHub** `WXL-1871/NLP`。当前**唯一阻塞点**：用户在 GitHub 网页端手动启用 Pages 失败，需要排查页面设置。

## 部署目标

| 项 | 值 |
| --- | --- |
| GitHub 用户名 | `WXL-1871` |
| 仓库 | `WXL-1871/NLP`（项目页，不是 `<user>.github.io`） |
| Pages URL | https://wxl-1871.github.io/NLP/ |
| 源码分支 | `hexo-source` |
| Pages 分支 | `main`（commit `0c7148b` 已含 `public/` 产物） |

`blog/_config.yml` 已写好：
```yaml
url: https://WXL-1871.github.io/NLP
root: /NLP/
deploy:
  type: git
  repo: https://github.com/WXL-1871/NLP.git
  branch: main
```

## 已完成 ✅

- 3 篇示例文章：
  - `source/_posts/linear-regression.md`（ML / 数学公式 + Python 代码）
  - `source/_posts/transformer-explained.md`（DL / Transformer + PyTorch）
  - `source/_posts/llm-prompt-engineering.md`（LLM / Prompt + ReAct + RAG）
- 分类页 `source/categories/index.md`、标签页 `source/tags/index.md`、关于页 `source/about/index.md`
- 主题 `themes/next/`（NexT 8.29，Gemini 方案）
- 主题开关：TOC、代码块（折叠 + 复制 + 语言标签）、MathJax（ams, linebreak）、darkmode、avatar、footer.since=2024
- 插件：`hexo-deployer-git`、`hexo-generator-feed`、`hexo-generator-sitemap`、`hexo-abbrlink`、`hexo-symbols-count-time`、`hexo-filter-mathjax`
- 源码分支 `hexo-source` 已 push 到 origin（commit `9e7d7c9`）
- public/ 已通过 `hexo deploy` 推到 `main`（commit `0c7148b`，强制更新）
- `.gitignore` 正确忽略 `node_modules/`、`public/`、`db.json`、`.deploy*/`
- README.md 已写

## 本地验证记录

```bash
hexo clean && hexo generate    # 73 文件、无 FATAL
hexo server -p 4000            # 6 个 URL 均 200
# 资源前缀全部以 /NLP/ 开头
```

## 当前阻塞 ⚠️

**GitHub Pages 未启用**。证据：
- `GET https://api.github.com/repos/WXL-1871/NLP` → `has_pages: False`
- `GET .../pages` → 404
- `GET .../pages/builds` → 404
- `https://wxl-1871.github.io/NLP/...` 全部 404

仓库状态：public、未归档、未禁用、main 分支未保护、无 workflow 抢占。用户在 Settings→Pages 操作失败，未提供截图。

## 接手者第一动作：诊断 Pages 启用失败

请让用户按以下步骤操作并把**整页截图**发回来：

1. 浏览器登录 GitHub
2. 打开 https://github.com/WXL-1871/NLP/settings/pages
3. **截图整页**（包括顶部横幅、Build and deployment 区块、任何报错）

### 截图诊断表

| 截图看到的内容 | 原因 | 修复 |
| --- | --- | --- |
| 下拉框显示 `None` 或 `GitHub Actions` | Source 未正确选 | 改选 `Deploy from a branch` + Branch=`main` + Folder=`/(root)` → Save |
| 已是 `Deploy from a branch`，Save 无响应 | 浏览器缓存 / 权限 | Ctrl+F5 强刷；换浏览器；换账号试 |
| 顶部绿条 `Your site is live at https://wxl-1871.github.io/NLP/` | **已启用** | 立刻跑下面的验证脚本 |
| 仅显示 `GitHub Actions only` | 仓库锁定 Actions 模式 | 改用 GitHub Actions 部署（需新写 `.github/workflows/deploy.yml`） |
| 显示 `Pro plan required` | 仓库被设私有 | Settings → General → Danger Zone → Change visibility → public |
| 显示 `Pages disabled at the organization level` | 组织策略 | 联系 org admin |
| 整个页面 404 | 没权限或路径错 | 确认登录 `WXL-1871` 账号 |

## 接手者第二动作：Pages 启用后跑这段验证

PowerShell：

```powershell
$r = Invoke-WebRequest -Uri "https://api.github.com/repos/WXL-1871/NLP" -UseBasicParsing -TimeoutSec 10
$j = $r.Content | ConvertFrom-Json
"has_pages=$($j.has_pages)"  # 期望 True

$api = Invoke-WebRequest -Uri "https://api.github.com/repos/WXL-1871/NLP/pages" -UseBasicParsing -TimeoutSec 10
$j = $api.Content | ConvertFrom-Json
"state=$($j.state) | url=$($j.html_url)"  # 期望 state=built

$urls = @(
  "https://wxl-1871.github.io/NLP/",
  "https://wxl-1871.github.io/NLP/2024/05/20/2034/",
  "https://wxl-1871.github.io/NLP/2024/06/10/7769/",
  "https://wxl-1871.github.io/NLP/2024/07/05/504f/",
  "https://wxl-1871.github.io/NLP/categories/",
  "https://wxl-1871.github.io/NLP/tags/",
  "https://wxl-1871.github.io/NLP/about/",
  "https://wxl-1871.github.io/NLP/css/main.css",
  "https://wxl-1871.github.io/NLP/js/main.js",
  "https://wxl-1871.github.io/NLP/images/avatar.gif",
  "https://wxl-1871.github.io/NLP/atom.xml"
)
foreach($u in $urls){
  try { $r = Invoke-WebRequest -Uri $u -UseBasicParsing -TimeoutSec 10; "$u : $($r.StatusCode)" }
  catch { "$u : $($_.Exception.Response.StatusCode.value__)" }
}
# 期望：全部 200
```

## 接手者第三动作：日常写作流程

```bash
cd E:\code\boke\blog
git checkout hexo-source

# 写新文章
hexo new post "<文章标题>"
# 编辑 source/_posts/<slug>.md（Front-matter 中加 mathjax: true 若含公式）

# 本地预览
hexo server -p 4000

# 备份源码
git add . && git commit -m "post: 新增 <文章标题>"
git push origin hexo-source

# 上线
hexo clean && hexo deploy
```

## 环境与命令速查

| 命令 | 用途 |
| --- | --- |
| `hexo clean` | 清 `public/` 与 `db.json` |
| `hexo generate` | 生成静态站点 |
| `hexo server -p 4000` | 本地预览 http://localhost:4000 |
| `hexo deploy` | 把 `public/` 推到 origin/main |
| `hexo new post <title>` | 新建文章 |
| `npm install --registry=https://registry.npmmirror.com` | 安装依赖（国内镜像） |

工作目录始终是 `E:\code\boke\blog\`。PowerShell 用 `workdir=` 参数而非 `cd && cmd`。

## 避坑（已踩过的坑，不要重蹈）

- ❌ 不要装 `hexo-renderer-pandoc`（本机无 pandoc → FATAL）
- ❌ sitemap 不要写 `template: ./sitemap_template.xml`（FATAL）
- ❌ git push `https://github.com/...` 走 openssl 会证书失败；用 schannel
  ```bash
  git config --global http.sslbackend schannel
  ```
- ❌ 不要直接在 main 分支改源码，`hexo deploy` 会冲掉它；源码改 hexo-source

## 关键文件清单

- `blog/_config.yml`：站点主配置（已配 url/root/deploy）
- `blog/themes/next/_config.yml`：主题配置（avatar / footer 已设）
- `blog/source/_posts/*.md`：3 篇示例文章
- `blog/source/{categories,tags,about}/index.md`：聚合页
- `blog/README.md`：项目说明
- `blog/HANDOFF.md`：**本文档**
- `blog/STATUS.md`：进度状态
- `blog/.gitignore`：hexo 默认已满足