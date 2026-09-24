# INDEX

> [!IMPORTANT]
> Headline 2 为主题聚类, 无主题内容默认放在 Headline 1 中. 同主题条目达到 3 条时, 为它们创建独立主题.
> 每页一行, 先查阅编译后的知识, 按需查阅源材料:
> `- [title](<file>.md) | <context>k | <一句话描述> | tag, tag`

- [编排者轮询导致上下文暴涨](orchestrator-polling-context-bloat.md) | 2k | 后台子代理等待期轮询烧掉约 2/3 token；解药是回合结束不终止任务，由完成通知触发职责链下一环 | subagent, context-bloat
- [多代理运行的费用核算与价值基准](multi-agent-run-cost-accounting.md) | 2k | 账单 = turns × 平均上下文；核算须含全部嵌套子代理会话，并以 VD、TPVD、CPVD、CPTK 区分交付效率与价格构成 | subagent, cost, pricing, token-accounting
- [Coding workflow 重构决策](coding-workflow-redesign-decisions.md) | 3k | coding 消费已批准计划并编排施工：Coder Plus 在 Milestone 内整合叶子并创建原子提交，以 start/delivery HEAD 和 owned-clean 状态稳定交付 gate | subagent, orchestration, coding, workflow
- [Coding 评估 R1 观察与 R2 边界](coding-eval-r1-r2-observation.md) | 1k | R1 证明主要流程合规但暴露质量探针与 oracle 缺口；R2 先修评估包并暂缓 skill 变更 | subagent, orchestration, coding, evaluation, eval
- [增量复审契约的 fresh-agent 盲区](delta-review-fresh-agent-blindspot.md) | 2k | 增量复审只在修复为局部编辑时安全；结构性返工下产出只对 payload 成立的 PASS，须以骨架字段触发器机械回落全量重审 | review, orchestration, contract
- [Strict 飞轮四轮上限与生命周期重启决策](strict-flywheel-four-round-decision.md) | 1k | 一次生命周期为 1 次初审加最多 4 次修正；停滞早停，耗尽后须用户授权且工件有实质变化才用 fresh reviewer 重启 | review, orchestration, strict, flywheel
