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

4. **生成 code-review 配置**（询问用户是否启用，默认同意）：

   a. **读取模板**：从 my-superpowers 仓库读取以下两个模板文件：
      - `templates/code-review/agent.md`
      - `templates/code-review/rule.md`

   b. **观察目标项目**：检查目标项目根目录，识别以下信息（不询问用户，直接观察）：
      - 项目名称（从 `package.json` / `Cargo.toml` / `pyproject.toml` / `go.mod` 等读取）
      - 技术栈（manifest 文件类型 + 主要依赖）
      - 关键目录结构（`list_dir` 看一级目录）
      - 性能/规模硬指标（如 manifest 或 README 中有提及则记录，否则填"无"）
      - 构建命令（按技术栈推断，如 `cargo build` / `npm run build` / `pytest`；多技术栈项目逐项列出；无明显构建步骤则注明"本项目无编译步骤"）

   c. **创建 subagent 定义**：在目标项目创建 `.codebuddy/agents/code-reviewer.md`：
      - 复制 `templates/code-review/agent.md` 内容
      - 填充 `{{PROJECT_NAME}}` / `{{TECH_STACK}}` / `{{KEY_DIRECTORIES}}` / `{{PERFORMANCE_CONSTRAINTS}}` 占位符
      - 删除 `<!-- ... -->` 注释段（含 SETUP 时需填充提示）
      - 保留审查维度、问题分级、输出格式等通用内容不动

   d. **创建触发 rule**：在目标项目创建 `.codebuddy/rules/requesting-code-review.md`：
      - 复制 `templates/code-review/rule.md` 内容
      - 用第 b 步推断的构建命令替换 `{{BUILD_COMMANDS}}` 占位符
      - 保留 `enabled: false`（用户后续主动开启）
      - 删除 `<!-- ... -->` 注释段

   e. **如已有同名文件**：检查目标项目 `.codebuddy/agents/code-reviewer.md` 和 `.codebuddy/rules/requesting-code-review.md` 是否已存在：
      - 存在 → 跳过生成，告知用户"已存在，未覆盖，如需重新生成请先手动删除"
      - 不存在 → 按上述步骤生成

5. **告知用户**：报告修改了哪些文件、加了什么内容：
   - `AGENTS.md` 注入了哪些触发说明
   - 生成了 `.codebuddy/agents/code-reviewer.md` 和 `.codebuddy/rules/requesting-code-review.md`
   - 提醒 code-review rule **默认 `enabled: false`**，需要手动改为 `true` 才会自动触发
   - 提醒重启会话让触发生效
   - 告知用户可自行编辑两个生成文件，加入项目特定审查维度或编译命令
