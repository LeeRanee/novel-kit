# PaperKit Research (联网引证) Capability

## ADDED Requirements

### Requirement: 文献搜索
系统 SHALL 支持联网搜索真实学术文献。

#### Scenario: 关键词搜索
- **WHEN** 用户执行 `/paper.research.search <query>`
- **THEN** 系统在学术数据库中搜索相关文献
- **AND** 返回前 10 条最相关结果
- **AND** 显示标题、作者、年份、摘要、DOI/URL

#### Scenario: 搜索范围优先级
- **WHEN** 系统执行文献搜索
- **THEN** 按以下优先级搜索：
  1. 知网、万方（中文学术文献）
  2. Google Scholar、Semantic Scholar（英文学术文献）
  3. 百度学术、必应学术（补充来源）

#### Scenario: 一键添加文献
- **WHEN** 用户在搜索结果中选择某篇文献
- **THEN** 用户可以执行"添加到文献库"
- **AND** 系统自动获取文献元数据
- **AND** 生成 BibTeX 条目并添加到 `sources/references.bib`

#### Scenario: 高级搜索
- **WHEN** 用户执行 `/paper.research.search --author "张三" --year 2020-2024`
- **THEN** 系统按指定条件过滤搜索结果
- **AND** 支持作者、年份范围、期刊、会议等过滤条件

### Requirement: 引用建议
系统 SHALL 自动分析章节内容并提供引用建议。

#### Scenario: 生成引用建议
- **WHEN** 用户执行 `/paper.research.suggest <section-id>`
- **THEN** 系统分析该章节的内容
- **AND** 识别需要引用支撑的论点
- **AND** 自动搜索相关文献
- **AND** 生成引用位置和建议文献列表

#### Scenario: 标记需要引用的论点
- **WHEN** 系统分析章节内容
- **THEN** 识别以下需要引用的情况：
  - 事实性陈述（如"研究表明..."）
  - 数据引用（如"占比达到 80%"）
  - 他人观点（如"有学者认为..."）
  - 定义和概念（如"X 是指..."）

#### Scenario: 提供引用选项
- **WHEN** 系统为某个论点生成引用建议
- **THEN** 提供 3-5 篇候选文献
- **AND** 显示每篇文献与论点的相关度评分
- **AND** 用户可以选择接受某个引用

### Requirement: 引用验证
系统 SHALL 验证论文中所有引用的真实性。

#### Scenario: 验证 DOI 有效性
- **WHEN** 用户执行 `/paper.research.verify`
- **THEN** 系统检查 `.bib` 文件中所有条目的 DOI
- **AND** 验证 DOI 是否有效（能否解析到真实文献）
- **AND** 标记无效的 DOI

#### Scenario: 验证 URL 有效性
- **WHEN** 用户执行 `/paper.research.verify`
- **THEN** 系统检查所有 URL 是否可访问
- **AND** 标记失效的链接
- **AND** 建议替代链接（如 archive.org 快照）

#### Scenario: 检测可疑引用
- **WHEN** 用户执行 `/paper.research.verify`
- **THEN** 系统标记以下可疑引用：
  - 无法在任何数据库中找到的文献
  - 元数据不完整的条目
  - 年份或作者信息异常的条目
- **AND** 提示"该引用可能是 AI 编造，请人工核实"

#### Scenario: 生成验证报告
- **WHEN** 验证完成
- **THEN** 系统生成验证报告
- **AND** 按状态分类（有效、可疑、失效）
- **AND** 提供修复建议

### Requirement: 实时引证
系统 SHALL 在扩写过程中自动搜索并插入引用。

#### Scenario: 扩写时自动搜索
- **WHEN** AI 生成章节内容时遇到需要引用的论点
- **THEN** 系统自动在后台搜索相关文献
- **AND** 插入引用占位符 `[@placeholder:topic]`

#### Scenario: 替换引用占位符
- **WHEN** 用户执行 `/paper.research.resolve`
- **THEN** 系统扫描文档中的引用占位符
- **AND** 为每个占位符提供候选文献
- **AND** 用户确认后替换为真实引用 `[@citation-key]`

#### Scenario: 批量解析占位符
- **WHEN** 用户执行 `/paper.research.resolve --auto`
- **THEN** 系统自动为所有占位符选择最相关的文献
- **AND** 替换为真实引用
- **AND** 生成解析报告供用户审核

### Requirement: 搜索缓存
系统 SHALL 缓存搜索结果以提高性能和可靠性。

#### Scenario: 缓存搜索结果
- **WHEN** 用户执行文献搜索
- **THEN** 系统将搜索结果缓存到 `.paperkit/cache/research/`
- **AND** 缓存有效期为 7 天

#### Scenario: 使用缓存结果
- **WHEN** 用户搜索相同关键词
- **THEN** 系统优先返回缓存结果
- **AND** 显示"结果来自缓存，上次更新于 X"

#### Scenario: 刷新缓存
- **WHEN** 用户执行 `/paper.research.search <query> --refresh`
- **THEN** 系统忽略缓存，重新搜索
- **AND** 更新缓存内容

### Requirement: 参考文献格式
系统 SHALL 支持多种参考文献格式。

#### Scenario: 生成 GB/T 7714 格式
- **WHEN** 用户配置引文格式为 GB/T 7714
- **THEN** 参考文献按国标格式生成
- **AND** 示例：[1] 作者. 标题[J]. 期刊名, 年份, 卷(期): 页码.

#### Scenario: 生成 APA 格式
- **WHEN** 用户配置引文格式为 APA
- **THEN** 参考文献按 APA 格式生成
- **AND** 示例：Author, A. A. (Year). Title. Journal, Volume(Issue), Pages.

#### Scenario: 一键生成参考文献列表
- **WHEN** 用户执行 `/paper.bibliography.generate`
- **THEN** 系统扫描论文中所有引用
- **AND** 按选定格式生成参考文献列表
- **AND** 输出到 `references.md` 文件

### Requirement: 引用审计
系统 SHALL 审计论文中的引用使用情况。

#### Scenario: 检查未使用的文献
- **WHEN** 用户执行 `/paper.citation.audit`
- **THEN** 系统检查 `.bib` 文件中哪些文献未被引用
- **AND** 提示用户是否删除未使用的条目

#### Scenario: 检查悬空引用
- **WHEN** 用户执行 `/paper.citation.audit`
- **THEN** 系统检查论文中哪些 `[@key]` 在 `.bib` 中找不到
- **AND** 标记悬空引用
- **AND** 建议添加对应文献

#### Scenario: 引用分布统计
- **WHEN** 用户执行 `/paper.citation.audit`
- **THEN** 系统统计每个章节的引用数量
- **AND** 标记引用过少的章节（可能需要补充文献支撑）
