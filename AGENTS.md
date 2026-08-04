# yes-subagents

子代理编排技能，帮助 agent 主动判断场景、组合子代理、安排多轮迭代与门禁检查。

## 当前状态

- **版本真源：** `skill/SKILL.md` frontmatter 的 `metadata.version`
- **当前状态：** 迭代 4 收尾完成——R1（eval 13 重标记 3/3 通过）、R2（id 17 薄层跳过 3/3 通过）、R3（回归 id 5/6/7/10/11 全过）已执行完毕，with_skill 28/28 通过；待用户确认 evals.json 改动（eval 13 断言改写 + id 17 新增）后提交
- **迭代记录：** `yes-subagents-workspace/iteration-1/` → `iteration-2/` → `iteration-3/` → `iteration-4/`
- **评估项目：** 迭代 1 在自己项目上测试、迭代 2/3 在 `cli_dotlink` 上测试、迭代 4 在 `app_viz-jzy`（kaisa-event-dashboard）测试
- **迭代 3：** 重写 b10b8bd 的虚构用例（id 6-12 落地 cli_dotlink + 沙盒）；发现 BLOCKED 补全漏洞（eval11 FAIL）并已修复验证通过；全部 12 用例 with_skill 数据完整，33/33 已评分断言通过；详见 `yes-subagents-workspace/iteration-3/REPORT.md`
- **迭代 4：** 基于 Sketch 插件任务真实反馈，新增「计划审查阈值」（薄计划层可跳过 critic-strict 但需显式声明理由；偏离已审草案为机械触发），v0.3.0 已落地并完成评估。厚层拦截（id 14/15）与偏离机械触发（id 16）3/3 全部通过；eval 13 的 2 个 FAIL 判定为用例前提错误（单文件改动但编号规则实为非平凡生成性选择，agent 正确触发审查且抓到 without_skill 漏掉的真实缺陷），非 skill 缺陷；详见 `yes-subagents-workspace/iteration-4/REPORT.md`

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
