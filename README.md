# my-superpowers

个人精简版 superpowers，专为 CodeBuddy 适配。

## 包含技能

- **using-superpowers** — 引导层，教 agent 任何动作前先查技能
- **brainstorming** — 写代码前先把想法变成设计
- **test-driven-development** — 红绿重构，先写测试

## 安装

在 CodeBuddy 中将此目录注册为插件市场，然后启用 my-superpowers 插件。

## 与原版 superpowers 的区别

- 只保留 3 个技能（原版 14 个）
- 使用 CodeBuddy 原生 `.codebuddy-plugin` 格式
- 无 SessionStart 钩子注入，靠项目 AGENTS.md 触发
- 无跨平台适配（只支持 CodeBuddy）

## 触发方式

本插件不做全局引导注入。在需要使用的项目里，于 `AGENTS.md` 或 `CLAUDE.md` 中添加技能触发说明，例如：

```markdown
## 使用 my-superpowers（特别重要）

会话开始时先调用 using-superpowers 技能，它会指导你在合适时机使用其他技能。

- 构建/改造功能前 → 先用 brainstorming 技能
- 写实现代码前 → 先用 test-driven-development 技能
```
