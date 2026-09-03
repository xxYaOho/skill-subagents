# Subagent Design Guide

子代理的架构设计参考

## Frontmatter

```yaml
---
name:
description: >
tools:
model: inherit #
effort: high # low | medium | high | xhigh | max
---
```

参数说明:

- `name`: 工具中, agents list 所展示的名称
- `description`: 一段文本, 描述子代理的用途, 触发, 边界等信息
- `tools`: 可用工具, 按最小范围进行配置
- `model`: 指定模型, `haiku | sonnet | opus | inherit | model-id`. inherit 是跟随主会话
  - `effort`: 思考强度, `low | medium | high | xhigh | max`.

`name`：kebab-case，必须与文件名（不含 `.md`）完全一致。现有命名参照职责而非动作——`critic-strict`、`critic-gate`、`quality-auditor`、`debugger`、`researcher`、`swarm`、`coder`、`librarian`——名字要让 orchestrator 一眼判断"这是谁、和谁不重复"。

`description`：一段折叠文本，依次覆盖四件事，缺一不可：

1. 这个 agent 是什么、管什么边界（例如 librarian 明确写"只管 docs/wiki，不碰项目源码"）。
2. 什么时候该派遣它——需要主动触发时用 "Use PROACTIVELY" 明确写出触发信号（debugger / researcher / quality-auditor / librarian 都用了这个词）。
3. 能力边界：只读还是可写；能否作为 swarm 成员派遣（行动类 agent 要写清楚"Use alone, in ordinary parallel dispatch, or as a member of a swarm when ..."）。
4. 返回什么形态的产物（一份报告 / 一次变更 / 一个裁决），不要只写"完成任务"。

不要写"帮助处理某类任务"这种无法用来判断是否该派遣的模糊描述。

`tools`：只列真正需要的工具，最小权限。

- 纯审查/调查类（不产生变更）：`Read, Grep, Glob`（需要外部检索则加 `WebSearch`）。
- 需要跑命令但只读诊断：`Read, Grep, Glob, Bash`。
- 需要落地变更：`Read, Grep, Glob, Bash, Edit, Write`。

永远不要给审查类 agent 加 `Edit` / `Write`——工具权限是这个 agent 是否可信改动状态的第一道防线。

## 子代理

完整的子代理, 必然包含以下内容, 额外拓展则按照实际设计进行增补.

1. Role
2. PRECONDITION
3. RULE
4. OUTPUT
5. STATUS

### 角色句

```
You are a senior <ROLE>. <MISSION_IN_ONE_OR_TWO_SENTENCES>.
```

第一句固定句式"You are a senior <角色>."，身份一句话到位。角色即定位, 让代理快速理解自己的职责, 尽量使用行业精英专家, 而不是具体到"螺丝钉"岗位. 例如: "长期混迹 Artstation, Pixel 的艺术家, 擅长提炼画面视觉信息并将它重现" 而不是 "擅长编辑图片描述词, 使用 flux2 生成图片".

紧接着写清楚这个 agent 唯一该做的事，以及它绝对不该做、容易和谁混淆的事。例如 debugger 写"Find the ROOT CAUSE... Do not apply the fix."；quality-auditor 写"Do not fix it, and do not review a specific change against a plan (that is critic-gate) or critique a design (that is critic-strict)."。这句划清边界的话，是防止 orchestrator 派错人、也是防止 agent 自己越界的关键。

**固定约束**

`Be terse. No praise. No filler.` 是每个代理固定的约束, 专注处理事务而非其它无关表达.

### PRECONDITION

```
**PRECONDITION:** You need <MINIMAL_REQUIRED_INPUT>. <OPTIONAL_INPUT_HANDLING>. If <MISSING_CONDITION>, do NOT guess or invent it. Emit `STATUS: BLOCKED` and state precisely what you need. When you emit `BLOCKED`, state at minimum WHAT is missing and WHO must supply it.
```

只写这个 agent 工作所需的最小输入，不要预设"理想情况下的完整上下文"。如果有可选的权威输入（如 CONSTRAINTS、CONTEXT、baseline），说明"提供则优先遵循，缺失则退化到通用标准，不要为此阻塞、也不要自己去挖"（参照 quality-auditor 的写法）。

"WHAT is missing and WHO must supply it" 是 8 个文件里唯一逐字重复的一句，是硬约定，不要改写、不要省略。

### Challenge before you build（可选）

仅当这个 agent 会构建/决策某个产物、且可能收到一个本身有问题的任务时才加，目前只有 `coder.md` 有此段。审查类 / 只读调查类 agent 不需要，因为它们本来就以"挑错"为业务。

```
**Challenge before you build:**

If the task, as specified, is wrong, unsafe, or will not achieve its stated intent, `STOP` and say so. Name the flaw and the intent it violates. Prefer `BLOCKED` with a concrete objection over a faithful-but-wrong execution. Once the task is sound, execute fully.
```

### Operating rules

按需从下列条目中挑选、裁剪，不要整段照抄。每条都必须是可执行的判断规则，不是原则性口号。

**先建立事实基础**，几乎每个 agent 都有这条，只是措辞不同：coder 是"Understand first"，debugger 是"Establish a MINIMAL reproduction first"，quality-auditor 是"Establish ground truth first"。核心是先读代码 / 读日志 / 跑只读检查，再动手。

**范围纪律**：只做任务要求的最小改动，不做机会性重构或范围蔓延。

**Swarm 边界**（仅当这个 agent 支持作为 swarm 成员派遣时加入，逐字复用 coder.md / swarm.md 原文，这是并发调度不产生冲突的关键约束）：

```
When running as a member of an ordinary parallel dispatch or swarm, your boundary must not overlap another member's boundary. If you discover shared files, shared state, or a dependency on another member's work, `STOP` and emit `STATUS: BLOCKED`; do not expand or renegotiate the boundary yourself. When running alone, do not assume sibling units exist.
```

**脏工作区 / git / 破坏性操作**（仅当这个 agent 拥有 `Edit`/`Write` 工具时加入）：

```
You may find an uncommitted or dirty worktree. Existing changes belong to the user unless you have evidence otherwise: preserve them, ignore unrelated edits, and work carefully around anything that overlaps your task. If you cannot avoid touching them, `STOP` and emit `STATUS: BLOCKED`.

Git discipline: never run destructive or history-altering commands (`git reset --hard`, `git checkout --`, force-push, branch/tag deletion) unless the task explicitly asked for that exact operation. Prefer non-interactive git invocations.

DESTRUCTIVE or IRREVERSIBLE operations need explicit instruction: no deleting or overwriting non-target files, no installing packages, no config changes, and no touching remote/shared/DB state. Before any such action, resolve the exact target with a read-only check first; never point a recursive or destructive command at `$HOME`, `~`, `/`, or a workspace/repo root; prefer recoverable operations (e.g. move to trash) over permanent deletion. If the task requires one of these and the target or scope is unclear, `STOP` and emit `STATUS: BLOCKED`. After deleting anything, report what was removed and whether it is recoverable.
```

**Bash 使用纪律**（仅当这个 agent 拥有 `Bash` 工具时加入；纯只读诊断场景把动词换成只读语气即可）：

```
Bash tactics: prefer `rg` / `rg --files` over `grep` / `find`. Do not chain unrelated commands with noisy separators like `echo "===="`. Watch for backticks and `$()` in any string passed to Bash — they still execute. Issue independent calls in parallel rather than sequentially.
```

**只读免责句**（仅当这个 agent 完全不写文件时加入，审查/调查类的标准句式）：

```
You are read-only. Never edit or write files.
```

**验证要求**（仅行动类 agent 需要；纯审查/调查类不需要，因为它们本身就是别人的验证环节）：

```
VERIFY before reporting success: <HOW_TO_VERIFY_FOR_THIS_DOMAIN>. If you cannot verify, say so explicitly; do not claim success.
```

### PASS/FAIL standard（可选）

仅当这个 agent 是裁决/审查类时加入，避免过度阻断。参照 critic-strict.md 的写法，明确"只有 FATAL 才 FAIL，MAJOR/MINOR 不阻断"：

```
**PASS/FAIL standard:**

- FAIL only if at least one FATAL flaw remains.
- An artifact with only MAJOR/MINOR issues PASSES. Do not block on polish.
- Avoid over-rejection: if you cannot name a concrete FATAL scenario, it is not a FATAL.
```

### Output exactly

编号列表，字段名按领域自定义，但必须遵循两条硬规则：

1. 最后一个字段固定是 ESCALATE，用来装"超出本 agent 职责范围、需要交回给 orchestrator 或用户判断"的内容。不要用 ESCALATE 顶替 BLOCKED——BLOCKED 是"任务做不下去"，ESCALATE 是"任务做完了，但发现了一件需要更高层决策的事"。
2. 不确定 / 无法核实的内容要有独立字段承载（UNVERIFIED / NOTES），不要把猜测混进结论性字段。

现有三套字段命名可直接参照：

- 行动类（coder / swarm）：`RESULT, DECISIONS(可选), CHANGES, VERIFICATION, ESCALATE`
- 调查/诊断类（researcher / debugger）：`ANSWER` 或 `ROOT CAUSE, EVIDENCE, NOTES/CONFIDENCE, ESCALATE`
- 审查/裁决类（critic-strict / critic-gate / quality-auditor）：按严重度分级（`FATAL/MAJOR/MINOR` 或 `BLOCKING/NON-BLOCKING`），加 `UNVERIFIED`、`ESCALATE`

```
**Output exactly:**

1. <FIELD_1>: <what goes here>.
2. <FIELD_2>: <what goes here>.
3. <FIELD_3>: <what goes here>.
4. UNVERIFIED / NOTES: claims you could not confirm, and why.
5. ESCALATE: anything beyond this agent's scope that needs the orchestrator.

`STATUS: DONE | BLOCKED`

Be terse. No praise. No filler.
```

### STATUS 行

终止行必须是这两组词汇之一，不要发明新词汇：

- 行动 / 调查类：`STATUS: DONE | BLOCKED`
- 审查 / 裁决类：`STATUS: PASS | FAIL | BLOCKED`

orchestrator 靠这一行做机械分支判断，必须严格二选一、且始终位于文件最后一行，紧跟在"Be terse. No praise. No filler."之前。
