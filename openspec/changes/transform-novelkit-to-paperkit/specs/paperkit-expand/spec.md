# PaperKit Expand (分步式内容扩写) Capability

## ADDED Requirements

### Requirement: 章节扩写
系统 SHALL 支持基于大纲标题的分步式内容扩写，每次生成 500-1000 字。

#### Scenario: 成功扩写章节
- **WHEN** 用户执行 `/paper.expand.section <section-id>`
- **THEN** 系统读取论文上下文（thesis、outline、相邻章节、相关文献）
- **AND** AI 生成 500-1000 字的章节内容
- **AND** 内容自动插入引用占位符 `[@placeholder:topic]`
- **AND** 将生成内容写入对应的 section 文件

#### Scenario: 选择扩写模式
- **WHEN** 用户执行 `/paper.expand.modes`
- **THEN** 系统显示三种扩写模式：快速草稿、标准模式、详细模式
- **AND** 用户可以选择切换模式

#### Scenario: 大纲中选择章节扩写
- **WHEN** 用户查看大纲并选择某个标题
- **THEN** 系统提示"是否扩写该章节？"
- **AND** 用户确认后执行扩写

### Requirement: 思路引导
系统 SHALL 提供写作思路引导，帮助学生理解该章节应该写什么。

#### Scenario: 获取写作思路
- **WHEN** 用户执行 `/paper.expand.guide <section-id>`
- **THEN** 系统分析该章节在论文中的位置和作用
- **AND** 生成写作步骤建议（如：先定义概念，再列举现状，最后分析问题）
- **AND** 生成段落骨架（每段的主题句）

#### Scenario: 不想让 AI 代写
- **WHEN** 用户表示想自己写但不知道写什么
- **THEN** 系统仅提供写作思路和结构建议
- **AND** 不生成具体内容

### Requirement: 渐进式完善
系统 SHALL 支持对已生成段落的追问、补充和修改。

#### Scenario: 追问完善
- **WHEN** 用户执行 `/paper.expand.follow-up`
- **THEN** 系统读取上一次生成的内容
- **AND** 用户可以提出追问（如：能否再详细解释一下 X 概念？）
- **AND** AI 生成补充内容并追加到章节

#### Scenario: 添加例子
- **WHEN** 用户执行 `/paper.expand.follow-up --add-example`
- **THEN** 系统为上一段内容添加具体的例子或案例
- **AND** 生成内容自然融入原文

#### Scenario: 深化论证
- **WHEN** 用户执行 `/paper.expand.follow-up --deepen`
- **THEN** 系统对上一段的论点进行深化论证
- **AND** 添加更多论据和分析

### Requirement: 上下文感知
系统 SHALL 在扩写时感知论文的整体上下文，确保内容连贯。

#### Scenario: 读取论文宪法
- **WHEN** 系统执行扩写
- **THEN** 首先读取 thesis.md 获取论文的核心论点
- **AND** 确保生成内容符合论文主题

#### Scenario: 读取相邻章节
- **WHEN** 系统执行扩写
- **THEN** 读取已完成的前后章节内容
- **AND** 确保内容衔接自然，避免重复

#### Scenario: 读取相关文献
- **WHEN** 系统执行扩写
- **THEN** 读取 sources-index.json 中与当前章节相关的文献
- **AND** 在生成内容中合理引用这些文献

### Requirement: 扩写历史管理
系统 SHALL 保留扩写历史，支持回滚和对比。

#### Scenario: 保留扩写历史
- **WHEN** 用户完成一次扩写
- **THEN** 系统保存扩写记录到 `.paperkit/memory/expand-history.json`
- **AND** 记录时间、章节 ID、扩写模式、生成内容

#### Scenario: 回滚扩写
- **WHEN** 用户对生成内容不满意
- **THEN** 用户可以执行 `/paper.expand.rollback`
- **AND** 系统恢复到扩写前的内容

#### Scenario: 对比扩写版本
- **WHEN** 用户执行 `/paper.expand.diff`
- **THEN** 系统显示当前内容与上一版本的对比
- **AND** 高亮显示变更部分
