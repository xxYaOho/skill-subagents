# yes-subagents

子代理编排技能，帮助 agent 主动判断场景、组合子代理、安排多轮迭代与门禁检查。

## 当前状态

- **版本真源：** `skill/SKILL.md` frontmatter 的 `metadata.version`
- **当前状态：** 迭代 5 收尾（版本保持 0.4.0）。新增 pruner（收尾清理）/ scout（删除安全核验）两子代理并完成全部验证：v2 复跑 9 PASS / 1 FAIL / 1 NOT GRADED；id 20 scout 首次取证（派遣链 tool_use_id 实证）；四项决策落地后 v3 复跑 6 PASS / 1 未取得数据。关键契约：pruner 持 Agent 工具嵌套派遣 scout（自检不替代）、收尾范围文件级、删除纪律（禁 rm -rf/通配符）、原子提交为里程碑退出条件。架构发现 12：capable pruner 起草即全仓 grep，scout 价值定位为「独立双确认 + 动态引用盲区覆盖 + 越界发现外溢上报」。下轮候选：description 自然触发 A/B、grep 不可见引用形态的甄别 fixture。详见 `yes-subagents-workspace/iteration-5/REPORT.md`（本地工作区，已 gitignore）
- **迭代记录：** `yes-subagents-workspace/iteration-1/` → `iteration-2/` → `iteration-3/` → `iteration-4/` → `iteration-5/`
- **评估项目：** 迭代 1 在自己项目上测试、迭代 2/3 在 `cli_dotlink` 上测试、迭代 4 在 `app_viz-jzy`（kaisa-event-dashboard）测试、迭代 5 在 `app_peony`（双沙盒克隆 + eval-base tag）测试
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
- 子代理分类：监察类（critic-strict / critic-gate / scout）、调查类（quality-auditor / debugger / researcher）、行动类（swarm / coder / pruner）
- 子代理契约与设计规范见 `skill/subagents/SCHEMA.md`；工作流模板见 `skill/workflow/`

## 迭代 2 关键验证结果

- 无 self-critique、无异步竞态、PASS 后正确停止
- 在 cli_dotlink 上 with_skill 发现了 `STORE_FILES.writeStore` 运行时崩溃，without_skill 漏掉了

## 语言策略

- SKILL.md 保持中文（匹配触发和交互习惯）
- 发布英文版单独维护 `SKILL.en.md`
