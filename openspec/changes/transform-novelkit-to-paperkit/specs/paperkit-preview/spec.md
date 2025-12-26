# PaperKit Preview (Web 预览系统) Capability

## ADDED Requirements

### Requirement: Web 服务器启动
系统 SHALL 提供 `/paper.preview.start` 命令启动本地 Web 服务器。

#### Scenario: 成功启动 Web 服务器
- **WHEN** 用户执行 `/paper.preview.start`
- **THEN** 系统启动本地 Web 服务器（默认端口 5000）
- **AND** 在浏览器中自动打开预览页面
- **AND** 显示服务器地址（如 `http://localhost:5000`）

#### Scenario: 端口被占用时的处理
- **WHEN** 默认端口 5000 被占用
- **THEN** 系统尝试使用其他端口（5001、5002 等）
- **AND** 提示用户实际使用的端口

### Requirement: 论文全文预览
系统 SHALL 实时渲染论文全文，包括所有已完成和未完成的章节。

#### Scenario: 渲染已完成章节
- **WHEN** 用户访问预览页面
- **THEN** 系统合并所有 `sections/*.md` 文件
- **AND** 按照 `outline.json` 中定义的顺序排列
- **AND** 渲染为 HTML 格式
- **AND** 应用引用格式（Pandoc citeproc）

#### Scenario: 渲染未完成章节（占位符）
- **WHEN** 某个章节在 `outline.json` 中定义但文件不存在
- **THEN** 系统显示占位符：`[待撰写：<章节标题>]`
- **AND** 占位符使用特殊样式（灰色、虚线边框）

#### Scenario: 实时更新预览
- **WHEN** 用户修改 `sections/*.md` 文件
- **THEN** 系统自动检测文件变化（文件监听）
- **AND** 在 2 秒内刷新预览页面
- **AND** 保持用户当前的滚动位置

### Requirement: 版本对比视图
系统 SHALL 提供版本对比功能，允许用户在两个版本之间切换查看。

#### Scenario: 选择对比版本
- **WHEN** 用户点击"版本对比"按钮
- **THEN** 系统显示版本列表（从 Git 历史或版本快照）
- **AND** 用户可以选择两个版本进行对比

#### Scenario: 并排对比视图
- **WHEN** 用户选择两个版本
- **THEN** 系统以并排方式显示两个版本的论文全文
- **AND** 高亮显示差异部分（新增、删除、修改）
- **AND** 支持同步滚动

#### Scenario: 差异统计
- **WHEN** 用户查看版本对比
- **THEN** 系统显示差异统计：新增行数、删除行数、修改行数
- **AND** 显示变更的章节列表

### Requirement: 导航与目录
系统 SHALL 提供交互式目录导航。

#### Scenario: 显示目录树
- **WHEN** 用户访问预览页面
- **THEN** 系统在左侧显示目录树（基于 `outline.json`）
- **AND** 目录树包含所有章节和子章节
- **AND** 标记已完成和未完成的章节（不同图标）

#### Scenario: 点击目录跳转
- **WHEN** 用户点击目录中的某个章节
- **THEN** 页面滚动到对应章节
- **AND** 高亮显示当前章节

### Requirement: 样式与主题
系统 SHALL 提供多种预览样式和主题。

#### Scenario: 切换预览样式
- **WHEN** 用户点击"样式"按钮
- **THEN** 系统显示样式列表（学术论文、期刊模板、简洁模式等）
- **AND** 用户可以切换样式
- **AND** 样式立即应用到预览页面

#### Scenario: 切换主题
- **WHEN** 用户点击"主题"按钮
- **THEN** 系统显示主题列表（亮色、暗色）
- **AND** 用户可以切换主题
- **AND** 主题立即应用到预览页面

### Requirement: 导出预览
系统 SHALL 支持从预览页面直接导出论文。

#### Scenario: 从预览页面导出
- **WHEN** 用户在预览页面点击"导出"按钮
- **THEN** 系统显示导出格式选项（PDF、Word、LaTeX）
- **AND** 用户选择格式后，系统生成文件
- **AND** 自动下载到 `exports/` 目录
