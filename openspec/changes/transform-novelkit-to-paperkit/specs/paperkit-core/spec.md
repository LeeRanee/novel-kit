# PaperKit Core Capability

## ADDED Requirements

### Requirement: 项目初始化
系统 SHALL 提供 `/paper.setup` 命令来初始化论文项目结构。

#### Scenario: 成功初始化新项目
- **WHEN** 用户在空目录中执行 `/paper.setup`
- **THEN** 系统创建以下目录结构：
  - `.paperkit/memory/`（状态和配置）
  - `.paperkit/templates/`（模板文件）
  - `.paperkit/scripts/`（脚本文件）
  - `.paperkit/formats/`（引文格式配置）
  - `.paperkit/sections/`（章节元数据）
  - `.paperkit/cache/`（缓存）
  - `sections/`（论文正文）
  - `sources/`（文献与附件）
  - `figures/`（图片）
  - `tables/`（表格）
  - `appendices/`（附录）
  - `exports/`（导出产物）
- **AND** 创建初始配置文件 `.paperkit/memory/config.json`
- **AND** 创建 `.gitignore` 文件（忽略 `.paperkit/cache/` 和 `exports/`）

#### Scenario: 项目已存在时的处理
- **WHEN** 用户在已有 `.paperkit/` 目录的项目中执行 `/paper.setup`
- **THEN** 系统提示"项目已初始化"
- **AND** 询问用户是否重新初始化（会覆盖现有配置）

### Requirement: 命令命名规范
所有 PaperKit 命令 SHALL 遵循 `/paper.<module>.<action>` 格式。

#### Scenario: 命令命名一致性
- **WHEN** 用户查看命令列表
- **THEN** 所有命令都以 `/paper.` 开头
- **AND** 命令按模块分组（setup、thesis、format、section、source、outline、citation、bibliography、export）

### Requirement: 跨平台兼容性
系统 SHALL 支持 Linux、macOS 和 Windows 平台。

#### Scenario: Linux 平台执行
- **WHEN** 用户在 Linux 系统上执行命令
- **THEN** 系统使用 Bash 脚本执行操作

#### Scenario: Windows 平台执行
- **WHEN** 用户在 Windows 系统上执行命令
- **THEN** 系统使用 PowerShell 脚本执行操作

#### Scenario: macOS 平台执行
- **WHEN** 用户在 macOS 系统上执行命令
- **THEN** 系统使用 Bash 脚本执行操作

### Requirement: 状态管理
系统 SHALL 维护项目状态，包括当前章节、当前格式、统计信息等。

#### Scenario: 读取项目状态
- **WHEN** 用户执行任何命令
- **THEN** 系统从 `.paperkit/memory/config.json` 读取当前状态
- **AND** 状态包括：`current_format`、`current_section`、`word_count`、`citation_count`、`last_modified`

#### Scenario: 更新项目状态
- **WHEN** 用户完成某个操作（如撰写章节、添加文献）
- **THEN** 系统更新 `.paperkit/memory/config.json`
- **AND** 记录操作时间戳

### Requirement: 错误处理
系统 SHALL 提供清晰的错误提示和恢复建议。

#### Scenario: 依赖缺失时的提示
- **WHEN** 用户执行需要 Pandoc 的命令，但 Pandoc 未安装
- **THEN** 系统提示"Pandoc 未安装"
- **AND** 提供针对当前操作系统的安装指引

#### Scenario: 文件不存在时的提示
- **WHEN** 用户尝试操作不存在的文件
- **THEN** 系统提示"文件不存在：<file_path>"
- **AND** 建议用户检查文件路径或使用相关命令创建文件
