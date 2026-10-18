# Fork Notes（本 fork 说明）

This is [@leether](https://github.com/leether)'s fork of [deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness).

## 为什么 fork / Why this fork exists

上游（upstream）目前**不接受外部 PR**（见上游 `CONTRIBUTING.md`，最后修改 2026-08-12）。本 fork 用于：

- 保留一个**已修复**的源码副本，便于本地从源码运行 / 调试 / 复用
- 等官方开放 PR 后，可直接从修复分支开 PR

## 分支结构 / Branches

| 分支 | 内容 |
|------|------|
| `master` | 与上游 master 一致（纯镜像）。2026-10-18 快进到 `687f282e`（对应 npm `0.2.1-rc.2`） |
| `fix/tool-runtime-scheduler-symbol` | 唯一修复，一个 commit（`59344e1e`，2026-10-18 rebase 到 `687f282e`） |
| `fork-notes` | 本说明文件所在分支；master 保持纯镜像 |

## 修复分支 / The fix branch

`fix/tool-runtime-scheduler-symbol`（commit `59344e1e`）：

> `fix(tools): key TOOL_RUNTIME_SCHEDULER via Symbol.for for cross-module stability`

- **文件**: `packages/core/tools/src/index.ts`（上游 master 第 481 行，一行改动）
  ```diff
  - export const TOOL_RUNTIME_SCHEDULER: unique symbol = Symbol('@deepseek-ai/dsh-tools.scheduler')
  + export const TOOL_RUNTIME_SCHEDULER: unique symbol = Symbol.for('@deepseek-ai/dsh-tools.scheduler')
  ```
- **根因（为什么改）**: `TOOL_RUNTIME_SCHEDULER` 是一把**跨模块共享**的 Symbol key。当同一进程存在多份物理 `@deepseek-ai/dsh-tools`（例如 `dsh plugin add` 装了一个把它声明为 dependency 的插件），两份副本各造一把**互不相等**的 `Symbol()`，导致 `dsh-agent-loop` 手里的 key 与 `ToolRuntime` 字段的 key 对不上，工具调度时 `ctx.tools[TOOL_RUNTIME_SCHEDULER]` 解析为 `undefined`，抛 `Cannot read properties of undefined (reading 'prepare')`。
- **验证**: 原验证在 `0.1.0-rc.6`（带重复副本的第二实例，工具调用恢复 `turn/end completed`）。2026-10-18 rebase 到 `0.2.1-rc.2` 后**未重新验证**。
- **状态**: 上游未采纳。官方在 2026-09-19 的设计笔记中明确拒绝该方向（`Symbol.for` 无法共享 scope parents / carrier keys / registration tables），改为解析层拦截。本分支仅作本地 unblocker 与参考实现保留。

## 关联 / Related

- 本修复的上游报告: https://github.com/deepseek-ai/deepseek-harness/discussions/2078
- 同根因报告: #572、#1418、#1337、#1849
- 官方设计笔记: `.agents/notes/implemented/architecture/2026-09-19-profile-resolution-lookup-order.md`
- 官方作者文档更新: `docs/user/develop/basic/publish.md`（2026-09-20，要求宿主包写 peer + dev）

## 与上游同步 / Syncing with upstream

```sh
# 方式一：服务端直接同步（会硬重置 master）
gh repo sync leether/deepseek-harness --branch master --force

# 方式二：本地
git fetch origin
git rebase origin/master fix/tool-runtime-scheduler-symbol
git push fork master fix/tool-runtime-scheduler-symbol --force-with-lease
```

---
- fork 日期: 2026-08-16
- 最近同步: 2026-10-18，基于上游 commit `687f282e`
