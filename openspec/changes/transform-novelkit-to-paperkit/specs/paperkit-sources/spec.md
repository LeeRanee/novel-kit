# PaperKit Sources (文献管理) Capability

## ADDED Requirements

### Requirement: .bib 文件管理
系统 SHALL 支持 `.bib` 文件的解析、索引和管理。

#### Scenario: 解析 .bib 文件
- **WHEN** 用户执行 `/paper.source.sync`
- **THEN** 系统解析 `sources/references.bib` 文件
- **AND** 生成 `.paperkit/memory/sources-index.json` 索引文件
- **AND** 索引包含：`citation_key`、`doi`、`url`、`title`、`authors`、`year`、`tags`

#### Scenario: 处理解析错误
- **WHEN** `.bib` 文件包含格式错误
- **THEN** 系统记录警告到 `parse_warnings` 字段
- **AND** 继续解析其他条目
- **AND** 提示用户检查有问题的条目

### Requirement: DOI 自动解析
系统 SHALL 支持通过 DOI 自动获取文献信息。

#### Scenario: 成功解析 DOI
- **WHEN** 用户执行 `/paper.source.import` 并提供有效的 DOI
- **THEN** 系统调用 doi.org API 获取 CSL JSON 格式的文献信息
- **AND** 将信息转换为 BibTeX 格式
- **AND** 追加到 `sources/references.bib` 文件
- **AND** 更新索引文件

#### Scenario: DOI 解析失败时的降级
- **WHEN** DOI 解析失败（网络错误或 DOI 不存在）
- **THEN** 系统提示用户手动粘贴 BibTeX 条目
- **AND** 提供手动录入的交互式界面

#### Scenario: DOI 解析结果缓存
- **WHEN** 用户解析同一个 DOI 多次
- **THEN** 系统从 `.paperkit/cache/doi/<doi-hash>.json` 读取缓存
- **AND** 避免重复请求 API

### Requirement: URL 自动解析
系统 SHALL 支持通过 URL 自动获取文献信息。

#### Scenario: 从 URL 提取 DOI
- **WHEN** 用户提供的 URL 包含 DOI
- **THEN** 系统提取 DOI 并走 DOI 解析流程

#### Scenario: 解析 arXiv URL
- **WHEN** 用户提供 arXiv URL（如 `https://arxiv.org/abs/2301.12345`）
- **THEN** 系统调用 arXiv API 获取文献信息
- **AND** 生成 BibTeX 条目（`@article` 类型）

#### Scenario: 通用 HTML 元数据提取
- **WHEN** URL 不属于已知站点
- **THEN** 系统提取 HTML 页面的元数据（`citation_*`、`dc.*`、schema.org）
- **AND** 生成 `@misc` 类型的 BibTeX 条目
- **AND** 提示用户补齐缺失字段

### Requirement: Citation Key 生成
系统 SHALL 自动生成唯一的 citation key。

#### Scenario: 生成标准 citation key
- **WHEN** 系统从 DOI/URL 获取文献信息
- **THEN** 生成格式为 `familyYYYYshorttitle` 的 citation key
- **AND** 例如：`smith2023diffusion`

#### Scenario: 处理 citation key 冲突
- **WHEN** 生成的 citation key 已存在
- **THEN** 系统追加后缀 `a/b/c`
- **AND** 例如：`smith2023diffusiona`

### Requirement: 文献检索
系统 SHALL 支持按多种条件检索文献。

#### Scenario: 按作者检索
- **WHEN** 用户执行 `/paper.source.list --author "Smith"`
- **THEN** 系统返回所有作者包含"Smith"的文献

#### Scenario: 按年份检索
- **WHEN** 用户执行 `/paper.source.list --year 2023`
- **THEN** 系统返回所有 2023 年发表的文献

#### Scenario: 按标签检索
- **WHEN** 用户执行 `/paper.source.list --tag "related-work"`
- **THEN** 系统返回所有标记为"related-work"的文献

### Requirement: 文献完整性检查
系统 SHALL 提供文献完整性检查功能。

#### Scenario: 检查必填字段
- **WHEN** 用户执行 `/paper.bibliography.check`
- **THEN** 系统检查每个条目的必填字段（title、author、year）
- **AND** 报告缺失字段的条目

#### Scenario: 检查重复 DOI
- **WHEN** 用户执行 `/paper.bibliography.check`
- **THEN** 系统检查是否存在重复的 DOI
- **AND** 报告重复的条目

#### Scenario: 检查重复 citation key
- **WHEN** 用户执行 `/paper.bibliography.check`
- **THEN** 系统检查是否存在重复的 citation key
- **AND** 报告重复的条目

### Requirement: 智能导入
系统 SHALL 支持智能识别文献类型并自动导入。

#### Scenario: 自动识别 DOI
- **WHEN** 用户执行 `/paper.source.import <input>` 且 input 是 DOI 格式
- **THEN** 系统自动识别为 DOI 并调用 DOI 解析流程

#### Scenario: 自动识别 URL
- **WHEN** 用户执行 `/paper.source.import <input>` 且 input 是 URL 格式
- **THEN** 系统自动识别为 URL 并调用 URL 解析流程

#### Scenario: 自动识别 PDF 文件
- **WHEN** 用户执行 `/paper.source.import <file-path>` 且文件是 PDF
- **THEN** 系统提取 PDF 元数据（标题、作者、DOI 等）
- **AND** 如果 PDF 包含 DOI，则调用 DOI 解析流程补全信息
- **AND** 如果无 DOI，则使用提取的元数据创建 BibTeX 条目
- **AND** 将 PDF 文件复制到 `sources/pdfs/` 目录

#### Scenario: 自动识别 ISBN
- **WHEN** 用户执行 `/paper.source.import <input>` 且 input 是 ISBN 格式
- **THEN** 系统调用图书 API（如 Google Books API）获取书籍信息
- **AND** 生成 `@book` 类型的 BibTeX 条目

#### Scenario: 批量导入
- **WHEN** 用户执行 `/paper.source.import --batch <file-or-dir>`
- **THEN** 系统扫描文件或目录中的所有 PDF 文件
- **AND** 对每个 PDF 执行智能导入流程
- **AND** 显示导入进度和结果

### Requirement: PDF 文件管理
系统 SHALL 支持 PDF 文件的存储、索引和内容提取。

#### Scenario: 存储 PDF 文件
- **WHEN** 用户导入包含 PDF 的文献
- **THEN** 系统将 PDF 复制到 `sources/pdfs/<citation-key>.pdf`
- **AND** 在 BibTeX 条目中记录 PDF 路径
- **AND** 在索引中记录 PDF 附件信息

#### Scenario: 提取 PDF 文本内容
- **WHEN** 用户执行 `/paper.source.extract <citation-key>`
- **THEN** 系统使用 PDF 解析库（如 PyPDF2 或 pdfplumber）提取文本
- **AND** 将文本保存到 `sources/texts/<citation-key>.txt`
- **AND** 更新索引，标记文本已提取

#### Scenario: 提取 PDF 元数据
- **WHEN** 系统导入 PDF 文件
- **THEN** 系统提取 PDF 元数据（标题、作者、创建日期、DOI）
- **AND** 如果元数据中包含 DOI，则尝试从 DOI 获取完整信息
- **AND** 如果无 DOI，则使用元数据创建 BibTeX 条目

### Requirement: 文献笔记系统
系统 SHALL 支持为每篇文献创建和管理笔记。

#### Scenario: 创建文献笔记
- **WHEN** 用户执行 `/paper.source.note <citation-key>`
- **THEN** 系统创建 `sources/notes/<citation-key>.md` 文件
- **AND** 使用模板填充笔记（标题、作者、摘要、关键点等）
- **AND** 在编辑器中打开笔记文件

#### Scenario: AI 生成文献摘要
- **WHEN** 用户执行 `/paper.source.summarize <citation-key>`
- **THEN** 系统读取文献的文本内容（从 PDF 或 URL）
- **AND** 调用 AI 生成摘要（关键观点、方法、结论）
- **AND** 将摘要写入 `sources/notes/<citation-key>.md`

#### Scenario: 查看文献笔记
- **WHEN** 用户执行 `/paper.source.show <citation-key> --with-notes`
- **THEN** 系统显示文献的基本信息
- **AND** 显示文献笔记内容（如果存在）

### Requirement: 附件管理
系统 SHALL 支持多种类型的附件管理。

#### Scenario: 添加附件
- **WHEN** 用户执行 `/paper.source.attach <citation-key> <file-path>`
- **THEN** 系统将文件复制到 `sources/attachments/<citation-key>/`
- **AND** 在索引中记录附件信息（文件名、类型、大小）

#### Scenario: 列出附件
- **WHEN** 用户执行 `/paper.source.show <citation-key> --attachments`
- **THEN** 系统列出该文献的所有附件
- **AND** 显示附件类型、大小、路径

#### Scenario: 打开附件
- **WHEN** 用户执行 `/paper.source.open <citation-key> --attachment <filename>`
- **THEN** 系统使用默认应用打开附件文件

### Requirement: 网页快照
系统 SHALL 支持保存网页文献的快照。

#### Scenario: 保存网页快照
- **WHEN** 用户导入 URL 类型的文献
- **THEN** 系统提示用户是否保存网页快照
- **AND** 如果用户同意，则下载网页 HTML 并保存到 `sources/snapshots/<citation-key>.html`
- **AND** 在索引中记录快照路径

#### Scenario: 查看网页快照
- **WHEN** 用户执行 `/paper.source.show <citation-key> --snapshot`
- **THEN** 系统在浏览器中打开保存的网页快照

### Requirement: 文献类型支持
系统 SHALL 支持多种文献类型的管理。

#### Scenario: 支持的文献类型
- **WHEN** 用户导入文献
- **THEN** 系统支持以下类型：
  - `@article`：期刊论文
  - `@book`：书籍
  - `@inproceedings`：会议论文
  - `@phdthesis`：博士论文
  - `@mastersthesis`：硕士论文
  - `@techreport`：技术报告
  - `@misc`：其他类型（网页、博客等）

#### Scenario: 类型特定字段验证
- **WHEN** 用户执行 `/paper.bibliography.check`
- **THEN** 系统根据文献类型检查必填字段
- **AND** 例如：`@article` 需要 journal、volume、pages
- **AND** 例如：`@book` 需要 publisher、isbn

