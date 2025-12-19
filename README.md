# NovelKit

AI 辅助小说写作工具包，帮助作者使用 AI 助手创作长篇小说。

## 安装

```bash
pip install novel-kit-cli
```

或从源码安装：

```bash
git clone https://github.com/t59688/novel-kit.git
cd novel-kit
pip install -e .
```

## 快速开始

### 1. 初始化项目

```bash
novel-kit init my-novel
```

会交互式选择 AI 环境（当前支持 cursor），然后自动下载构建产物并初始化项目。

### 2. 项目设置

在 Cursor（或其他 AI 环境）中使用：

```
/novel.setup
```

这会创建必要的目录结构（`chapters/`, `world/`, `plots/` 等）。

### 3. 创建 Writer

Writer 定义了写作风格。创建第一个 writer：

```
/novel.writer.new
```

按提示交互式创建，或直接提供描述：

```
/novel.writer.new mystery thriller writer, fast-paced, third person limited
```

### 4. 开始写作

```
/novel.chapter.new    # 创建新章节
/novel.chapter.plan   # 规划章节
/novel.chapter.write  # 撰写章节
```

## 核心概念

### Writer

Writer 是写作风格配置，定义了：
- 叙述视角和时态
- 语言风格和节奏
- 角色发展和对话风格
- 世界构建方式

每个项目可以有多个 writer，通过 `/novel.writer.switch` 切换。

### 项目结构

```
my-novel/
├── .novelkit/          # 系统文件（自动管理）
│   ├── memory/         # 状态和配置
│   ├── templates/      # 模板文件
│   ├── scripts/        # 自动化脚本
│   ├── writers/        # Writer 配置
│   └── chapters/       # 章节元数据
├── .cursor/            # Cursor 命令（或其他 AI 环境）
├── chapters/           # 章节正文
├── world/              # 世界构建（角色、地点、势力等）
└── plots/              # 剧情数据
```

## 主要命令

### Writer 管理
- `/novel.writer.new` - 创建 writer
- `/novel.writer.list` - 列出所有 writers
- `/novel.writer.show <id>` - 查看 writer 详情
- `/novel.writer.switch <id>` - 切换活动 writer
- `/novel.writer.update <id> <changes>` - 更新 writer

### 章节管理
- `/novel.chapter.new` - 创建新章节
- `/novel.chapter.plan` - 规划章节内容
- `/novel.chapter.write` - 撰写章节
- `/novel.chapter.review` - 审查章节
- `/novel.chapter.polish` - 润色章节
- `/novel.chapter.confirm` - 确认章节完成

### 世界构建
- `/novel.character.new` - 创建角色
- `/novel.location.new` - 创建地点
- `/novel.faction.new` - 创建势力
- `/novel.plot.main.new` - 创建主线剧情
- `/novel.plot.side.new` - 创建支线剧情
- `/novel.plot.foreshadow.new` - 创建伏笔

### 项目设置
- `/novel.setup` - 初始化项目目录结构
- `/novel.constitution.create` - 创建小说宪法
- `/novel.constitution.check` - 检查是否符合宪法

完整命令列表见 `commands/` 目录。

## 工作原理

1. **命令文件**（`commands/*.md`）：定义 AI 命令的行为和交互流程
2. **脚本文件**（`scripts/bash/*.sh` 和 `scripts/powershell/*.ps1`）：执行实际的文件操作
3. **模板文件**（`templates/*.md`）：定义生成内容的结构

构建脚本（`build.py`）将这些文件打包成发布包，CLI 工具负责下载和初始化。

## 文档

- [贡献指南](CONTRIBUTING.md) - 如何参与开发
- [问题反馈](https://github.com/t59688/novel-kit/issues)
