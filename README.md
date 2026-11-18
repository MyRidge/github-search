# GitHub 搜索

一个仿百度风格的单文件 GitHub 项目搜索页。双击 `index.html` 就能用，不需要安装、不需要构建、不需要后端。

![单文件](https://img.shields.io/badge/单文件-56KB-blue) ![无依赖](https://img.shields.io/badge/依赖-无-success) ![协议](https://img.shields.io/badge/协议-MIT-lightgrey)

## 快速开始

```
双击 index.html
```

或者用任意方式托管到静态服务器（GitHub Pages、Nginx、Vercel 都行），整个项目只有一个文件。

首次打开会停在一个居中的搜索框，回车或点「搜索」进入结果页。

## 功能

### 搜索

- 关键词搜索仓库，或切到「用户」页签搜用户
- 四种排序：最佳匹配、星数最多、Fork 最多、最近更新
- 每页 10 条，最多 100 页（受 GitHub 搜索接口 `page × per_page ≤ 1000` 限制）
- 支持 GitHub 原生搜索语法，例如 `language:python stars:>10000`、`topic:llm`
- 首页四个快捷词：最热门项目、Python 神器、LLM 相关、前端轮子

### 结果卡片

每张卡片展示：

| 字段 | 说明 |
| --- | --- |
| 仓库名 | `owner/name`，点击新标签打开 GitHub |
| 描述 | 作者填写的项目简介，缺失时提示 |
| **最新 Release** | 版本号、发布时间、附件数量；点击展开更新说明 |
| **README 摘要** | 自动清洗后取前 300 字，两行截断，点击展开全文 |
| topics | 点击后按该 topic 重新搜索 |
| 语言 | 带 GitHub 官方配色圆点 |
| stars / forks / issues | 紧凑数字，如 `253k`、`52.8k` |
| 更新时间 | 相对时间，悬停显示精确日期 |
| 标记 | 已归档、Fork 会额外标注 |

### 中文翻译

README 摘要和 Release 更新说明默认显示中文，右下角「原文」按钮可切回英文原文。

翻译走浏览器内置的 **Translator API**：

- 模型在本地设备运行，**内容不出本机**，不联网、不花钱、无请求数限制
- 需要 Chrome / Edge **138 或更高版本**
- 首次使用会下载语言模型，首页会显示下载进度
- 浏览器不支持时自动回退显示英文原文，其他功能不受影响
- 只翻摘要和更新说明，不翻仓库名、描述、topics、语言名（避免对不上原项目）

### 性能

所有请求都做了缓存，翻页、切排序、重搜、刷新都不会重复打接口：

| 内容 | 存储位置 | 生命周期 |
| --- | --- | --- |
| 搜索结果 | 内存 | 当前页面会话 |
| README 摘要 | sessionStorage | 当前标签页会话 |
| Release 信息 | sessionStorage | 当前标签页会话 |
| 中文译文 | localStorage | 持久 |

README 和 Release 采用懒加载：首屏前 3 条立即取，其余滚动到视口附近才取（`IntersectionObserver` + `rootMargin: 200px`）。

### Token 与限流

GitHub 搜索接口对匿名请求限制约 **每分钟 10 次**。点击 ⚙ 填入 Personal Access Token 可提升到每分钟 30 次。

- Token 只存在你自己的浏览器 `localStorage`（键名 `gh_search_token`），不会发往除 `api.github.com` 之外的任何地方
- 建议创建时**不勾任何权限**，或只勾 `public_repo`——搜索公开仓库不需要额外权限
- 被限流时页面会提示剩余额度和重置时间

### 快捷键

| 按键 | 作用 |
| --- | --- |
| `/` | 聚焦搜索框 |
| `Esc` | 关闭设置面板 |

### URL 状态同步

搜索条件会写进 URL hash，可以直接分享或收藏：

```
index.html#q=react&sort=stars&page=3&tab=users
```

支持的参数：`q`（关键词）、`sort`（stars / forks / updated）、`page`（页码）、`tab`（repos / users）。

## 技术说明

- **零依赖**：原生 HTML + CSS + JavaScript，没有框架、没有构建步骤、没有 npm install
- **浏览器直连** GitHub REST API v3（`https://api.github.com`），没有中间层
- 用到的接口：
  - `GET /search/repositories`
  - `GET /search/users`
  - `GET /repos/{owner}/{repo}/readme`
  - `GET /repos/{owner}/{repo}/releases?per_page=1`
- 响应式布局，窄屏可用

## 已知限制

- **受 GitHub 匿名限流约束**：不加 Token 时频繁搜索会被临时拒绝，提示里会给出重置时间
- **最多 100 页**：这是 GitHub 搜索接口的硬上限，翻不到第 101 页
- **中文翻译依赖新版本浏览器**：Chrome / Edge 138 以下没有 Translator API，会显示英文原文
- **摘要由正则清洗**：README 的 markdown 结构千奇百怪，个别项目可能残留意译不通的片段或表格痕迹

## 目录结构

```
github-search/
├── index.html   # 全部内容：样式、逻辑、图标都在里面
├── README.md    # 本文件
├── LICENSE      # MIT 开源协议全文
└── .gitignore   # Git 忽略规则
```

## 常见问题

**双击打开是白屏？**
用浏览器打开，不要用文本编辑器。如果浏览器极旧（不支持 ES2020），请升级。

**为什么搜索结果和 github.com 网页上不一样？**
调的是 GitHub 官方搜索接口，排序算法与网页版略有差异，尤其在「最佳匹配」下。

**翻译没生效？**
先确认浏览器版本 ≥ 138（在地址栏输入 `chrome://version` 查看）。不满足就会显示英文原文，这是预期行为。

**能离线用吗？**
搜索本身必须联网（要请求 GitHub）。已经缓存过的摘要和译文在离线时仍能显示。

## 协议

本项目基于 [MIT License](LICENSE) 开源。

你可以自由使用、修改、分发，包括商用，只需保留版权声明与许可声明。软件按「原样」提供，不附带任何担保。
