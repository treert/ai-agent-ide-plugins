# my-superpowers

个人精简版 superpowers，专为 CodeBuddy 适配。

## 包含

### Skills（11 个）

- **using-superpowers** — 引导层，教 agent 任何动作前先查技能
- **brainstorming** — 写代码前先把想法变成设计
- **writing-plans** — 把设计文档转成可执行的分步实现计划
- **executing-plans** — 按计划逐步实现，带 review checkpoint
- **finishing-a-development-branch** — 分支收尾：跑测试、提交、合并
- **test-driven-development** — 红绿重构，先写测试
- **systematic-debugging** — 系统化定位 bug 根因
- **verification-before-completion** — 完成前必须自我验证
- **receiving-code-review** — 收到 review 反馈后的处理规范
- **writing-skills** — 编写新的 skill 文档
- **setup-my-superpowers** — 在目标项目注入触发说明 + 生成 code-review 配置（由 `setup-skills` command 调用）

### 模板（1 套，位于 `Skills/setup-my-superpowers/templates/code-review/`）

- **agent.md** — 生成目标项目 `.codebuddy/agents/code-reviewer.md` 的源模板（subagent 定义）
- **rule.md** — 生成目标项目 `.codebuddy/rules/requesting-code-review.md` 的源模板（触发 rule）

由 `setup-my-superpowers` skill 在执行时读取并填充 placeholder 后落地，详见 `ADAPTATION-NOTES.md` 的"Code Review 适配方案"。

### Commands

- **setup-skills** — 调用 `setup-my-superpowers` skill，在目标项目注入 my-superpowers 触发说明到 `AGENTS.md`，并生成项目级 code-review subagent + rule

## 安装

在 CodeBuddy 中将此目录注册为插件市场，然后启用 my-superpowers 插件。

## 与原版 superpowers 的区别

- 原版 14 个 skill，本仓库取 11 个 skill（含 `setup-my-superpowers` 这个自创 skill）+ 1 套 code-review 模板（来自原版 `requesting-code-review`），排除 3 个
- 排除原因：
  - `subagent-driven-development` / `dispatching-parallel-agents` — 强依赖 `general-purpose` subagent，CodeBuddy 无此 subagent
  - `using-git-worktrees` — 用户决定不需要
- `requesting-code-review` 转为模板生成器模式（不再作为 skill），由 `setup-skills` 命令在目标项目生成项目级 subagent + rule，方便各项目独立定制审查规范
- `writing-plans` / `brainstorming` 中原版的 spec/plan document reviewer 模板已删除（未被引用，且依赖 `general-purpose` 语法），保留主 agent 自执行的 Self-Review 步骤
- 使用 CodeBuddy 原生 `.codebuddy-plugin` 格式
- 无 SessionStart 钩子注入，靠项目 `AGENTS.md` 触发
- 无跨平台适配（只支持 CodeBuddy）

## 触发方式

本插件不做全局引导注入。在需要使用的项目里，运行：

```
@command://my-superpowers:setup-skills
```

该命令会在目标项目的 `AGENTS.md` 注入技能触发说明，并生成 code-review subagent + rule（rule 默认 `enabled: false`，需用户主动开启）。

`AGENTS.md` 中注入的触发段落示例：

```markdown
## 使用 my-superpowers（特别重要）

会话开始时先调用 using-superpowers 技能，它会指导你在合适时机使用其他技能。

- 构建/改造功能前 → 先用 brainstorming 技能
- 写实现代码前 → 先用 test-driven-development 技能
```

更多适配细节见 [`ADAPTATION-NOTES.md`](./ADAPTATION-NOTES.md)。
