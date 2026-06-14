# 个人博客搭建完整计划

> **项目名称**: SiliconBlog（暂定，可更改）
> **技术栈**: Hugo + PaperMod + GitHub Pages + GitHub Actions
> **目标受众**: 半导体 CAD 工程师
> **总费用**: ¥0

## AI 工具分工

| 角色 | 负责阶段 | 理由 |
|------|---------|------|
| **Cowork** | 总体方案、提示词编写、Review、调试 | 架构规划与质量把控 |
| **Claude Code** | Phase 1（项目初始化）、Phase 4（内容）、Phase 5（SEO）、Phase 6（CI/CD） | 文件操作、CLI 执行、配置生成 |
| **Gemini** | Phase 2（CSS 主题设计）、Phase 3-SVG（Logo、电路纹理） | 前端设计 WebDev Arena #1 |
| **Codex** | Phase 3-IMG（OG 社交图、Favicon 光栅图） | 图像生成 |

---

## Phase 1：环境准备与项目初始化

### 目标
在本地创建完整的 Hugo 项目骨架，集成 PaperMod 主题，配置基础参数，确保 `hugo server` 可本地预览。

### Claude Code 提示词

```
请帮我完成以下任务（按顺序执行）：

1. 检查系统是否已安装 Hugo（运行 `hugo version`）。如果未安装：
   - Windows: 用 `winget install Hugo.Hugo.Extended` 或 `choco install hugo-extended`
   - 确认安装的是 Extended 版本（支持 SCSS）

2. 检查系统是否已安装 Git（运行 `git version`）。如果未安装则提示我手动安装。

3. 在我指定的目录下创建 Hugo 新站点：
   ```bash
   hugo new site silicon-blog
   cd silicon-blog
   ```

4. 初始化 Git 仓库并以 submodule 方式添加 PaperMod 主题：
   ```bash
   git init
   git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
   ```

5. 创建 `hugo.yaml` 配置文件（删除默认的 hugo.toml），内容如下：

   ```yaml
   baseURL: "https://YOUR_GITHUB_USERNAME.github.io/"
   languageCode: zh-cn
   defaultContentLanguage: zh
   title: "SiliconBlog — 半导体 CAD 工程师的技术笔记"
   theme: PaperMod
   paginate: 8

   params:
     env: production
     defaultTheme: dark
     ShowReadingTime: true
     ShowShareButtons: true
     ShowPostNavLinks: true
     ShowBreadCrumbs: true
     ShowCodeCopyButtons: true
     ShowWordCount: true
     ShowRssButtonInSectionTermList: true
     UseHugoToc: true
     disableSpecial1stPost: false
     hidemeta: false
     showtoc: true
     tocopen: false
     comments: false

     homeInfoParams:
       Title: "欢迎来到 SiliconBlog"
       Content: >
         面向半导体 CAD 工程师的技术博客。
         分享 EDA 工具、PDK 开发、DRC/LVS、Pcell 编程、
         自动化脚本等实战经验。

     socialIcons:
       - name: github
         url: "https://github.com/YOUR_GITHUB_USERNAME"
       - name: email
         url: "mailto:YOUR_EMAIL"

     assets:
       favicon: "/favicon.ico"
       favicon16x16: "/favicon-16x16.png"
       favicon32x32: "/favicon-32x32.png"
       apple_touch_icon: "/apple-touch-icon.png"

     fuseOpts:
       isCaseSensitive: false
       shouldSort: true
       location: 0
       distance: 1000
       threshold: 0.4
       minMatchCharLength: 0
       keys: ["title", "permalink", "summary", "content"]

   menu:
     main:
       - identifier: posts
         name: 文章
         url: /posts/
         weight: 10
       - identifier: categories
         name: 分类
         url: /categories/
         weight: 20
       - identifier: tags
         name: 标签
         url: /tags/
         weight: 30
       - identifier: archives
         name: 归档
         url: /archives/
         weight: 40
       - identifier: search
         name: 搜索
         url: /search/
         weight: 50
       - identifier: about
         name: 关于
         url: /about/
         weight: 60

   outputs:
     home:
       - HTML
       - RSS
       - JSON

   markup:
     highlight:
       codeFences: true
       guessSyntax: true
       lineNos: true
       style: dracula
     goldmark:
       renderer:
         unsafe: true
   ```

6. 创建归档页面 `content/archives.md`：
   ```markdown
   ---
   title: "归档"
   layout: "archives"
   url: "/archives/"
   summary: archives
   ---
   ```

7. 创建搜索页面 `content/search.md`：
   ```markdown
   ---
   title: "搜索"
   layout: "search"
   url: "/search/"
   summary: search
   placeholder: "搜索文章..."
   ---
   ```

8. 创建关于页面 `content/about.md`：
   ```markdown
   ---
   title: "关于"
   url: "/about/"
   summary: about
   ---

   ## 关于本站

   这是一个面向半导体 CAD 工程师的技术博客，主要分享：

   - EDA 工具使用技巧（Cadence Virtuoso、Synopsys 等）
   - PDK 开发与维护经验
   - DRC/LVS 规则编写
   - Pcell / SKILL 编程
   - 自动化脚本（Python、Shell、SKILL）
   - 半导体行业技术洞察

   ## 关于作者

   一名半导体 CAD 工程师，热爱技术分享。
   ```

9. 创建第一篇测试文章 `content/posts/hello-world.md`：
   ```markdown
   ---
   title: "Hello World — 博客启航"
   date: 2024-01-01T00:00:00+08:00
   draft: false
   tags: ["博客", "公告"]
   categories: ["公告"]
   summary: "SiliconBlog 正式启航！记录半导体 CAD 工程师的技术探索之路。"
   ---

   欢迎来到 SiliconBlog！

   这是本站的第一篇文章，未来将在这里分享半导体 CAD 领域的技术经验。

   ## 计划内容

   - SKILL 编程实战
   - Python 自动化 EDA 工作流
   - PDK 开发笔记
   - DRC/LVS 规则调试技巧

   敬请期待！
   ```

10. 运行本地预览服务器：
    ```bash
    hugo server -D
    ```
    确认网站可以在 http://localhost:1313 正常访问。

完成后告诉我每一步的执行结果。
```

### 验收标准
- [ ] `hugo version` 输出 Extended 版本号
- [ ] `hugo server -D` 成功启动，无报错
- [ ] 浏览器访问 localhost:1313 可见暗色主题的博客首页
- [ ] 导航栏显示：文章、分类、标签、归档、搜索、关于
- [ ] 搜索功能可用

---

## Phase 2：主题深度定制 — 半导体科技风

> **执行者：Gemini** — 前端 CSS/HTML 设计

### 目标
在 PaperMod 基础上，通过自定义 CSS 和 partial 模板，打造具有半导体/电路风格的科技感界面。

### 设计语言
- **主色调**: 深空黑 `#0a0e17` + 电路板绿 `#00ff88` / 芯片蓝 `#00d4ff`
- **辅助色**: 硅晶紫 `#a855f7` 用于标签/高亮
- **字体**: 正文用系统无衬线，代码用 `JetBrains Mono`（Google Fonts 免费）
- **装饰**: 电路线纹理作为背景微装饰，不喧宾夺主

### Gemini 提示词

```
我正在用 Hugo 静态站点生成器 + PaperMod 主题搭建一个半导体 CAD 工程师的技术博客（名为 "SiliconBlog"）。
我需要你帮我设计一套完整的自定义 CSS 主题 + Hugo partial 模板，打造半导体/集成电路科技风视觉。

## 项目背景

- Hugo 博客，主题为 PaperMod（https://github.com/adityatelange/hugo-PaperMod）
- PaperMod 的自定义 CSS 入口：`assets/css/extended/custom.css`（放在此路径会被自动加载）
- PaperMod 的扩展 partial 入口：`layouts/partials/extend_head.html` 和 `layouts/partials/extend_footer.html`
- 目标受众：半导体行业工程师，内容包含大量代码（SKILL、Python、Shell）

## 设计要求

### 配色
- 主背景：深空黑 #0a0e17
- 卡片/区块背景：#111827
- 主强调色：电路板绿 #00ff88（用于 Logo、链接 hover、h2 标题）
- 次强调色：芯片蓝 #00d4ff（用于 h3、链接）
- 第三强调色：硅晶紫 #a855f7（用于标签）
- 正文：#d1d5db
- 边框：#1e293b

### 视觉效果（请发挥你的前端设计能力，做出精致的效果）
1. **背景装饰**：用纯 CSS 实现微妙的电路板网格线（不使用图片），透明度极低，不影响阅读
2. **导航栏**：毛玻璃效果（backdrop-filter: blur），Logo 文字为电路绿色
3. **文章卡片**：hover 时有绿色荧光边框 glow + 微上浮 + 顶部渐变彩条（绿→蓝→紫）淡入
4. **代码块**：使用 JetBrains Mono 字体（Google Fonts），左侧 3px 渐变色条（绿→蓝），深色背景 #0d1117
5. **行内代码**：绿色文字 + 半透明绿色背景
6. **标签**：紫色系胶囊样式，hover 加深
7. **目录（TOC）**：与卡片同色背景，hover 变绿
8. **滚动条**：自定义为窄条 + 主题色
9. **选中文字**：绿色半透明高亮
10. **页脚标语**：居中显示 "Powered by Silicon & Code — 用代码驱动半导体设计"
11. **搜索框**：focus 时绿色边框 + 辉光

### 额外要求
- 同时提供亮色模式的变量覆盖（:root 下），保持科技感但适合日间阅读
- 响应式适配：移动端网格背景缩小
- CSS 动画要轻量，不影响性能
- 所有代码以注释清晰分区

## 输出格式

请输出以下 3 个完整文件的代码：

1. **`assets/css/extended/custom.css`** — 完整的自定义 CSS
2. **`layouts/partials/extend_head.html`** — Google Fonts preconnect
3. **`layouts/partials/extend_footer.html`** — 站点标语

请直接输出可用的完整代码，不需要解释原理。我会将代码文件放入 Hugo 项目对应路径。
```

### Claude Code 后续操作提示词（将 Gemini 输出的文件放入项目）

```
请帮我将以下三个文件放入 Hugo 博客项目对应路径：

1. 将 [Gemini 输出的 CSS 代码] 保存到 `assets/css/extended/custom.css`
2. 将 [Gemini 输出的 head partial] 保存到 `layouts/partials/extend_head.html`
3. 将 [Gemini 输出的 footer partial] 保存到 `layouts/partials/extend_footer.html`

然后运行 `hugo server -D` 验证效果。
```

### 验收标准
- [ ] 深色背景 + 电路网格装饰可见
- [ ] 文章卡片 hover 效果正常（绿色辉光 + 顶部彩条）
- [ ] 代码块使用 JetBrains Mono 字体 + 左侧渐变条
- [ ] 标签为紫色风格
- [ ] 移动端正常显示

---

## Phase 3：Logo 与图形资产设计

### 目标
为博客设计 Logo、Favicon、OG 社交分享图和装饰性 SVG。

### 3A — SVG 图形（Gemini 执行）

#### Gemini 提示词：Logo + 电路纹理 SVG

```
我需要为一个名为 "SiliconBlog" 的半导体技术博客设计 SVG 图形资产。博客主题色为深空黑 #0a0e17 背景 + 电路板绿 #00ff88 + 芯片蓝 #00d4ff。

请为我生成以下 SVG 代码：

### 1. 横版 Logo（用于网站 header）

设计思路：
- 将"芯片/IC"图形符号与博客名称结合
- 可以考虑：芯片引脚 + 尖括号 </> 融合、晶圆切片图形、电路走线连接文字等创意方案
- 文字 "SiliconBlog" 使用无衬线体，字母间距略宽
- 主色 #00ff88，辅助 #00d4ff
- 适合深色背景，高度约 40px 的使用场景
- 输出干净、可缩放的 SVG 代码

### 2. 正方形图标（用于 Favicon/App Icon）

- 从 Logo 中提取核心图形符号（不含文字）
- 512x512 viewBox
- 在 16x16 的极小尺寸下仍可辨认
- 输出 SVG 代码

### 3. 可平铺的电路板纹理图案（用于页面背景装饰）

- 200x200 viewBox，可无缝平铺
- 包含：直线走线、过孔（via）圆点、拐角、T 型交叉
- 颜色：#00ff88，透明度 3-5%（极淡）
- 背景透明
- 线宽 1px
- 作为网页 background-image 使用时不影响正文阅读

请直接输出 3 个完整的 SVG 代码块，不需要解释。
```

### 3B — 光栅图像（Codex 执行）

#### Codex 提示词：OG 社交分享图
```
设计一张博客社交分享预览图（Open Graph Image）。

要求：
- 尺寸：1200x630 像素
- 背景：深空黑 #0a0e17，带有微妙的电路板走线纹理
- 中央显示博客名 "SiliconBlog"（使用无衬线粗体，白色或 #00ff88 绿色）
- 副标题："半导体 CAD 工程师的技术笔记"（较小字号，#8899aa 灰色）
- 底部有一条从左到右的渐变线（绿 #00ff88 → 蓝 #00d4ff → 紫 #a855f7）
- 角落点缀简化的芯片/晶圆图形元素
- 风格：科技、专业、简洁
- 输出 PNG 格式
```

#### Codex 提示词：Favicon PNG 系列
```
根据以下 SVG 图标 [粘贴 Gemini 生成的正方形图标 SVG]，
生成以下尺寸的 PNG 文件：
- favicon-16x16.png
- favicon-32x32.png
- apple-touch-icon.png（180x180）
- favicon.ico（包含 16、32、48 多尺寸）
背景透明，保持 #00ff88 和 #00d4ff 配色。
```

### 资产放置位置
Logo 和图形资产由 Codex 生成后，放入项目的 `static/` 目录：
```
static/
├── favicon.ico
├── favicon-16x16.png
├── favicon-32x32.png
├── apple-touch-icon.png
├── logo.svg
├── og-image.png
└── circuit-pattern.svg   (可选)
```

---

## Phase 4：内容结构与示例文章

### 目标
建立面向半导体 CAD 工程师的分类体系，创建文章模板。

### Claude Code 提示词

```
请帮我在 Hugo 博客项目中完成以下内容结构搭建：

1. 创建文章模板 `archetypes/posts.md`，用于 `hugo new posts/xxx.md` 时自动填充：

   ```markdown
   ---
   title: "{{ replace .Name "-" " " | title }}"
   date: {{ .Date }}
   draft: true
   author: "SiliconBlog"
   tags: []
   categories: []
   summary: ""
   description: ""
   cover:
     image: ""
     alt: ""
     caption: ""
   ShowToc: true
   TocOpen: false
   ---

   <!--more-->
   ```

2. 创建一篇示例技术文章 `content/posts/skill-programming-basics.md`：

   ```markdown
   ---
   title: "SKILL 编程入门：Cadence Virtuoso 自动化的第一步"
   date: 2024-01-15T10:00:00+08:00
   draft: false
   author: "SiliconBlog"
   tags: ["SKILL", "Cadence", "Virtuoso", "自动化", "EDA"]
   categories: ["SKILL 编程"]
   summary: "SKILL 是 Cadence Virtuoso 平台的内置脚本语言。本文介绍 SKILL 的基础语法、常用函数和一个实际的自动化示例。"
   ShowToc: true
   TocOpen: true
   ---

   SKILL 是 Cadence Virtuoso 平台的内置脚本语言，基于 Lisp 方言设计。掌握 SKILL 编程是半导体 CAD 工程师提升效率的关键技能之一。

   ## 为什么学 SKILL？

   在日常工作中，CAD 工程师经常需要：

   - 批量修改版图属性
   - 自动生成 Pcell（参数化单元）
   - 定制 DRC/LVS 检查流程
   - 开发自定义工具菜单

   这些任务如果手动完成，效率低且容易出错。SKILL 可以将这些操作自动化。

   ## 基础语法

   ### 变量与赋值

   ```skill
   ; 定义变量
   myVar = 42
   myStr = "Hello, Silicon!"

   ; 列表
   myList = '(1 2 3 4 5)
   ```

   ### 函数定义

   ```skill
   procedure(addNumbers(a b)
     a + b
   )

   ; 调用
   addNumbers(3 4)  ; => 7
   ```

   ### 条件判断

   ```skill
   procedure(checkWidth(w)
     if(w < 0.1 then
       printf("Warning: width %.3f below minimum!\n" w)
     else
       printf("Width %.3f is OK.\n" w)
     )
   )
   ```

   ## 实用示例：批量获取 Cell 信息

   ```skill
   procedure(listAllCells(libName)
     let((lib cellIds)
       lib = ddGetObj(libName)
       when(lib
         cellIds = lib~>cells
         foreach(cell cellIds
           printf("Cell: %s\n" cell~>name)
         )
       )
     )
   )

   ; 使用
   listAllCells("myDesignLib")
   ```

   ## 小结

   SKILL 编程是 CAD 工程师的核心技能。后续文章将深入介绍 Pcell 开发、DRC 规则编写等进阶主题。

   ---

   *本文是 SKILL 编程系列的第一篇，敬请关注后续更新。*
   ```

3. 创建第二篇示例文章 `content/posts/python-eda-automation.md`：

   ```markdown
   ---
   title: "Python 在半导体 EDA 自动化中的应用"
   date: 2024-01-20T10:00:00+08:00
   draft: false
   author: "SiliconBlog"
   tags: ["Python", "EDA", "自动化", "脚本"]
   categories: ["Python 自动化"]
   summary: "Python 正在成为半导体行业自动化的重要工具。本文介绍如何用 Python 处理 GDS 文件、解析日志和自动化 EDA 工作流。"
   ShowToc: true
   TocOpen: true
   ---

   Python 凭借丰富的生态和简洁的语法，正在半导体 EDA 领域发挥越来越重要的作用。

   ## 常用 Python 库

   | 库名 | 用途 |
   |------|------|
   | `gdstk` / `gdspy` | GDS II 文件读写与操作 |
   | `klayout.db` | KLayout 脚本引擎 |
   | `pya` | KLayout Python API |
   | `pandas` | 数据分析与日志解析 |
   | `matplotlib` | 数据可视化 |

   ## 示例：用 gdstk 创建简单版图

   ```python
   import gdstk

   # 创建库和顶层 Cell
   lib = gdstk.Library()
   cell = lib.new_cell("DEMO")

   # 画一个矩形（Layer 1, Datatype 0）
   rect = gdstk.rectangle((0, 0), (10, 5), layer=1)
   cell.add(rect)

   # 保存 GDS 文件
   lib.write_gds("demo.gds")
   print("GDS file created successfully!")
   ```

   ## 示例：解析 DRC 日志

   ```python
   import re
   from collections import Counter

   def parse_drc_log(filepath):
       violations = []
       with open(filepath, 'r') as f:
           for line in f:
               match = re.search(r'Rule:\s+(\S+).*Count:\s+(\d+)', line)
               if match:
                   violations.append({
                       'rule': match.group(1),
                       'count': int(match.group(2))
                   })
       return violations

   # 统计 Top 10 违规规则
   results = parse_drc_log("drc_results.log")
   counter = Counter({v['rule']: v['count'] for v in results})
   for rule, count in counter.most_common(10):
       print(f"{rule}: {count}")
   ```

   ## 小结

   Python 在 EDA 领域的应用远不止于此。后续将介绍更多实战案例。
   ```

4. 运行 `hugo server -D` 确认所有文章正常渲染。

完成后告诉我结果。
```

### 分类体系（已嵌入文章 front matter）
- **SKILL 编程**: Cadence SKILL/SKILL++ 相关
- **Python 自动化**: Python 在 EDA 中的应用
- **PDK 开发**: PDK 维护与开发经验
- **DRC/LVS**: 设计规则检查
- **工具技巧**: EDA 工具使用 Tips
- **行业观察**: 半导体技术趋势

---

## Phase 5：SEO 与搜索引擎收录

### 目标
确保博客能被 Google 等搜索引擎发现和索引。

### Claude Code 提示词

```
请帮我为 Hugo 博客配置 SEO 优化：

1. 在 `hugo.yaml` 中追加以下 SEO 相关配置（如果尚未存在）：

   ```yaml
   params:
     description: "SiliconBlog — 面向半导体 CAD 工程师的技术博客，分享 EDA、SKILL 编程、PDK 开发、DRC/LVS 等实战经验"
     images:
       - "/og-image.png"

   sitemap:
     changefreq: weekly
     filename: sitemap.xml
     priority: 0.5
   ```

2. 创建 `layouts/robots.txt` 模板：
   ```
   User-agent: *
   Allow: /

   Sitemap: {{ .Site.BaseURL }}sitemap.xml
   ```

   并在 `hugo.yaml` 中启用：
   ```yaml
   enableRobotsTXT: true
   ```

3. 创建（或更新）`layouts/partials/extend_head.html`，添加 Open Graph 和结构化数据：

   ```html
   <!-- Preconnect for Google Fonts -->
   <link rel="preconnect" href="https://fonts.googleapis.com">
   <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

   <!-- Additional SEO meta -->
   <meta name="author" content="SiliconBlog">
   <meta name="keywords" content="半导体, CAD, EDA, SKILL, Cadence, Virtuoso, PDK, DRC, LVS, Pcell, 自动化">

   <!-- Structured Data: WebSite -->
   <script type="application/ld+json">
   {
     "@context": "https://schema.org",
     "@type": "WebSite",
     "name": "{{ .Site.Title }}",
     "url": "{{ .Site.BaseURL }}",
     "description": "{{ .Site.Params.description }}",
     "potentialAction": {
       "@type": "SearchAction",
       "target": "{{ .Site.BaseURL }}search/?q={search_term_string}",
       "query-input": "required name=search_term_string"
     }
   }
   </script>
   ```

4. 运行 `hugo` 构建，确认以下文件正确生成：
   - `public/sitemap.xml` — 包含所有文章 URL
   - `public/robots.txt` — 包含 Sitemap 声明
   - 文章页面的 HTML `<head>` 中包含 og:title, og:description, og:image

完成后告诉我结果。
```

### 手动步骤（部署上线后由你操作）
1. 访问 [Google Search Console](https://search.google.com/search-console)
2. 添加资源 → URL 前缀 → 输入 `https://yourusername.github.io/`
3. 验证方式选择 "HTML 标签"，将 meta 标签加入 `extend_head.html`
4. 提交 sitemap：`https://yourusername.github.io/sitemap.xml`

---

## Phase 6：GitHub Actions 自动部署

### 目标
配置 CI/CD，实现 `git push` 后自动构建并部署到 GitHub Pages。

### 前置步骤（你手动操作）
1. 在 GitHub 创建仓库，命名为 `yourusername.github.io`（公开仓库）
2. 在仓库 Settings → Pages → Source 选择 "GitHub Actions"

### Claude Code 提示词

```
请帮我创建 GitHub Actions 部署工作流：

1. 创建 `.github/workflows/deploy.yml`：

   ```yaml
   name: Deploy Hugo to GitHub Pages

   on:
     push:
       branches: [main]
     workflow_dispatch:

   permissions:
     contents: read
     pages: write
     id-token: write

   concurrency:
     group: "pages"
     cancel-in-progress: false

   defaults:
     run:
       shell: bash

   jobs:
     build:
       runs-on: ubuntu-latest
       env:
         HUGO_VERSION: 0.147.0
       steps:
         - name: Install Hugo CLI
           run: |
             wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
             && sudo dpkg -i ${{ runner.temp }}/hugo.deb

         - name: Checkout
           uses: actions/checkout@v4
           with:
             submodules: recursive
             fetch-depth: 0

         - name: Setup Pages
           id: pages
           uses: actions/configure-pages@v5

         - name: Build with Hugo
           env:
             HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
             HUGO_ENVIRONMENT: production
             TZ: Asia/Shanghai
           run: |
             hugo \
               --gc \
               --minify \
               --baseURL "${{ steps.pages.outputs.base_url }}/"

         - name: Upload artifact
           uses: actions/upload-pages-artifact@v3
           with:
             path: ./public

     deploy:
       environment:
         name: github-pages
         url: ${{ steps.deployment.outputs.page_url }}
       runs-on: ubuntu-latest
       needs: build
       steps:
         - name: Deploy to GitHub Pages
           id: deployment
           uses: actions/deploy-pages@v4
   ```

2. 创建 `.gitignore`（如不存在）：
   ```
   public/
   resources/_gen/
   .hugo_build.lock
   .DS_Store
   ```

3. 告诉我接下来的 Git 推送命令。

完成后告诉我结果。
```

### 推送命令（你手动执行）
```bash
cd silicon-blog
git add .
git commit -m "Initial commit: Hugo blog with PaperMod theme"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/YOUR_USERNAME.github.io.git
git push -u origin main
```

---

## Phase 7：验收与调试

### 验收清单

**功能验收**
- [ ] 首页正常加载，显示文章列表
- [ ] 暗色主题默认启用，电路网格背景可见
- [ ] 文章卡片 hover 效果（绿色辉光）
- [ ] 单篇文章页：目录、代码高亮、标签正常
- [ ] 搜索功能可用
- [ ] 归档页按时间排列
- [ ] 分类/标签页正常
- [ ] RSS 订阅可用（/index.xml）

**SEO 验收**
- [ ] /sitemap.xml 可访问且包含所有文章
- [ ] /robots.txt 正确声明 sitemap
- [ ] 文章页 HTML 包含 og:title, og:description, og:image
- [ ] Google Search Console 已提交 sitemap

**性能验收**
- [ ] Lighthouse Performance ≥ 90
- [ ] Lighthouse SEO ≥ 90
- [ ] 移动端响应式正常

**部署验收**
- [ ] GitHub Actions 构建成功（绿色 ✓）
- [ ] https://username.github.io 可正常访问
- [ ] 新 push 后自动更新

---

## 后续扩展（可选）

- **评论系统**: 接入 Giscus（基于 GitHub Discussions，免费）
- **自定义域名**: 购买域名后在 GitHub Pages 设置 CNAME
- **访问统计**: 接入 Google Analytics 或 Umami（自托管免费）
- **数学公式**: 启用 KaTeX 支持
- **Mermaid 图表**: 流程图/架构图渲染
