# Change: 将 NovelKit 改造为 PaperKit（论文写作工具）

## Why

NovelKit 当前专注于小说创作场景（角色、情节、世界观），但学术论文写作有完全不同的需求：文献管理、引用规范、论证结构、学术润色。用户需要一个通用的论文写作工具，支持从本科课程论文到科研期刊论文的全流程 AI 辅助。

## What Changes

### 核心架构保留
- ✅ 保留"命令 + 脚本 + 模板"的模块化架构
- ✅ 保留 slash 命令交互方式（`/paper.xxx`）
- ✅ 保留 Markdown 为主要写作格式

### 重命名与语义迁移
- **项目名称**：NovelKit → PaperKit
- **目录结构**：`.novelkit/` → `.paperkit/`
- **核心概念映射**：
  - `chapter` → `section`（章节）
  - `writer` → `format`（引文格式与排版风格）
  - `constitution` → `thesis`（论文论点宪法）
- **命令重命名**：40+ 个 `/novel.*` 命令 → `/paper.*` 命令

### 移除功能（小说特有）
- ❌ 角色管理（`character.*`）
- ❌ 地点管理（`location.*`）
- ❌ 势力管理（`faction.*`）
- ❌ 剧情管理（`plot.*`）

### 新增功能（论文特有）
- ✨ **分步式扩写**（`expand.*`）：按标题扩写段落（500-1000字）、写作思路引导、渐进式完善 ⭐ 核心
- ✨ **学术润色**（`polish.*`）：口语转书面语、降重改写、学术规范检查 ⭐ 核心
- ✨ **联网引证**（`research.*`）：自动搜索真实文献、实时引用验证、引用建议 ⭐ 核心
- ✨ **文献管理**（`source.*`）：.bib 文件管理、DOI/URL 自动解析、PDF 导入、文献检索
- ✨ **大纲生成**（`outline.*`）：基于论文类型生成标准结构（IMRAD 等）
- ✨ **引用管理**（`citation.*`）：引用插入、引用审计、参考文献生成（GB/T 7714、APA）
- ✨ **导出系统**（`export.*`）：支持 Markdown/Word/LaTeX/PDF 多格式导出
- ✨ **Web 预览系统**（`preview.*`）：实时预览论文全文、占位符管理、版本对比
- ✨ **版本控制**（`version.*`）：Git 集成、版本快照、版本回滚

### 技术栈增强
- 集成 Pandoc 作为转换引擎
- 集成 bibtexparser/pybtex 进行 .bib 解析
- 支持 CSL（Citation Style Language）引用格式
- 支持 DOI.org/Crossref API 进行文献解析
- 集成 Web 服务器（Flask/FastAPI）用于实时预览
- 集成 Git 进行版本控制

## Impact

### 破坏性变更（**BREAKING**）
- **命令命名空间完全变更**：所有 `/novel.*` 命令改为 `/paper.*`
- **目录结构变更**：`.novelkit/` → `.paperkit/`，`chapters/` → `sections/`，新增 `sources/`、`figures/`、`tables/`、`exports/`
- **数据模型变更**：移除角色/地点/势力/剧情相关的 JSON 数据结构
- **配置文件变更**：`writer` 配置改为 `format` 配置（引用风格 + 导出模板）

### 受影响的规格（Affected Specs）
- **新增 capability**：
  - `paperkit-core`：核心项目结构与命令系统
  - `paperkit-expand`：分步式内容扩写系统 ⭐ 核心
  - `paperkit-polish`：学术化润色系统 ⭐ 核心
  - `paperkit-research`：联网引证系统 ⭐ 核心
  - `paperkit-sources`：文献管理系统
  - `paperkit-outline`：大纲生成与管理
  - `paperkit-citation`：引用插入与参考文献生成
  - `paperkit-export`：多格式导出系统
  - `paperkit-section`：章节管理（从 NovelKit 的 chapter 演化）
  - `paperkit-format`：引文格式与排版风格管理（从 NovelKit 的 writer 演化）
  - `paperkit-thesis`：论文论点宪法（从 NovelKit 的 constitution 演化）
  - `paperkit-preview`：Web 预览系统
  - `paperkit-version`：版本控制系统

### 受影响的代码（Affected Code）
- `commands/*.md`：所有命令文件需要重写或重命名
- `scripts/bash/*.sh` 和 `scripts/powershell/*.ps1`：所有脚本需要适配新的目录结构和数据模型
- `templates/*.md`：所有模板需要从小说写作转换为学术写作
- `src/`：CLI 工具需要更新项目初始化逻辑
- `build_novelkit.py`：构建脚本需要重命名为 `build_paperkit.py`
- `README.md`、`pyproject.toml`、`package.json` 等：项目元信息需要更新

### 迁移路径
- 不提供从 NovelKit 到 PaperKit 的自动迁移工具（两者领域差异过大）
- 建议用户将 PaperKit 作为全新项目使用
- 可选：提供 `/paper.import.novelkit` 命令，尝试将 NovelKit 的 chapter 转换为 section（实验性功能）

## User Requirements

### 目标用户
- **主要用户**：基础较差的学生（本科生、专科生），需要 AI 深度辅助完成论文全文
- **次要用户**：研究生（学位论文初稿）、需要快速产出论文草稿的写作者

### 核心需求

#### 1. 分步式内容扩写（核心写作能力）
- **按标题扩写**：用户点击大纲中的某个小标题，AI 自动生成该段落的 500-1000 字草稿（避免一次生成全文导致内容崩坏）
- **思路引导**：如果学生想自己写，AI 提供"本段应该写什么"的写作提示（如：先定义概念，再列举现状，最后分析问题）
- **渐进式完善**：支持对已生成的段落进行追问、补充、修改

#### 2. 真实文献引用（解决学术硬伤）
- **联网引证**：AI 生成内容时，自动搜索并关联真实的互联网/学术数据库文献，避免"瞎编引用"
- **文献验证**：所有引用的文献都可追溯，提供 DOI/URL 链接
- **参考文献一键生成**：自动按国标（GB/T 7714）或 APA 格式生成文末的参考文献列表

#### 3. 学术化润色（语言提升）
- **口语转书面语**：学生输入大白话，AI 将其改写为学术用语（如将"这个东西很有用"改为"该机制具有显著的应用价值"）
- **降重/改写**：针对已有段落进行语义重组，降低查重率
- **学术规范检查**：检查格式、用词、表述是否符合学术规范

#### 4. 文献管理（基础能力）
- 内置 .bib 文件管理，支持手动添加和 DOI/URL 自动解析
- 支持 PDF 导入和元数据提取

#### 5. 导出与预览
- **写作格式**：使用 Markdown 进行写作
- **导出格式**：支持导出为 Markdown/Word/LaTeX/PDF
- **Web 预览**：实时预览论文全文，未完成章节显示占位符
- **版本控制**：支持版本快照、版本对比、版本回滚

### 最终目标
帮助基础较差的学生快速生成符合学术规范的完整论文全文，通过分步扩写、真实引用、学术润色三大核心能力，让学生能够产出一篇查重率低、引用真实、语言规范的论文
