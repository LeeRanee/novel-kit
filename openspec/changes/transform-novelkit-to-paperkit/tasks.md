# Implementation Tasks

## 1. 项目重命名与基础设施
- [ ] 1.1 将项目名称从 NovelKit 改为 PaperKit
- [ ] 1.2 更新 `pyproject.toml`：包名、版本、描述、关键词
- [ ] 1.3 更新 `README.md`：项目介绍、功能列表、使用说明
- [ ] 1.4 重命名 `build_novelkit.py` → `build_paperkit.py`
- [ ] 1.5 更新 `build-config.json`：构建配置
- [ ] 1.6 更新 `.github/workflows/`：CI/CD 配置

## 2. 目录结构改造
- [ ] 2.1 创建新的目录结构模板（`.paperkit/` 替代 `.novelkit/`）
- [ ] 2.2 设计 `sections/`、`sources/`、`figures/`、`tables/`、`exports/` 目录
- [ ] 2.3 更新 CLI 初始化逻辑（`src/` 中的项目初始化代码）
- [ ] 2.4 创建 `.paperkit/memory/` 数据结构（config.json、thesis.md、outline.json、sources-index.json）

## 3. 命令系统改造
- [ ] 3.1 重命名核心命令文件（`commands/` 目录）
  - [ ] 3.1.1 `novel-setup.md` → `paper-setup.md`
  - [ ] 3.1.2 `novel-constitution-*.md` → `paper-thesis-*.md`（4 个文件）
  - [ ] 3.1.3 `novel-writer-*.md` → `paper-format-*.md`（5 个文件）
  - [ ] 3.1.4 `novel-chapter-*.md` → `paper-section-*.md`（6 个文件）
- [ ] 3.2 移除小说特有命令文件
  - [ ] 3.2.1 删除 `novel-character-*.md`（4 个文件）
  - [ ] 3.2.2 删除 `novel-location-*.md`（5 个文件）
  - [ ] 3.2.3 删除 `novel-faction-*.md`（6 个文件）
  - [ ] 3.2.4 删除 `novel-plot-*.md`（10 个文件）
- [ ] 3.3 创建新的论文特有命令文件
  - [ ] 3.3.1 创建 `paper-source-*.md`（6 个文件：add、import、list、show、update、sync）
  - [ ] 3.3.2 创建 `paper-outline-*.md`（3 个文件：generate、show、update）
  - [ ] 3.3.3 创建 `paper-citation-*.md`（3 个文件：search、insert、audit）
  - [ ] 3.3.4 创建 `paper-bibliography-*.md`（2 个文件：generate、check）
  - [ ] 3.3.5 创建 `paper-export.md`（1 个文件）

## 4. 脚本系统改造
- [ ] 4.1 更新 Bash 脚本（`scripts/bash/` 目录）
  - [ ] 4.1.1 适配新的目录结构（`.paperkit/`、`sections/`、`sources/` 等）
  - [ ] 4.1.2 更新状态管理脚本（读写 config.json、outline.json 等）
  - [ ] 4.1.3 移除角色/地点/势力/剧情相关脚本
- [ ] 4.2 更新 PowerShell 脚本（`scripts/powershell/` 目录）
  - [ ] 4.2.1 适配新的目录结构
  - [ ] 4.2.2 更新状态管理脚本
  - [ ] 4.2.3 移除角色/地点/势力/剧情相关脚本
- [ ] 4.3 创建 Python 脚本（`scripts/python/` 目录，新增）
  - [ ] 4.3.1 创建 `.bib` 文件解析脚本（使用 bibtexparser 或 pybtex）
  - [ ] 4.3.2 创建 DOI 解析脚本（调用 doi.org 和 Crossref API）
  - [ ] 4.3.3 创建 URL 解析脚本（提取元数据）
  - [ ] 4.3.4 创建引用审计脚本（扫描 Markdown 中的 `[@key]`）
  - [ ] 4.3.5 创建 Pandoc 导出脚本（Markdown → Word/LaTeX/PDF）

## 5. 模板系统改造
- [ ] 5.1 更新核心模板（`templates/` 目录）
  - [ ] 5.1.1 `chapter.md` → `section.md`（学术章节模板）
  - [ ] 5.1.2 `constitution.md` → `thesis.md`（论文论点宪法模板）
  - [ ] 5.1.3 `writer.md` → `format.md`（引文格式与排版风格模板）
- [ ] 5.2 移除小说特有模板
  - [ ] 5.2.1 删除 `character.md`
  - [ ] 5.2.2 删除 `location.md`
  - [ ] 5.2.3 删除 `faction.md`
  - [ ] 5.2.4 删除 `plot.md`
- [ ] 5.3 创建新的论文特有模板
  - [ ] 5.3.1 创建 `paper.md`（论文主文档骨架）
  - [ ] 5.3.2 创建 `source-note.md`（文献阅读笔记模板）
  - [ ] 5.3.3 创建 `outline.md`（大纲模板）
  - [ ] 5.3.4 创建 CSL 引用格式文件（APA、IEEE、GB7714 等）
  - [ ] 5.3.5 创建 Word 参考模板（`reference.docx`）
  - [ ] 5.3.6 创建 LaTeX 模板（`template.tex`）

## 6. 文献管理系统实现
- [ ] 6.1 实现 `.bib` 文件解析与索引
  - [ ] 6.1.1 解析 `sources/references.bib` 并生成 `sources-index.json`
  - [ ] 6.1.2 支持多个 `.bib` 文件
  - [ ] 6.1.3 处理解析错误和警告
- [ ] 6.2 实现 DOI/URL 自动解析
  - [ ] 6.2.1 DOI 解析（doi.org + Crossref API）
  - [ ] 6.2.2 URL 解析（arXiv、PubMed、ACM、IEEE 等）
  - [ ] 6.2.3 通用 HTML 元数据提取
  - [ ] 6.2.4 缓存机制（`.paperkit/cache/doi/` 和 `.paperkit/cache/url/`）
- [ ] 6.3 实现文献条目管理
  - [ ] 6.3.1 添加文献条目（手动或自动）
  - [ ] 6.3.2 更新文献条目
  - [ ] 6.3.3 删除文献条目
  - [ ] 6.3.4 文献检索（按作者、标题、年份、标签）
- [ ] 6.4 实现 Citation Key 生成算法（`familyYYYYshorttitle`）

## 7. 大纲生成系统实现
- [ ] 7.1 设计大纲数据结构（`outline.json`）
- [ ] 7.2 实现大纲生成算法
  - [ ] 7.2.1 IMRAD 结构模板（Introduction、Methods、Results、Discussion）
  - [ ] 7.2.2 学位论文结构模板
  - [ ] 7.2.3 综述论文结构模板
- [ ] 7.3 实现大纲可视化（树形展示）
- [ ] 7.4 实现大纲更新与同步（自动创建/更新 section 文件）

## 8. 引用管理系统实现
- [ ] 8.1 实现引用插入
  - [ ] 8.1.1 文献检索（从 `sources-index.json`）
  - [ ] 8.1.2 生成 Pandoc 引用语法（`[@key]`、`[@key1; @key2]`、`[@key, p. 12]`）
- [ ] 8.2 实现引用审计
  - [ ] 8.2.1 扫描 Markdown 文件中的 `[@key]`
  - [ ] 8.2.2 检查 key 是否存在于 `.bib` 文件
  - [ ] 8.2.3 检查未使用的文献
  - [ ] 8.2.4 检查重复引用
- [ ] 8.3 实现参考文献生成
  - [ ] 8.3.1 生成 `references.md`（预览用）
  - [ ] 8.3.2 集成 Pandoc citeproc（导出时自动生成）

## 9. 导出系统实现
- [ ] 9.1 集成 Pandoc
  - [ ] 9.1.1 检测 Pandoc 安装
  - [ ] 9.1.2 提供安装指引
- [ ] 9.2 实现 Markdown 导出（合并所有 section）
- [ ] 9.3 实现 Word 导出
  - [ ] 9.3.1 使用 `reference.docx` 样式模板
  - [ ] 9.3.2 集成 CSL 引用格式
- [ ] 9.4 实现 LaTeX 导出
  - [ ] 9.4.1 使用 `template.tex` 模板
  - [ ] 9.4.2 处理中文支持（xelatex）
- [ ] 9.5 实现 PDF 导出
  - [ ] 9.5.1 检测 TeX 引擎安装
  - [ ] 9.5.2 提供安装指引
  - [ ] 9.5.3 支持 xelatex/pdflatex

## 10. 分步式内容扩写系统实现 ⭐ 核心
- [ ] 10.1 实现章节扩写命令（`/paper.expand.section`）
  - [ ] 10.1.1 实现上下文构建（读取 thesis、outline、相邻章节、文献）
  - [ ] 10.1.2 实现 AI 内容生成（500-1000 字）
  - [ ] 10.1.3 实现引用占位符插入（`[@placeholder:topic]`）
  - [ ] 10.1.4 实现三种扩写模式（快速草稿、标准、详细）
- [ ] 10.2 实现思路引导命令（`/paper.expand.guide`）
  - [ ] 10.2.1 分析章节在论文中的位置和作用
  - [ ] 10.2.2 生成写作步骤建议
  - [ ] 10.2.3 生成段落骨架（主题句）
- [ ] 10.3 实现渐进式完善命令（`/paper.expand.follow-up`）
  - [ ] 10.3.1 实现追问完善
  - [ ] 10.3.2 实现添加例子（`--add-example`）
  - [ ] 10.3.3 实现深化论证（`--deepen`）
- [ ] 10.4 实现扩写历史管理
  - [ ] 10.4.1 保存扩写记录到 `expand-history.json`
  - [ ] 10.4.2 实现回滚功能
  - [ ] 10.4.3 实现版本对比
- [ ] 10.5 创建扩写命令文件
  - [ ] 10.5.1 创建 `paper-expand-section.md`
  - [ ] 10.5.2 创建 `paper-expand-guide.md`
  - [ ] 10.5.3 创建 `paper-expand-follow-up.md`
  - [ ] 10.5.4 创建 `paper-expand-modes.md`

## 11. 学术化润色系统实现 ⭐ 核心
- [ ] 11.1 实现口语转书面语命令（`/paper.polish.formalize`）
  - [ ] 11.1.1 构建口语-学术用语转换规则库
  - [ ] 11.1.2 实现 AI 改写引擎
  - [ ] 11.1.3 实现批量转换（整章节）
- [ ] 11.2 实现降重改写命令（`/paper.polish.rewrite`）
  - [ ] 11.2.1 实现轻度改写（同义词替换）
  - [ ] 11.2.2 实现中度改写（句式调整）
  - [ ] 11.2.3 实现重度改写（语义重组）
  - [ ] 11.2.4 实现多方案生成与选择
  - [ ] 11.2.5 实现相似度预估
- [ ] 11.3 实现学术规范检查命令（`/paper.polish.check`）
  - [ ] 11.3.1 实现人称检查
  - [ ] 11.3.2 实现口语词汇检查
  - [ ] 11.3.3 实现格式规范检查（数字、单位、标点）
  - [ ] 11.3.4 实现引用格式检查
  - [ ] 11.3.5 生成检查报告
- [ ] 11.4 实现润色历史管理
  - [ ] 11.4.1 保存润色记录到 `polish-history.json`
  - [ ] 11.4.2 实现回滚功能
  - [ ] 11.4.3 实现版本对比
- [ ] 11.5 创建润色命令文件
  - [ ] 11.5.1 创建 `paper-polish-formalize.md`
  - [ ] 11.5.2 创建 `paper-polish-rewrite.md`
  - [ ] 11.5.3 创建 `paper-polish-check.md`

## 12. 联网引证系统实现 ⭐ 核心
- [ ] 12.1 实现文献搜索命令（`/paper.research.search`）
  - [ ] 12.1.1 集成知网/万方搜索（中文文献）
  - [ ] 12.1.2 集成 Google Scholar/Semantic Scholar（英文文献）
  - [ ] 12.1.3 实现搜索结果解析和展示
  - [ ] 12.1.4 实现一键添加到文献库
  - [ ] 12.1.5 实现高级搜索过滤
- [ ] 12.2 实现引用建议命令（`/paper.research.suggest`）
  - [ ] 12.2.1 分析章节内容
  - [ ] 12.2.2 识别需要引用的论点
  - [ ] 12.2.3 自动搜索相关文献
  - [ ] 12.2.4 生成引用建议列表
- [ ] 12.3 实现引用验证命令（`/paper.research.verify`）
  - [ ] 12.3.1 验证 DOI 有效性
  - [ ] 12.3.2 验证 URL 有效性
  - [ ] 12.3.3 检测可疑引用（AI 编造）
  - [ ] 12.3.4 生成验证报告
- [ ] 12.4 实现引用占位符解析命令（`/paper.research.resolve`）
  - [ ] 12.4.1 扫描文档中的占位符
  - [ ] 12.4.2 为占位符匹配候选文献
  - [ ] 12.4.3 实现自动解析模式
- [ ] 12.5 实现搜索缓存
  - [ ] 12.5.1 缓存搜索结果（7 天有效期）
  - [ ] 12.5.2 实现缓存刷新
- [ ] 12.6 创建联网引证命令文件
  - [ ] 12.6.1 创建 `paper-research-search.md`
  - [ ] 12.6.2 创建 `paper-research-suggest.md`
  - [ ] 12.6.3 创建 `paper-research-verify.md`
  - [ ] 12.6.4 创建 `paper-research-resolve.md`

## 13. 数据模型与状态管理
- [ ] 13.1 设计并实现 SourceEntry 数据模型（文献条目）
- [ ] 13.2 设计并实现 OutlineTree 数据模型（大纲结构）
- [ ] 13.3 设计并实现 SectionMeta 数据模型（章节元数据）
- [ ] 13.4 设计并实现 FormatProfile 数据模型（引文格式配置）
- [ ] 13.5 设计并实现 ThesisConstitution 数据模型（论文论点宪法）
- [ ] 13.6 实现状态管理脚本（读写 JSON 文件）

## 14. Web 预览系统实现
- [ ] 14.1 选择 Web 框架（Flask 或 FastAPI）
- [ ] 14.2 实现 Web 服务器启动命令（`/paper.preview.start`）
- [ ] 14.3 实现论文全文渲染
  - [ ] 14.3.1 合并所有 section 文件
  - [ ] 14.3.2 按 outline.json 排序
  - [ ] 14.3.3 渲染为 HTML（使用 Pandoc 或 Markdown 库）
  - [ ] 14.3.4 应用引用格式（Pandoc citeproc）
- [ ] 14.4 实现占位符管理
  - [ ] 14.4.1 检测未完成章节
  - [ ] 14.4.2 生成占位符 HTML
  - [ ] 14.4.3 应用占位符样式
- [ ] 14.5 实现实时更新
  - [ ] 14.5.1 文件监听（watchdog 库）
  - [ ] 14.5.2 WebSocket 或 Server-Sent Events
  - [ ] 14.5.3 自动刷新预览
- [ ] 14.6 实现版本对比视图
  - [ ] 14.6.1 版本选择界面
  - [ ] 14.6.2 并排对比视图
  - [ ] 14.6.3 差异高亮（diff 算法）
  - [ ] 14.6.4 同步滚动
- [ ] 14.7 实现导航与目录
  - [ ] 14.7.1 生成目录树（基于 outline.json）
  - [ ] 14.7.2 目录点击跳转
  - [ ] 14.7.3 当前章节高亮
- [ ] 14.8 实现样式与主题
  - [ ] 14.8.1 创建多种预览样式（CSS）
  - [ ] 14.8.2 创建亮色/暗色主题
  - [ ] 14.8.3 样式切换功能
- [ ] 14.9 实现导出功能（从预览页面）
- [ ] 14.10 前端界面开发（HTML/CSS/JavaScript）

## 15. 版本控制系统实现
- [ ] 15.1 实现 Git 集成
  - [ ] 15.1.1 自动初始化 Git 仓库（`/paper.setup`）
  - [ ] 15.1.2 创建 `.gitignore` 文件
  - [ ] 15.1.3 检查 Git 状态（`/paper.version.status`）
- [ ] 15.2 实现版本快照
  - [ ] 15.2.1 创建快照命令（`/paper.version.snapshot`）
  - [ ] 15.2.2 自动快照触发机制
  - [ ] 15.2.3 快照元数据记录（`versions.json`）
  - [ ] 15.2.4 Git tag 创建
- [ ] 15.3 实现版本列表
  - [ ] 15.3.1 列出所有版本（`/paper.version.list`）
  - [ ] 15.3.2 版本过滤（按日期、标签）
  - [ ] 15.3.3 版本统计信息显示
- [ ] 15.4 实现版本对比
  - [ ] 15.4.1 对比两个版本（`/paper.version.diff`）
  - [ ] 15.4.2 对比当前版本与历史版本
  - [ ] 15.4.3 文件级别差异
  - [ ] 15.4.4 内容级别差异
- [ ] 15.5 实现版本回滚
  - [ ] 15.5.1 回滚命令（`/paper.version.rollback`）
  - [ ] 15.5.2 安全回滚（创建备份）
  - [ ] 15.5.3 确认提示
- [ ] 15.6 实现版本导出
  - [ ] 15.6.1 导出历史版本（`/paper.version.export`）
  - [ ] 15.6.2 临时切换版本
  - [ ] 15.6.3 生成导出文件
- [ ] 15.7 实现版本标签与注释
  - [ ] 15.7.1 添加标签（`/paper.version.tag`）
  - [ ] 15.7.2 添加注释（`/paper.version.annotate`）
  - [ ] 15.7.3 更新 `versions.json`
- [ ] 15.8 实现分支管理
  - [ ] 15.8.1 创建分支（`/paper.version.branch`）
  - [ ] 15.8.2 切换分支（`/paper.version.switch`）
  - [ ] 15.8.3 合并分支（`/paper.version.merge`）
  - [ ] 15.8.4 冲突处理提示

## 16. 测试与验证
- [ ] 16.1 创建测试项目（使用 `/paper.setup` 初始化）
- [ ] 16.2 测试文献管理功能
  - [ ] 16.2.1 测试手动添加文献
  - [ ] 16.2.2 测试 DOI 自动解析
  - [ ] 16.2.3 测试 URL 自动解析
  - [ ] 16.2.4 测试文献检索
- [ ] 16.3 测试大纲生成功能
- [ ] 16.4 测试分步式扩写功能 ⭐
  - [ ] 16.4.1 测试章节扩写
  - [ ] 16.4.2 测试思路引导
  - [ ] 16.4.3 测试渐进式完善
- [ ] 16.5 测试学术润色功能 ⭐
  - [ ] 16.5.1 测试口语转书面语
  - [ ] 16.5.2 测试降重改写
  - [ ] 16.5.3 测试学术规范检查
- [ ] 16.6 测试联网引证功能 ⭐
  - [ ] 16.6.1 测试文献搜索
  - [ ] 16.6.2 测试引用建议
  - [ ] 16.6.3 测试引用验证
- [ ] 16.7 测试引用插入与审计
- [ ] 16.8 测试导出功能（Markdown/Word/LaTeX/PDF）
- [ ] 16.9 测试 Web 预览功能
  - [ ] 16.9.1 测试服务器启动
  - [ ] 16.9.2 测试论文全文渲染
  - [ ] 16.9.3 测试占位符显示
  - [ ] 16.9.4 测试实时更新
  - [ ] 16.9.5 测试版本对比视图
- [ ] 16.10 测试版本控制功能
  - [ ] 16.10.1 测试版本快照
  - [ ] 16.10.2 测试版本列表
  - [ ] 16.10.3 测试版本对比
  - [ ] 16.10.4 测试版本回滚
  - [ ] 16.10.5 测试分支管理
- [ ] 16.11 测试跨平台兼容性（Linux、macOS、Windows）

## 17. 文档更新
- [ ] 17.1 更新 `README.md`
  - [ ] 17.1.1 更新项目介绍
  - [ ] 17.1.2 更新功能列表（添加扩写、润色、联网引证）
  - [ ] 17.1.3 更新安装说明
  - [ ] 17.1.4 更新快速开始指南
  - [ ] 17.1.5 更新命令列表
- [ ] 17.2 更新 `docs/commands.md`（完整命令文档）
- [ ] 17.3 更新 `docs/build.md`（构建文档）
- [ ] 17.4 创建 `docs/expand.md`（分步式扩写详细说明）⭐
- [ ] 17.5 创建 `docs/polish.md`（学术润色详细说明）⭐
- [ ] 17.6 创建 `docs/research.md`（联网引证详细说明）⭐
- [ ] 17.7 创建 `docs/bibliography.md`（文献管理详细说明）
- [ ] 17.8 创建 `docs/export.md`（导出系统详细说明）
- [ ] 17.9 创建 `docs/preview.md`（Web 预览系统详细说明）
- [ ] 17.10 创建 `docs/version.md`（版本控制详细说明）

## 18. 发布准备
- [ ] 18.1 更新版本号（0.1.0，表示重大变更）
- [ ] 18.2 生成 CHANGELOG.md
- [ ] 18.3 构建发布包（运行 `build_paperkit.py`）
- [ ] 18.4 测试发布包安装
- [ ] 18.5 发布到 PyPI（`novel-kit-cli` → `paper-kit-cli`）
- [ ] 18.6 更新 GitHub 仓库信息
- [ ] 18.7 创建 Release 和 Tag
