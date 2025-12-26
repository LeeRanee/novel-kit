# PaperKit Version (版本控制系统) Capability

## ADDED Requirements

### Requirement: Git 集成
系统 SHALL 集成 Git 进行版本控制。

#### Scenario: 自动初始化 Git 仓库
- **WHEN** 用户执行 `/paper.setup`
- **THEN** 系统检查当前目录是否已是 Git 仓库
- **AND** 如果不是，则自动执行 `git init`
- **AND** 创建 `.gitignore` 文件（忽略 `.paperkit/cache/`、`exports/` 等）

#### Scenario: 检查 Git 状态
- **WHEN** 用户执行 `/paper.version.status`
- **THEN** 系统显示当前 Git 状态（已修改、已暂存、未跟踪的文件）
- **AND** 显示当前分支和最近的提交

### Requirement: 版本快照
系统 SHALL 提供版本快照功能，自动保存论文的重要版本。

#### Scenario: 创建版本快照
- **WHEN** 用户执行 `/paper.version.snapshot` 并提供描述
- **THEN** 系统执行 `git add .`
- **AND** 执行 `git commit -m "<描述>"`
- **AND** 创建 Git tag（格式：`v<timestamp>-<short-desc>`）
- **AND** 记录快照到 `.paperkit/memory/versions.json`

#### Scenario: 自动快照触发
- **WHEN** 用户完成某个重要操作（如完成章节、完成大纲）
- **THEN** 系统提示用户是否创建快照
- **AND** 如果用户同意，则自动创建快照

#### Scenario: 快照元数据
- **WHEN** 系统创建快照
- **THEN** 记录以下元数据：
  - 快照 ID（Git commit hash）
  - 快照时间
  - 快照描述
  - 论文统计（字数、章节数、引用数）
  - 完成度（已完成章节 / 总章节）

### Requirement: 版本列表
系统 SHALL 提供版本列表查看功能。

#### Scenario: 列出所有版本
- **WHEN** 用户执行 `/paper.version.list`
- **THEN** 系统显示所有版本快照
- **AND** 按时间倒序排列
- **AND** 显示每个版本的：ID、时间、描述、统计信息

#### Scenario: 过滤版本列表
- **WHEN** 用户执行 `/paper.version.list --since "2025-01-01"`
- **THEN** 系统仅显示指定日期之后的版本

### Requirement: 版本对比
系统 SHALL 提供版本对比功能。

#### Scenario: 对比两个版本
- **WHEN** 用户执行 `/paper.version.diff <version1> <version2>`
- **THEN** 系统使用 `git diff` 对比两个版本
- **AND** 显示文件级别的差异（哪些文件被修改、新增、删除）
- **AND** 显示内容级别的差异（具体修改了什么）

#### Scenario: 对比当前版本与历史版本
- **WHEN** 用户执行 `/paper.version.diff <version>`
- **THEN** 系统对比当前工作目录与指定版本
- **AND** 显示差异

### Requirement: 版本回滚
系统 SHALL 提供版本回滚功能。

#### Scenario: 回滚到指定版本
- **WHEN** 用户执行 `/paper.version.rollback <version>`
- **THEN** 系统提示用户确认（显示将要丢失的修改）
- **AND** 如果用户确认，则执行 `git reset --hard <version>`
- **AND** 更新工作目录到指定版本

#### Scenario: 安全回滚（创建备份）
- **WHEN** 用户执行 `/paper.version.rollback <version> --safe`
- **THEN** 系统先创建当前状态的快照（作为备份）
- **AND** 然后执行回滚
- **AND** 提示用户备份快照的 ID

### Requirement: 版本导出
系统 SHALL 支持导出指定版本的论文。

#### Scenario: 导出历史版本
- **WHEN** 用户执行 `/paper.version.export <version> --format pdf`
- **THEN** 系统切换到指定版本（临时）
- **AND** 生成 PDF 文件
- **AND** 恢复到当前版本
- **AND** 将 PDF 保存到 `exports/versions/<version>/`

### Requirement: 版本标签
系统 SHALL 支持为版本添加标签和注释。

#### Scenario: 添加版本标签
- **WHEN** 用户执行 `/paper.version.tag <version> <tag-name>`
- **THEN** 系统为指定版本创建 Git tag
- **AND** 更新 `.paperkit/memory/versions.json`

#### Scenario: 添加版本注释
- **WHEN** 用户执行 `/paper.version.annotate <version> <note>`
- **THEN** 系统将注释添加到 `.paperkit/memory/versions.json`
- **AND** 注释在版本列表中显示

### Requirement: 分支管理
系统 SHALL 支持 Git 分支管理。

#### Scenario: 创建实验分支
- **WHEN** 用户执行 `/paper.version.branch <branch-name>`
- **THEN** 系统执行 `git checkout -b <branch-name>`
- **AND** 提示用户已切换到新分支

#### Scenario: 切换分支
- **WHEN** 用户执行 `/paper.version.switch <branch-name>`
- **THEN** 系统执行 `git checkout <branch-name>`
- **AND** 更新工作目录

#### Scenario: 合并分支
- **WHEN** 用户执行 `/paper.version.merge <branch-name>`
- **THEN** 系统执行 `git merge <branch-name>`
- **AND** 如果有冲突，提示用户手动解决
- **AND** 如果无冲突，自动完成合并
