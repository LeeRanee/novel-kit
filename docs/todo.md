# 开发进度

## 已完成 ✅

### 核心功能
- Writer 管理（创建、列表、查看、更新、切换）
- 章节生命周期（规划、写作、润色、评审、确认）
- 角色管理（CRUD）
- 地点管理（CRUD + 地图）
- 势力管理（CRUD + 成员 + 关系）
- 剧情管理（主线、支线、伏笔）
- 项目设置和宪法管理

### 基础设施
- CLI 工具（安装、初始化、远程下载）
- 构建系统（build.py）
- GitHub Actions 自动化发布

## 进行中 🟡

- 状态机扩展（current_character, current_location 等字段）
- 世界构建数据自动更新（chapter.confirm 时）

## 待实现 ⏳

### 世界构建
- `/novel.item.*` - 物品管理
- `/novel.worldview.*` - 世界观管理
- `/novel.rule.*` - 规则管理
- `/novel.relationship.*` - 关系网络管理

### 综合功能
- `/novel.search` - 全局搜索
- `/novel.stats` - 统计信息
- `/novel.overview` - 总览
- `/novel.consistency.check` - 一致性检查

### AI 环境扩展
- Claude Code 支持
- 其他 AI 环境支持
