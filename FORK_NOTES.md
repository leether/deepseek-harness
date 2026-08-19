# Fork Notes（本 fork 说明）

This is [@leether](https://github.com/leether)'s fork of [deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness).

## 为什么 fork / Why this fork exists

上游（upstream）目前**不接受外部 PR**（见上游 `CONTRIBUTING.md`）。本 fork 用于：

- 保留一个**已修复**的源码副本，便于本地从源码运行 / 调试 / 复用
- 等官方开放 PR 后，可直接从修复分支开 PR

## 分支结构 / Branches

| 分支 | 内容 |
|------|------|
| `master` | 与上游 master 保持一致（干净，无本地改动；2026-08-20 重置回 origin/master=rc.8） |
| `fix/tool-runtime-scheduler-symbol` | 唯一的修复（一个 commit，可直接复用 / 开 PR；rc.8 上游仍未修复，继续有效） |
| `fork-notes` | 本说明文件所在分支（2026-08-20 从 master 挪出，master 恢复纯镜像） |

## 修复分支 / The fix branch

`fix/tool-runtime-scheduler-symbol`（commit `b72ee99907`）：

> `fix(tools): key TOOL_RUNTIME_SCHEDULER via Symbol.for for cross-module stability`

- **文件**: `packages/core/tools/src/index.ts`（一行改动）
  ```diff
  - export const TOOL_RUNTIME_SCHEDULER: unique symbol = Symbol('@deepseek-ai/dsh-tools.scheduler')
  + export const TOOL_RUNTIME_SCHEDULER: unique symbol = Symbol.for('@deepseek-ai/dsh-tools.scheduler')
  ```
- **根因（为什么改）**: `TOOL_RUNTIME_SCHEDULER` 是一把**跨模块共享**的 Symbol key。当同一进程存在多份物理 `@deepseek-ai/dsh-tools`（例如 `dsh plugin add` 装了一个把它声明为 dependency / peerDependency 的插件），两份副本各造一把**互不相等**的 `Symbol()`，导致 `dsh-agent-loop` 手里的 key 与 `ToolRuntime` 字段的 key 对不上，工具调度时 `ctx.tools[TOOL_RUNTIME_SCHEDULER]` 解析为 `undefined`，抛 `Cannot read properties of undefined (reading 'prepare')`。`Symbol.for` 走全局注册表，跨任意份副本返回同一把 key，同时仍是 Symbol（保留原作者「不进命名服务 API」的 `@internal` 意图）。
- **验证**: 0.1.0-rc.6；带重复副本的第二实例，工具调用恢复 `turn/end completed`。

## 关联 / Related

- 本修复的上游报告: https://github.com/deepseek-ai/deepseek-harness/discussions/2078
- 前人的同根因报告: #572、#1418、#1337、#1849

## 与上游同步 / Syncing with upstream

```sh
git fetch origin
git rebase origin/master                        # master 保持与上游一致
git push fork master
git rebase origin/master fix/tool-runtime-scheduler-symbol   # 需要时把修复 rebase 到最新上游
git push fork fix/tool-runtime-scheduler-symbol
```

---
- fork 日期: 2026-08-16
- 基于上游 commit: `47f943859b`
