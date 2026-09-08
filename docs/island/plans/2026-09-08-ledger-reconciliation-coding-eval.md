# 银行 CSV 对账 Coding Workflow 评估包 Plan

> 2026-09-08 · 状态：已对齐

## Problem Statement

现有 yes-subagents 评估把需求发现、计划生成、计划审查、施工编排和业务实现放在同一条长链中，单次运行变量过多。计划层反复全文审查、施工线程继承过多历史、薄厚执行映射不稳定，使失败难以归因，也放大 token 与费用波动。

本轮需要一套新的评估包，单独验证 coding workflow 是否能消费已经批准的产品设计，正确生成施工计划，并按薄计划或厚计划组织执行、gate 与上下文传递。评估包只准备协议和 fixture，不在本轮运行评估。

## Solution

以“本地账本导入银行 CSV，真实目标是稳定对账”为统一领域 fixture，建立薄、厚两条隔离流水线。每条流水线预置已经冻结的 go-spec Decision 与 Draft；Step 1 只生成并审查执行层，Step 2 在新线程中消费批准计划并施工。

薄计划验证 Orchestrator 自己实施与独立持久 gate。厚计划验证 Orchestrator 按 Milestone 派遣 Coder Plus、Coder Plus 自执行或有收益地分派叶子、同一 Orchestrator 持续 gate。Truth Card、业务 oracle 与评分规则仅供 Eval Host 和评分者使用。

## User Stories

1. 作为评估设计者，我希望冻结产品决策和 Draft，只测试执行层计划与施工编排，以便把结果归因到 coding workflow。
2. 作为 Eval Host，我希望获得明确的线程、输入和工件传递合同，以便在不泄漏隐藏真值的情况下调用完整评估流程。
3. 作为 skill 维护者，我希望同时观察薄计划自执行、厚计划 Coder Plus 分派判断、strict 与 gate 生命周期，以便定位实际编排偏差。
4. 作为评估报告读者，我希望业务正确性、流程合规性、裁量观察和运行故障分开记录，以便基于证据讨论，而不是把所有现象压成一个 PASS 分数。
5. 作为成本观察者，我希望记录主线程与全部嵌套子代理的 token、费用和已验证交付单元，以便统一计算 TPVD、CPVD 与 CPTK。

## Constraints

- Step 1 的计划作者与 fresh `critic-strict` 必须是不同线程；self-review 不替代 strict。
- Step 2 必须使用新线程，只接收批准计划、strict PASS 凭证和未阻断事项，不继承 Step 1 transcript。
- 薄计划由 Step 2 Orchestrator 自己实施，不派 `coder`；最终 gate 使用独立且持续复用的 `critic-gate`。
- 厚计划每个 Milestone 只由 Orchestrator 首层派遣一次 `coder-plus`，不把 Milestone 内 Task 扁平直派；整个计划由同一个 Orchestrator gate。
- Coder Plus 默认自己实施，叶子分派只在并行、上下文隔离或专长收益明确时使用；`researcher` 不构成必经角色或评分要求。
- Truth Card、oracle、断言和陷阱说明不得提供给计划作者、strict、施工 Orchestrator、Coder Plus 或其叶子。
- 评估期间不修改 yes-subagents；skill 变更只能在 REPORT 完成并与用户讨论后进行。
- fixture 使用 Python 标准库，避免依赖安装、网络和包管理器成为评估噪音。
- 业务范围仅含 CNY，业务时区固定为 `Asia/Shanghai`。
- 本轮不运行 eval，不生成 REPORT 结论，不编译 swiki，不提交。

## Implementation Decisions

1. 新评估包独立于历史 `evals.json`，使用自己的 runbook、manifest、cases、fixture、oracle 与报告模板；旧评估保持历史状态。
2. Eval Host 是最外层主持者，只负责准备隔离副本、按阶段派遣、依据 Truth Card 回答被问到的问题、采集 transcript 与 token、运行机械验证并汇总 REPORT。它不替被测线程完成计划或施工判断。
3. 薄、厚各有一套可见的冻结 Decision 与 Draft。它们是 Step 1 的输入，不参与产品设计质量评分；Step 1 产出 Implement Plan、必要切片、自检记录和 strict 凭证。
4. 薄流水线包含 `P-thin -> E-thin`。厚流水线包含 `P-thick -> E-thick-M1 -> E-thick-M2`，以两个 Milestone 观察同一 gate reviewer 的连续性。
5. Step 1 与 Step 2 之间使用显式 handoff 清单。禁止传递原始需求讨论、Truth Card、评分断言、strict transcript 或后续 Milestone 内容。
6. 账本以 `bank + account + transaction_id` 作为来源唯一键。重复键且规范化业务字段相同为 duplicate；相同键但金额、时间、币种或 target 不同为 conflict，不静默覆盖。
7. 金额使用分为单位的整数。保留 bank、account、transaction ID、原始行号与原始字段。无效行进入 rejected；合法行仍可导入。
8. `target` 表示交易对象、商户或收付款对方，保留原值并只做空白规范化。`category` 优先读取 CSV 字段并映射到固定分类；无法映射时为“未分类”，不做 AI 推断或用户规则系统。
9. 周统计按周一至周日，月统计按自然月，均以交易发生时间和 `Asia/Shanghai` 归属周期；退款以负向金额计入。
10. 业务 oracle 同时覆盖重复导入、重复外观但不同交易号、同键冲突、无效行、多银行命名空间、退款、周/月边界与分类汇总。
11. 评分分为四层：机械业务正确性、机械流程合规性、裁量观察、运行有效性。Coder Plus 是否分派某种叶子只进入裁量观察；只有越界、错误层级或缺失整体验证才构成流程失败。
12. token 统计覆盖 Eval Host、计划作者、strict、施工 Orchestrator、gate、Coder Plus 及全部嵌套叶子；报告 VD、TPVD、CPVD、CPTK，不使用旧称 TPD。

## Testing Decisions

- fixture 基线测试只验证已有行为与种子数据可读，不提前实现被测需求。
- 提供独立的 oracle 校验脚本，从黑盒 CLI、账本输出和 git diff 读取结果，不依赖被测实现的内部函数。
- 提供 package doctor，检查必需文件、JSON 格式、CSV 编码、fixture 基线测试、隐藏材料未被 Step prompts 引用。
- prompts 和 manifest 使用稳定 case ID；评分结果必须引用 transcript、文件或命令输出证据。
- invalid run 单独判定：模型或供应商失败、超时、缺少 transcript、fixture 污染、隐藏材料泄漏或阶段输入错误。invalid 不计入 skill PASS/FAIL。
- 每条流水线默认单次运行，用于首次协议校准；是否增加重复样本由首份 REPORT 的方差证据决定，不在包内预设虚假统计显著性。

## Out of Scope

- 不重新评估 go-spec 的需求发现、Problem/Solution 重构或 Draft 产品设计质量。
- 不与 without-skill、历史版本或其他 planning skill 做 A/B 比较。
- 不测试真实银行格式、外汇、转账配对、余额校验、数据库迁移、可视化图表或交互式分类管理。
- 不要求 Coder Plus 使用 `researcher`、`scout`、`coder` 或 `swarm` 中的任何指定角色。
- 不在本轮修改 `skill/SKILL.md`、`workflow/coding.md`、子代理合同或历史 eval 工件。

## Further Notes

“薄计划由 agent 自己实施”中的 agent 指 E-thin 施工线程内的 Orchestrator，不是最外层 Eval Host。厚计划的 gate reviewer 是 E-thick 施工 Orchestrator；该线程应跨 M1 与 M2 持续存在。

## Draft Plan

- 结构总览：评估包分为主持协议、隐藏真值、薄/厚 case、可复制 fixture、机械 oracle、采集 schema 与报告模板。Eval Host 从 manifest 路由阶段，各阶段只读取显式白名单输入。
- 关键契约：case 使用稳定 ID；阶段结果写入各自 artifact 目录；handoff 只列批准工件和凭证；机械评分与观察记录分开；invalid run 在汇总前隔离。
- 文件结构：`skill/evals/coding-ledger/README.md` 说明入口；`manifest.json` 描述阶段图；`host/` 保存 Truth Card、采集与报告合同；`cases/thin/`、`cases/thick/` 保存冻结 Decision/Draft 与阶段 prompt；`fixture/ledger-cli/` 保存基线项目；`fixture/inputs/` 保存 CSV；`oracle/` 保存预期结果和验证脚本。
- 风险与未决：子代理工具在不同 harness 中的角色名可能不同，prompt 以责任合同为准并允许等价 built-in role；token 字段可能缺失，必须标记缺测而非填零；报告不得用业务失败推断编排失败。
- 里程碑草案：M1 冻结协议、真值与可见上游计划；M2 建立确定性 fixture 和数据 oracle；M3 完成阶段 prompts、机械检查与 REPORT 模板。

## Implement Plan

按验证主题顺序完成三个里程碑。M1 先锁定信息边界，M2 再提供所有阶段共享的确定性运行环境，M3 最后把阶段调用和评分接到已冻结合同上。文件职责以 Draft Plan 为准。

验证主题地图：

1. M1：评估职责与信息隔离可独立审阅。
2. M2：基线 fixture 与 mock 数据确定、可复制、可机械验真。
3. M3：fresh Eval Host 可仅依赖包内说明完成全流程并产出可讨论 REPORT。

## Milestone-1

冻结协议、Truth Card 与上游计划

- 验证主题：评估对象、线程拓扑、输入白名单、业务真值以及薄/厚可见上游设计互相一致。
- 核验基线：README、manifest、Truth Card、两套 Decision/Draft 经静态检查无职责冲突、无隐藏材料泄漏、无未决占位。
- 来源：User Story 1、2、3、4；Constraints 1-6。

### Task-1

建立 Eval Host 协议与隐藏真值（来源：User Story 1、2、4）

- 验收标准：入口说明可由 fresh-agent 独立执行；Truth Card 逐项定义对账、统计与应答口径；阶段输入白名单和 invalid 条件明确。
- 可动文件范围：`skill/evals/coding-ledger/README.md`、`manifest.json`、`host/`。
- 依赖：本计划的 Constraints 与 Implementation Decisions。
- 固有风险：Truth Card 泄漏给被测线程会使评估失效；主持职责过宽会污染被测行为。

- [x] 写入主持运行合同和目录约定。
- [x] 写入隐藏 Truth Card、证据采集 schema 与 REPORT 模板骨架。
- [x] 定义四层评分与 invalid run 分流。

### Task-2

冻结薄、厚两套 Decision 与 Draft（来源：User Story 1、3）

- 验收标准：两套材料只包含决策层和 Draft；薄方案是一条整体可验收路径且无 Milestone 暗示；厚方案明确两个验证主题但不预写 Task 或施工步骤。
- 可动文件范围：`skill/evals/coding-ledger/cases/thin/Decision.md`、`Draft.md`、`cases/thick/Decision.md`、`Draft.md`。
- 依赖：Task-1 的 Truth Card；go-spec 0.1.2 分层合同。
- 固有风险：早期材料过度暗示执行形态会把被测答案写进 prompt；材料不足又会迫使 Step 1 重新做产品设计。

- [x] 编写薄计划可见上游材料。
- [x] 编写厚计划可见上游材料。
- [x] 对照 Truth Card 检查覆盖与非泄漏边界。

> [!IMPORTANT]
> Milestone-1 闸门：主持合同、Truth Card 与四份上游材料必须一致；任何被测线程都不能通过可见路径读取 Truth Card、oracle 或评分规则。

## Milestone-2

建立确定性账本 fixture

- 验证主题：fixture 基线、mock CSV 与预期业务结果形成可重复的黑盒环境。
- 核验基线：基线测试全绿；输入文件覆盖所有 Truth Card 场景；复制后运行不依赖网络或外部包。
- 来源：User Story 2、4；Implementation Decision 6-10。

### Task-1

实现最小账本 CLI 基线（来源：User Story 2）

- 验收标准：Python 标准库 CLI 能读取种子账本、列出交易并执行既有 Alpha 银行导入；测试不包含待实现的 dry-run、多银行、冲突报告或统计能力。
- 可动文件范围：`skill/evals/coding-ledger/fixture/ledger-cli/`。
- 依赖：M1 的业务 Truth Card。
- 固有风险：基线过强会吞掉被测任务，过弱会把薄计划膨胀成隐含重构。

- [x] 建立 CLI、账本存储、Alpha CSV 适配与基线测试。
- [x] 固定种子账本和命令输出。
- [x] 运行标准库测试确认基线。

### Task-2

构造 CSV 数据集与 oracle（来源：User Story 4、5）

- 验收标准：输入覆盖 duplicate、look-alike distinct、conflict、rejected、Beta bank、refund、周/月边界和分类；expected JSON 给出逐行处置和汇总真值。
- 可动文件范围：`skill/evals/coding-ledger/fixture/inputs/`、`oracle/expected.json`、`oracle/README.md`。
- 依赖：Task-1 的基线 schema 与 CLI。
- 固有风险：日期、金额正负或重复键计算错误会让 oracle 自身成为噪音源。

- [x] 写入 Alpha、Beta、冲突与无效数据。
- [x] 手工推导并结构化记录预期账本与周月汇总。
- [x] 用独立脚本复算关键金额与条数。

> [!IMPORTANT]
> Milestone-2 闸门：fixture 基线测试全绿，所有 oracle 数字可由输入和种子账本独立复算，且基线尚未实现被测功能。

## Milestone-3

接通阶段调用、验证与报告

- 验证主题：fresh Eval Host 可从 README 出发完成 P-thin、E-thin、P-thick、E-thick-M1/M2，并把事实写入统一报告。
- 核验基线：package doctor 全绿；每个 prompt 的可见输入与输出合同符合 manifest；报告模板能区分业务、流程、裁量与 invalid。
- 来源：User Story 2、3、4、5；Testing Decisions。

### Task-1

编写 Step 1 与 Step 2 prompts（来源：User Story 2、3）

- 验收标准：计划线程完成 Implement/self-review/fresh strict；施工线程只消费批准工件；薄层自执行，厚层按 Milestone 派 Coder Plus；不强制 researcher 或其他叶子。
- 可动文件范围：`skill/evals/coding-ledger/cases/thin/`、`cases/thick/`、`host/dispatch-contract.md`。
- 依赖：M1 全部工件、M2 fixture 路径与命令。
- 固有风险：prompt 直接复述评分点会把评估退化为关键词服从测试。

- [x] 编写 P-thin、E-thin、P-thick、E-thick prompts。
- [x] 定义 Step 1 -> Step 2 handoff 及 Milestone 增量输入。
- [x] 检查 prompt 不包含隐藏断言或指定叶子角色答案。

### Task-2

实现 package doctor 与业务 oracle 校验（来源：User Story 2、4）

- 验收标准：doctor 检查目录、JSON、禁漏规则和基线测试；oracle verifier 能分别检查 thin、thick M1、thick M2 的黑盒结果，并对未完成阶段给出明确状态。
- 可动文件范围：`skill/evals/coding-ledger/scripts/`、`oracle/`。
- 依赖：Task-1 prompts 与 M2 fixture。
- 固有风险：验证器耦合实现细节会误罚合法实现；脚本不得修改被测工作区。

- [x] 编写只读 package doctor。
- [x] 编写分阶段黑盒 verifier。
- [x] 对未修改基线运行 verifier，确认待实现能力按预期失败而 doctor 仍通过。

### Task-3

完成采集字段与 REPORT 模板（来源：User Story 4、5）

- 验收标准：模板记录阶段工件、线程身份、派遣树、gate reviewer 连续性、验证结果、偏差、invalid 事件和全树 token；TPVD、CPVD、CPTK 公式明确，裁量项不伪装为硬断言。
- 可动文件范围：`skill/evals/coding-ledger/host/observation-schema.json`、`host/REPORT_TEMPLATE.md`、根 README。
- 依赖：Task-1、Task-2。
- 固有风险：缺失嵌套会话或把未返回 token 记为零会扭曲效率结论。

- [x] 固定观察记录 schema。
- [x] 完成 REPORT 模板与指标公式。
- [x] 运行 package doctor 和 `git diff --check`，保留 eval 未运行声明。

> [!IMPORTANT]
> Milestone-3 闸门：包内静态检查与 fixture 基线测试全绿；业务 verifier 对未实施基线呈现预期缺口；没有任何 eval 运行或结果被伪造。
