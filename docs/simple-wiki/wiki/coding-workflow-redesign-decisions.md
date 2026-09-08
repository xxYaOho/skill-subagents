---
title: Coding workflow 重构决策
type: decision
created: 2026-09-08
updated: 2026-09-09
sources: [raw/coding-workflow-redesign-discussion-2026-09-08.md, raw/coder-plus-milestone-lead-decision-2026-09-08.md, raw/coding-workflow-plan-consumer-implementation-2026-09-08.md, raw/coding-workflow-thin-plan-execution-correction-2026-09-08.md]
topic: Coding workflow
tags: [subagent, orchestration, coding, workflow, evaluation]
status: current
context: 2
---

# Coding workflow 重构决策

本页记录 2026-09-08 已落地的 coding workflow 合同。`workflow/coding.md` 是计划消费与施工编排器：它消费已批准计划，选择施工协作形态，推进计划级 gate 与收尾；计划生产和结构修正由 `go-spec` 负责。完整背景见[《Coding workflow 重构讨论纪要》](../raw/coding-workflow-redesign-discussion-2026-09-08.md)，Coder Plus 合同见[《Coder Plus 里程碑负责人设计决策》](../raw/coder-plus-milestone-lead-decision-2026-09-08.md)，施工消费合同见[《Coding workflow 计划消费实现合同》](../raw/coding-workflow-plan-consumer-implementation-2026-09-08.md)。

## 职责边界

- `SKILL.md` 保留角色、组合、飞轮、待机、监察与完成条件等编排原语和全局原则。
- `go-spec` 生产并维护决策层、Draft、Implement Plan、Milestone 及派生切片；发现结构性偏差时接收回流并重新定稿。
- `workflow/coding.md` 消费已批准计划，判断薄厚实施层，选择施工协作形态，推进 gate、偏差分流和计划级收尾。
- `subagents/*.md` 定义各角色收到任务后的内部执行契约。

coding 不重复通用飞轮、最大轮次、等待机制、计划生产过程或子代理完整契约。

## 流式多线程评估

Step 是评估阶段，不等于线程。计划作者 self-review、fresh strict、施工 Orchestrator、执行者和 gate reviewer 分属不同上下文，评估重点是职责分离和工件传递。薄、厚计划分别进入以下流水线：

```text
P-thin  -> 通过审查的薄计划  -> E-thin
P-thick -> 通过审查的厚计划  -> E-thick-M1 -> E-thick-M2（可选）
```

self-review 只检查完整性、一致性和格式，不替代独立 critique；strict 对两种计划都交给 fresh-agent。Step 1 只向 Step 2 交付批准计划、PASS 凭证和未阻断事项，不传完整 transcript；Step 2 不重复 strict，只观察实施、自执行、分派和 gate。厚实施默认只跑一个 Milestone，只有需要观察跨节点流转时才增加第二个。稳定回归使用冻结的已批准计划，动态接力只承担集成观察。

## 计划消费与施工入口

开工必须取得完整计划、作者 self-review、作者之外 fresh `critic-strict` 的 PASS，以及用户的实施授权。施工线程只接收批准计划、PASS 凭证和未阻断事项，不继承计划作者或 strict 的完整 transcript，也不重复计划审查。

计划结构决定实施方式：薄计划只有一条连贯的实施路径，可作为整体直接验收，由 Orchestrator 自行实施，不人为拆成 Task 或派遣 `coder`；厚计划包含 Milestone，Orchestrator 每个 Milestone 只派遣一次 `coder-plus`，不把其内部 Task 扁平直派。风险、迁移、新颖性和验证难度只改变 gate 的核验重点，不替代薄厚计划的结构判断。该薄计划规则取代早先材料中的 Task→coder 映射（[Coding workflow 薄计划执行纠正](../raw/coding-workflow-thin-plan-execution-correction-2026-09-08.md)）。

每次派遣只交付当前节点。Task 的最小施工包包含目标与实施内容、验收标准、可动文件范围、依赖、约束和下游接口承诺；Milestone 使用 `go-spec` 的 `Milestone-<n>.md` 切片或等价内容。后续节点和历史 transcript 不进入执行者上下文。

## Coder Plus 里程碑负责人

`coder-plus` 是已批准 Milestone 的独立负责人，偏向代码执行的简易 Orchestrator。它默认亲自完成里程碑内全部工作；只有分派能带来明确的并行收益、上下文隔离或专门能力时，才调用叶子代理。无论是否分派，它都负责整合结果并统一验证整个 Milestone。

允许的非编排型叶子角色包括：边界明确的 `coder`；多个简单、同质且不重叠单元的 `swarm`；事实性不确定项的 `researcher`；根因未知故障的 `debugger`；仅用于核验迁移、改写、删除或破坏性动作静态安全前提的 `scout`。禁止派遣 `coder-plus`、`critic-strict`、`critic-gate`、`quality-auditor`、`pruner` 或任何仍可继续派遣的代理，代理树保持两层：Orchestrator -> Coder Plus -> leaves。

关键路径、强耦合、共享文件或状态、需要连续判断，或简报与整合成本不低于直接完成的工作，默认不分派。叶子返回 `STATUS: DONE` 只代表局部完成；Coder Plus 必须核对证据与改动，完成 Milestone 验收后再交付。可补充事实输入的 BLOCKED 可在同一单元补充后继续；若需扩大边界、修改验收标准、改变计划决策或取得用户授权，必须向 Orchestrator 返回 BLOCKED。

Coder Plus 输出必须包含 `DELEGATION`：记录助手、分派收益和整合结果；全部亲自完成时写 `None`。`DECISIONS` 只记录无法从 diff 恢复的非显然取舍。Coder Plus 不负责计划审查、最终 gate、提交或计划级清理。

## 计划级 Gate 与偏差

一个批准计划只建立一个持久 gate reviewer，跨全部节点、FAIL 修正和补证轮次复用同一 reviewer。子代理实施时由 Orchestrator 兼任；Orchestrator 自行实施任一节点时，从计划开始创建独立 `critic-gate` 并持续复用。实施者可 self-verify，但不能作最终 gate。

局部实施问题交回原执行者并尽量延续原会话，由同一 gate reviewer 复核。若出现结构性偏差，或需要修改目标、范围、验收、设计或 Milestone 边界，则停止施工并回到 `go-spec`，重新取得 fresh strict PASS，同时重建 gate 生命周期。计划结构性变更、reviewer 不可用或上下文失真时也重建 gate。

## 计划级收尾与提交

完整计划的全部实施节点 gate PASS 后，才运行一次 `pruner -> scout`；评估只观察前一至两个 Milestone 时不触发计划级收尾。Pruner 只处理 Scout 判定为 SAFE 的目标，随后直接验证并完成收束，不在每个 Milestone 后重复运行。

执行者不创建提交。节点 PASS 后是否提交及提交粒度，由 Orchestrator 按用户授权和仓库规则决定。

## 关联

- [多代理运行的费用核算与价值基准](multi-agent-run-cost-accounting.md) —— 计划级流水线和嵌套派遣的效率评估，应按 TPVD、CPVD、CPTK 的全会话核算口径观察。
