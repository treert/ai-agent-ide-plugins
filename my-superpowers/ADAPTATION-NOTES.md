# Skill 适配说明

> 本文件记录 my-superpowers 从原版 superpowers 筛选、复制、适配的情况。
> 供新会话继续工作时参考。最后更新：2026-08-13。

## 移植来源

- **源仓库目录**：`F:\MyGit\superpowers\skills\`
- **目标目录**：`F:\MyGit\ai-agent-ide-plugins\my-superpowers\Skills\`

## 当前已包含的 Skills（10 个）

> 注：`requesting-code-review` 已从 Skills 目录迁出，转为 `templates/code-review/` 下的模板，由 `setup-skills` 命令在目标项目生成项目级 subagent + rule。详见下文"Code Review 适配方案"。

| Skill | 来源 | subagent 依赖 | 状态 |
|-------|------|--------------|------|
| `using-superpowers` | 原版 | 无 | ✅ 可直接用 |
| `brainstorming` | 原版 | 无 | ✅ 可直接用 |
| `test-driven-development` | 原版 | 无 | ✅ 可直接用 |
| `systematic-debugging` | 原版 | 无 | ✅ 可直接用 |
| `verification-before-completion` | 原版 | 无 | ✅ 可直接用 |
| `finishing-a-development-branch` | 原版 | 无 | ✅ 可直接用 |
| `receiving-code-review` | 原版 | 无 | ✅ 可直接用 |
| `writing-plans` | 原版 | 无 | ✅ 可直接用 |
| `executing-plans` | 原版 | 无 | ✅ 可直接用 |
| `writing-skills` | 原版 | 无 | ✅ 可直接用 |

## 明确不包含的 Skills（3 个）

均因强依赖 `general-purpose` subagent，CodeBuddy IDE 无此 subagent（仅有 `code-explorer`），暂不复制：

| Skill | 排除原因 |
|-------|---------|
| `subagent-driven-development` | 4 处 subagent dispatch（implementer + task-reviewer + re-reviewer + final reviewer）+ 3 个 shell script，核心流程完全建立在 subagent 之上 |
| `dispatching-parallel-agents` | 核心就是并行 dispatch N 个 `general-purpose` subagent |
| `using-git-worktrees` | 用户决定不需要（CodeBuddy 有自己的 worktree 管理或不需要此层隔离） |

## Code Review 适配方案

### 演进历史

1. **原版问题**：`requesting-code-review/SKILL.md` 和 `code-reviewer.md` 都用 Claude Code 的 `Dispatch a general-purpose subagent` 语法，CodeBuddy 没有 `general-purpose` subagent。

2. **2026-08-12 验证**：测试了 4 种调度方式，结论是只能用 `code-explorer` 替代，且 prompt 内嵌完整指令最稳。

3. **2026-08-13 重构**：放弃"改写 SKILL.md"思路，改为**模板生成器**模式——my-superpowers 不再包含 review skill，而是提供模板，在 `setup-skills` 执行时生成项目级 `.codebuddy/agents/code-reviewer.md` + `.codebuddy/rules/requesting-code-review.md`，由各项目独立维护、独立演进。

### 为什么不用 skill / command 包装

- 原 SKILL.md 是通用的，无法承载项目特定内容（编译命令、技术栈关注点、性能硬指标等）
- 不同项目 review 规范不同，集中维护一份会影响所有项目
- 参考项目（`F:\MyGit\ai-mylua-lsp\.codebuddy\`）已验证：subagent + rule 的项目级组合更可控、可定制

### 模板文件位置

```
my-superpowers/templates/code-review/
├── agent.md   ← 生成目标项目 .codebuddy/agents/code-reviewer.md 的源模板
└── rule.md    ← 生成目标项目 .codebuddy/rules/requesting-code-review.md 的源模板
```

两个模板都带 placeholder（`{{PROJECT_NAME}}` / `{{TECH_STACK}}` / `{{BUILD_COMMANDS}}` 等），由 `commands/setup-skills.md` 第 4 步在执行时填充。

### 生成后的目标项目结构

```
目标项目/.codebuddy/
├── agents/
│   └── code-reviewer.md           ← subagent 定义（readonly: true，agentMode: agentic）
└── rules/
    └── requesting-code-review.md  ← 触发 rule（默认 enabled: false，用户主动开启）
```

### 关键设计决策

- **subagent readonly: true**：天然符合 reviewer "Read-Only" 原则，不会误改代码
- **rule 默认 enabled: false**：避免用户未准备好就被强制 review，setup 后主动评估再开启
- **receiving-code-review 仍是 skill**：处理反馈是主 agent 职责，不 dispatch subagent
- **项目背景由 setup 时 AI 主动观察填充**：不询问用户，从 manifest 和目录树推断；观察不到则留 placeholder 注释让用户后填
- **如已有同名文件不覆盖**：保护用户已定制的版本，需重新生成时手动删除

### 已删除的孤立 reviewer 文件

适配过程中发现 2 个孤立文件，均含 Claude Code 专属的 `Subagent (general-purpose):` 语法，且无任何 SKILL.md 引用，已删除：

| 被删文件 | 原作用 | 删除理由 |
|---------|--------|---------|
| `Skills/brainstorming/spec-document-reviewer-prompt.md` | 审查已写好的 spec 文档 | 孤儿文件 + subagent 语法 + 已有 Self-Review 替代 |
| `Skills/writing-plans/plan-document-reviewer-prompt.md` | 审查已写好的 plan 文档 | 孤儿文件 + subagent 语法 + 已有 Self-Review 替代 |

**澄清：被删的文件不是用来"写" spec/plan 的，是用来"审查"已写好的 spec/plan 的。** 写和审两件事都在各自 SKILL.md 里完整保留：

| 流程 | 所在文件 | 关键段落 |
|------|---------|---------|
| 写 spec | `brainstorming/SKILL.md` | "## The Process" → "Documentation" 段（写 `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` + git commit） |
| 审查 spec（Self-Review） | `brainstorming/SKILL.md` | "## Spec Self-Review" 段（主 agent 自查 placeholder/一致性/范围/歧义） |
| 写 plan | `writing-plans/SKILL.md` | "## File Structure" + "## Task Structure" + "## Plan Document Header" |
| 审查 plan（Self-Review） | `writing-plans/SKILL.md` | "## Self-Review" 段（主 agent 自查 spec 覆盖/placeholder/类型一致性） |

**Self-Review 比 subagent review 更适合 CodeBuddy**：
- 不依赖 `general-purpose` subagent
- 主 agent 自执行，无调度开销
- 原版设计本就如此（两个 SKILL.md 都明确写了"This is a checklist you run yourself — not a subagent dispatch"）

**当前完整文档流程（无 subagent 依赖）：**

```
brainstorming/SKILL.md
  ↓ 探索 → 提问 → 提方案 → 呈现设计 → 用户批准
  ↓ 写 spec 到 docs/superpowers/specs/
  ↓ Spec Self-Review（主 agent 自查）
  ↓ 用户 review spec
  ↓
writing-plans/SKILL.md
  ↓ 写 plan 到 docs/superpowers/plans/
  ↓ Self-Review（主 agent 自查）
  ↓
executing-plans/SKILL.md
  ↓ 读 plan → 执行任务
```

## 依赖关系图（适配后）

```
  using-superpowers（引导层，会话开始必调）

  ┌─ 主线工作流 ─────────────────────────────────────────────┐
  │                                                          │
  │  brainstorming ──> writing-plans                         │
  │                        │                                  │
  │                        └──> executing-plans ──> finishing-a-development-branch
  │                                                          │
  └──────────────────────────────────────────────────────────┘

  ┌─ Debug/质量保证支线 ─────────────────────────────────────┐
  │                                                          │
  │  systematic-debugging ──> test-driven-development        │
  │                      └──> verification-before-completion │
  │                                                          │
  │  writing-skills ──> test-driven-development              │
  └──────────────────────────────────────────────────────────┘

  ┌─ Code Review 支线 ───────────────────────────────────────┐
  │                                                          │
  │  一次性 setup（运行 @command://my-superpowers:setup-skills）  │
  │  ─────────────────────────────────────────               │
  │  → 从 templates/code-review/ 读取 agent.md + rule.md     │
  │  → 观察目标项目 manifest / 目录树，填充 placeholder        │
  │  → 生成 .codebuddy/agents/code-reviewer.md (subagent)     │
  │  → 生成 .codebuddy/rules/requesting-code-review.md (rule) │
  │    （默认 enabled: false，用户主动开启）                  │
  │                                                          │
  │  运行时（rule 启用后）                                    │
  │  ──────────                                              │
  │  完成功能性修改 → 编译验证 → Task(code-reviewer) 调度     │
  │       → receiving-code-review skill 处理反馈              │
  │         - Critical → 立即修复                             │
  │         - Important → 修复后继续                          │
  │         - Minor → 记录稍后处理                            │
  │         - 错误反馈 → 技术 pushback                        │
  │                                                          │
  │  receiving-code-review (保留为 skill，主 agent 自执行)    │
  └──────────────────────────────────────────────────────────┘
```

注：
- 原版的 `subagent-driven-development` 介于 `writing-plans` 和 `executing-plans` 之间，是 subagent 化的执行模式。因不包含此 skill，主线工作流走 `executing-plans` 的手动执行模式。
- `requesting-code-review` 已从 Skills 目录迁出转为模板（详见上文"Code Review 适配方案"）。

## 待办事项

- [x] 适配 `requesting-code-review`（已完成：转为模板生成器模式，详见"Code Review 适配方案"）
- [x] 更新 `commands/setup-skills.md`，注入新的技能触发说明 + 生成 code-review 配置（第 4 步已加）
- [x] 更新 `README.md`（已改为 10 个 skill + 1 套 code-review 模板）
- [x] 检查 `writing-plans` / `brainstorming` 的 subagent 依赖（已完成）
  - 发现并删除 2 个孤立文件：`writing-plans/plan-document-reviewer-prompt.md` 和 `brainstorming/spec-document-reviewer-prompt.md`（均含 `Subagent (general-purpose):` 语法，且无任何 SKILL.md 引用，原版用主 agent 自执行的 Self-Review 替代）
  - 清理 `writing-plans/SKILL.md` 中对已排除 skill 的引用：删除 `using-git-worktrees` context 行、`subagent-driven-development` 选项与路径
  - 清理 `executing-plans/SKILL.md` 中对已排除 skill 的引用：删除"subagents 可用时改用 subagent-driven-development"段、`using-git-worktrees` workspace 步骤
- [x] 考虑是否需要为 `finishing-a-development-branch` 适配 worktree 相关逻辑（已完成）
  - 检查结论：该 skill 的 worktree 逻辑是**检测 + 兼容**（Detect Environment + Cleanup），不是**创建** worktree（后者在已排除的 `using-git-worktrees` 里）
  - 即使 CodeBuddy 不主动创建 worktree，用户可能手动用 `git worktree` 或 CodeBuddy 内部用 worktree 隔离——这 skill 能正确处理两种场景，保留更安全
  - 唯一改动：Step 6 注释里的 `Superpowers` → `my-superpowers`
- [ ] 在某个真实目标项目实测 `setup-skills` 第 4 步生成的两个文件是否工作正常
- [x] 检查 `writing-skills/render-graphs.js` 第 96-97 行示例命令（已完成）
  - 示例参数 `../subagent-driven-development` → 改为 `../brainstorming`（本仓库实际存在且有 `.dot` 块的 skill）

## 原版 superpowers 参考

- 源仓库：`f:\MyGit\superpowers`
- 原版共 14 个 skill，本仓库取 10 个 skill + 1 套 code-review 模板（来自原版 `requesting-code-review`），排除 3 个
- 原版所有 prompt 模板中的 `Subagent (general-purpose):` 语法是 Claude Code 专属，CodeBuddy 需用 `Task(subagent_name="code-explorer")` 或项目级自定义 subagent 替代
- 本仓库的 code-review 适配走"项目级 subagent + rule"路线（参考 `F:\MyGit\ai-mylua-lsp\.codebuddy\` 的实践）
