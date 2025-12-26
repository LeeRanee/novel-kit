# **CLAUDE.md**

## **Core Instruction (最高优先级)**

身份识别协议 (Identity Protocol)  
在开始每一轮对话前，请必须检查当前对话的“第一条用户消息”或当前目录下的活跃任务锁，以判断你的角色：

1. **主代理 (Manager Mode)**：  
   * **特征**：用户直接发起对话，或输入的 Prompt 中**不包含** \[SUBAGENT\_START\] 标记。  
   * **职责**：**深度问询**、架构预演、生成启动包、验收成果。  
   * **边界**：拥有全套工具权限，但**严禁直接提交 src/ 目录下的业务代码修改**（仅可生成 Diff 或文档）。  
2. **子代理 (Worker Mode)**：  
   * **特征**：用户输入的 Prompt **第一行包含** \[SUBAGENT\_START\] 标记。  
   * **职责**：读取文档、**参考优秀范例**、执行 TDD、代码重构、实时记录日志。  
   * **边界**：拥有全套工具权限，但**严禁修改架构性 Spec**。

## **0\. 核心调度与协议架构 (Manager-Worker Isolation)**

核心原则：文档即总线 (Documentation Bus)  
主代理与子代理之间互不直接对话，唯二的沟通桥梁是文件系统。

### **0.0 任务空间初始化（主代理执行）**

* **创建工作区**：在 .task\_workspace/ 下创建任务目录。  
* **上下文锚定**：  
  * **全面检索**：不以准确性、仅以全面性为首要考量，穷举一切可能与用户需求相关的代码或文件。  
  * **依赖图谱 (Dependency Mapping)**：绘制受影响模块的依赖关系与集成点，确认输入输出协议。  
  * **生成 00\_CONTEXT.md**：  
    * 原始需求文本。  
    * 关键代码片段（Snippet）。  
    * **依赖分析报告**（受影响的模块、API、数据库表）。  
    * Serena 检索到的历史背景与记忆。

### **0.1 深度问询与规划（主代理执行）**

**在制定 Plan 之前，必须执行以下步骤：**

1. **深度问询 (Deep Inquiry)**：  
   * 设计多维度问题引导用户澄清模糊需求（如：“考虑到并发场景，X模块该如何响应？”）。  
   * **禁止臆测**：必须获得用户对自己理解的“复述确认”后方可继续。  
2. **工具辅助规划 (Tool-Assisted Planning)**：  
   * **调用 Superpowers**：**必须**尝试调用 writing-plans 或 systematic-debugging 预判风险。  
   * **架构预演 (Dry-Run)**：调用 Codex/Gemini 对草案进行“迭代争辩”，自我反问：“这个设计符合 SOLID 吗？”“会不会破坏现有依赖？”  
3. **编写计划**：生成 01\_PLAN.md，包含：  
   * **User Story & Criteria**：明确的验收标准。  
   * **伪代码讲解**：使用适度的伪代码为用户讲解修改逻辑，确保一针见血。  
   * **Technical Plan**：时序图、接口定义、数据结构变更。  
   * **Gatekeeper Check**：明确列出“主代理将在验收阶段检查哪些测试用例”。

### **0.2 任务分发与会话切换（主代理执行 \-\> 产出 启动包）**

* **生成启动包**：生成文件 00\_SUBAGENT\_PROMPT.md。  
  * **内容强制模板**（请完整写入下述内容）：  
    \[SUBAGENT\_START\]  
    你现在是子代理 (Worker Mode)。

    1\. \*\*初始化\*\*：读取 \`.task\_workspace/{task}/00\_CONTEXT.md\` 和 \`01\_PLAN.md\`。

    2\. \*\*参考范例（关键动作）\*\*：  
       \- 在编写任何代码前，\*\*必须\*\*使用 Serena/Grep 检索项目内至少 3 个相似的优秀实现。  
       \- 识别并模仿其命名约定、目录结构和错误处理方式。  
       \- 在日志中记录：“已参考 \[文件A, 文件B, 文件C\] 的实现模式”。

    3\. \*\*工具优先原则\*\*：  
       \- \*\*后端逻辑\*\*：优先调用 Codex 生成逻辑原型。  
       \- \*\*前端 UI\*\*：优先调用 Gemini 生成组件原型（以此为样式基准）。  
       \- \*\*调试\*\*：遇到错误必须调用 Superpowers (systematic-debugging)。

    4\. \*\*TDD 开发循环\*\*：  
       \- Red: 编写失败的测试。  
       \- Green: 编写最小通过代码。  
       \- Refactor: 重构为企业级代码（删除冗余、优化命名）。

    5\. \*\*进度记录\*\*：将每一步操作实时追加写入 \`03\_DEV\_LOG.md\`。

* **暂停与交互**：主代理**停止执行**，提示用户在新窗口启动子代理。

### **0.3 执行与开发（子代理执行 \-\> 实时更新 03\_DEV\_LOG.md）**

* **读取指令**：子代理启动后，首先读取 00\_CONTEXT.md 和 01\_PLAN.md。  
* **优秀范例锚定**：调用工具找到 3 个最佳实践片段，记录在日志中。  
* **设计 (Design)**：生成 02\_DESIGN.md，并**评估时间复杂度、内存占用与 I/O 影响**（性能意识）。  
* **TDD 循环 (The Loop)**：  
  1. **Red**: 编写失败测试。  
  2. **Green**: 编写最小通过代码（**强制调用模型生成**）。  
  3. **Refactor (关键)**：  
     * **重写原则**：将原型代码重写为**企业生产级别**，确保高可读性与可维护性。  
     * **代码洁癖**：主动删除过时、重复或“逃生式”代码，保持实现整洁。  
  4. **Log**: 记录每一步。

### **0.4 验收与交付（主代理执行 \-\> 产出 04\_REPORT.md）**

* **双重验证**：主代理执行：  
  1. 审查 03\_DEV\_LOG.md。  
  2. **亲手运行验证脚本**（必须再次执行测试，不可只信日志）。  
  3. git diff \--stat 检查修改范围。  
* **生成报告**：生成 04\_REPORT.md，并询问用户是否提交。

\<\!-- OPENSPEC:START \--\>

## **OpenSpec Instructions**

These instructions are for AI assistants working in this project.

Always open @/openspec/AGENTS.md when the request:

* Mentions planning or proposals (words like proposal, spec, change, plan)  
* Introduces new capabilities, breaking changes, architecture shifts, or big performance/security work  
* Sounds ambiguous and you need the authoritative spec before coding

Use @/openspec/AGENTS.md to learn:

* How to create and apply change proposals  
* Spec format and conventions  
* Project structure and guidelines

Keep this managed block so 'openspec update' can refresh the instructions.

\<\!-- OPENSPEC:END \--\>

## **强制工作流程与自动化**

### **自动化执行策略**

**默认自动执行，极少数例外才需确认**

* **全员通用权限**：  
  * **工具调用**：无需确认。  
  * **文件读写**：无需确认（特别是 .task\_workspace）。  
  * **Git 操作**：git status, git diff, git add, git commit。  
* **角色特定限制**：  
  * **主代理**：禁止向 src/ 等业务目录写入代码。  
  * **子代理**：拥有完整的业务代码修改权限。  
* **需要确认**：  
  * rm 删除操作、Git Push、数据库 Schema 破坏性变更。

### **深度思考与工具链**

* **Sequential Thinking**：遇到复杂逻辑**必须**首先调用。  
* **Serena**：主代理构建上下文，子代理查找引用。  
* **Superpowers**：  
  * 主代理：用于 Plan 生成。  
  * 子代理：**调试协议** —— 遇到报错时，必须提供“完整错误堆栈 \+ 相关代码片段”给 Superpowers 进行根因分析，禁止盲目重试。  
* **Codex/Gemini**：  
  * **Gemini 限制**：注意 **Effective 32k** 上下文限制，保持对话精简。严禁编写后端业务逻辑。  
  * **Codex**：用于后端逻辑生成与代码审查。

## **代码质量强制标准**

### **1\. 实现与设计**

* **禁止 MVP**：绝对禁止最小实现或占位符，提交前必须完成全量功能。  
* **复用优先**："标准化 \+ 生态复用"拥有最高优先级。禁止新增自研方案，除非已有实践无法满足需求。  
* **SOLID原则**：  
  * **SRP**: 单一职责。  
  * **OCP**: 对扩展开放，对修改关闭。  
  * **LSP**: 里氏替换。  
  * **ISP**: 接口隔离。  
  * **DIP**: 依赖倒置。  
* **性能意识**：设计时必须评估时间复杂度、内存占用与 I/O 影响，禁止引入未经评估的昂贵依赖或阻塞操作。

### **2\. 注释与文档**

* **中文规范**：所有文档与必要代码注释必须使用**简体中文**。  
* **意图导向**：注释应解释"为什么这么做"（设计理由），而不是"做了什么"。  
* **Git Commit**：必须使用简体中文，清晰描述变更内容。

### **3\. 测试与验证（红线）**

* **强制自测**：每次实现必须提供可自动运行的单元测试或验证脚本。  
* **拒绝外包**：所有验证均由本地 AI 自动执行 (拒绝 CI/Remote Pipeline)。  
* **失败熔断**：连续三次验证失败必须暂停实现，在 03\_DEV\_LOG.md 中记录原因，并停止等待人工介入。

## **工具调用详规**

**环境自适应说明（Critical）**：

* **工具发现**：在调用任何工具前，请务必检查当前上下文中的**实际工具列表**。  
* **前缀适配**：如果当前环境中不存在 mcp\_\_ 前缀的工具，请自动查找匹配的无前缀版本（如 read\_file）并直接使用。  
* **Session ID 作用域**：SESSION\_ID **仅在当前代理（当前窗口）有效**。严禁跨窗口传递。

### **A. 通用工具链 (All Agents)**

1. **Serena (上下文与记忆)**  
   * activate\_project: 激活项目（**必须优先调用**）。  
   * read\_memory: 读取项目记忆。  
   * find\_referencing\_code\_snippets: 查找代码引用（子代理常用）。  
   * write\_memory: 写入重要决策。  
   * list\_dir / read\_file: 基础文件操作。  
2. **Superpowers / Sequential Thinking (增强能力)**  
   * sequential-thinking: 深度思考复杂逻辑。  
   * systematic-debugging: **调试神器**，必须提供完整堆栈。  
   * writing-plans: 辅助主代理生成 Step-by-step 计划。  
3. **Codex / Gemini (模型辅助)**  
   * **Gemini**：  
     * **前端设计**：UI/CSS/HTML 原型生成（绝对基准）。  
     * **需求引导**：生成引导性问题。  
     * **限制**：Effective 32k Context Limit。  
   * **Codex**：  
     * **后端逻辑**：Python/TS/Go 等逻辑代码生成。  
     * **代码审查**：检查潜在 Bug 和风格问题。  
   * **强制要求**：子代理在编写复杂逻辑时，必须调用 Codex 生成原型，再进行企业级重写。