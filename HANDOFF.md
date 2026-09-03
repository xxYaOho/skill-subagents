# Handoff Kanban

> 接手说明: 从下方全局动态及其关联文档中恢复工作状态, 向用户简要说明已知现状与下一步.
> 存在课题组时, 额外提供课题组列表 (代号和一句话描述).
> 若引用的 commit / 分支 / 文件已不存在, 或工作区存在文档未提及的改动, 先向用户指出差异.
> 接手后先做一次简单复验: 派遣子代理 (skill: yes-subagent) 复查文档的真实性, 核对其陈述与工作区实际状态是否一致, 差异之处向用户指出.
> 说明完毕后等待用户的进一步安排, 不要自行开工.

v0.5.0（coder-plus 里程碑执行者 + gate 默认化）已落地提交 `44530a2`；AGENTS.md / coding.md / SKILL.md 含杨桃手改（勿回退，SKILL.md L80 有笔误待其确认）。当前停在 iteration-6 evals：三用例设计稿（E1 全流程/E2 单工作单元/E3 不可逆外派 gate，micro-note fixture）已呈现**待确认**，确认后才改写 evals.json、搭 fixture、单轮 spawn 3 个 with-skill 运行（无 baseline、无 viewer）。

[交接文档 V6：v0.5.0 实现完成与 evals 执行包](.tmp/handoff/260903171256-v050-coding-evals.md)
