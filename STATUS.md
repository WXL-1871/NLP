# 项目状态 · STATUS

> 滚动更新：完成 / 当前阻塞 / 下一步

**最后更新**：Hexo 站点上线调试中（GitHub Pages 启用阻塞）

---

## 阶段进度

| 阶段 | 状态 | 备注 |
| --- | --- | --- |
| 1. 环境（Node 24 / npm 11 / git 2.39） | ✅ | |
| 2. Hexo 8 + NexT 8.29 安装 | ✅ | Gemini scheme |
| 3. 主题配置（菜单、TOC、MathJax、代码块、avatar、footer） | ✅ | |
| 4. 3 篇示例文章（ML / DL / LLM） | ✅ | linear-regression / transformer-explained / llm-prompt-engineering |
| 5. 分类 / 标签 / 关于页 | ✅ | |
| 6. `.gitignore` + `README.md` + `HANDOFF.md` | ✅ | hexo-starter 默认 + 补全 |
| 7. 本地预览（`hexo server`） | ✅ | 6 个 URL 均 200 |
| 8. 源码推送到 GitHub | ✅ | `hexo-source` 分支 commit `9e7d7c9` |
| 9. public 推送到 GitHub | ✅ | `main` 分支 commit `0c7148b` |
| 10. **GitHub Pages 启用** | ❌ 阻塞 | 用户在网页端操作未成功，未提供截图 |
| 11. 站点在线验证 | ⏸ 待 Pages 启用 | 11 个 URL 待测 |

## 关键 SHA

| 分支 | SHA | 含义 |
| --- | --- | --- |
| `hexo-source` | `9e7d7c9` | 完整源码 |
| `main` | `0c7148b` | public 产物（已生成 73 文件） |

## 关键 URL（启用后应当 200）

```
https://wxl-1871.github.io/NLP/
https://wxl-1871.github.io/NLP/2024/05/20/2034/   ← 线性回归
https://wxl-1871.github.io/NLP/2024/06/10/7769/   ← Transformer
https://wxl-1871.github.io/NLP/2024/07/05/504f/   ← LLM Prompt
https://wxl-1871.github.io/NLP/categories/
https://wxl-1871.github.io/NLP/tags/
https://wxl-1871.github.io/NLP/about/
https://wxl-1871.github.io/NLP/css/main.css
https://wxl-1871.github.io/NLP/js/main.js
https://wxl-1871.github.io/NLP/images/avatar.gif
https://wxl-1871.github.io/NLP/atom.xml
```

## 下一步

1. 用户在 https://github.com/WXL-1871/NLP/settings/pages 启用 Pages（Source = `Deploy from a branch`、Branch = `main`、Folder = `/(root)`）
2. 截整页图回传 → AI 根据截图判断失败原因
3. Pages 启用后跑 `STATUS.md` 验证脚本（见 HANDOFF.md 第二节）
4. 若启用成功，更新本文档 "10. GitHub Pages 启用" → ✅
5. 交付完成

## 已知坑

- 不要装 `hexo-renderer-pandoc`
- 不要写 `sitemap.template`
- Windows git push HTTPS 走 schannel
- 改源码用 `hexo-source`，发布用 `main`