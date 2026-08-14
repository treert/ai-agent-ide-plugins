## 作用

在当前项目注入 my-superpowers 技能触发说明到 `AGENTS.md`，并生成项目级 code-review 配置（subagent + rule）。

## 执行

调用 `setup-my-superpowers` skill 执行完整流程。该 skill 会：

1. 在 `AGENTS.md`（或 `CLAUDE.md`）注入技能触发说明
2. 观察目标项目结构，推断技术栈和构建命令
3. 读取 skill 目录下的 `templates/code-review/agent.md` 和 `rule.md` 模板
4. 在目标项目生成 `.codebuddy/agents/code-reviewer.md`（subagent 定义）
5. 在目标项目生成 `.codebuddy/rules/requesting-code-review.md`（触发 rule，默认 `enabled: false`）

详细执行步骤见 `Skills/setup-my-superpowers/SKILL.md`。

## 后续操作

执行完成后，用户需要：

- 手动将 `.codebuddy/rules/requesting-code-review.md` 中的 `enabled: false` 改为 `true` 才会自动触发 review
- 重启会话让 AGENTS.md 触发生效
- 可自行编辑生成的两个文件，加入项目特定审查维度或编译命令
