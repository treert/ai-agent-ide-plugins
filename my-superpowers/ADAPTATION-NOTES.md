# Skill 适配说明

> 本文件记录 my-superpowers 从原版 superpowers 筛选、复制、适配的情况。
> 供新会话继续工作时参考。最后更新：2026-08-13。

## 当前已包含的 Skills（11 个）

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
| `requesting-code-review` | 原版 | ⚠️ 见下文 | 需适配 |

## 明确不包含的 Skills（3 个）

均因强依赖 `general-purpose` subagent，CodeBuddy IDE 无此 subagent（仅有 `code-explorer`），暂不复制：

| Skill | 排除原因 |
|-------|---------|
| `subagent-driven-development` | 4 处 subagent dispatch（implementer + task-reviewer + re-reviewer + final reviewer）+ 3 个 shell script，核心流程完全建立在 subagent 之上 |
| `dispatching-parallel-agents` | 核心就是并行 dispatch N 个 `general-purpose` subagent |
| `using-git-worktrees` | 用户决定不需要（CodeBuddy 有自己的 worktree 管理或不需要此层隔离） |

## `requesting-code-review` 的适配问题

### 问题

`requesting-code-review/SKILL.md` 第 34 行和 `code-reviewer.md` 模板内部都写的是：

```
Dispatch a `general-purpose` subagent, filling the template at code-reviewer.md
```

这是 Claude Code 的 subagent 调度语法。CodeBuddy IDE 没有 `general-purpose` subagent。

### CodeBuddy 环境验证结论（2026-08-12 测试）

1. `Task(subagent_name="general-purpose")` → **失败**，subagent 未注册
2. `Task(subagent_name="code-reviewer", subagent_path="code-reviewer.md")` → **失败**，subagent_path 不能凭空创建 subagent
3. `Task(subagent_name="code-explorer", subagent_path="code-reviewer.md")` + prompt 内含完整 reviewer 指令 → **成功**，但 subagent_path 参数本身无效，reviewer 行为来自 prompt 内容
4. `Task(subagent_name="code-explorer", subagent_path="code-reviewer.md")` + prompt 只写"按照 code-reviewer.md 里的指令 review" → **成功**，subagent 主动 `read_file` 读取了模板并遵守格式

### 可行的适配方向（待实施）

**方案 A：用 command 包装**
在 `commands/` 下新建 `code-review.md`，内部指导 AI 用 `Task(subagent_name="code-explorer")` 调度，prompt 里让 subagent 先 `read_file` 读取 `requesting-code-review/code-reviewer.md` 模板再执行。

**方案 B：改写 SKILL.md**
把 `requesting-code-review/SKILL.md` 里的 `general-purpose` 改成 `code-explorer`，并在 dispatch 指令里加上"先 read_file 读取 code-reviewer.md"的步骤。

**方案 C：组合**
同时做 A + B——SKILL.md 适配自动触发场景，command 供手动调用。

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
  │  requesting-code-review ──> [code-reviewer.md 模板]      │
  │       (⚠️ 需适配)              │                          │
  │                                 └──> code-explorer subagent (适配后)
  │                                                          │
  │  receiving-code-review (独立，处理 review 反馈)           │
  └──────────────────────────────────────────────────────────┘
```

注：原版的 `subagent-driven-development` 介于 `writing-plans` 和 `executing-plans` 之间，是 subagent 化的执行模式。因不包含此 skill，主线工作流走 `executing-plans` 的手动执行模式。

## 待办事项

- [ ] 适配 `requesting-code-review`（选 A/B/C 方案之一）
- [ ] 更新 `commands/setup-skills.md`，注入新的技能触发说明（目前只注入了 brainstorming + TDD）
- [ ] 更新 `README.md`（目前还写着"只保留 3 个技能"）
- [ ] 考虑是否需要为 `finishing-a-development-branch` 适配 worktree 相关逻辑（原版依赖 `using-git-worktrees`，已排除）
- [ ] 考虑 `writing-plans` 的 `plan-document-reviewer-prompt.md` 是否也有 subagent 依赖需检查

## 原版 superpowers 参考

- 源仓库：`f:\MyGit\superpowers`
- 原版共 14 个 skill，本仓库取 11 个，排除 3 个
- 原版所有 prompt 模板中的 `Subagent (general-purpose):` 语法是 Claude Code 专属，CodeBuddy 需用 `Task(subagent_name="code-explorer")` 替代
