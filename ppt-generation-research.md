# PPT/PDF 生成 Agent 技术调研

## 目标

构建一个能从文本、图片、Excel 生成 **精美排版** 商业报告 PPT/PDF 的 Agent。

---

## 一、现有 SaaS 产品对比

| 产品 | 特点 | API | 审美评分 |
|------|------|-----|---------|
| **[Gamma.app](https://gamma.app)** | AI生成精美PPT，设计感强，商业风格 | ✅ 有 | ⭐⭐⭐⭐⭐ |
| **[Tome](https://tome.app)** | 叙事性强，动态展示，现代感 | ✅ 有 | ⭐⭐⭐⭐⭐ |
| **[Beautiful.ai](https://beautiful.ai)** | 智能模板，自动排版对齐 | ✅ 有 | ⭐⭐⭐⭐ |
| **[Canva](https://canva.com)** | 海量模板，AI Magic Design | ✅ 有 | ⭐⭐⭐⭐ |
| **[Pitch](https://pitch.com)** | 协作PPT，设计精美 | ✅ 有 | ⭐⭐⭐⭐ |
| **Microsoft Copilot PPT** | Office集成，企业场景 | ✅ 有 | ⭐⭐⭐ |

---

## 二、开源/自建技术方案

### 方案 1: Markdown → Slides（推荐探索）

```
内容 → LLM → Markdown → Slidev/Marp → PDF/PPTX
```

| 工具 | 说明 | 优点 | 缺点 |
|------|------|------|------|
| **[Slidev](https://sli.dev)** | Vue驱动的Markdown演示工具 | 动画丰富、代码高亮、主题美观 | 需要Node环境 |
| **[Marp](https://marp.app)** | Markdown演示生态系统 | 极简、导出PPTX/PDF、VSCode插件 | 样式控制有限 |
| **[reveal.js](https://revealjs.com)** | HTML演示框架 | 功能强大、插件多 | 学习曲线 |

### 方案 2: HTML/CSS → PDF（高度可控）

```
内容 → LLM → HTML + Tailwind CSS → Puppeteer/Playwright → PDF
```

- **优点**: 完全控制每个像素，前端技术栈
- **缺点**: 分页控制需要 CSS Paged Media
- **工具**: WeasyPrint, Puppeteer, Playwright

### 方案 3: Typst（现代排版）

```
内容 → LLM → Typst 代码 → typst compile → PDF
```

- **优点**: 语法简洁、编译快、排版质量高
- **缺点**: 生态较新、商业模板少
- **官网**: [typst.app](https://typst.app)

### 方案 4: python-pptx + 模板（当前方案改进）

```
内容 → LLM → JSON结构 → python-pptx 填充预制模板 → PPTX
```

- **关键**: 需要设计师预制精美模板
- **优点**: 直接生成PPTX、企业兼容性好
- **缺点**: 代码量大、模板维护成本

---

## 三、核心问题：审美从哪来？

LLM 本身不懂视觉设计，审美需要外部注入：

### 1. 预制模板库

设计师制作多套模板，LLM 根据内容选择合适模板填充：

```
模板类型:
├── 封面页 (Cover)
├── 目录页 (TOC)
├── 标题+正文 (Title + Body)
├── 图文对照 (Image + Text)
├── 数据图表 (Charts)
├── 对比页 (Comparison)
├── 时间线 (Timeline)
├── 团队介绍 (Team)
├── 引用页 (Quote)
└── 结束页 (End)
```

### 2. Design Token 系统

定义设计变量，约束 LLM 输出：

```json
{
  "colors": {
    "primary": "#1a365d",
    "secondary": "#2b6cb0",
    "accent": "#ed8936",
    "text": "#2d3748",
    "background": "#ffffff"
  },
  "typography": {
    "title": { "size": "44px", "weight": "700", "font": "Inter" },
    "subtitle": { "size": "24px", "weight": "600", "font": "Inter" },
    "body": { "size": "18px", "weight": "400", "font": "Inter" }
  },
  "spacing": {
    "page-margin": "60px",
    "element-gap": "24px",
    "section-gap": "48px"
  }
}
```

### 3. 设计规则系统

编码设计原则，作为 LLM 的约束：

- **对齐**: 所有元素基于 8px 网格对齐
- **层级**: 最多 3 级标题层级
- **留白**: 页面边距不小于 10%
- **配色**: 单页不超过 3 种颜色
- **图文比**: 图片占比 40-60%

### 4. 参考设计系统

| 设计系统 | 来源 | 特点 |
|---------|------|------|
| Material Design 3 | Google | 完整、现代、动效丰富 |
| Carbon Design | IBM | 企业级、数据可视化强 |
| Polaris | Shopify | 商业友好、清晰 |
| Primer | GitHub | 开发者风格、简洁 |

---

## 四、相关研究论文

| 论文 | 方向 |
|------|------|
| **LayoutGPT** | LLM 理解空间布局，生成场景/文档/UI 布局 |
| **LayoutTransformer** | Transformer 生成文档布局 |
| **CanvasLLM** | LLM 驱动的画布设计生成 |
| **DocLayNet** | 文档布局数据集 |

---

## 五、推荐架构

```
┌─────────────────────────────────────────────────────────────────┐
│                   PPT Generation Agent                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐          │
│  │ 文本解析     │    │ Excel解析   │    │ 图片分析     │          │
│  │ (LLM提取)   │    │ (pandas)    │    │ (多模态LLM)  │          │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘          │
│         │                  │                  │                  │
│         └──────────────────┼──────────────────┘                  │
│                            ▼                                     │
│                   ┌─────────────────┐                            │
│                   │  结构化内容      │                            │
│                   │  (JSON Schema)  │                            │
│                   └────────┬────────┘                            │
│                            ▼                                     │
│                   ┌─────────────────┐                            │
│                   │  大纲生成 LLM    │                            │
│                   │  (10-15页规划)  │                            │
│                   └────────┬────────┘                            │
│                            ▼                                     │
│                   ┌─────────────────┐                            │
│                   │  页面设计 LLM    │                            │
│                   │  + Design Rules │                            │
│                   └────────┬────────┘                            │
│                            ▼                                     │
│         ┌──────────────────┼──────────────────┐                  │
│         ▼                  ▼                  ▼                  │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐          │
│  │ Slidev      │    │ python-pptx │    │ Typst       │          │
│  │ (Markdown)  │    │ (Template)  │    │ (Modern)    │          │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘          │
│         │                  │                  │                  │
│         └──────────────────┼──────────────────┘                  │
│                            ▼                                     │
│                   ┌─────────────────┐                            │
│                   │  PDF / PPTX     │                            │
│                   └─────────────────┘                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 六、实现路径建议

### Phase 1: 快速验证 (MVP)

1. 使用 **Slidev + 预制主题** 验证流程
2. LLM 生成 Markdown 内容
3. 手动调整模板样式
4. 验证商业报告场景

### Phase 2: 模板系统

1. 设计 **10 种页面模板**
2. 建立 **Design Token 系统**
3. LLM 学习模板选择规则
4. 实现自动模板匹配

### Phase 3: 智能布局

1. 研究 **LayoutGPT** 等论文
2. 实现布局约束系统
3. 多模态理解图片内容
4. 智能图文排版

### Phase 4: 品牌定制

1. 品牌色/字体配置
2. 自定义模板上传
3. 风格迁移能力

---

## 七、技术选型建议

| 场景 | 推荐方案 |
|------|---------|
| 快速验证 | Slidev + LLM |
| 企业交付 | python-pptx + 模板库 |
| 高质量排版 | Typst + LLM |
| Web展示 | HTML + Tailwind → PDF |

---

## 八、可复用的本地 Skills

在 `~/.claude/skills/` 目录下发现以下可复用的设计相关 skill：

### 1. `frontend-design` ⭐⭐⭐⭐⭐ (强烈推荐)

**核心理念**: 创建独特、生产级的前端界面，避免"AI 审美"

**设计原则**:
- **Typography**: 选择独特字体，避免 Arial/Inter，使用有性格的展示字体
- **Color**: 大胆配色，主色+锐利点缀，避免平淡分布
- **Motion**: CSS动画、staggered reveals、scroll-triggering
- **Spatial**: 非对称布局、重叠、对角线、打破网格、留白
- **Backgrounds**: 渐变网格、噪点纹理、几何图案、透明层叠

**关键洞察**:
```
❌ 禁止: Inter/Roboto/Arial, 紫色渐变+白底, 预设布局, 千篇一律
✅ 推荐: 极简主义 OR 极繁主义, 明确风格方向, 大胆执行
```

### 2. `ui-designer`

**功能**: Design Token 生成器

```bash
python scripts/design_token_generator.py [brand_color] [style] [format]
# style: modern, classic, playful
# format: json, css, scss
```

**输出**:
- 完整色板生成
- 模块化字体比例
- 8pt 间距网格
- 阴影/动画 token
- 响应式断点

### 3. `app-ui-design`

**功能**: 移动端 UI 设计规范

**可借鉴内容**:
- 8-Point Grid 系统
- iOS/Android Typography Scale
- 60-30-10 配色法则
- WCAG 2.2 无障碍标准
- Dark Mode 最佳实践
- 微交互设计原则

### 4. `artifacts-builder`

**功能**: React + Tailwind + shadcn/ui 单文件打包

**技术栈**: React 18 + TypeScript + Vite + Parcel + Tailwind CSS + shadcn/ui

**潜在用途**: 可用于生成可交互的 HTML 版 PPT 预览

---

## 九、推荐方案：HTML → PDF (结合 Skills)

基于现有 skills，推荐以下架构：

```
┌─────────────────────────────────────────────────────────────────┐
│                   PPT Generation Agent v2                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    Design System Layer                    │    │
│  │  (基于 ui-designer skill 的 Design Token)                 │    │
│  │  colors + typography + spacing + shadows                  │    │
│  └─────────────────────────────────────────────────────────┘    │
│                              ↓                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    Content Layer                          │    │
│  │  文本 → LLM 提取结构化内容                                  │    │
│  │  Excel → pandas 解析为图表数据                             │    │
│  │  图片 → 多模态 LLM 生成描述                                │    │
│  └─────────────────────────────────────────────────────────┘    │
│                              ↓                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    Layout Layer                           │    │
│  │  (基于 frontend-design skill 的设计原则)                   │    │
│  │  页面模板选择 + 布局生成 + 样式应用                          │    │
│  └─────────────────────────────────────────────────────────┘    │
│                              ↓                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    Render Layer                           │    │
│  │  React + Tailwind → HTML → Puppeteer → PDF               │    │
│  │  (可选: artifacts-builder 打包为单 HTML)                   │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 核心 Prompt 设计

借鉴 `frontend-design` skill 的风格指导：

```markdown
## 风格方向 (选择一个极端)
- Brutalist (粗野主义): 原始、工业、无装饰
- Editorial (杂志风): 大留白、精致字体、网格
- Luxury (奢华): 深色、金色点缀、衬线字体
- Corporate Modern (现代商务): 清晰、专业、数据驱动
- Minimalist (极简): 减法设计、呼吸空间

## 避免 (AI 审美陷阱)
- 紫色渐变 + 白色背景
- Inter/Roboto 字体
- 居中对齐一切
- 统一圆角
- 无层次的灰色
```

---

## 十、开源 AI PPT 生成项目 (网络调研结果)

### ⭐ Presenton (3.6k stars) - 强烈推荐

**GitHub**: [presenton/presenton](https://github.com/presenton/presenton)

**定位**: Gamma / Beautiful AI / Decktopus 的开源替代方案

**核心功能**:
- 通过 Prompt 或上传文档生成 PPT
- **上传现有 PPTX 作为模板，生成品牌一致的 PPT** ⭐
- 导出 PPTX / PDF
- 使用 HTML + Tailwind CSS 创建自定义模板
- 可调语气 (默认/休闲/专业/幽默/教育/销售)
- 可调详细度 (简洁/标准/详尽)

**技术栈**:
| 组件 | 技术 |
|------|------|
| 前端 | TypeScript (68%) |
| 后端 | Python (30.8%) |
| 部署 | Docker + GPU 加速 |
| 许可 | Apache 2.0 |

**AI 支持**:
- LLM: OpenAI GPT-4.1 / Gemini / Claude / Ollama (本地)
- 图像: DALL-E 3 / Gemini / Pexels / ComfyUI

**API 示例**:
```bash
POST /api/v1/ppt/presentation/generate
{
  "content": "商业地产投资报告",
  "n_slides": 10,
  "language": "Chinese",
  "template": "corporate",
  "export_as": "pptx"
}
```

**快速启动**:
```bash
docker run -it --name presenton -p 5000:80 \
  -v "./app_data:/app_data" \
  ghcr.io/presenton/presenton:latest
```

---

### ALLWEONE Presentation AI (2.3k stars)

**GitHub**: [allweonedev/presentation-ai](https://github.com/allweonedev/presentation-ai)

**特点**:
- 9 个内置主题，计划增加 15+
- 支持 Ollama/LM Studio 本地模型
- Next.js + React + Tailwind
- MIT 许可

**不足**: PPTX/PDF 导出功能不完整

---

### PowerPoint Generator Python (387 stars)

**GitHub**: [otahina/PowerPoint-Generator-Python-Project](https://github.com/otahina/PowerPoint-Generator-Python-Project)

**特点**:
- 纯 Python + Flask
- GPT-3.5 生成内容
- Pexels 图片搜索
- 可定制颜色主题
- MIT 许可

**适合**: 快速理解实现原理

---

### 其他相关项目

| 项目 | Stars | 说明 |
|------|-------|------|
| PptxGenJS | 4.4k | JavaScript PPT 生成库 (纯库，无 AI) |
| python-pptx | - | Python PPT 基础库 |
| Marp | - | Markdown → Slides |
| Slidev | - | Vue + Markdown Slides |

---

## 十一、推荐实现路径

### 方案 A: 直接用 Presenton (最快)

```
优点:
- 功能完整，开箱即用
- 支持自定义模板 (HTML + Tailwind)
- Docker 部署简单
- 有 API 可调用

缺点:
- TypeScript 为主，Python 部分较少
- 需要适配你的 OM 场景
```

### 方案 B: 基于 Presenton 二次开发

```
1. Fork Presenton
2. 添加 OM 专用模板 (封面/财务/租户/...)
3. 集成你的 RAGFlow 数据源
4. 定制 API 满足 CRE 场景
```

### 方案 C: 从零构建 (参考 Presenton 架构)

```
1. 内容层: RAGFlow + LLM 生成结构化内容
2. 模板层: HTML + Tailwind 定义页面模板
3. 渲染层: Puppeteer → PDF 或 python-pptx → PPTX
4. API 层: FastAPI 提供生成接口
```

---

## 十二、开源 PDF 生成项目 (网络调研结果)

### ⭐ PDFME (3.9k stars) - 强烈推荐

**GitHub**: [pdfme/pdfme](https://github.com/pdfme/pdfme)

**核心功能**:
- **可视化模板设计器** (WYSIWYG)
- JSON 模板系统，易于 LLM 生成
- 支持 Node.js 和浏览器
- 基于 pdf-lib + fontkit
- MIT 许可

**技术栈**: TypeScript + React + Ant Design

**使用流程**:
```
1. 可视化设计器创建模板 → JSON
2. LLM 生成填充数据 → JSON
3. pdfme.generate(template, data) → PDF
```

**Playground**: [playground.pdfme.com](https://playground.pdfme.com)

---

### Carbone (1.9k stars)

**GitHub**: [carboneio/carbone](https://github.com/carboneio/carbone)

**核心功能**:
- 使用 LibreOffice/Word/Google Docs 创建模板
- JSON 数据填充 → PDF/DOCX/XLSX/PPTX
- 模板语法: `{d.companyName}`, `{d.items[i].name}`
- 性能: ~50ms/报告 (含 PDF 转换)

**适合场景**: 需要复杂排版的商业文档

**限制**: 需要安装 LibreOffice

---

### 其他 PDF 工具

| 项目 | Stars | 语言 | 说明 |
|------|-------|------|------|
| pandoc-latex-template | 7k | LaTeX | Markdown → PDF (学术风格) |
| html2pdf.js | 4.7k | JS | HTML → PDF (浏览器端) |
| borb | 3.6k | Python | 纯 Python PDF 库 |
| xhtml2pdf | 2.4k | Python | HTML → PDF (ReportLab) |
| WeasyPrint | - | Python | HTML+CSS → PDF (高质量) |

---

## 十三、网络上的 Claude Code Skills (审美/文档相关)

### ⭐ UI UX Pro Max Skill (11.9k stars) - 强烈推荐

**GitHub**: [nextlevelbuilder/ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill)

**核心功能**:
- **57 种 UI 风格** - Glassmorphism、Claymorphism、Minimalism、Brutalism、Neumorphism、Bento Grid 等
- **95 套配色方案** - 针对 SaaS、电商、医疗、金融科技、美妆等行业定制
- **56 组字体搭配** - 带 Google Fonts 导入链接
- **24 种图表类型** - 仪表盘和数据分析
- **98 条 UX 指南** - 最佳实践、反模式、无障碍规则

**安装**:
```bash
npm install -g uipro-cli
cd /path/to/your/project
uipro init --ai claude
```

**支持技术栈**: HTML+Tailwind, React, Next.js, Vue, Nuxt, SwiftUI, Flutter

---

### Awesome Claude Skills (18.1k stars)

**GitHub**: [ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)

**文档处理相关 Skills**:

| Skill | 功能 |
|-------|------|
| **pptx** | 读取、生成、调整幻灯片、布局、模板 |
| **pdf** | 提取文本/表格/元数据、合并、注释 |
| **docx** | 创建编辑 Word 文档、修订追踪、格式化 |
| **xlsx** | 电子表格操作、公式、图表、数据转换 |

**创意/设计相关 Skills**:

| Skill | 功能 |
|-------|------|
| **Theme Factory** | 为 artifacts 应用专业字体和配色主题 |
| **Brand Guidelines** | 应用品牌色彩和排版保持视觉一致 |
| **Canvas Design** | 创建 PNG/PDF 格式的视觉艺术 |
| **Image Enhancer** | 提升图片质量用于专业展示 |
| **D3.js Visualization** | 生成 D3 图表和交互式数据可视化 |

---

### Awesome Claude Code (20k stars)

**GitHub**: [hesreallyhim/awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code)

**设计/UI 相关资源**:

| 资源 | 说明 |
|------|------|
| **Design Review Workflow** | 自动化 UI/UX 设计审查 |
| **Web Assets Generator** | 生成 favicons、PWA 图标、社交媒体元图像 |
| **artifacts-builder** | React + Tailwind + shadcn/ui 组件构建 |

---

### 其他高星 Claude Skills 项目

| 项目 | Stars | 说明 |
|------|-------|------|
| [wshobson/agents](https://github.com/wshobson/agents) | 25.1k | 多代理编排，含 claude-code-plugin |
| [winfunc/opcode](https://github.com/winfunc/opcode) | 19.9k | Claude Code GUI + 自定义代理 |
| [CherryHQ/cherry-studio](https://github.com/CherryHQ/cherry-studio) | 37.6k | AI Agent 桌面应用 |
| [thedotmack/claude-mem](https://github.com/thedotmack/claude-mem) | 13.4k | 自动捕获上下文插件 |

---

## 十四、本地可用的审美相关 Skills

### 汇总表

| Skill 名称 | 主要用途 | 审美相关性 |
|-----------|---------|-----------|
| `frontend-design` | 避免 AI 审美陷阱 | ⭐⭐⭐⭐⭐ |
| `ui-designer` | Design Token 生成 | ⭐⭐⭐⭐ |
| `app-ui-design` | 移动端 UI 规范 | ⭐⭐⭐⭐ |
| `product-ux-expert` | UX 评估 + 认知心理学 | ⭐⭐⭐ |
| `sage-ui-design` | CLI 终端 UI 规范 | ⭐⭐ |
| `artifacts-builder` | React+Tailwind 打包 | ⭐⭐ |

### frontend-design (核心审美 Skill)

**避免 AI 审美陷阱**:
```
❌ 禁止:
- Inter/Roboto/Arial 字体
- 紫色渐变 + 白色背景
- 居中对齐一切
- 统一圆角
- 无层次的灰色

✅ 推荐:
- 选择一个极端风格 (极简 OR 极繁)
- 独特字体 + 大胆配色
- 非对称布局 + 留白
- 有意识的设计决策
```

**风格方向选择**:
- Brutalist (粗野主义): 原始、工业、无装饰
- Editorial (杂志风): 大留白、精致字体、网格
- Luxury (奢华): 深色、金色点缀、衬线字体
- Corporate Modern (现代商务): 清晰、专业、数据驱动
- Minimalist (极简): 减法设计、呼吸空间

### product-ux-expert (UX 评估)

**Nielsen's 10 Heuristics**:
1. Visibility of System Status
2. Match System & Real World
3. User Control & Freedom
4. Consistency & Standards
5. Error Prevention
6. Recognition over Recall
7. Flexibility & Efficiency
8. **Aesthetic & Minimalist Design** ← 审美相关
9. Help Users with Errors
10. Help & Documentation

**认知心理学法则**:
- Hick's Law: 选项越多，决策越慢
- Miller's Law: 工作记忆 7±2 项
- Fitts's Law: 大目标更易点击
- Von Restorff: 不同的更易记忆

**Gestalt 原则**:
- Proximity (接近)
- Similarity (相似)
- Continuity (连续)
- Closure (闭合)

---

## 十五、综合推荐方案

### 方案对比

| 方案 | PPT | PDF | 审美 | 复杂度 | 推荐度 |
|------|-----|-----|------|--------|--------|
| Presenton | ✅ | ✅ | ⭐⭐⭐⭐ | 低 | ⭐⭐⭐⭐⭐ |
| UI UX Pro Max + python-pptx | ✅ | ❌ | ⭐⭐⭐⭐⭐ | 中 | ⭐⭐⭐⭐ |
| PDFME + 模板 | ❌ | ✅ | ⭐⭐⭐⭐⭐ | 中 | ⭐⭐⭐⭐ |
| Carbone | ✅ | ✅ | ⭐⭐⭐ | 中 | ⭐⭐⭐ |
| HTML+Tailwind→PDF | ❌ | ✅ | ⭐⭐⭐⭐⭐ | 高 | ⭐⭐⭐ |

### 最终推荐: Presenton + UI UX Pro Max Skill

```
架构:
┌─────────────────────────────────────────────────────────────┐
│                   OM Generation Agent                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. 安装 UI UX Pro Max Skill (提供审美能力)                   │
│     - 57 种 UI 风格                                          │
│     - 95 套配色方案                                          │
│     - 56 组字体搭配                                          │
│     ↓                                                        │
│  2. 内容解析 (RAGFlow + LLM)                                  │
│     ↓                                                        │
│  3. 大纲生成 (结合 UI/UX 审美原则)                             │
│     ↓                                                        │
│  4. 调用 Presenton API 生成 PPT                              │
│     - 使用预制 OM 模板                                        │
│     - 支持 PPTX/PDF 导出                                     │
│     ↓                                                        │
│  5. 输出精美商业报告                                          │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 十六、待调研

- [x] 开源 AI PPT 生成项目调研 ✅
- [x] 开源 PDF 生成项目调研 ✅
- [x] 本地审美 Skills 整理 ✅
- [x] 网络 Claude Code Skills 调研 ✅
- [ ] 安装测试 UI UX Pro Max Skill
- [ ] Presenton 部署测试
- [ ] PDFME 可视化设计器试用
- [ ] pptx / pdf Skill 详细功能测试
- [ ] 商业报告模板设计
- [ ] OM 场景专用模板开发
- [ ] Presenton 源码分析 (模板系统如何实现)
- [ ] Presenton 部署测试
- [ ] PDFME 可视化设计器试用
- [ ] 商业报告模板设计
- [ ] OM 场景专用模板开发
