---
name: setup-my-superpowers
description: INTERNAL — 手动调用专用，仅由 setup-skills command 主动触发，不响应任何用户消息被动触发。
---

# Setup my-superpowers in Target Project

在目标项目注入 my-superpowers 技能触发说明，并生成项目级 code-review 配置（subagent + rule）。

本 skill 由 `commands/setup-skills.md` 调用，模板文件位于本 skill 目录下的 `templates/code-review/`。

**Announce at start:** "I'm using the setup-my-superpowers skill to configure this project."

## Step 1: Inject AGENTS.md Triggers

1. **找到目标文件**：检查当前项目根目录，优先 `AGENTS.md`，其次 `CLAUDE.md`。都没有则创建 `AGENTS.md`。

2. **检查是否已有触发段落**：搜索文件中是否包含 `## 使用 my-superpowers` 标题。
   - 已有 → 替换该段落内容
   - 没有 → 追加到文件末尾

3. **注入以下内容**：

```markdown
## 使用 my-superpowers（特别重要）

会话开始时先调用 using-superpowers 技能，它会指导你在合适时机使用其他技能。

- 构建/改造功能前 → 先用 brainstorming 技能
- 写实现代码前 → 先用 test-driven-development 技能
```

## Step 2: Observe Target Project

检查目标项目根目录，识别以下信息（**不询问用户，直接观察**）：

- **项目名称**：从 `package.json` / `Cargo.toml` / `pyproject.toml` / `go.mod` 等读取
- **技术栈**：manifest 文件类型 + 主要依赖
- **关键目录结构**：`list_dir` 看一级目录
- **性能/规模硬指标**：如 manifest 或 README 中有提及则记录，否则填"无"
- **构建命令**：按技术栈推断（如 `cargo build` / `npm run build` / `pytest`）；多技术栈项目逐项列出；无明显构建步骤则注明"本项目无编译步骤"

## Step 3: Read Templates

读取本 skill 目录下的两个模板文件（路径相对于本 SKILL.md）：

- `templates/code-review/agent.md` — code-reviewer subagent 定义模板
- `templates/code-review/rule.md` — code-review 触发 rule 模板

## Step 4: Generate code-reviewer subagent

在目标项目创建 `.codebuddy/agents/code-reviewer.md`：

- 以 `templates/code-review/agent.md` 内容为来源
- 填充占位符：
  - `{{PROJECT_NAME}}` ← Step 2 观察到的项目名称
  - `{{TECH_STACK}}` ← Step 2 观察到的技术栈
  - `{{KEY_DIRECTORIES}}` ← Step 2 观察到的关键目录
  - `{{PERFORMANCE_CONSTRAINTS}}` ← Step 2 观察到的硬指标（无则填"无"）
- 删除所有 `<!-- ... -->` 注释段
- 保留审查维度、问题分级、输出格式等通用内容不动

## Step 5: Generate Triggering Rule

在目标项目创建 `.codebuddy/rules/requesting-code-review.md`：

- 以 `templates/code-review/rule.md` 内容为来源
- 用 Step 2 推断的构建命令替换 `{{BUILD_COMMANDS}}` 占位符
- 保留 `enabled: false`（用户后续主动开启）
- 删除所有 `<!-- ... -->` 注释段

## Step 6: Idempotency Check

检查目标项目 `.codebuddy/agents/code-reviewer.md` 和 `.codebuddy/rules/requesting-code-review.md` 是否已存在：

- **存在** → 跳过生成，告知用户"已存在，未覆盖，如需重新生成请先手动删除"
- **不存在** → 按 Step 4-5 生成

## Step 7: Report to User

报告修改了哪些文件、加了什么内容：

- `AGENTS.md` 注入了哪些触发说明
- 生成了 `.codebuddy/agents/code-reviewer.md` 和 `.codebuddy/rules/requesting-code-review.md`
- 提醒 code-review rule **默认 `enabled: false`**，需要手动改为 `true` 才会自动触发
- 提醒重启会话让触发生效
- 告知用户可自行编辑两个生成文件，加入项目特定审查维度或编译命令

## Template Files

模板文件位于本 skill 目录下：

```
templates/code-review/
├── agent.md   ← 生成 .codebuddy/agents/code-reviewer.md 的源模板
└── rule.md    ← 生成 .codebuddy/rules/requesting-code-review.md 的源模板
```

两个模板都带 `<!-- ... -->` 注释段说明填充规则，以及 `{{...}}` 占位符需在生成时替换。修改模板只需编辑这两个文件，不需要改 SKILL.md。
