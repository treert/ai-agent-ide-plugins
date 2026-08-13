<!--
模板用途：setup-skills 命令执行时复制此文件到目标项目 `.codebuddy/rules/requesting-code-review.md`
           并按下方"SETUP 时需填充"段落的占位符补充项目特定内容。

生成后的文件由目标项目独立维护，不再受 my-superpowers 影响。
默认 enabled: false，由用户在 setup 后主动改 true 才生效。
-->

---
description: 代码修改完成后自动审查，确保质量。也可在用户提到 review、审查代码、检查 bug 时调用。
alwaysApply: false
enabled: false
---

# 代码修改后自动 Review

每次完成一组**功能性代码修改**后（即将告知用户"完成"之前），必须执行以下流程。

## 何时触发

<!-- 此章节定义 review 触发时机，与 my-superpowers/Skills/requesting-code-review 的设计理念一致。 -->

- 完成一组功能性代码修改后（即将告知用户"完成"之前）
- 实现完一个 major feature 后
- 合并到 main 之前

可选但推荐：
- 卡住时（换个视角）
- 重构前（基线检查）
- 修复复杂 bug 后

## 跳过条件

以下场景**不需要** review：
- 单行修改
- 注释/文档变更
- 配置文件微调

## 流程

### 第 1 步：编译验证

运行构建命令，确保**零新增 error**；新增 warning 须评估并尽量消除。

<!-- SETUP 时需填充：根据项目 manifest 推断技术栈，填入对应构建命令。多技术栈项目逐项列出。若项目无明确构建步骤则删除本步并注明"本项目无编译步骤"。 -->

{{BUILD_COMMANDS}}

### 第 2 步：获取变更范围

```bash
# 获取本次变更的文件列表
git diff --name-only

# 如需对比特定范围
# BASE_SHA={{BASE_SHA}}
# HEAD_SHA={{HEAD_SHA}}
# git diff --stat $BASE_SHA..$HEAD_SHA
# git diff $BASE_SHA..$HEAD_SHA
```

明确**本次改动的目标**（用户需求摘要）。

### 第 3 步：调度 code-reviewer subagent

使用 `Task(subagent_name="code-reviewer")` 调度项目级 subagent，prompt 内传递：

- **变更文件列表**（来自第 2 步）
- **改动目标**（来自用户需求摘要）
- **Git 范围**（BASE_SHA / HEAD_SHA，如适用）

subagent 自带项目背景、审查维度、输出格式（定义在 `.codebuddy/agents/code-reviewer.md`），不需要在 prompt 内重复。

subagent 是 `readonly: true`，只读 review，不会修改工作树。

### 第 4 步：处理 Review 结果

按 `receiving-code-review` skill 的规范处理反馈：

- **Critical（Must Fix）** → 立即修复
- **Important（Should Fix）** → 修复后才能继续推进
- **Minor（Nice to Have）** → 记录稍后处理
- **错误反馈** → 用技术理由 push back，不要表演性同意
- **结论为 APPROVED** → 告知用户完成
- **结论为 CHANGES_REQUESTED** → 修复所有 Critical 问题后重新审查，直到通过

## 行为准则

- 不要因为"看起来简单"就跳过 review
- 不要忽略 Critical 问题
- 不要带未修复的 Important 问题继续推进
- 对正确的技术反馈直接修复并简述改动，不要表演性致谢
- 如 reviewer 错了，用技术理由 push back
