<!-- README-I18N:START -->

[English](./README.en.md) · **简体中文**

<!-- README-I18N:END -->

<div align="center">

<img src="./assets/cover.zh.png" alt="Luv Letter — 写给自己内在世界的一封情书" width="100%">

</div>

# Luv Letter

*写给自己内在世界的一封情书 · A slow reflection space for AI agents*

<br>

Portable **Agent Skill**，让 AI 不再急于回答，而是陪你把念头慢慢展开——像一封写给内在世界的信，有倾听、有回响、有留白，也有被郑重收好的记忆。

> We do not give answers. We extend thinking.

<br>

[项目结构](#项目结构) · [安装](#安装) · [系统](#系统) · [使用](#使用) · [记忆](#记忆) · [示例](#示例) · [常见问题](#常见问题) · [文档](#文档)

<br>

---

<br>

## 项目结构

本仓库是 **Luv Letter 开源项目**，兼容 [Agent Skills 开放标准](https://github.com/vercel-labs/skills)——可在 **Cursor、Claude Code、Codex、OpenCode** 等 Agent 中使用。Skill 源码在 `skills/`，安装时由 CLI 或手动复制到各 Agent 对应目录。

```text
Luv-Letter/
├── README.md                          # 中文说明
├── README.en.md                       # English
├── LICENSE
├── LuvLetterProduct-Specification.md  # 产品愿景
├── assets/
│   ├── cover.zh.png
│   └── cover.en.png
└── skills/
    └── luv-letter/                    # Agent Skill 源码（分发目录）
        ├── SKILL.md
        ├── orchestrator.md
        ├── memory.md
        ├── memory-tool.md
        ├── react-loop.md
        ├── examples.md
        └── modules/
```

运行时记忆（用户数据，不入库）默认落在使用方项目的 `.luv-letter/memory/`。

<br>

---

<br>

## 安装

Luv Letter 通过 [`npx skills add`](https://github.com/vercel-labs/skills) 安装——与 [taste-skill](https://github.com/Leonxlnx/taste-skill) 相同，CLI 会扫描本仓库 `skills/` 目录并写入对应 Agent 路径。

### 一键安装（推荐）

```bash
# 安装到 CLI 检测到的 Agent（交互式选择）
npx skills add https://github.com/SanbaoAI/Luv-letter --skill luv-letter

# 指定 Cursor / Claude Code / Codex / OpenCode
npx skills add https://github.com/SanbaoAI/Luv-letter \
  --skill luv-letter \
  -a cursor -a claude-code -a codex -a opencode \
  -y

# 全局安装（所有项目可用）
npx skills add https://github.com/SanbaoAI/Luv-letter \
  --skill luv-letter -g \
  -a cursor -a claude-code -a codex -a opencode \
  -y

# 本地开发（在本仓库根目录）
npx skills add . --skill luv-letter -a cursor -a claude-code -a codex -a opencode -y
```

仓库地址：[github.com/SanbaoAI/Luv-letter](https://github.com/SanbaoAI/Luv-letter)。可用 `npx skills add ... --list` 预览 skill 列表。

### 各平台路径与唤起

| Agent | CLI `-a` 参数 | 项目路径 | 全局路径 | 如何唤起 |
| --- | --- | --- | --- | --- |
| **Cursor** | `cursor` | `.agents/skills/luv-letter/` | `~/.cursor/skills/luv-letter/` | 对话中 `@luv-letter` |
| **Claude Code** | `claude-code` | `.claude/skills/luv-letter/` | `~/.claude/skills/luv-letter/` | 输入 `/luv-letter` |
| **Codex** | `codex` | `.agents/skills/luv-letter/` | `~/.codex/skills/luv-letter/` | 描述任务，或提及「Luv Letter 模式」 |
| **OpenCode** | `opencode` | `.agents/skills/luv-letter/` | `~/.config/opencode/skills/luv-letter/` | 重启后说「进入 Luv Letter 模式」 |

> 路径来源：[vercel-labs/skills 支持列表](https://github.com/vercel-labs/skills#supported-agents)。部分 Agent 也兼容 `.cursor/skills/` 等旧路径，以 CLI 实际写入为准。

安装后 **Codex / OpenCode 建议重启** 以加载新 skill。验证：`npx skills list`。

### 手动安装

若不用 CLI，将 `skills/luv-letter/` **整个文件夹**（含 `modules/` 等全部文件）复制到对应路径：

<details>
<summary><strong>Cursor</strong></summary>

```bash
# 项目级
mkdir -p .cursor/skills   # 或 .agents/skills
cp -r /path/to/Luv-Letter/skills/luv-letter .cursor/skills/

# 全局
cp -r /path/to/Luv-Letter/skills/luv-letter ~/.cursor/skills/
```

唤起：`@luv-letter`

</details>

<details>
<summary><strong>Claude Code</strong></summary>

```bash
# 项目级
mkdir -p .claude/skills
cp -r /path/to/Luv-Letter/skills/luv-letter .claude/skills/

# 全局
cp -r /path/to/Luv-Letter/skills/luv-letter ~/.claude/skills/
```

唤起：`/luv-letter`（目录名即命令）

文档：[Claude Code Skills](https://code.claude.com/docs/en/skills)

</details>

<details>
<summary><strong>Codex</strong></summary>

```bash
# 项目级
mkdir -p .agents/skills
cp -r /path/to/Luv-Letter/skills/luv-letter .agents/skills/

# 全局
mkdir -p ~/.codex/skills
cp -r /path/to/Luv-Letter/skills/luv-letter ~/.codex/skills/
```

唤起：重启 Codex 后，自然描述「Send into Luv」或 slow reflection 需求；Codex 按 `SKILL.md` 的 `description` 自动匹配。

文档：[Codex Skills](https://developers.openai.com/codex/skills)

</details>

<details>
<summary><strong>OpenCode</strong></summary>

```bash
# 项目级
mkdir -p .agents/skills
cp -r /path/to/Luv-Letter/skills/luv-letter .agents/skills/

# 全局
mkdir -p ~/.config/opencode/skills
cp -r /path/to/Luv-Letter/skills/luv-letter ~/.config/opencode/skills/
```

唤起：退出并重启 OpenCode → 说「进入 Luv Letter 模式」或 `list installed skills` 确认已加载。

文档：[OpenCode Skills](https://opencode.ai/docs/skills)

</details>

> 只粘贴 [SKILL.md](./skills/luv-letter/SKILL.md) 可以开口，但完整体验需要 `skills/luv-letter/` 下**全部文件**（orchestrator、memory、modules 等）。

<br>

---

<br>

## 系统

Luv Letter 不是一条 prompt，而是一套会**呼吸**的反思系统。每个部件做一件事；你不需要一次理解全部，只要送出一句话，它会自己运转。

| 部件 | 名字 | 它在做什么 |
| --- | --- | --- |
| **Orchestrator** | 心跳调度 | 每一轮先感受语境，再决定：继续深入、轻轻停步、还是先把你的话收好 |
| **Memory** | 切片记忆 | 把值得留下的片段，像信笺一样逐条收藏——不是堆聊天记录，而是收「被你确认过的真实」 |
| **ReAct** | 慢循环 | 先观察，再理解，再回应，再校验，最后才把记忆写入纸页 |
| **Safety** | 温柔边界 | 在你需要现实帮助时，停止拆解，先把安全放在前面 |
| **Reflection Engine** | 三层回响 | 听见你 → 试探性拆开 → 把问题还给你 |
| **Gate** | 深度门 | 走够远时，轻声问：还要再往里吗，还是先歇一歇 |
| **Image** | 画面触发 | 不解读物体，只问：这张图在你心里，像什么 |
| **Pause** | 落款 | 会话结束时的 mirror 与告别，把确认过的洞察写入长期记忆 |

完整架构见 [SKILL.md](./skills/luv-letter/SKILL.md)。

<br>

---

<br>

## 使用

安装完成后，在任意支持的 Agent 中进入 Luv Letter 模式：

| Agent | 唤起方式 |
| --- | --- |
| **Cursor** | `@luv-letter`，或说「进入 Luv Letter 模式」 |
| **Claude Code** | `/luv-letter`，或说「Send into Luv」 |
| **Codex** | 「Enter Luv Letter mode」/ 「Send into Luv」 |
| **OpenCode** | 「进入 Luv Letter 模式」/ 「Send into Luv」 |

以下步骤在所有平台相同。

### 第一步 · 写下一个片段

不必完整。一个词、一种身体感觉、半句没说完的话，都可以：

```text
最近脑子停不下来，消息一来就要回，不回就焦虑。
```

### 第二步 · 跟着信的节奏

每一轮，你会收到融在文字里的三层回响——不会标注「第几层」，只是慢慢展开：

| | |
| --- | --- |
| **Echo · 回响** | 你的话被换种方式说回来——先被听见 |
| **Deconstruction · 拆开** | 情绪、假设、张力被试探性地呈现；随时可以说「不对」 |
| **Extension · 延伸** | 一两个开放式问题，把意义留给你自己 |

大约 **四轮** 后，会轻声问：

> 你想继续往深处走，还是先在这里停一停？

### 第三步 · 停，也是一种完成

```text
先停在这里吧。
```

值得留下的，会被写入记忆。下次回来，可以轻轻接上——不会替你总结人生，只会记得**你说过的那些字**。

<br>

### 你可以这样开口

```text
Send into Luv

最近总是很累，但不是身体累，是脑子停不下来。
```

```text
[上传一张图片]

这张画面让我有点说不出的感觉。
```

```text
不对，不是焦虑，我只是烦。
```

它会收回框架，重新听你说——**你的词，比它的结构更重要。**

<br>

---

<br>

## 记忆

记忆不是把对话塞进数据库，而是像整理抽屉里的信：

| 存储 | 名字 | 存什么 |
| --- | --- | --- |
| **INNER.md** | 内在纸页 | 被你确认过的主题与感受——你的原话，比系统的猜测更优先 |
| **LUV.md** | 反思笔记 | 曾误判的地方、你的校正、这个空间的惯例 |
| **THREAD.md** | 当前线程 | 这一次谈话走到哪了——蒸馏过的，不是原文 |
| **archive** | 旧信归档 | 跨会话的主题与 pause 快照；需要时才被翻找 |
| **sessions** | 回合留底 | 完整对话存档，供「上次我们谈过……」时召回 |

每条记忆是一个 **Slice（切片）**：可单独添加、替换、合并、搜索。  
Pause 或换题前，系统会先 **flush**——把值得留下的蒸馏进纸页，再让上下文轻轻退去。

详见 [memory.md](./skills/luv-letter/memory.md) · [memory-tool.md](./skills/luv-letter/memory-tool.md)

<br>

---

<br>

## 示例

完整场景（含内部 ReAct trace）见 [examples.md](./skills/luv-letter/examples.md)。

**你：**

> 最近脑子停不下来，消息一来就要回，不回就焦虑。

**Luv Letter：**

> 你描述的似乎不只是疲惫，而是一种脑子被持续占满、很难真正空下来的状态——即使身体在休息，注意力仍被「随时要回应」拉着走。
>
> 如果试着把这段话拆开，我暂时看到几层——不一定对，你可以校正我：
>
> - 情绪：耗竭、紧绷
> - 某个可能的假设：「我应该随时在线、随时回应」
> - 张力：休息的需要 ↔ 被期待立刻可用的压力
>
> 哪一层最先触碰到你？

<br>

---

<br>

## 它不会做什么

- 不给人生建议或行动清单
- 不做心理诊断
- 不替你做决定
- 不快速来回问答

Luv Letter 是**自我对话空间**，不是心理咨询或医疗建议。  
若你处于危机中，请优先联系现实中的人或专业援助。

<br>

---

<br>

## 常见问题

**这和普通 Chat 有什么区别？**  
Chat 追求快与全；Luv Letter 追求慢与深。它有调度、有记忆、有停步的门——更像一封会回复的信，而不是搜索框。

**为什么说「切片记忆」？**  
因为值得记住的从来不是整段对话，而是几个被你点头的瞬间。切片化，是为了只收那些真实。

**可以只用 SKILL.md 吗？**  
可以开口，但完整行为依赖 orchestrator、memory、react 与 modules。像只读封面，信便无法寄达。

**支持图片吗？**  
支持。但不描述「这是什么物体」——只问它在你内在世界里，唤起什么。

**记忆存在哪？**  
设计路径为使用方项目内 `.luv-letter/memory/`（见 memory 文档）。各 Agent 按 skill 规范维护，与安装路径无关。

**支持哪些 Agent？**  
Cursor、Claude Code、Codex、OpenCode（通过 [vercel-labs/skills](https://github.com/vercel-labs/skills) CLI）。其他兼容 Agent Skills 标准的工具也可尝试 `npx skills add ... -a <agent>`。

<br>

---

<br>

## 文档

| 文件 | 内容 |
| --- | --- |
| [SKILL.md](./skills/luv-letter/SKILL.md) | 系统入口与执行协议 |
| [orchestrator.md](./skills/luv-letter/orchestrator.md) | 调度与路由 |
| [memory.md](./skills/luv-letter/memory.md) | 切片化记忆架构 |
| [react-loop.md](./skills/luv-letter/react-loop.md) | ReAct 循环 |
| [examples.md](./skills/luv-letter/examples.md) | 场景示例 |
| [LuvLetterProduct-Specification.md](./LuvLetterProduct-Specification.md) | 产品愿景 |

<br>

---

<br>

<div align="center">

*Take your time. You are worth it.*

<br>

**亲爱的自己，谢谢你一直在这里。**

</div>
