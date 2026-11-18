<div align="center">
  <h1>Yes! Subagents</h1>
  <p><strong>强化子代理分派能力, 维护主会话中的上下文健康</strong></p>
  <p>单次的高消耗换取长久的稳定</p>
</div>

健康的上下文空间, 可以让 Agent 能持久地推进工作计划, 稍微缓解记忆幻觉和长文腐化. 其中不可避免地增加 token 开销, 粗略计算下来是 2 倍左右, 需要酌情使用.

## 如何添加

```bash
git clone https://github.com/xxYaOho/skill-subagents.git
cd skill-subagents
mkdir -p ~/.agents/skills
ln -s "$(pwd)/skill" ~/.agents/skills/yes-subagents
```

- 技能本体在 `skill/` 目录（含 `SKILL.md` 与 `subagents/`）。
- `~/.agents/skills/` 是跨工具通用目录：Claude Code、Kimi Code、Codex、DeepSeek Harness 都会扫描；用符号链接挂载，仓库更新后即时生效。

如果使用的是其它工作, 则自行调整 Symlink 的位置

| CLI | 用户级技能目录 |
| --- | --- |
| Claude Code | `~/.claude/skills/`，也读 `~/.agents/skills/` |
| Kimi Code CLI | `~/.kimi-code/skills/`（随 `$KIMI_CODE_HOME` 移动），也读 `~/.agents/skills/` |
| Codex CLI | `~/.agents/skills/`（跨工具通用目录；项目级为 `<repo>/.agents/skills/`） |
| DeepSeek Harness | `~/.agents/skills/`（作为共享 Agent 技能根加载） |

```bash
mkdir -p ~/.claude/skills
ln -s "$(pwd)/skill" ~/.claude/skills/yes-subagents
```

注意：链接名必须叫 `yes-subagents`，与 `skill/SKILL.md` frontmatter 里的 `name` 一致；链接目标必须是包含 `SKILL.md` 的目录本身，不要链到仓库根。

验证安装：

```bash
ls ~/.agents/skills/yes-subagents/SKILL.md
```

## 移除

```bash
unlink ~/.agents/skills/yes-subagents   # 只删链接，不影响仓库
```

## 子代理们

| 名称             | ID              | 描述                                                            |
|----------------|-----------------|---------------------------------------------------------------|
| Coder Plus     | coder-plus      | 把一个 Milestone 从实现、原子提交到自验一手包完；主会话负责 gate 核验，不代为提交；需要并行或查证时，它自己安排人手 |
| Coder          | coder           | 边界清楚的实现活, 丢过去就行, 不用你盯着。跨文件、按项目老规矩改也不含糊, 需求站不住脚它还会顶回来          |
| Swarm          | swarm           | 一批互相独立的简单活, 开一群同时跑, 互不踩脚, 各自交结果。怎么分工怎么排队, 不归它管                |
| Critic Strict  | critic-strict   | 方案定稿前请它挑刺。它没被你的思路带过, 最容易看见你自己看不见的洞, 问题还按轻重排好序                |
| Critic Gate    | critic-gate     | 验收门卫, 记性好, 改几轮都记得之前卡在哪、答应过什么。只问这次交付达没达标, 不翻架构旧账               |
| Scout          | scout           | 删东西、动结构之前, 先让它确认这么干到底安不安全。它只回答安不安全, 值不值得干不归它说                 |
| Debugger       | debugger        | 怪毛病自己查不出来时让它上, 去复现、去试错, 最后给你一句有证据的"病根在哪"。只诊断, 不开药             |
| Researcher     | researcher      | 调研、查资料这种慢活让它在旁边跑, 主线继续干正事。回来只带结论和证据, 不带一堆原始材料                 |
| Pruner         | pruner          | 活干完的收尾交给它, 脚手架和临时代码都收走。每一刀之前先让 Scout 确认安全, 只在任务碰过的地盘里动手       |
| Quality Auditor | quality-auditor | 给仓库定期体检, 先按你们自己定的规矩挑毛病, 不够再看通用标准, 最后告诉你先还哪笔债                  |
