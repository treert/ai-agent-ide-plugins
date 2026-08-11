## 用法

```
@command://my-superpowers:setup-skills
```

在当前项目根目录的 `AGENTS.md`（或 `CLAUDE.md`）中注入技能触发说明。如果已有触发段落则更新，不重复添加。

## 执行步骤

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

4. **告知用户**：报告修改了哪个文件、加了什么内容。提醒用户重启会话让触发生效。
