# PaperKit Polish (学术化润色) Capability

## ADDED Requirements

### Requirement: 口语转书面语
系统 SHALL 支持将口语化表达改写为学术用语。

#### Scenario: 转换口语表达
- **WHEN** 用户执行 `/paper.polish.formalize`
- **THEN** 系统读取当前选中的文本或整个章节
- **AND** 识别口语化表达（如"这个东西很有用"）
- **AND** 改写为学术用语（如"该机制具有显著的应用价值"）
- **AND** 保留原文语义

#### Scenario: 常见口语转换示例
- **WHEN** 文本包含口语化表达
- **THEN** 系统进行以下转换：
  - 「我们发现」→「研究表明」
  - 「很多人认为」→「学界普遍认为」
  - 「这个方法不错」→「该方法具有较高的可行性」
  - 「问题很大」→「存在显著的问题」
  - 「然后」→「随后」
  - 「所以」→「因此」

#### Scenario: 批量转换
- **WHEN** 用户执行 `/paper.polish.formalize --section <section-id>`
- **THEN** 系统对整个章节进行口语转书面语转换
- **AND** 生成修改报告，列出所有转换项

### Requirement: 降重改写
系统 SHALL 支持对已有段落进行语义重组，降低查重率。

#### Scenario: 轻度改写
- **WHEN** 用户执行 `/paper.polish.rewrite --level light`
- **THEN** 系统仅进行同义词替换
- **AND** 保持原文句式结构不变
- **AND** 预估降低查重率 5-10%

#### Scenario: 中度改写
- **WHEN** 用户执行 `/paper.polish.rewrite --level medium`
- **THEN** 系统进行句式调整和同义词替换
- **AND** 调整句子顺序，拆分长句
- **AND** 预估降低查重率 15-25%

#### Scenario: 重度改写
- **WHEN** 用户执行 `/paper.polish.rewrite --level heavy`
- **THEN** 系统进行语义重组
- **AND** 完全重写段落，保留核心语义
- **AND** 预估降低查重率 30-50%

#### Scenario: 提供多个方案
- **WHEN** 用户执行降重改写
- **THEN** 系统提供 2-3 个改写方案
- **AND** 显示原文对照
- **AND** 用户可以选择接受某个方案

#### Scenario: 显示相似度预估
- **WHEN** 系统完成改写
- **THEN** 计算原文与改写后的文本相似度
- **AND** 显示预估查重率变化

### Requirement: 学术规范检查
系统 SHALL 检查论文是否符合学术写作规范。

#### Scenario: 人称检查
- **WHEN** 用户执行 `/paper.polish.check`
- **THEN** 系统扫描文本中的第一人称使用（我、我们、本文作者）
- **AND** 提示用户修改为第三人称或被动语态
- **AND** 提供修改建议

#### Scenario: 口语词汇检查
- **WHEN** 用户执行 `/paper.polish.check`
- **THEN** 系统识别口语化词汇（然后、所以、好像、差不多）
- **AND** 提供学术化替代词

#### Scenario: 格式规范检查
- **WHEN** 用户执行 `/paper.polish.check`
- **THEN** 系统检查以下格式规范：
  - 数字格式（阿拉伯数字 vs 汉字）
  - 单位格式（标准 SI 单位）
  - 标点使用（中文标点 vs 英文标点）
  - 引号使用（正确的引号配对）

#### Scenario: 引用格式检查
- **WHEN** 用户执行 `/paper.polish.check`
- **THEN** 系统检查引用格式是否符合所选样式（GB/T 7714、APA 等）
- **AND** 标记格式错误的引用

#### Scenario: 生成检查报告
- **WHEN** 用户执行 `/paper.polish.check`
- **THEN** 系统生成检查报告
- **AND** 按类别列出所有问题
- **AND** 提供修改建议
- **AND** 支持一键修复简单问题

### Requirement: 润色历史管理
系统 SHALL 保留润色历史，便于对比和回滚。

#### Scenario: 保留润色记录
- **WHEN** 用户完成一次润色操作
- **THEN** 系统保存润色记录到 `.paperkit/memory/polish-history.json`
- **AND** 记录时间、操作类型、原文、修改后文本

#### Scenario: 对比润色前后
- **WHEN** 用户执行 `/paper.polish.diff`
- **THEN** 系统显示润色前后的文本对比
- **AND** 高亮显示修改部分

#### Scenario: 回滚润色
- **WHEN** 用户对润色结果不满意
- **THEN** 用户可以执行 `/paper.polish.rollback`
- **AND** 系统恢复到润色前的内容

### Requirement: 润色范围选择
系统 SHALL 支持灵活选择润色范围。

#### Scenario: 润色选中文本
- **WHEN** 用户选中部分文本并执行润色命令
- **THEN** 系统仅对选中的文本进行润色

#### Scenario: 润色整个章节
- **WHEN** 用户执行 `/paper.polish.formalize --section <section-id>`
- **THEN** 系统对整个章节进行润色

#### Scenario: 润色全文
- **WHEN** 用户执行 `/paper.polish.check --all`
- **THEN** 系统对论文全文进行规范检查
- **AND** 生成全文检查报告
