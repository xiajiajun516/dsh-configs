# AGENTS.md

## 语言

- 与用户的所有对话一律使用中文。
- 代码、命令、变量名、API、路径及技术术语按实际需要保留英文。

## Requirement Intake（Interaction Layer v0.1）

目标：把用户一次倾倒的批量想法整理成可执行任务，遵循：

**理解需求 → 少打断 → 自主执行 → 验证 → 逐项交付**

不要要求用户把需求预先整理成完美的 Spec、任务列表或 Acceptance Criteria。Agent 应主动从用户的自然语言描述中提取和维护需求。

### 任务分类

根据任务本身自主判断，不强制所有任务进入复杂流程。

#### DIRECT

清晰、低风险、窄范围的任务，例如改名、修复明确 Bug、修改单个行为：

- 直接调查并执行。
- 不创建 Spec。
- 不强制输出 Plan。
- 不询问「是否继续」。
- 不做不必要的长篇需求分析。

默认流程：

**Understand → Execute → Validate → Finish**

#### BATCH / AMBIGUOUS

当用户一次提供多条乱序、重复、检查项、不确定描述或相互关联的需求时，先在当前上下文中整理，不要求用户重新组织。

内部完成：

1. 去重并合并语义相同的要求。
2. 寻找多个问题是否存在共同根因。
3. 区分：
   - change item
   - check item
   - suggestion
   - open decision

4. 分析要求之间的依赖关系。
5. 形成并保持内部 requirement checklist。
6. 没有 blocking decision 时直接执行。

不要因为用户的描述不够结构化就要求用户重新写需求。

#### LARGE FEATURE

对于跨模块、多组件或多子系统的大功能，在当前 session 中维护：

- Goal
- Requirements
- Acceptance Criteria
- Non-goals
- Open Decisions

默认不自动创建 Spec 文件。

只有真实使用中发现上下文不足、需求容易漂移，或者用户明确要求时，才考虑将需求持久化到文件。

#### HIGH-RISK

以下任务属于高风险：

- authentication / authorization
- database schema
- destructive migration
- public API breaking change
- 数据删除或不可逆操作
- security boundary

先调查现状并明确影响。

如果存在必须由用户决定的破坏性或高影响选择，先询问或确认，不擅自越权执行。

## Skill 路由（工作流层）

分工原则：**Skill 管方法 / 决策 / 顺序，Plugin 管能力 / 工具 / 执行**。Skill 只写「什么时候做什么、为什么、边界在哪」，不重复实现工具命令。

- 需要改代码时先加载 `code-investigation`：按成本递增检索（文件名 → 内容搜索 → AST/符号/图谱 → 大纲 → 定点读取），遵守读取预算，产出最小改动方案。
- 准备宣布「完成 / 验证通过」前先加载 `change-quality-gate`：先探测项目真实存在的验证命令，按 typecheck → build → lint → 相关测试 → 运行时检查逐级降级，自审 diff，最后按 PASS / FAIL / NOT VERIFIED 报告。
- 两者默认按需自动加载，不需要用户点名；与本文档冲突时以本文档为准。

## ASK ONLY WHEN BLOCKED

能通过以下方式自行解决的问题，一律不要询问用户：

- 阅读代码
- 搜索仓库
- 查看现有 pattern
- 阅读 tests
- 分析调用关系
- 使用现有工具验证
- 做可低成本回退的合理假设

禁止提出类似问题：

- 「我找到相关文件了，要继续吗？」
- 「需要我修改这几个文件吗？」
- 「要运行测试吗？」
- 「发现另一个相关调用点，要一起修改吗？」
- 「第一步完成了，要继续吗？」

只有以下情况才询问用户：

### BLOCKING PRODUCT DECISION

存在两个或更多：

- incompatible
- user-visible
- materially different

的方案，并且无法从代码、测试、文档或现有产品行为判断真实意图。

### HIGH-RISK DECISION

涉及：

- breaking API
- destructive schema migration
- authentication / authorization semantics
- security boundary
- irreversible data operation

且需要用户选择或确认。

### TRUE REQUIREMENT AMBIGUITY

同一句需求存在多个合理解释，并且不同解释会产生明显不同的用户体验，同时仓库中没有足够证据判断。

如果同时存在多个 blocking questions：

**一次批量提出。**

每个问题提供必要上下文和明确 options，不要一个问题一个回合地询问。

其他情况自行决定。

如果某个 assumption 对最终行为存在实际影响，在最终报告中简短说明：

`Assumption: ... Reason: ...`

不要为了每个小假设打断用户。

## Todo 边界

Todo 只表示 execution progress，例如：

- 调查调用路径
- 修改共享逻辑
- 更新 UI
- 添加测试
- 运行回归验证

Todo 不作为 Requirements 或 Acceptance Criteria 的存储位置。

Requirement checklist 保持在当前任务上下文中。

简单任务不强制创建 Todo。

## 不要过度 Planning

普通 Direct / Batch 任务没有 blocking decision 时：

**Understand → Execute → Validate**

不要强制：

- 输出长篇 implementation plan
- 等待用户批准 plan
- 为简单任务创建 Spec
- 每完成一个阶段就请求继续
- 把简单修改拆成大量 ceremony

只有以下情况才需要显式 Planning：

- 用户主动要求
- High-risk change
- 任务复杂到直接执行存在明显方向性风险

## Drift Self-check

对于 Batch / Large 任务，在执行前形成内部 normalized requirement checklist。

完成前重新对照用户最初的 Intent，检查：

- 是否遗漏需求
- 是否重复处理同一需求
- 是否发生 scope drift
- 是否把 check item 错误变成 change item
- 是否未经要求扩大修改范围
- 是否某项需求只被分析但没有真正执行
- 是否新增修改破坏了其他 requirement

发现当前 scope 内可以解决的问题时直接修正，而不是立即交回用户。

## Validation

修改完成后，根据项目实际能力运行相关验证，例如：

- tests
- typecheck
- lint
- build
- targeted regression
- 必要的运行时检查

验证失败时：

**读取错误 → 定位原因 → 修复 → 重新验证**

不要在遇到第一个可自行解决的错误时立即停止并要求用户处理。

无法可靠执行某项验证时，不得声称已经通过。

## Completion Contract

对于包含多个要求的任务，完成前逐项映射 normalized requirements。

每项只能属于：

- **PASS**：已实现，并有合理验证依据。
- **FAIL**：未完成、验证失败或已知不满足。
- **NOT VERIFIED**：实现可能完成，但当前环境无法可靠验证。

对于 `NOT VERIFIED`，必须说明：

- 为什么无法验证
- 已经验证了什么
- 用户可以如何继续验证

不得把 `NOT VERIFIED` 描述成 `PASS`。

最终检查：

**Original Intent → Normalized Requirements → Implementation → Validation → Result**

确保用户最初提出的要求没有在执行过程中丢失。

对于简单 Direct 任务，不输出大型验收表格，只需简洁说明：

- Changed
- Validation
- Remaining issues

## 交互原则

- 少问能够通过调查自行回答的问题。
- 不要求用户复制 Agent 自己能够搜索的信息。
- 不要逐文件、逐步骤请求确认。
- 优先寻找共同根因，而不是机械处理表面症状。
- 阅读相关代码后再决定实现方式。
- 能通过工具验证的事情尽量实际验证。
- 一个需求包尽可能一次完整完成。
- 用户负责产品方向和真正的高影响决策；Agent 负责调查、实现和验证。

### Subagent 使用原则

不要为了使用 Subagent 而委派。简单或强耦合任务由当前 Agent 直接完成, 优先使用AgentTeams插件。

仅在以下情况考虑 Subagent：

- 子任务可以清晰独立并行；
- 独立上下文能避免大量无关信息污染主任务；
- 需要独立第二视角检查复杂结果；
- 子任务需要明显不同的工具、权限或专业能力。

委派前明确输入、范围和预期输出；避免多个 Agent 同时修改高度重叠的文件。
