# GitHub 个人主页重设计（暗夜霓虹）实现计划

> **面向 AI 代理的工作者：** 必需子技能：使用 superpowers:subagent-driven-development（推荐）或 superpowers:executing-plans 逐任务实现此计划。步骤使用复选框（`- [ ]`）语法来跟踪进度。

**目标：** 按规格 `docs/superpowers/specs/2026-08-03-github-profile-redesign-design.md` 将 GitHub 个人主页 README 重设计为暗夜霓虹风格。

**架构：** GitHub README 不允许自定义 CSS，视觉效果全部通过图片素材实现：自制深色底 SVG（hero、footer，插画 base64 内嵌）+ readme-typing-svg 换色 + shields 徽章换色 + metrics workflow 注入深色 CSS。B 版（整幅融入）为默认 hero，A 版（人物立绘）作为备用素材入库。

**技术栈：** SVG（手写）、shields.io、readme-typing-svg、lowlighter/metrics（GitHub Actions）、sips（macOS 图片处理）、Swift Vision（抠图，仅备用素材需要）

**规格文件：** `docs/superpowers/specs/2026-08-03-github-profile-redesign-design.md`

---

## 文件结构

| 文件 | 职责 |
| --- | --- |
| `assets/header.svg` | 新建。Hero 横幅 B 版（整幅融入），README 引用 |
| `assets/header-standee.svg` | 新建。Hero 备用 A 版（人物立绘），README 不引用 |
| `assets/footer.svg` | 新建。页尾收尾带，README 引用 |
| `README.md` | 重写。新页面结构 |
| `.github/workflows/metrics.yml` | 修改。两个 job 注入深色 `config_css` |
| `.superpowers/build/` | 临时目录（已 gitignore）。压缩图、SVG 分片等中间产物，不入库 |

临时素材来源（已存在，可直接使用；若缺失则按任务 1 重新生成）：
- `.superpowers/brainstorm/50586-1785770870/chara.png` — 已抠好的人物立绘（902×1059）

---

### 任务 1：准备图片素材（临时产物）

**文件：**
- 创建：`.superpowers/build/illust-800.jpg`（由 `image/138888613_p0.png` 压缩）
- 创建：`.superpowers/build/chara-700.png`（由已抠图的 `chara.png` 缩放）

- [ ] **步骤 1：生成 800px 压缩插画**

```bash
cd /Users/frewily/GithubProjects/frewily
mkdir -p .superpowers/build
sips -Z 800 -s format jpeg -s formatOptions 78 image/138888613_p0.png --out .superpowers/build/illust-800.jpg
```

- [ ] **步骤 2：验证压缩结果**

运行：`sips -g pixelWidth -g pixelHeight .superpowers/build/illust-800.jpg && ls -la .superpowers/build/illust-800.jpg`
预期：宽 800、高约 417；文件大小约 88KB（不超过 150KB）

- [ ] **步骤 3：生成 700px 立绘 PNG（保留透明通道）**

```bash
sips -Z 700 .superpowers/brainstorm/50586-1785770870/chara.png --out .superpowers/build/chara-700.png
```

若 `.superpowers/brainstorm/50586-1785770870/chara.png` 不存在，先用 Swift Vision 重新抠图——创建 `.superpowers/build/cutout.swift`：

```swift
import Vision
import AppKit

let args = CommandLine.arguments
let inputURL = URL(fileURLWithPath: args[1])
let outputURL = URL(fileURLWithPath: args[2])
guard let nsImage = NSImage(contentsOf: inputURL),
      let cgImage = nsImage.cgImage(forProposedRect: nil, context: nil, hints: nil) else { exit(1) }
let request = VNGenerateForegroundInstanceMaskRequest()
let handler = VNImageRequestHandler(cgImage: cgImage, options: [:])
try handler.perform([request])
guard let result = request.results?.first else { exit(1) }
let buffer = try result.generateMaskedImage(ofInstances: result.allInstances, from: handler, croppedToInstancesExtent: true)
let ciImage = CIImage(cvPixelBuffer: buffer)
guard let masked = CIContext().createCGImage(ciImage, from: ciImage.extent) else { exit(1) }
let rep = NSBitmapImageRep(cgImage: masked)
try rep.representation(using: .png, properties: [:])!.write(to: outputURL)
```

然后运行：

```bash
swift .superpowers/build/cutout.swift image/138888613_p0.png .superpowers/build/chara.png
sips -Z 700 .superpowers/build/chara.png --out .superpowers/build/chara-700.png
```

- [ ] **步骤 4：验证立绘**

运行：`sips -g pixelWidth -g pixelHeight .superpowers/build/chara-700.png && ls -la .superpowers/build/chara-700.png`
预期：高约 700、宽约 596；文件不超过 700KB；`sips -g hasAlpha` 返回 yes

（本任务产物为 gitignore 的临时文件，不 commit）

---

### 任务 2：页尾 `assets/footer.svg`

**文件：**
- 创建：`assets/footer.svg`

- [ ] **步骤 1：编写 footer.svg**

```xml
<svg viewBox="0 0 1000 140" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Thanks for visiting">
  <defs>
    <linearGradient id="bgGradF" x1="0" y1="0" x2="1" y2="1">
      <stop offset="0%" stop-color="#1a1030"/>
      <stop offset="45%" stop-color="#131a2c"/>
      <stop offset="100%" stop-color="#0d1117"/>
    </linearGradient>
    <linearGradient id="neonGradF" x1="0" y1="0" x2="1" y2="0">
      <stop offset="0%" stop-color="#c792ff"/>
      <stop offset="50%" stop-color="#7ee0ff"/>
      <stop offset="100%" stop-color="#c792ff"/>
    </linearGradient>
    <filter id="softGlowF" x="-60%" y="-60%" width="220%" height="220%">
      <feGaussianBlur stdDeviation="4" result="b"/>
      <feMerge><feMergeNode in="b"/><feMergeNode in="SourceGraphic"/></feMerge>
    </filter>
  </defs>
  <rect x="1" y="1" width="998" height="138" rx="16" fill="url(#bgGradF)" stroke="#7ee0ff" stroke-opacity="0.18"/>
  <circle cx="120" cy="110" r="1.5" fill="#7ee0ff" opacity="0.6"/>
  <circle cx="880" cy="36" r="1.5" fill="#c792ff" opacity="0.6"/>
  <circle cx="760" cy="104" r="1.2" fill="#ffffff" opacity="0.5"/>
  <rect x="180" y="69" width="640" height="2" rx="1" fill="url(#neonGradF)" filter="url(#softGlowF)"/>
  <text x="500" y="52" text-anchor="middle" font-family="ui-monospace, 'SF Mono', Menlo, Consolas, monospace" font-size="14" letter-spacing="3" fill="#7ee0ff" opacity="0.9">// thanks for visiting</text>
</svg>
```

- [ ] **步骤 2：验证 XML 合法**

运行：`xmllint --noout assets/footer.svg && echo OK`
预期：输出 `OK`，无报错

- [ ] **步骤 3：Commit**

```bash
git add assets/footer.svg
git commit -m "Add dark neon footer SVG"
```

---

### 任务 3：Hero `assets/header.svg`（B 版整幅融入，默认）

SVG 需要 base64 内嵌插画。做法：把 SVG 拆成头尾两个分片文件，中间用命令拼入 base64 串，避免手工粘贴大段编码。

**文件：**
- 创建：`.superpowers/build/header-head.part`
- 创建：`.superpowers/build/header-tail.part`
- 创建：`assets/header.svg`（拼接产物）

- [ ] **步骤 1：编写头部分片 `.superpowers/build/header-head.part`**

注意：文件最后一行以 `data:image/jpeg;base64,` 结尾，**不要换行之后再加内容**（拼接时 base64 串紧跟其后）。

```xml
<svg viewBox="0 0 1000 300" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Frewily - Java backend and AI application developer">
  <defs>
    <linearGradient id="bgGrad" x1="0" y1="0" x2="1" y2="1">
      <stop offset="0%" stop-color="#0d1117"/>
      <stop offset="55%" stop-color="#131a2c"/>
      <stop offset="100%" stop-color="#1a1030"/>
    </linearGradient>
    <linearGradient id="neonGrad" x1="0" y1="0" x2="1" y2="0">
      <stop offset="0%" stop-color="#7ee0ff"/>
      <stop offset="100%" stop-color="#c792ff"/>
    </linearGradient>
    <linearGradient id="fade" x1="0" y1="0" x2="1" y2="0">
      <stop offset="0%" stop-color="#0d1117" stop-opacity="1"/>
      <stop offset="45%" stop-color="#0d1117" stop-opacity="0.85"/>
      <stop offset="80%" stop-color="#0d1117" stop-opacity="0.15"/>
      <stop offset="100%" stop-color="#0d1117" stop-opacity="0"/>
    </linearGradient>
    <pattern id="grid" width="40" height="40" patternUnits="userSpaceOnUse">
      <path d="M40 0H0V40" fill="none" stroke="#7ee0ff" stroke-opacity="0.05" stroke-width="1"/>
    </pattern>
    <filter id="softGlow" x="-60%" y="-60%" width="220%" height="220%">
      <feGaussianBlur stdDeviation="5" result="b"/>
      <feMerge><feMergeNode in="b"/><feMergeNode in="SourceGraphic"/></feMerge>
    </filter>
    <clipPath id="heroClip"><rect x="1" y="1" width="998" height="298" rx="16"/></clipPath>
  </defs>
  <g clip-path="url(#heroClip)">
    <rect x="1" y="1" width="998" height="298" fill="url(#bgGrad)"/>
    <image href="data:image/jpeg;base64,
```

- [ ] **步骤 2：编写尾部分片 `.superpowers/build/header-tail.part`**

第一行以 `"` 开头（闭合 href 属性）：

```xml
" x="320" y="0" width="680" height="300" preserveAspectRatio="xMidYMid slice"/>
    <rect x="200" y="0" width="420" height="300" fill="url(#fade)"/>
    <rect x="320" y="0" width="680" height="300" fill="#0d1117" opacity="0.08"/>
    <rect x="1" y="1" width="998" height="298" fill="url(#grid)"/>
  </g>
  <rect x="1" y="1" width="998" height="298" rx="16" fill="none" stroke="#7ee0ff" stroke-opacity="0.18"/>
  <text x="64" y="92" font-family="ui-monospace, 'SF Mono', Menlo, Consolas, monospace" font-size="15" letter-spacing="2" fill="#7ee0ff" opacity="0.9">// welcome to my profile</text>
  <text x="62" y="152" font-family="-apple-system, 'Segoe UI', 'PingFang SC', 'Microsoft YaHei', sans-serif" font-size="56" font-weight="800" fill="url(#neonGrad)" filter="url(#softGlow)">Frewily</text>
  <text x="64" y="192" font-family="-apple-system, 'Segoe UI', 'PingFang SC', 'Microsoft YaHei', sans-serif" font-size="20" font-weight="600" fill="#e6edf3">Java 后端 · AI 应用开发者</text>
  <text x="64" y="222" font-family="-apple-system, 'Segoe UI', 'PingFang SC', 'Microsoft YaHei', sans-serif" font-size="14" fill="#a8b3c0">构建能解决真实问题的 AI 应用 — RAG / Agent / LLM 工程化</text>
  <g font-family="ui-monospace, 'SF Mono', Menlo, Consolas, monospace" font-size="13">
    <rect x="64" y="242" width="118" height="30" rx="15" fill="#7ee0ff" fill-opacity="0.08" stroke="#7ee0ff" stroke-opacity="0.6"/>
    <text x="123" y="262" text-anchor="middle" fill="#7ee0ff">frewily.top</text>
    <rect x="194" y="242" width="150" height="30" rx="15" fill="#c792ff" fill-opacity="0.08" stroke="#c792ff" stroke-opacity="0.6"/>
    <text x="269" y="262" text-anchor="middle" fill="#c792ff">github.com/frewily</text>
  </g>
</svg>
```

- [ ] **步骤 3：拼接生成 header.svg**

```bash
cd /Users/frewily/GithubProjects/frewily
{ printf '%s' "$(cat .superpowers/build/header-head.part)"; base64 -i .superpowers/build/illust-800.jpg | tr -d '\n'; printf '%s' "$(cat .superpowers/build/header-tail.part)"; } > assets/header.svg
```

（`$(cat ...)` 会去掉分片文件末尾的换行，保证 base64 串与 `data:image/jpeg;base64,` 前缀之间、以及与闭合引号之间不混入换行）

- [ ] **步骤 4：验证**

运行：

```bash
xmllint --noout assets/header.svg && echo XML_OK
grep -c "data:image/jpeg;base64" assets/header.svg
ls -la assets/header.svg
```

预期：`XML_OK`；grep 输出 `1`；文件大小约 120KB（不超过 250KB）

- [ ] **步骤 5：浏览器目检**

运行：`open "file:///Users/frewily/GithubProjects/frewily/assets/header.svg"`
预期：右侧清晰可见亚托莉海边插画（人物不被渐变压住），左侧文字与两个 pill 正常，整体深空底色

- [ ] **步骤 6：Commit**

```bash
git add assets/header.svg
git commit -m "Add full-bleed hero header SVG with embedded illustration"
```

---

### 任务 4：备用 Hero `assets/header-standee.svg`（A 版立绘，README 不引用）

立绘底部残留原图光斑和 "My Dear Moments" 文字，通过 SVG clipPath 裁掉显示区域底部 8% 解决，不改图片本身。chara-700.png 约 596×700，以高 252 显示时宽约 215，clip 高度 232（=252×0.92）。

**文件：**
- 创建：`.superpowers/build/standee-head.part`
- 创建：`.superpowers/build/standee-tail.part`
- 创建：`assets/header-standee.svg`

- [ ] **步骤 1：编写头部分片 `.superpowers/build/standee-head.part`**

最后一行以 `data:image/png;base64,` 结尾：

```xml
<svg viewBox="0 0 1000 300" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Frewily - Java backend and AI application developer (standee variant)">
  <defs>
    <linearGradient id="bgGradS" x1="0" y1="0" x2="1" y2="1">
      <stop offset="0%" stop-color="#0d1117"/>
      <stop offset="55%" stop-color="#131a2c"/>
      <stop offset="100%" stop-color="#1a1030"/>
    </linearGradient>
    <linearGradient id="neonGradS" x1="0" y1="0" x2="1" y2="0">
      <stop offset="0%" stop-color="#7ee0ff"/>
      <stop offset="100%" stop-color="#c792ff"/>
    </linearGradient>
    <radialGradient id="glowCyanS" cx="50%" cy="50%" r="50%">
      <stop offset="0%" stop-color="#7ee0ff" stop-opacity="0.28"/>
      <stop offset="100%" stop-color="#7ee0ff" stop-opacity="0"/>
    </radialGradient>
    <radialGradient id="glowPurpleS" cx="50%" cy="50%" r="50%">
      <stop offset="0%" stop-color="#c792ff" stop-opacity="0.3"/>
      <stop offset="100%" stop-color="#c792ff" stop-opacity="0"/>
    </radialGradient>
    <pattern id="gridS" width="40" height="40" patternUnits="userSpaceOnUse">
      <path d="M40 0H0V40" fill="none" stroke="#7ee0ff" stroke-opacity="0.05" stroke-width="1"/>
    </pattern>
    <filter id="softGlowS" x="-60%" y="-60%" width="220%" height="220%">
      <feGaussianBlur stdDeviation="5" result="b"/>
      <feMerge><feMergeNode in="b"/><feMergeNode in="SourceGraphic"/></feMerge>
    </filter>
    <clipPath id="charaClip"><rect x="700" y="34" width="215" height="232"/></clipPath>
  </defs>
  <rect x="1" y="1" width="998" height="298" rx="16" fill="url(#bgGradS)" stroke="#7ee0ff" stroke-opacity="0.18"/>
  <rect x="1" y="1" width="998" height="298" rx="16" fill="url(#gridS)"/>
  <circle cx="170" cy="70" r="180" fill="url(#glowCyanS)"/>
  <text x="64" y="92" font-family="ui-monospace, 'SF Mono', Menlo, Consolas, monospace" font-size="15" letter-spacing="2" fill="#7ee0ff" opacity="0.9">// welcome to my profile</text>
  <text x="62" y="152" font-family="-apple-system, 'Segoe UI', 'PingFang SC', 'Microsoft YaHei', sans-serif" font-size="56" font-weight="800" fill="url(#neonGradS)" filter="url(#softGlowS)">Frewily</text>
  <text x="64" y="192" font-family="-apple-system, 'Segoe UI', 'PingFang SC', 'Microsoft YaHei', sans-serif" font-size="20" font-weight="600" fill="#e6edf3">Java 后端 · AI 应用开发者</text>
  <text x="64" y="222" font-family="-apple-system, 'Segoe UI', 'PingFang SC', 'Microsoft YaHei', sans-serif" font-size="14" fill="#8b949e">构建能解决真实问题的 AI 应用 — RAG / Agent / LLM 工程化</text>
  <g font-family="ui-monospace, 'SF Mono', Menlo, Consolas, monospace" font-size="13">
    <rect x="64" y="242" width="118" height="30" rx="15" fill="#7ee0ff" fill-opacity="0.08" stroke="#7ee0ff" stroke-opacity="0.6"/>
    <text x="123" y="262" text-anchor="middle" fill="#7ee0ff">frewily.top</text>
    <rect x="194" y="242" width="150" height="30" rx="15" fill="#c792ff" fill-opacity="0.08" stroke="#c792ff" stroke-opacity="0.6"/>
    <text x="269" y="262" text-anchor="middle" fill="#c792ff">github.com/frewily</text>
  </g>
  <circle cx="810" cy="160" r="150" fill="url(#glowPurpleS)"/>
  <circle cx="810" cy="160" r="108" fill="none" stroke="url(#neonGradS)" stroke-width="2.5" filter="url(#softGlowS)" opacity="0.9"/>
  <circle cx="810" cy="160" r="126" fill="none" stroke="#7ee0ff" stroke-opacity="0.25" stroke-width="1" stroke-dasharray="4 8"/>
  <image href="data:image/png;base64,
```

- [ ] **步骤 2：编写尾部分片 `.superpowers/build/standee-tail.part`**

```xml
" x="700" y="34" width="215" height="252" preserveAspectRatio="xMidYMid meet" clip-path="url(#charaClip)"/>
  <circle cx="904" cy="70" r="4" fill="#7ee0ff" filter="url(#softGlowS)"/>
  <circle cx="712" cy="248" r="3" fill="#c792ff" filter="url(#softGlowS)"/>
</svg>
```

- [ ] **步骤 3：拼接生成 header-standee.svg**

```bash
cd /Users/frewily/GithubProjects/frewily
{ printf '%s' "$(cat .superpowers/build/standee-head.part)"; base64 -i .superpowers/build/chara-700.png | tr -d '\n'; printf '%s' "$(cat .superpowers/build/standee-tail.part)"; } > assets/header-standee.svg
```

- [ ] **步骤 4：验证**

运行：

```bash
xmllint --noout assets/header-standee.svg && echo XML_OK
grep -c "data:image/png;base64" assets/header-standee.svg
open "file:///Users/frewily/GithubProjects/frewily/assets/header-standee.svg"
```

预期：`XML_OK`；grep 输出 `1`；浏览器中人物立绘完整（含头顶发丝），底部无残留光斑与文字，双层光环正常

- [ ] **步骤 5：Commit**

```bash
git add assets/header-standee.svg
git commit -m "Add standee hero variant as backup asset"
```

---

### 任务 5：metrics workflow 深色化

**文件：**
- 修改：`.github/workflows/metrics.yml`（两个 job 的 `with:` 块各加一行 `config_css`）

- [ ] **步骤 1：给 `github-metrics-left` job 添加深色 CSS**

在 `plugin_people_types: followers` 之后追加：

```yaml
          config_css: |
            svg { background: #0d1117; border-radius: 12px; }
            text, tspan { fill: #c9d1d9; }
            .field, .field svg { color: #7ee0ff; fill: #7ee0ff; }
            a { color: #7ee0ff; }
```

- [ ] **步骤 2：给 `github-metrics-right` job 添加同样的深色 CSS**

在 `plugin_steam_recent_limit: 4` 之后追加与上一步完全相同的 `config_css` 块。

- [ ] **步骤 3：验证 YAML 结构**

运行：`grep -n "config_css" .github/workflows/metrics.yml`
预期：输出 2 行，分别在两个 job 内

- [ ] **步骤 4：Commit**

```bash
git add .github/workflows/metrics.yml
git commit -m "Dark theme for metrics SVGs via config_css"
```

说明：metrics 生成的 SVG 内部结构依插件而异，`config_css` 选择器可能需要按实际产出微调。线上验证在任务 7 进行；若产出仍是浅色背景，在 `config_css` 开头追加一条 `svg > rect:first-child { fill: #0d1117; }` 再重新触发 workflow。

---

### 任务 6：重写 `README.md`

**文件：**
- 修改：`README.md`（整体重写）

- [ ] **步骤 1：写入新 README.md（完整替换）**

```markdown
<div align="center">

<img src="assets/header.svg" width="100%" alt="Frewily — Java 后端 · AI 应用开发者" />

<img src="https://readme-typing-svg.herokuapp.com?font=Noto+Sans+SC&weight=600&size=26&duration=3000&pause=1000&color=7EE0FF&background=00000000&center=true&vCenter=true&width=760&height=60&lines=Java+%E5%90%8E%E7%AB%AF+%7C+AI+%E5%BA%94%E7%94%A8%E5%BC%80%E5%8F%91%E8%80%85;%E6%9E%84%E5%BB%BA%E8%83%BD%E8%A7%A3%E5%86%B3%E7%9C%9F%E5%AE%9E%E9%97%AE%E9%A2%98%E7%9A%84+AI+%E5%BA%94%E7%94%A8" alt="Typing SVG" />

<a href="https://frewily.top"><img src="https://img.shields.io/badge/%E4%B8%AA%E4%BA%BA%E7%BD%91%E7%AB%99-frewily.top-0d1117?style=flat-square&logo=google-chrome&logoColor=7EE0FF" alt="个人网站" /></a>
<a href="https://github.com/frewily"><img src="https://img.shields.io/badge/GitHub-frewily-0d1117?style=flat-square&logo=github&logoColor=C792FF" alt="GitHub" /></a>

</div>

## 👋 关于我

> 在 AI 时代持续学习，把想法做成可运行的产品。

- 🔭 专注于 **Java 后端** 与 **AI 应用开发**（RAG / Agent / LLM 工程化）
- 🌱 正在深入算法、Flutter 与开源协作；热爱二次元文化

## 🧰 技术栈

<div align="center">

![Java](https://img.shields.io/badge/Java-0d1117?style=for-the-badge&logo=openjdk&logoColor=ED8B00)
![Spring Boot](https://img.shields.io/badge/Spring_Boot-0d1117?style=for-the-badge&logo=springboot&logoColor=6DB33F)
![MySQL](https://img.shields.io/badge/MySQL-0d1117?style=for-the-badge&logo=mysql&logoColor=4479A1)
![Redis](https://img.shields.io/badge/Redis-0d1117?style=for-the-badge&logo=redis&logoColor=DC382D)
![Python](https://img.shields.io/badge/Python-0d1117?style=for-the-badge&logo=python&logoColor=3776AB)
![LangChain](https://img.shields.io/badge/LangChain-0d1117?style=for-the-badge&logo=langchain&logoColor=white)
![Vue.js](https://img.shields.io/badge/Vue.js-0d1117?style=for-the-badge&logo=vuedotjs&logoColor=4FC08D)

</div>

## 🚀 精选项目

| 项目 | 简介 | 技术亮点 |
| --- | --- | --- |
| [AI 知识工单平台](https://github.com/frewily/langchain_springboot_vue3_exemple_project) | 面向企业知识管理与智能客服的全栈平台。 | LangChain4j、RAG、Agent 工具调用、SSE 流式输出、Spring Boot、Vue 3 |
| [智能扫地机器人客服](https://github.com/frewily/Agent_exercise_project) | 可检索知识库、查询外部数据并生成报告的 ReAct 智能客服。 | LangChain、LangGraph、Chroma、Streamlit、动态 Prompt 中间件 |
| [RepoGuide Skill](https://github.com/frewily/RepoGuide-Skill) | 以引导式对话帮助开发者系统学习开源项目，并沉淀为 Obsidian 笔记。 | Agent Skill、项目架构分析、结构化 Markdown 知识沉淀 |

<div align="center">
  <a href="https://github.com/frewily?tab=repositories">查看全部项目 →</a>
</div>

## 📊 开发活动

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/frewily/frewily/refs/heads/output/github-contribution-grid-snake-dark.svg" />
  <img src="https://raw.githubusercontent.com/frewily/frewily/refs/heads/output/github-contribution-grid-snake.svg" width="100%" alt="GitHub Contribution Grid Snake Animation" />
</picture>

<div align="center">
  <img src="https://raw.githubusercontent.com/frewily/frewily/main/metrics.left.svg" width="49%" alt="GitHub metrics" />
  <img src="https://raw.githubusercontent.com/frewily/frewily/main/metrics.right.svg" width="49%" alt="Interests metrics" />
</div>

<img src="assets/footer.svg" width="100%" alt="Thanks for visiting" />
```

变更点对照：typing `color` 由 `FF6B9D` 改为 `7EE0FF`；删除 `image/138888613_p0.png` 的 `<img>`（插画已进 hero）；删除 komarev Profile Views 徽章与"感谢你的来访"；snake 改 `<picture>` 明暗切换；项目表格内容未动。

- [ ] **步骤 2：验证旧元素已移除、新元素已就位**

运行：

```bash
grep -c "komarev" README.md; grep -c "感谢你的来访" README.md; grep -c "138888613" README.md
grep -c "assets/header.svg" README.md; grep -c "assets/footer.svg" README.md; grep -c "snake-dark" README.md
```

预期：前三条均输出 `0`；后三条均输出 `1`

- [ ] **步骤 3：Commit**

```bash
git add README.md
git commit -m "Restyle profile README with dark neon theme"
```

---

### 任务 7：push 与线上验证

- [ ] **步骤 1：向用户确认后 push**

```bash
git push origin main
```

（push 是对外可见操作，执行前必须得到用户明确同意）

- [ ] **步骤 2：确认 metrics workflow 运行成功**

push 会自动触发 Metrics workflow。运行：

```bash
gh run list --workflow=metrics.yml --limit 2
```

预期：最新一条 run 的 conclusion 为 `success`。若无 `gh` 命令，让用户在 GitHub Actions 页面确认。

- [ ] **步骤 3：验证 metrics SVG 已深色化**

运行：

```bash
curl -s https://raw.githubusercontent.com/frewily/frewily/main/metrics.left.svg | grep -c "0d1117"
```

预期：输出 ≥ `1`。若为 `0`（仍是浅色背景）：按任务 5 说明在 `config_css` 开头追加 `svg > rect:first-child { fill: #0d1117; }`，commit + push，再用 `gh workflow run metrics.yml` 手动触发并重验。

- [ ] **步骤 4：请用户目检线上效果**

让用户打开 `https://github.com/frewily`，分别在系统明暗两种模式下检查：
- hero 插画清晰（亚托莉可辨）、中文渲染正常
- 打字动画为青色
- 徽章为深色底
- snake 在深色模式下为 dark 版
- metrics 两张图为深色
- 页尾收尾带正常，无 Profile Views 徽章

如有问题，回滚方式：`git revert` 对应 commit 后 push。
