# Technical Design: NovelKit → PaperKit 改造

## Context

NovelKit 是一个面向小说创作的 AI 辅助工具，采用"命令 + 脚本 + 模板"的模块化架构。用户需要将其改造为论文写作工具（PaperKit），以支持学术论文的全流程创作。

**关键约束**：
- 保留现有的模块化架构和 slash 命令交互方式
- 支持通用学术写作场景（本科生、研究生、科研人员）
- 使用 Markdown 作为主要写作格式
- 支持多格式导出（Markdown/Word/LaTeX/PDF）

**技术栈**：
- Python 3.11+
- Bash/PowerShell 脚本
- Pandoc（文档转换）
- bibtexparser/pybtex（.bib 解析）
- DOI.org/Crossref API（文献解析）

## Goals / Non-Goals

### Goals
- ✅ 将 NovelKit 的核心架构迁移到论文写作场景
- ✅ 实现完整的文献管理系统（.bib 文件管理、DOI/URL 自动解析）
- ✅ 实现大纲生成系统（支持 IMRAD 等标准结构）
- ✅ 实现引用管理系统（引用插入、审计、参考文献生成）
- ✅ 实现多格式导出系统（Markdown/Word/LaTeX/PDF）
- ✅ 保持跨平台兼容性（Linux、macOS、Windows）

### Non-Goals
- ❌ 不提供从 NovelKit 到 PaperKit 的自动迁移工具（领域差异过大）
- ❌ 不集成专业文献管理软件（Zotero、Mendeley）的 API（MVP 阶段）
- ❌ 不提供图形界面（保持命令行工具定位）
- ❌ 不支持实时协作编辑（保持单用户本地工作流）

## Decisions

### Decision 1: 目录结构设计

**选择**：`.paperkit/` 作为 meta-space，`sections/`、`sources/`、`figures/`、`tables/`、`exports/` 作为 user-space。

**理由**：
- 清晰分离系统文件和用户内容
- 便于版本控制（`.paperkit/` 可以被 `.gitignore`）
- 符合学术写作的文件组织习惯

**目录结构**：
```
./
├─ .paperkit/                      # meta-space（系统文件）
│  ├─ memory/                      # 状态和配置
│  │  ├─ config.json               # 全局配置
│  │  ├─ thesis.md                 # 论文论点宪法
│  │  ├─ outline.json              # 大纲结构
│  │  └─ sources-index.json        # 文献索引缓存
│  ├─ templates/                   # 模板文件
│  ├─ scripts/                     # 脚本文件
│  ├─ formats/                     # 引文格式配置
│  ├─ sections/                    # 章节元数据
│  └─ cache/                       # 缓存（DOI/URL 解析结果）
├─ sections/                       # user-space（论文正文）
├─ sources/                        # user-space（文献与附件）
│  ├─ references.bib               # 主 .bib 文件
│  ├─ notes/                       # 文献阅读笔记
│  └─ pdfs/                        # PDF 附件
├─ figures/                        # user-space（图片）
├─ tables/                         # user-space（表格）
├─ appendices/                     # user-space（附录）
└─ exports/                        # user-space（导出产物）
```

**替代方案**：
- 方案 A：将所有文件放在项目根目录（被拒绝，因为会导致目录混乱）
- 方案 B：使用 `paper/` 作为主目录（被拒绝，因为不符合学术写作习惯）

### Decision 2: 文献管理方案

**选择**：`sources/references.bib` 作为事实来源（source of truth），`.paperkit/memory/sources-index.json` 作为可重建索引缓存。

**理由**：
- `.bib` 文件是学术界标准格式，易于与其他工具互操作
- 纯文本格式便于版本控制和 AI 解析
- 索引缓存提高检索性能，避免每次全量解析

**实现细节**：
- 使用 `bibtexparser` 或 `pybtex` 进行解析
- 新增条目采用"追加写入"策略，保留用户原有格式和注释
- 更新条目优先提示用户手改 `.bib`，再运行 `/paper.source.sync` 重建索引
- 索引字段：`citation_key`、`doi`、`url`、`title`、`authors`、`year`、`tags`、`attachments`

**替代方案**：
- 方案 A：使用 JSON 格式存储文献（被拒绝，因为不符合学术界标准）
- 方案 B：直接集成 Zotero API（被拒绝，因为增加复杂度，MVP 阶段不需要）

### Decision 3: DOI/URL 自动解析方案

**选择**：多级解析管线 + 缓存 + 离线降级。

**DOI 解析流程**：
1. 优先使用 `doi.org` 内容协商（`Accept: application/vnd.citationstyles.csl+json`）
2. Fallback 到 Crossref API（`api.crossref.org/works/<doi>`）
3. 失败降级：提示用户粘贴 BibTeX 或手工录入

**URL 解析流程**：
1. 从 URL/页面中提取 DOI（若存在则走 DOI 流程）
2. 特判站点：arXiv、PubMed、ACM、IEEE（使用其公开 API 或页面 meta）
3. 通用 HTML 元数据抽取：`citation_*`（Highwire）、`dc.*`（DublinCore）、schema.org JSON-LD
4. 失败降级：保存为 `@misc`（至少保留 title/url/accessed），提示用户补齐字段

**缓存策略**：
- `.paperkit/cache/doi/<doi-hash>.json`：保存 DOI 解析结果
- `.paperkit/cache/url/<url-hash>.json`：保存 URL 解析结果
- 缓存有效期：30 天（可配置）

**Citation Key 生成规则**：
- 格式：`familyYYYYshorttitle`（如 `smith2023diffusion`）
- 冲突处理：追加 `a/b/c`（如 `smith2023diffusiona`）
- 记录 `generated_from` 字段以便追溯

**替代方案**：
- 方案 A：仅支持手动添加（被拒绝，因为不满足用户需求）
- 方案 B：使用第三方 API 服务（被拒绝，因为增加依赖和成本）

### Decision 4: 引用插入与参考文献生成方案

**选择**：使用 Pandoc 引用语法（`[@key]`）+ Pandoc citeproc 生成参考文献。

**引用语法**：
- 单引用：`[@smith2023diffusion]`
- 多引用：`[@smith2023diffusion; @wang2022survey]`
- 页码：`[@smith2023diffusion, p. 12]`
- 作者年份：`@smith2023diffusion says...`

**参考文献生成**：
- 导出时：`pandoc --citeproc --bibliography sources/references.bib --csl <style.csl>`
- 预览时：`/paper.bibliography.generate` 生成 `references.md`

**理由**：
- Pandoc 是学术写作的标准工具，支持多种引用格式（CSL）
- 引用语法简洁，易于 AI 生成和人工编辑
- 与 Markdown 生态完美集成

**替代方案**：
- 方案 A：使用 LaTeX 引用语法（`\cite{key}`）（被拒绝，因为不适合 Markdown）
- 方案 B：自己实现引用解析器（被拒绝，因为重复造轮子）

### Decision 5: 多格式导出方案

**选择**：以 Pandoc 作为唯一转换引擎，format profile 决定 CSL + docx reference + LaTeX template + pdf-engine。

**导出流程**：
1. 合并所有 section 文件为单个 Markdown 文件
2. 根据 format profile 选择 CSL 文件、docx 模板、LaTeX 模板
3. 调用 Pandoc 进行转换
4. 输出到 `exports/` 目录

**格式支持**：
- **Markdown**：直接合并 section 文件
- **Word（.docx）**：使用 `reference.docx` 样式模板
- **LaTeX（.tex）**：使用 `template.tex` 模板，支持中文（xelatex）
- **PDF**：依赖 TeX 引擎（xelatex/pdflatex）

**依赖检测**：
- 检测 Pandoc 安装：`pandoc --version`
- 检测 TeX 引擎安装：`xelatex --version` 或 `pdflatex --version`
- 提供安装指引（针对不同操作系统）

**理由**：
- Pandoc 是学术写作的标准工具，支持多种格式转换
- 避免重复造轮子，降低维护成本
- 用户可以自定义模板和样式

**替代方案**：
- 方案 A：使用 Python 库（python-docx、pylatex）（被拒绝，因为功能有限）
- 方案 B：集成多个转换工具（被拒绝，因为增加复杂度）

### Decision 6: 命令命名规范

**选择**：`/paper.<module>.<action>` 格式。

**模块划分**：
- `setup`：项目初始化
- `thesis`：论文论点宪法
- `format`：引文格式与排版风格
- `section`：章节管理
- `source`：文献管理
- `outline`：大纲生成与管理
- `citation`：引用插入与审计
- `bibliography`：参考文献生成
- `export`：多格式导出

**命令示例**：
- `/paper.setup`
- `/paper.thesis.create`
- `/paper.source.add`
- `/paper.section.write`
- `/paper.export`

**理由**：
- 清晰的命名空间，避免命令冲突
- 符合用户的心理模型（模块 → 动作）
- 易于扩展和维护

### Decision 7: Web 预览系统架构

**选择**：使用 Flask 作为 Web 服务器，Pandoc 进行 Markdown 到 HTML 转换，watchdog 进行文件监听。

**实现细节**：
- **Web 框架**：Flask（轻量级，易于集成）
- **渲染引擎**：Pandoc（支持引用格式、数学公式、表格等）
- **文件监听**：watchdog 库（跨平台）
- **实时更新**：Server-Sent Events（SSE，比 WebSocket 更简单）
- **前端**：原生 HTML/CSS/JavaScript（避免引入复杂的前端框架）

**目录结构**：
```
.paperkit/
├─ server/
│  ├─ app.py              # Flask 应用
│  ├─ templates/
│  │  └─ preview.html     # 预览页面模板
│  ├─ static/
│  │  ├─ css/
│  │  │  ├─ academic.css  # 学术论文样式
│  │  │  ├─ journal.css   # 期刊模板样式
│  │  │  └─ themes.css    # 主题样式
│  │  └─ js/
│  │     ├─ preview.js    # 预览逻辑
│  │     └─ diff.js       # 差异对比逻辑
│  └─ utils/
│     ├─ renderer.py      # 渲染工具
│     └─ watcher.py       # 文件监听
```

**理由**：
- Flask 轻量级，易于集成到 CLI 工具
- Pandoc 是学术写作的标准工具，支持丰富的格式
- watchdog 跨平台，稳定可靠
- SSE 比 WebSocket 更简单，适合单向推送

**替代方案**：
- 方案 A：使用 FastAPI（被拒绝，因为对于简单的预览功能来说过于复杂）
- 方案 B：使用 Node.js + Express（被拒绝，因为增加技术栈复杂度）

### Decision 8: 版本控制方案

**选择**：直接使用 Git 作为版本控制后端，`.paperkit/memory/versions.json` 作为元数据索引。

**实现细节**：
- **版本快照**：使用 `git commit` + `git tag`
- **版本列表**：从 `git log` 和 `versions.json` 获取
- **版本对比**：使用 `git diff`
- **版本回滚**：使用 `git reset` 或 `git checkout`
- **元数据存储**：`versions.json` 记录快照的额外信息（描述、统计、完成度）

**versions.json 结构**：
```json
{
  "snapshots": [
    {
      "id": "abc123",
      "commit_hash": "abc123def456",
      "tag": "v1703577600-initial-draft",
      "timestamp": "2025-12-26T10:00:00Z",
      "description": "初稿完成",
      "stats": {
        "word_count": 5000,
        "section_count": 5,
        "citation_count": 20
      },
      "completion": {
        "completed_sections": 3,
        "total_sections": 5,
        "percentage": 60
      }
    }
  ]
}
```

**理由**：
- Git 是成熟的版本控制系统，无需重复造轮子
- Git 提供了强大的分支、合并、对比功能
- 用户可以使用标准的 Git 工具查看历史
- `versions.json` 提供额外的元数据，便于 AI 理解和展示

**替代方案**：
- 方案 A：自己实现版本控制系统（被拒绝，因为重复造轮子）
- 方案 B：使用数据库存储版本（被拒绝，因为增加复杂度）

### Decision 9: 智能文献导入方案

**选择**：统一的 `/paper.source.import` 命令，自动识别输入类型（DOI/URL/PDF/ISBN）并执行相应的导入流程。

**实现细节**：
- **输入识别**：
  - DOI：匹配正则表达式 `10\.\d{4,}/.*`
  - URL：匹配 `http(s)://` 开头
  - PDF：检查文件扩展名 `.pdf`
  - ISBN：匹配 ISBN-10 或 ISBN-13 格式
- **PDF 元数据提取**：
  - 使用 PyPDF2 或 pdfplumber 提取元数据
  - 优先从元数据中提取 DOI
  - 如果有 DOI，则调用 DOI 解析流程补全信息
  - 如果无 DOI，则使用元数据创建 BibTeX 条目
- **批量导入**：
  - 支持 `--batch` 参数扫描目录
  - 并行处理多个 PDF 文件
  - 显示进度条和导入结果

**目录结构**：
```
sources/
├─ references.bib          # 主 .bib 文件
├─ pdfs/                   # PDF 文件存储
│  ├─ smith2023diffusion.pdf
│  └─ wang2022survey.pdf
├─ texts/                  # 提取的文本内容
│  ├─ smith2023diffusion.txt
│  └─ wang2022survey.txt
├─ notes/                  # 文献笔记
│  ├─ smith2023diffusion.md
│  └─ wang2022survey.md
├─ snapshots/              # 网页快照
│  └─ blog2023example.html
└─ attachments/            # 其他附件
   └─ smith2023diffusion/
      ├─ supplementary.pdf
      └─ dataset.csv
```

**理由**：
- 统一的命令接口，降低用户学习成本
- 自动识别输入类型，减少用户操作步骤
- 支持多种文献来源，满足实际需求
- PDF 元数据提取提高导入准确性

**替代方案**：
- 方案 A：为每种类型提供单独的命令（被拒绝，因为增加复杂度）
- 方案 B：仅支持手动录入（被拒绝，因为不满足用户需求）

### Decision 10: AI 辅助文献阅读方案

**选择**：提取 PDF/网页文本内容，使用 AI 生成摘要和笔记，存储到 `sources/notes/` 目录。

**实现细节**：
- **文本提取**：
  - PDF：使用 PyPDF2 或 pdfplumber 提取文本
  - 网页：使用 BeautifulSoup 提取正文
  - 保存到 `sources/texts/<citation-key>.txt`
- **AI 摘要生成**：
  - 调用 AI（Claude/GPT）生成摘要
  - 摘要包含：研究问题、方法、主要发现、结论、局限性
  - 保存到 `sources/notes/<citation-key>.md`
- **笔记模板**：
```markdown
# [文献标题]

**作者**：[作者列表]
**年份**：[年份]
**来源**：[期刊/会议/网站]
**Citation Key**：[citation-key]

## 摘要
[AI 生成的摘要]

## 研究问题
[AI 提取的研究问题]

## 方法
[AI 提取的方法]

## 主要发现
[AI 提取的主要发现]

## 结论
[AI 提取的结论]

## 局限性
[AI 提取的局限性]

## 个人笔记
[用户手动添加的笔记]

## 引用建议
[AI 建议在论文中如何引用这篇文献]
```

**理由**：
- AI 能够快速阅读和理解大量文献
- 结构化的笔记便于后续引用和综述
- 文本提取后可以离线使用
- 笔记模板确保信息完整性

**替代方案**：
- 方案 A：不提供 AI 阅读功能（被拒绝，因为不满足用户需求）
- 方案 B：使用第三方文献管理服务（被拒绝，因为增加依赖）

### Decision 11: 分步式内容扩写系统 ⭐ 核心

**选择**：基于大纲结构的分段扩写，每次生成 500-1000 字，支持思路引导和渐进式完善。

**实现细节**：
- **扩写入口**：用户选择大纲中的某个标题，触发 `/paper.expand.section <section-id>`
- **上下文构建**：
  - 读取论文论点宪法（thesis.md）
  - 读取大纲结构（outline.json）
  - 读取已完成的相邻章节内容
  - 读取相关文献的摘要和笔记
- **生成策略**：
  - 单次生成 500-1000 字（避免一次性生成全文导致内容崩坏）
  - 生成内容自动插入 `[@placeholder]` 引用占位符
  - 提供 3 种扩写模式：快速草稿、标准模式、详细模式
- **思路引导**：`/paper.expand.guide <section-id>`
  - 分析该章节应该写什么
  - 提供写作步骤建议（如：先定义概念，再列举现状，最后分析问题）
  - 生成段落骨架（每段的主题句）
- **渐进式完善**：
  - 支持对已生成段落进行追问（`/paper.expand.follow-up`）
  - 支持补充细节、添加例子、深化论证
  - 保留修改历史，支持回滚

**命令设计**：
- `/paper.expand.section <section-id>` - 扩写指定章节
- `/paper.expand.guide <section-id>` - 获取写作思路引导
- `/paper.expand.follow-up` - 对上一段进行追问完善
- `/paper.expand.modes` - 查看/切换扩写模式

**理由**：
- 分段扩写避免 AI 生成长文时的内容崩坏问题
- 思路引导帮助学生理解论文写作逻辑
- 渐进式完善支持迭代优化

**替代方案**：
- 方案 A：一次性生成全文（被拒绝，因为容易导致内容重复、逻辑混乱）
- 方案 B：仅提供写作建议不生成内容（被拒绝，因为不满足目标用户需求）

### Decision 12: 学术化润色系统 ⭐ 核心

**选择**：多模式润色引擎，支持口语转书面语、降重改写、学术规范检查。

**实现细节**：
- **口语转书面语**：`/paper.polish.formalize`
  - 将口语化表达改写为学术用语
  - 保留原文语义，仅调整表达风格
  - 示例：「这个东西很有用」→「该机制具有显著的应用价值」
  - 示例：「我们发现」→「研究表明」
- **降重改写**：`/paper.polish.rewrite`
  - 三种改写强度：轻度（同义词替换）、中度（句式调整）、重度（语义重组）
  - 每次改写提供 2-3 个方案供选择
  - 显示预估查重率变化（基于文本相似度）
  - 保留原文对照，便于用户判断
- **学术规范检查**：`/paper.polish.check`
  - 检查人称使用（避免"我"、"我们"等第一人称）
  - 检查口语化词汇（"然后"、"所以"等）
  - 检查格式规范（数字、单位、标点）
  - 检查引用格式是否正确
  - 生成修改建议报告

**润色流程**：
1. 用户选择要润色的文本（段落或章节）
2. 选择润色模式（口语转书面语 / 降重改写 / 规范检查）
3. AI 生成润色结果
4. 用户选择接受、修改或拒绝
5. 应用修改并更新文件

**理由**：
- 口语转书面语解决学生学术表达能力不足的问题
- 降重改写解决查重率过高的痛点
- 学术规范检查帮助学生了解学术写作规范

**替代方案**：
- 方案 A：仅提供改写不保留原文（被拒绝，因为用户需要对比判断）
- 方案 B：使用第三方润色 API（被拒绝，因为增加成本和依赖）

### Decision 13: 联网引证系统 ⭐ 核心

**选择**：实时联网搜索真实文献，自动生成引用建议，验证引用真实性。

**实现细节**：
- **文献搜索**：`/paper.research.search <query>`
  - 搜索范围优先级：
    1. 知网、万方（中文学术文献）
    2. Google Scholar、Semantic Scholar（英文学术文献）
    3. 百度学术、必应学术（补充来源）
  - 返回前 10 条最相关结果
  - 显示标题、作者、年份、摘要、DOI/URL
  - 支持一键添加到 `.bib` 文件
- **引用建议**：`/paper.research.suggest <section-id>`
  - 分析当前章节内容
  - 识别需要引用支撑的论点
  - 自动搜索相关文献并提供引用建议
  - 生成引用位置和建议文献列表
- **引用验证**：`/paper.research.verify`
  - 检查所有 `[@key]` 引用的真实性
  - 验证 DOI/URL 是否有效
  - 检查引用内容与原文是否匹配
  - 标记可疑引用（可能是 AI 编造）
- **实时引证**：在扩写过程中自动搜索
  - AI 生成内容时，自动搜索相关文献
  - 插入 `[@placeholder:topic]` 占位符
  - 用户确认后替换为真实引用

**搜索 API 集成**：
- **知网/万方**：使用爬虫或第三方接口（需注意合规性）
- **Google Scholar**：使用 scholarly 库或 SerpAPI
- **Semantic Scholar**：使用官方 API（免费配额）
- **Crossref**：用于验证 DOI

**缓存策略**：
- 搜索结果缓存 7 天
- 文献元数据永久缓存
- 用户可手动刷新缓存

**理由**：
- 联网搜索确保引用的真实性，避免 AI 编造文献
- 引用建议帮助学生找到合适的参考文献
- 引用验证提供质量保障

**替代方案**：
- 方案 A：仅使用本地文献库（被拒绝，因为学生通常没有足够的文献积累）
- 方案 B：仅验证不搜索（被拒绝，因为不满足用户需求）

## Risks / Trade-offs

### Risk 1: Pandoc 依赖安装复杂

**风险**：Pandoc 和 TeX 引擎在不同平台的安装方式不同，用户可能遇到安装困难。

**缓解措施**：
- 提供详细的安装指引（针对 Linux、macOS、Windows）
- 在命令执行前检测依赖，给出明确的错误提示
- 提供 Docker 镜像（可选，后续版本）

### Risk 2: DOI/URL 解析不稳定

**风险**：DOI.org 和 Crossref API 可能限流、超时或返回不完整数据。

**缓解措施**：
- 实现缓存机制，避免重复请求
- 实现重试和退避策略
- 提供离线降级方案（手动粘贴 BibTeX）
- 记录解析警告，提示用户检查

### Risk 3: .bib 文件编辑冲突

**风险**：用户手动编辑 `.bib` 文件可能导致格式错误或与索引不一致。

**缓解措施**：
- 提供 `/paper.source.sync` 命令重建索引
- 提供 `/paper.bibliography.check` 命令检查 `.bib` 文件完整性
- 在解析时容错，记录警告而不是直接失败

### Risk 4: AI 生成引用的准确性

**风险**：AI 生成的文献综述或论证可能包含虚假引用或错误归因。

**缓解措施**：
- 提供 `/paper.citation.audit` 命令扫描引用
- 检查 `[@key]` 是否存在于 `.bib` 文件
- 检查关键结论是否有引用支撑
- 提示用户人工审查

### Risk 5: 通用用户群需求差异大

**风险**：本科生、研究生、科研人员对论文结构和规范的要求差异大。

**缓解措施**：
- 将结构和规范放入 `format profile`，而不是硬编码在命令中
- 提供多种预设模板（课程论文、学位论文、期刊论文）
- 允许用户自定义模板和样式

### Risk 6: Web 服务器端口冲突

**风险**：默认端口 5000 可能被其他应用占用。

**缓解措施**：
- 自动检测端口占用，尝试其他端口（5001、5002 等）
- 允许用户通过参数指定端口
- 提供清晰的错误提示

### Risk 7: 实时更新性能问题

**风险**：大型论文（100+ 章节）的实时渲染可能导致性能问题。

**缓解措施**：
- 实现增量渲染（仅重新渲染修改的章节）
- 添加防抖机制（文件修改后延迟 2 秒再渲染）
- 提供"暂停实时更新"选项

### Risk 8: Git 操作失败

**风险**：Git 操作可能失败（如冲突、权限问题）。

**缓解措施**：
- 在执行 Git 操作前检查状态
- 提供清晰的错误提示和恢复建议
- 实现"安全回滚"（先创建备份）
- 记录所有 Git 操作到日志文件

### Risk 9: PDF 文本提取质量问题

**风险**：PDF 文本提取可能失败或质量差（扫描版 PDF、复杂排版）。

**缓解措施**：
- 尝试多个 PDF 解析库（PyPDF2、pdfplumber、pdfminer）
- 对于扫描版 PDF，提示用户使用 OCR 工具
- 提供手动编辑提取文本的功能
- 记录提取质量评分，提示用户检查

### Risk 10: AI 摘要生成成本

**风险**：大量文献的 AI 摘要生成可能产生较高的 API 成本。

**缓解措施**：
- 提供本地模型选项（如 Ollama）
- 实现摘要缓存，避免重复生成
- 允许用户选择性生成摘要（仅对重要文献）
- 提供摘要质量评估，避免低质量摘要

### Risk 11: 批量导入性能问题

**风险**：批量导入大量 PDF 文件可能耗时较长。

**缓解措施**：
- 实现并行处理（多线程/多进程）
- 显示进度条和预计剩余时间
- 支持断点续传（记录已处理的文件）
- 提供"快速模式"（跳过文本提取和 AI 摘要）

### Risk 12: 联网引证的可靠性

**风险**：联网搜索可能返回不相关或低质量的文献。

**缓解措施**：
- 限制搜索范围（学术数据库优先：知网、万方、Google Scholar）
- 实现相关度排序和过滤
- 要求用户确认引用的文献
- 记录引用来源，便于追溯验证

### Risk 13: 降重效果不稳定

**风险**：AI 改写可能导致语义偏移或表达不自然。

**缓解措施**：
- 提供多种改写方案供用户选择
- 保留原文对照，便于用户判断
- 提供"保守模式"（仅替换同义词）和"激进模式"（重组句子结构）
- 提示用户检查改写后的语义准确性

## Migration Plan

### Phase 1: 核心架构迁移（Week 1-2）
1. 重命名项目和目录结构
2. 更新命令文件和脚本
3. 移除小说特有功能
4. 更新模板文件

### Phase 2: 文献管理系统（Week 3-4）
1. 实现 `.bib` 文件解析与索引
2. 实现 DOI/URL 自动解析
3. 实现文献条目管理命令
4. 实现缓存机制

### Phase 3: 大纲与章节系统（Week 5-6）
1. 实现大纲生成算法
2. 实现大纲可视化
3. 更新章节管理命令（适配学术写作）
4. 实现论文论点宪法（thesis）

### Phase 4: 引用与导出系统（Week 7-8）
1. 实现引用插入命令
2. 实现引用审计命令
3. 实现参考文献生成
4. 实现多格式导出（Markdown/Word/LaTeX/PDF）

### Phase 5: 测试与文档（Week 9-10）
1. 创建测试项目
2. 测试所有功能
3. 更新文档（README、命令文档、构建文档）
4. 发布 0.1.0 版本

### Rollback Plan
- 保留 NovelKit 的 Git 分支（`main-novelkit`）
- 如果改造失败，可以回退到 NovelKit
- 用户可以继续使用 NovelKit（通过旧版本）

## Open Questions

1. **是否需要支持多语言论文？**
   - 当前设计支持中文和英文，但其他语言（日语、韩语等）可能需要额外的字体和模板配置。

2. **是否需要支持协作编辑？**
   - 当前设计是单用户本地工作流，如果需要协作，可能需要集成 Git 或云存储。

3. **是否需要支持图表管理？**
   - 当前设计仅提供 `figures/` 和 `tables/` 目录，但没有专门的图表管理命令。后续可以考虑添加 `/paper.figure.*` 和 `/paper.table.*` 命令。

4. **是否需要支持数据管理？**
   - 科研论文可能需要管理实验数据、代码、结果等。后续可以考虑添加 `/paper.data.*` 命令。

5. **是否需要支持版本控制集成？**
   - 当前设计假设用户使用 Git 进行版本控制，但没有提供专门的 Git 集成命令。后续可以考虑添加 `/paper.git.*` 命令。
