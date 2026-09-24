---
date: 2026-09-24
state: 已对齐
dispatch: ~
---

# coder-plus-milestone-commit-autonomy Plan

## Problem Statement

当前 coding workflow 禁止 Coder Plus 创建提交，要求 Milestone 通过 gate 后再由 Orchestrator 决定提交。实际 Milestone 往往横跨多个厚重 Task；全部改动长期停留在工作区，会失去持续恢复点，扩大 diff，并增加故障定位和局部回退成本。

## Solution

将 commit 与 gate 解耦：Coder Plus 负责 Milestone 内部的原子提交，commit 只表示已整合并自验证的施工检查点；gate 仍对整个 Milestone 的最终实现作独立裁决。叶子不提交，由 Coder Plus 统一整合。gate PASS 后 Orchestrator 推进下一 Milestone 或计划级收尾，不强制造新的提交。

## User Stories

1. 作为 Milestone 实施负责人，我希望在施工过程中创建原子提交，以便厚重节点拥有可恢复、可定位和可回退的检查点。
2. 作为 Orchestrator，我希望 commit 与 gate 裁决保持分离，以便不把“已持久化”误认为“已验收”。
3. 作为 gate reviewer，我希望审查完整 Milestone 的最终状态，以便提交数量和拓扑不改变验收对象。

## Constraints

- Coder Plus 仍不拥有计划审查、最终 gate、远程推送或计划级清理职责。
- 叶子代理不创建提交；Coder Plus 检查、整合并统一验证后提交。
- 提交只包含当前 Milestone 拥有的改动，不混入用户或其他工作单元的变化。
- Orchestrator 派遣前记录开工 `HEAD`、dirty 状态与已知归属；DONE 时全部 Milestone-owned 改动必须已提交，gate handoff 由开工/交付 HEAD 和 owned-clean 状态稳定界定。
- gate 开始后冻结受审历史；FAIL 修复使用追加提交，不改写已审对象。
- 运行时内置角色若禁止必要提交能力，不得假装合同已满足；改用具备完整读写、命令和叶子分派能力的通用代理注入完整 Coder Plus 合同；无合格代理时 BLOCKED。

## Implementation Decisions

1. Coder Plus 自主决定原子提交粒度，不强制 Task 与 commit 一一对应；较重 Task 可拆成多个逻辑完整的提交。
2. workflow 定义 Git 生命周期与状态转换：Coder Plus 施工并提交，gate 审查整个 Milestone，PASS 后 Orchestrator 推进，FAIL 后原执行者追加修复提交。
3. Coder Plus 的可写叶子收到明确的“不提交”约束，局部结果由 Coder Plus 整合后提交。
4. Coder Plus 输出增加 `COMMITS`，报告本 Milestone 创建的提交和剩余未提交改动。
5. 薄计划仍由 Orchestrator 自行实施并按仓库规则提交；计划级 Pruner 仍不提交，其清理改动由 Orchestrator 收口。
6. Coder Plus 在已有提交后 BLOCKED 时保留恢复点并报告剩余状态；Orchestrator 不推进下游，事实补齐后续用原执行会话，结构变化则回流 go-spec。

## Testing Decisions

- 静态检查现行合同、当前 wiki 与活跃评估不再保留“Coder Plus 零提交”的要求；历史 raw 和旧计划保留被取代论断用于溯源。
- JSON 评估清单必须保持可解析。
- coding-ledger package doctor、仓库 diff check 与 Simple Wiki doctor 必须通过。
- 本轮只验证合同与评估包，不运行完整多代理行为 eval；运行时 fallback、叶子零提交、无 push 与 gate 后历史冻结留给独立 eval 验证。

## Out of Scope

- 不引入 Milestone 专用分支、worktree、squash 或 merge commit 协议。
- 不改变 gate reviewer 的选择和持续复用规则。
- 不改变 Milestone 间的串并行调度。
- 不授权自动 push 或其它远程操作。
- 不在本轮运行完整多代理行为 eval 或声称运行时行为已经得到证明。

## Further Notes

本变更只调整施工持久化职责。commit 不是验收凭证，gate 也不以 commit 数量或拓扑作为 PASS 条件。

Simple Wiki 写入前，`doctor.sh --detail ingest` 返回 `SCHEMA_STATUS=legacy` 且 10 份 raw 缺少 `integrated`。yes-swiki v0.6 禁止在 legacy schema 上继续写入，因此迁移是本次知识同步的前置；它只更新受管 SCHEMA 与历史 raw frontmatter，并作为独立原子提交，便于单独回退。

## Draft Plan

- 结构总览：`workflow/coding.md` 持有 Git 生命周期；`subagents/coder-plus.md` 持有 Milestone 内提交自治；评估包验证叶子零提交、Coder Plus 原子提交和 gate 状态流转；Simple Wiki 记录新决策。
- 关键契约：Coder Plus 可提交、叶子不可提交、gate 前后历史冻结边界、PASS 只触发推进、运行时角色能力冲突时使用完整合同注入。
- 文件结构：修改 `skill/SKILL.md`（含版本真源）、`skill/workflow/coding.md`、`skill/subagents/coder-plus.md`、`README.md` 与相关 eval 工件；新增本次 raw 决策并更新既有 Coding workflow wiki 页面。Simple Wiki schema 迁移独立提交。
- 风险与未决：评估包原有“执行者零提交”断言需要区分薄计划显式禁提交与厚计划 Coder Plus 提交；运行时内置角色可能保留更高优先级限制。本轮不实测多代理运行时行为，必须在交付中保留该未验证边界。
- 里程碑草案：单一 Milestone 完成合同、评估、知识库与验证闭环。

## Implement Plan

本计划只有一个合同变更 Milestone。先修改真源与角色契约，再同步所有已知消费者和评估，最后完成 wiki 收录、静态验证、提交与推送。

## Milestone-1

Coder Plus Milestone 内提交自治

- 验证主题：角色权限、workflow 状态流转、eval 断言和项目知识对同一提交/gate 模型达成一致。
- 核验基线：现行合同、当前 wiki 与活跃 eval 无旧要求残留；版本真源为 `0.7.0`；JSON、coding-ledger doctor、Simple Wiki doctor 与 `git diff --check` 通过。运行时行为明确标记为未独立 eval。
- 来源：User Story 1-3；Implementation Decision 1-6。

### Task-1

修改 workflow 与角色合同（来源：User Story 1-3）

- 验收标准：Coder Plus 拥有 Milestone 内原子提交自治；叶子不提交；gate 审查完整状态；PASS 推进而非强制提交。
- 可动文件范围：`skill/SKILL.md`、`skill/workflow/coding.md`、`skill/subagents/coder-plus.md`、`README.md`。
- 依赖：无。
- 固有风险：内置角色限制覆盖项目合同。

- [x] 修改合同真源与运行时兼容路由。

### Task-2

同步评估消费合同（来源：Testing Decisions）

- 验收标准：micro eval 与 coding-ledger 不再要求 Coder Plus 零提交，并明确叶子零提交、gate 后推进。
- 可动文件范围：`skill/evals/evals.json`、`skill/evals/coding-ledger/**`。
- 依赖：Task-1。
- 固有风险：旧断言残留导致新合同被误判失败。

- [x] 更新 prompt、rubric、truth card、报告模板与说明。

### Task-3

知识同步与验证（来源：User Story 1-3；Testing Decisions）

- 验收标准：legacy schema 前置迁移独立提交；raw-first 收录完成；wiki 页面与 LOG/INDEX 状态有效；全部静态验证通过；行为 eval 未运行边界保留。
- 可动文件范围：本计划、`docs/simple-wiki/**`。
- 依赖：Task-1、Task-2。
- 固有风险：Simple Wiki legacy schema 必须先迁移。

- [x] 完成 wiki schema 迁移、决策收录与共享状态接线。
- [x] 运行全部针对性检查。

> [!IMPORTANT]
> Milestone-1 闸门：合同真源、角色契约、评估断言与知识页必须一致；任何一层仍把 commit 当作 gate 结果，均不得交付。

## Strict MAJOR 台账

- `RESOLVED` Gate handoff 未稳定：新增开工/交付 HEAD、DONE owned-clean 与 gate 后历史冻结合同；owner：Orchestrator / Coder Plus；关闭点：Task-1。
- `RESOLVED` 脏工作区归属不可识别：施工包记录 dirty 基线和归属，同文件无法证明隔离时 BLOCKED；owner：Orchestrator / Coder Plus；关闭点：Task-1。
- `RESOLVED` 静态测试不能证明运行时行为：交付结论收窄为合同与评估包验证，完整行为 eval 明确未运行；owner：Orchestrator；关闭点：Task-2 / 最终交付。
- `RESOLVED` README 与版本验收遗漏：纳入文件边界并锁定 `skill/SKILL.md` 版本 `0.7.0`；owner：Orchestrator；关闭点：Task-1。
- `RESOLVED` Wiki schema 迁移混入功能提交：记录 doctor 前置证据并拆为独立原子提交；owner：Orchestrator；关闭点：Task-3。
