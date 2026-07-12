# yes-subagents

子代理编排技能，帮助 agent 主动判断场景、组合子代理、安排多轮迭代与门禁检查。

## 当前状态

- **版本：** v2（迭代 2 完成）
- **迭代记录：** `yes-subagents-workspace/iteration-1/` → `iteration-2/`
- **评估项目：** 迭代 1 在自己项目上测试、迭代 2 在 `cli_dotlink` 上测试

## 核心设计决策

1. **审查必须分离：** 草案、方案、报告不自己审，直接丢给 critic-strict。self-critique 是反模式。
2. **PASS 即停：** critic-strict/critic-gate 返回 PASS（0 FATAL）后停止迭代，剩余 MAJOR/MINOR 汇报用户判断。
3. **等结果再改：** 派完审查等返回，不预判结论提前修改。
4. **时期确认：** 每个流程阶段结束停住等用户确认，全自动需用户明确要求。
5. **流程是参考模式不是脚本：** agent 内化原则后自主判断。

## 子代理

- `./skill/subagents/` — 本地子代理定义
- 与内置子代理职责重复时优先内置，但在指令中注入输出契约（PRECONDITION / Output exactly / STATUS）
- 子代理分类：监察类（critic-strict / critic-gate）、调查类（quality-auditor / debugger / researcher）、行动类（swarm / coder）

## 迭代 2 关键验证结果

- 无 self-critique、无异步竞态、PASS 后正确停止
- 在 cli_dotlink 上 with_skill 发现了 `STORE_FILES.writeStore` 运行时崩溃，without_skill 漏掉了

## 语言策略

- SKILL.md 保持中文（匹配触发和交互习惯）
- 发布英文版单独维护 `SKILL.en.md`
