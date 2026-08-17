# Agent Note: contribution 提供的 Trajectory 调用定位与标签

Status: implemented

[English](2026-08-18-trajectory-contributed-call-placement.md) | 中文

## 问题

Trajectory 已经可以接收注册式 Conversation Definition 独立 contribution 的 Tool tree。Contribution 携带 Conversation Location，但 Trajectory 快照此前只保留普通 Event Node 的 Location。独立根 Tool 完成后，layout 因此会把它当成没有归属的结果，放入 Turn 序言。这样会让本应属于 Step 的调用树排到触发它的 User 与 Context 记录之前。

固定的 Tool 与 Subtool 标签也会隐藏外部 Definition 提供的业务角色。调用方可以把角色写入 Tool 名称，但标签仍然显示 `TOOL` 或 `SUBTOOL`；若修改核心种类，又会重复表现与交互行为。

## 决策

Trajectory Builder 为每个独立 contribution 的根调用保留其 Location。Layout 会为运行中与已完成的独立根调用查询该 Location。解析成功的 Step Location 会把完整调用树放入对应 Step；Location 缺失或尚未解析时保持原有回退行为。Assistant 自有的 Tool call 继续跟随其 Assistant Step，不需要单独 Location。

Tool contribution 可以提供可选 map，按根调用或后代调用 ID 指定语义显示标签。Layout 会把标签复制到对应记录；标签存在时，表格、检查器、无障碍行名称、tooltip 和搜索索引都会使用它。记录仍属于 `tool` 或 `subtool`；颜色、图标、折叠、嵌套、计时、选择与检查行为仍由种类决定。

这一扩展属于 [Trajectory 组装决策](2026-08-11-trajectory-conversation-context-assembly.md)所定义的既有 target 专属 contribution 约定。它不会引入第二套 history source、新的 Event 族，也不会让 Trajectory 了解任何外部工作流产品。

## 分发与上游同步

Chat 维护的派生源码位于私有 `later-3/deepseek-harness-chat` 仓库。`origin/main` 是已发布派生源码的事实来源，官方 `deepseek-ai/deepseek-harness` 仓库是只读 `upstream`，隔离开发使用 `codex/*` 分支。Chat 运行时仍固定 rc.6 npm 发行包并应用可审核补丁，因此本仓库拥有开发与上游汇合历史，Chat 仓库拥有部署版本、补丁 Hash 与运行时漂移检查。

上游发布版本或相关 Trajectory 变更会从`origin/main`的隔离 worktree 开始。维护者先判断上游是否提供等价的公开 contribution 字段；存在等价约定时删除派生差异，而不是继续保留。否则只重放 Location 保留和语义显示标签，再运行受影响测试、仓库 typecheck、bundle、lint 与文档检查。Chat 补丁、lock Hash、版本证据和浏览器验证从该已验证提交更新，之后私有默认分支才能前进。

## 考虑过的替代方案

**把每棵独立 Tool tree 都放到最后一条输入之后。** 不予采纳：缺少 Conversation Location 时，排序只能依赖启发式规则；它无法正确处理多个 Step，也会丢弃 assembler 已提供的归属事实。

**要求每棵外部调用树都带有 Assistant Tool-call 头。** 不予采纳：伪造 Assistant 消息会改变执行记录，并把外部工作流错误表现为模型发起的工具调用。

**新增 workflow 和 agent 记录种类。** 不予采纳：业务专属种类会复制核心表现行为，并把 Trajectory 耦合到单一集成。语义标签可以表达角色，同时不改变交互语义。

**只把角色写入 Tool 名称。** 不予采纳：事件标签、无障碍名称、检查器、tooltip 与搜索词汇仍只会报告通用种类。

## 验证

Builder 测试固定根 Location 与语义标签的保留行为。Layout 测试固定运行中及已完成调用的 Step 定位、后代顺序与标签投影。Cell 和表格测试固定可见标签与检查器，同时确认底层仍为 `tool` 或 `subtool` 种类。既有测试覆盖未提供两个可选输入的 contribution，因此原有行为保持不变。

## 后果

外部 Definition 可以在真实 Step 位置投影忠实的业务调用树，无需伪造 Conversation Event，也无需 fork 完整 Trajectory 视图。标签刻意仅用于表现，因此集成不能借它改变层级或交互规则。

快照现在多携带两个小型 map，其规模与独立 contribution 的调用树成正比。同一快照中的调用 ID 共用一个标签命名空间；Definition 作者必须使用稳定且无冲突的 ID，并把缺失 Location 或标签视为通用回退情形。
