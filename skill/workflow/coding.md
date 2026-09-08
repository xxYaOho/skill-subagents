# Coding

适用于已经完成需求讨论、准备按计划实施的编程工作。本 workflow 是**计划消费与施工编排器**：确认计划可施工，判断薄厚层级，组织执行、gate 与收尾。计划生产由 `go-spec` 负责；通用组合、飞轮、待机和完成条件由 `SKILL.md` 负责。

```text
取得已批准计划
  ↓
判断薄计划 / 厚计划
  ├─ 薄计划 → Orchestrator 自己实施
  └─ 厚计划 → Milestone → coder-plus
  ↓
同一 gate reviewer 持续核验
  ↓
全部节点 PASS → pruner → scout
  ↓
完整交付
```

## 1. 进入施工

默认输入是一份由 `go-spec` 或等价方法产出的完整计划。开工前确认：

- 目标、范围和验收标准可判定；
- 实施单元具有明确边界、依赖和下游接口承诺；
- 计划作者已经完成 self-review；
- 计划已经由作者之外的 fresh `critic-strict` 审查并取得 `STATUS: PASS`；
- 用户已经授权进入实施。

self-review 只检查完整性、一致性和格式，不替代独立 critique。无论计划薄厚，strict 都由 fresh-agent 承担。

Strict `PASS` 后，MAJOR/MINOR 作为非阻断事项进入施工；在批准基线内消化或披露不触发复审，处置若改变目标、范围、结论、节点、依赖、验收、证据集、交付物或外部契约，则作为结构性新版本重新进入 `FULL`。

缺少计划、计划仍不完整或尚未通过 strict 时，转交 `go-spec` 与用户补齐，不在本 workflow 内临时生产或修补计划。strict 修正轮次遵循 `critic-strict` 的增量输入契约。

施工线程只接收批准计划、strict PASS 凭证和未阻断事项，不继承计划作者或 strict 的完整 transcript，也不重复已经完成的计划审查。

## 2. 判断计划层级

厚薄描述施工结构，不描述改动行数，也不决定是否需要 strict。

**薄计划**：只有一条连贯的实施路径，可作为一个整体直接验收，不需要 Milestone 协调多个内部单元。

**厚计划**：执行层包含 Milestone；每个 Milestone 内含多个 Task、内部依赖，或需要统一验证的交付主题。

按计划已有结构判断：

- 薄计划由 Orchestrator 自己实施，不派 `coder`。
- Milestone 首层映射到 `coder-plus`；一个 Milestone 对应一次派遣。
- 不把 Milestone 扁平拆成多个 Task 后由 Orchestrator 逐项直派。
- `coder-plus` 自己决定 Milestone 内如何实施，并对整体验收负责。
- `coder` 可由 Coder Plus 用作 Milestone 内的叶子，不代表薄计划执行者。
- 风险、迁移、新颖性和验证难度影响 gate 的核验重点，不改变薄/厚施工模式。

若薄计划实际隐藏多个内部单元、依赖链或独立验证阶段，说明计划结构失真。停止施工并返回 `go-spec` 修正；不要在实施期自行发明 Milestone。

## 3. 准备施工上下文

薄计划由 Orchestrator 持有批准计划，按需读取相关代码并直接实施，不创建子代理施工包。

厚计划每次只向 Coder Plus 交付当前 Milestone，不发送完整计划、后续节点或历史讨论。施工包至少包含：

- 当前 Milestone 的目标与实施内容；
- 验收标准；
- 可动文件范围；
- 依赖、约束和下游接口承诺。

施工包使用 `go-spec` 的 `Milestone-<n>.md` 切片或等价内容，并满足 `coder-plus` 的 PRECONDITION。Task 归属于该 Milestone，只由 Coder Plus 消费。

若当前节点依赖上一节点的实际结果，只补充已经确认的接口或事实，不附上上一节点 transcript。

## 4. 推进施工

### 薄计划

Orchestrator 直接完成全部实施与直接验证，不为分派而拆分工作。完成后，将批准计划、改动、实现取舍和验证证据交给本计划的独立 `critic-gate`。未取得 `STATUS: PASS`，不进入收尾。

### 厚计划

按计划顺序逐个推进 Milestone：

1. 将当前 Milestone 施工包一次性交给 `coder-plus`。
2. Coder Plus 默认亲自实施；仅在并行、上下文隔离或专门能力收益明确时分派叶子。
3. 收到 `STATUS: DONE` 后，检查其 DELEGATION、DEVIATIONS、CHANGES 与 VERIFICATION。
4. 由当前计划的 gate reviewer 核验整个 Milestone。
5. 只有 `STATUS: PASS` 后，才进入下一 Milestone。

Orchestrator 不干预 Coder Plus 的内部 Task 分派，也不为其叶子增加独立 gate。叶子完成只代表局部输入；Milestone 是否完成，以 Coder Plus 的统一验证和计划级 gate 为准。

## 5. Gate 生命周期

一个批准计划只建立一个 gate reviewer，并在全部节点、FAIL 修正和 Gate 要求的补证轮次中持续复用。

- 薄计划由 Orchestrator 实施：从计划开始创建一个独立 `critic-gate`，并持续复用。
- 厚计划由 Coder Plus 实施：Orchestrator 兼任整个计划的 gate reviewer，审查他人产出不属于 self-critique。
- 实施者负责 self-verification，但不能作最终 gate。
- 同一 reviewer 只读取当前节点基线、实现结果、验证证据和已接受偏差；不重复 strict，也不重读无关计划层。

gate 对照批准计划当前节点判断：验收是否满足、实现是否忠实、下游契约是否保持、是否引入回归。实现方式可以不同，但必须向 reviewer 如实披露；Coder Plus 通过 DEVIATIONS 记录。

`STATUS: FAIL` 时，将阻断项交回原执行者修正；条件允许时延续同一执行会话。修正后仍由同一个 gate reviewer 复核。`STATUS: BLOCKED` 按 `SKILL.md` 的缺失项规则处理，补齐必要证据后复用原 reviewer。

`PASS + UNVERIFIED` 只允许包含不影响 Gate 结论的事项，由 Orchestrator 完成直接验证，不自动回传 reviewer。若补证与已审声明矛盾、暴露缺陷或导致实现变化，则恢复原 Gate 生命周期；决定结论所必需的证据未确认时，Gate 应返回 `BLOCKED` 而不是 `PASS`。

gate 发现下列情况时，不继续局部修补：

- 需要修改目标、范围或验收标准；
- 需要改变已批准的设计或 Milestone 边界；
- 当前实现暴露出计划无法成立。

此时停止施工，回到 `go-spec` 记录变更并重新取得 fresh `critic-strict` PASS。受影响计划重新批准后，重新建立 gate 生命周期。

## 6. 提交与计划收尾

子代理不创建提交。节点 gate PASS 后，是否提交以及提交粒度由 Orchestrator 按用户授权和仓库规则决定。

全部实施节点 PASS 后，只运行一次 `pruner`，并提供本计划实际触及的文件边界。Pruner 自行派 `scout` 核验清理目标，只执行 `SAFE` 项；完成后直接验证，不再回到 critic-gate。

Pruner 不在每个 Task 或 Milestone 后运行。若计划尚未完整执行，例如评估只观察前一至两个 Milestone，则不触发计划级收尾。

## 7. 交付

完成意味着：

- 所有计划节点均已通过同一 gate reviewer；
- 必要提交已由 Orchestrator 完成；
- 计划级 Pruner/Scout 收尾已完成；
- 最终验证覆盖完整交付，而不只是单个叶子或节点。

向用户汇报最终结果、关键偏差、验证证据、未阻断事项，以及任何未能完成的验证。不要复述子代理过程日志。
