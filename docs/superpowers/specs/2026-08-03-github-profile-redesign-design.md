# GitHub 个人主页重设计规格

日期：2026-08-03
状态：已与用户逐项确认

## 背景与目标

仓库 `frewily/frewily` 是 GitHub 个人主页 README。当前风格为粉色二次元 + 彩虹徽章，视觉不统一。目标：重设计为**暗夜霓虹**风格（深空底 `#0d1117`→`#1a1030` 渐变，青色 `#7ee0ff` 与紫色 `#c792ff` 霓虹强调色），提升独特性与视觉统一度。

约束：GitHub README 不允许自定义 CSS，所有视觉效果只能通过图片素材（自制 SVG、shields 徽章、第三方 SVG 服务）实现。SVG 通过 `<img>` 加载时不可点击、不可引用外部资源（图片必须 base64 内嵌）。

## 已确认的决策

| 决策点 | 结论 |
| --- | --- |
| 整体风格 | 暗夜霓虹（深空底 + 青紫霓虹渐变 + 终端气质） |
| 版块取舍 | 保留：关于我（精简）、技术栈、精选项目、动态数据区；砍掉：Profile Views 计数器、"感谢你的来访"文字 |
| 实现路线 | 混合方案：自制 SVG（hero、footer）+ readme-typing-svg + shields 徽章 + metrics 深色化 |
| Hero 右侧素材 | 两版都保留：B 整幅融入为默认，A 人物立绘为备用 |
| 页尾 | 渐变收尾带，不含 frewily.top 域名（网站入口只在顶部徽章） |
| 插画来源 | 使用仓库现有 `image/138888613_p0.png`（已与用户确认就是主页那张） |

## 页面结构（自上而下）

1. **Hero**：`assets/header.svg`（B 版整幅融入，新建）
2. **打字动画行**：readme-typing-svg，霓虹渐变配色
3. **徽章行**：个人网站 frewily.top（第一）→ GitHub，深色底霓虹描边 shields
4. **## 关于我**：精简到 2-3 行；删除原来的 `<img src="image/138888613_p0.png">`（插画已进入 hero）
5. **## 技术栈**：新配色徽章（见下）
6. **## 精选项目**：Markdown 表格原样不动
7. **## 开发活动**：snake `<picture>` 明暗切换 + metrics 左右两张 SVG 并排
8. **页尾**：`assets/footer.svg`（新建）

## 素材清单

### 新建：`assets/header.svg`（Hero，B 版整幅融入）

- viewBox `0 0 1000 300`，圆角 16px，自带深色渐变底（`#0d1117`→`#131a2c`→`#1a1030`），细网格纹理（`#7ee0ff` 5% 透明度），1px 青色 18% 透明度描边
- 右侧 68%（x=320 起）铺插画：`image/138888613_p0.png` 压缩为 800px JPEG（质量约 78，约 88KB）后 **base64 内嵌**；插图上压 8% 深色统一色调
- 左侧融入渐变：x=200 起 420px 宽，从 `#0d1117` 不透明渐变到完全透明（45% 处 0.85、80% 处 0.15），确保渐变不压人物——人物与海景必须在清晰区
- 左侧文字块（x=64）：
  - mono 字体 `// welcome to my profile`（15px，青色，字距 2）
  - `Frewily`（56px 800 粗细，青→紫渐变填充 + 发光滤镜）
  - `Java 后端 · AI 应用开发者`（20px，`#e6edf3`）
  - `构建能解决真实问题的 AI 应用 — RAG / Agent / LLM 工程化`（14px，`#a8b3c0`）
  - 两个装饰 pill（不可点击）：`frewily.top`（青描边）、`github.com/frewily`（紫描边）
- 字体栈：标题/正文 `-apple-system, 'Segoe UI', 'PingFang SC', 'Microsoft YaHei', sans-serif`；mono `ui-monospace, 'SF Mono', Menlo, Consolas, monospace`

### 新建：`assets/header-standee.svg`（备用 Hero，A 版立绘）

- 与 B 版同底色、同文字块；右侧为人物立绘 + 双层光环（r=108 实线渐变圈 + r=126 青色虚线轨道圈）
- 立绘来源：macOS Vision 框架对原图做主体抠图（原型脚本在 `.superpowers/brainstorm/` 会话目录，实现时重新生成并裁剪底部约 70px 去除残留光斑与 "My Dear Moments" 文字），base64 内嵌
- 不上主页，仅作备用素材存入仓库，README 不引用

### 新建：`assets/footer.svg`（页尾收尾带）

- viewBox `0 0 1000 140`，圆角 16px，渐变方向与 hero 镜像（`#1a1030`→`#131a2c`→`#0d1117`）
- 中间一条 640×2 发光渐变线（紫→青→紫），线上方居中 mono 文字 `// thanks for visiting`（14px 青色，字距 3）——**不含 frewily.top 域名**
- 少量星点装饰

### 修改：readme-typing-svg 配色

- 保留现有服务与文案（两行：身份行 + slogan），`color` 参数改为 `7EE0FF`（该服务仅支持单色），其余参数现状保持

### 修改：shields 徽章

- 顶部徽章行：个人网站（第一）+ GitHub，`style=flat-square`、底色 `0d1117`，logoColor 分别用青 `7EE0FF` / 紫 `C792FF`（shields 不支持描边，neon 感靠底色 + logo 色表达）
- 技术栈徽章：`style=for-the-badge`、统一底色 `0d1117`，logoColor 保留品牌色：Java `ED8B00`、Spring Boot `6DB33F`、MySQL `4479A1`、Redis `DC382D`、Python `3776AB`、LangChain `white`、Vue.js `4FC08D`

### 修改：`.github/workflows/metrics.yml`

- 两个 job 均加 `config_css` 注入深色样式（底 `#0d1117`、浅色文字、青紫强调色），具体色值以 Action 实际产出效果微调
- 产出文件名 `metrics.left.svg` / `metrics.right.svg` 不变，README 引用不变

### 修改：README 中 snake 引用

- 改用 `<picture>`：`prefers-color-scheme: dark` 时用 `github-contribution-grid-snake-dark.svg`（workflow 已在 output 分支生成），默认用浅色版

## 容错与兼容

- 所有 `<img>` 写 alt 文本
- 自制 SVG 自带深色底：浅色模式访客看到深色卡片，不会出现白底样式翻车
- typing / shields 外部服务不可用时仅影响对应局部，hero/footer 为仓库内文件不受影响
- metrics workflow 失败时保留上一次成功产出的 SVG（现状行为，不改）
- 回滚：`git revert`

## 验证

- push 后在 GitHub 查看真实渲染，明暗两种主题模式各检查一次
- metrics 配置改动后手动触发一次 workflow，确认产出正常再合入 README 变更
- 检查 hero 中插画清晰可辨（人物不被渐变压住）、中文字体渲染正常

## 明确不做（YAGNI）

- 不做分区标题自制 SVG / 分隔线素材（markdown 标题即可）
- 不做全自制徽章、项目卡片
- 不改动精选项目表格内容
- 不新增版块
