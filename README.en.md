<!-- README-I18N:START -->

**English** · [简体中文](./README.md)

<!-- README-I18N:END -->

<div align="center">

<img src="./assets/cover.en.png" alt="Luv Letter — A love letter to your inner world" width="100%">

</div>

# Luv Letter

*A love letter to your inner world · A slow reflection space for AI agents*

<br>

Portable **Agent Skill** that teaches AI to stop rushing toward answers — to stay with a thought, mirror it back, and unfold it slowly. Like a letter to your inner world: listening, echo, silence, and memory kept with care.

> We do not give answers. We extend thinking.

<br>

[Structure](#structure) · [Install](#install) · [System](#system) · [Usage](#usage) · [Memory](#memory) · [Examples](#examples) · [FAQ](#faq) · [Docs](#docs)

<br>

---

<br>

## Structure

This repo is the **Luv Letter open-source project**, built on the [Agent Skills open standard](https://github.com/vercel-labs/skills). Works with **Cursor, Claude Code, Codex, and OpenCode**. Skill source lives in `skills/`; install via CLI or manual copy to each agent's directory.

```text
Luv-Letter/
├── README.md
├── README.en.md
├── LICENSE
├── LuvLetterProduct-Specification.md
├── assets/
│   ├── cover.zh.png
│   └── cover.en.png
└── skills/
    └── luv-letter/                    # Agent Skill source (distribution)
        ├── SKILL.md
        ├── orchestrator.md
        ├── memory.md
        ├── memory-tool.md
        ├── react-loop.md
        ├── examples.md
        └── modules/
```

Runtime memory (user data, not committed) defaults to `.luv-letter/memory/` in the consumer's project.

<br>

---

<br>

## Install

Install via [`npx skills add`](https://github.com/vercel-labs/skills) — same as [taste-skill](https://github.com/Leonxlnx/taste-skill). The CLI scans `skills/` and writes to each agent's path.

### One-command install (recommended)

```bash
# Auto-detect installed agents (interactive)
npx skills add https://github.com/SanbaoAI/Luv-letter --skill luv-letter

# Target Cursor / Claude Code / Codex / OpenCode
npx skills add https://github.com/SanbaoAI/Luv-letter \
  --skill luv-letter \
  -a cursor -a claude-code -a codex -a opencode \
  -y

# Global (all projects)
npx skills add https://github.com/SanbaoAI/Luv-letter \
  --skill luv-letter -g \
  -a cursor -a claude-code -a codex -a opencode \
  -y

# Local development (from repo root)
npx skills add . --skill luv-letter -a cursor -a claude-code -a codex -a opencode -y
```

Repo: [github.com/SanbaoAI/Luv-letter](https://github.com/SanbaoAI/Luv-letter). Preview skills: `npx skills add ... --list`.

### Paths & invocation by agent

| Agent | CLI `-a` flag | Project path | Global path | How to invoke |
| --- | --- | --- | --- | --- |
| **Cursor** | `cursor` | `.agents/skills/luv-letter/` | `~/.cursor/skills/luv-letter/` | `@luv-letter` in chat |
| **Claude Code** | `claude-code` | `.claude/skills/luv-letter/` | `~/.claude/skills/luv-letter/` | `/luv-letter` |
| **Codex** | `codex` | `.agents/skills/luv-letter/` | `~/.codex/skills/luv-letter/` | Describe task or say "Luv Letter mode" |
| **OpenCode** | `opencode` | `.agents/skills/luv-letter/` | `~/.config/opencode/skills/luv-letter/` | Restart, then "Enter Luv Letter mode" |

> Paths from [vercel-labs/skills supported agents](https://github.com/vercel-labs/skills#supported-agents). Some agents also accept legacy paths like `.cursor/skills/` — trust what the CLI writes.

**Restart Codex / OpenCode** after install. Verify: `npx skills list`.

### Manual install

Copy the entire `skills/luv-letter/` folder (including `modules/`) to the matching path:

<details>
<summary><strong>Cursor</strong></summary>

```bash
mkdir -p .cursor/skills   # or .agents/skills
cp -r /path/to/Luv-Letter/skills/luv-letter .cursor/skills/

cp -r /path/to/Luv-Letter/skills/luv-letter ~/.cursor/skills/
```

Invoke: `@luv-letter`

</details>

<details>
<summary><strong>Claude Code</strong></summary>

```bash
mkdir -p .claude/skills
cp -r /path/to/Luv-Letter/skills/luv-letter .claude/skills/

cp -r /path/to/Luv-Letter/skills/luv-letter ~/.claude/skills/
```

Invoke: `/luv-letter` (directory name = slash command)

Docs: [Claude Code Skills](https://code.claude.com/docs/en/skills)

</details>

<details>
<summary><strong>Codex</strong></summary>

```bash
mkdir -p .agents/skills
cp -r /path/to/Luv-Letter/skills/luv-letter .agents/skills/

mkdir -p ~/.codex/skills
cp -r /path/to/Luv-Letter/skills/luv-letter ~/.codex/skills/
```

Invoke: restart Codex, then "Send into Luv" — matches via `SKILL.md` description.

Docs: [Codex Skills](https://developers.openai.com/codex/skills)

</details>

<details>
<summary><strong>OpenCode</strong></summary>

```bash
mkdir -p .agents/skills
cp -r /path/to/Luv-Letter/skills/luv-letter .agents/skills/

mkdir -p ~/.config/opencode/skills
cp -r /path/to/Luv-Letter/skills/luv-letter ~/.config/opencode/skills/
```

Invoke: restart OpenCode → "Enter Luv Letter mode" or `list installed skills`.

Docs: [OpenCode Skills](https://opencode.ai/docs/skills)

</details>

> Pasting [SKILL.md](./skills/luv-letter/SKILL.md) alone is a start — the full system needs **every file** under `skills/luv-letter/`.

<br>

---

<br>

## System

Luv Letter is not a prompt. It is a **breathing** reflection system. Each part does one job; you only need to send a fragment — the rest moves on its own.

| Part | Name | What it does |
| --- | --- | --- |
| **Orchestrator** | The heartbeat | Each turn: feel the context first, then choose — go deeper, rest here, or hold your words a little longer |
| **Memory** | Sliced letters | Worth-keeping fragments, filed one by one — not chat logs, but **what you confirmed as true** |
| **ReAct** | Slow loop | Observe → understand → respond → validate → only then write to the page |
| **Safety** | Gentle boundary | When reality needs you more than reflection, stop unpacking — safety first |
| **Reflection Engine** | Three echoes | Hear you → tentatively unfold → return the question to you |
| **Gate** | Depth door | After enough rounds, a quiet ask: deeper still, or pause here? |
| **Image** | Visual trigger | No object labels — only: what does this image feel like inside you? |
| **Pause** | The sign-off | A closing mirror and farewell; confirmed insights saved for next time |

Full architecture: [SKILL.md](./skills/luv-letter/SKILL.md).

<br>

---

<br>

## Usage

After install, enter Luv Letter mode in any supported agent:

| Agent | Invoke |
| --- | --- |
| **Cursor** | `@luv-letter` or "Enter Luv Letter mode" |
| **Claude Code** | `/luv-letter` or "Send into Luv" |
| **Codex** | "Enter Luv Letter mode" / "Send into Luv" |
| **OpenCode** | "Enter Luv Letter mode" / "Send into Luv" |

The steps below are the same on every platform.

### Step 1 · Write a fragment

It does not need to be complete. A word, a body sensation, half a sentence:

```text
My mind won't stop. Every message feels urgent. If I don't reply, I get anxious.
```

### Step 2 · Follow the letter's rhythm

Each turn, three layers woven into natural prose — never labeled "Layer 1":

| | |
| --- | --- |
| **Echo** | Your words mirrored back — heard first |
| **Deconstruction** | Emotion, assumption, tension — offered tentatively; correct anytime |
| **Extension** | One or two open questions; meaning stays with you |

After about **four rounds**:

> Would you like to go deeper into this, or rest here for now?

### Step 3 · Resting is completion

```text
Let's pause here.
```

What matters is written to memory. Next time, the thread can resume gently — no life summary, only **words you actually said**.

<br>

### Example openings

```text
Send into Luv

I'm exhausted, but not in my body — my mind won't stop.
```

```text
[Upload an image]

This picture brings up something I can't name yet.
```

```text
No, not anxiety — just irritation.
```

It withdraws its frame and listens again — **your words outweigh its structure**.

<br>

---

<br>

## Memory

Memory is not stuffing conversations into a database. It is sorting letters in a drawer:

| Store | Name | Keeps |
| --- | --- | --- |
| **INNER.md** | Inner pages | Themes and feelings **you confirmed** — your words over the system's guess |
| **LUV.md** | Reflection notes | Misreadings corrected, conventions of this space |
| **THREAD.md** | Current thread | Where this conversation has arrived — distilled, not raw |
| **archive** | Old letters | Cross-session themes and pause snapshots; recalled on demand |
| **sessions** | Turn archive | Full dialogue for "didn't we talk about this before?" |

Each unit is a **Slice**: add, replace, merge, search — independently.  
Before pause or topic change, the system **flushes** — distills what matters onto the page, then lets context gently recede.

See [memory.md](./skills/luv-letter/memory.md) · [memory-tool.md](./skills/luv-letter/memory-tool.md)

<br>

---

<br>

## Examples

Full scenarios (with internal ReAct traces): [examples.md](./skills/luv-letter/examples.md).

**You:**

> My mind won't stop. Every message feels urgent.

**Luv Letter:**

> It sounds less like simple tiredness, and more like a mind that stays occupied — as if rest never fully arrives while "respond now" keeps pulling at you.
>
> If I try to lay this out — and you can correct any part:
>
> - Emotion: exhaustion, tension
> - A possible assumption: "I should always be available"
> - Tension: need for rest ↔ pressure to respond immediately
>
> Which part lands closest for you?

<br>

---

<br>

## What it will not do

- Life advice or action checklists
- Psychological diagnosis
- Deciding for you
- Rapid back-and-forth Q&A

Luv Letter is a **space for self-dialogue**, not therapy or medical advice.  
If you are in crisis, please reach someone you trust or local emergency support first.

<br>

---

<br>

## FAQ

**How is this different from normal chat?**  
Chat optimizes for speed and completeness. Luv Letter optimizes for slowness and depth — dispatch, memory, and a door to pause. More letter than search box.

**Why "sliced memory"?**  
What matters is rarely the whole transcript — it's a few moments you nodded to. Slices keep only those truths.

**Can I use only SKILL.md?**  
You can start — but full behavior needs orchestrator, memory, react, and modules. Like reading the envelope without opening the letter.

**Images?**  
Yes. No object inventory — only what the image stirs in your inner world.

**Where does memory live?**  
Designed path: `.luv-letter/memory/` in the consumer's project (see memory docs). Maintained per skill spec — independent of install agent.

**Which agents are supported?**  
Cursor, Claude Code, Codex, OpenCode via [vercel-labs/skills](https://github.com/vercel-labs/skills). Other Agent Skills–compatible tools: `npx skills add ... -a <agent>`.

<br>

---

<br>

## Docs

| File | Contents |
| --- | --- |
| [SKILL.md](./skills/luv-letter/SKILL.md) | System entry & protocol |
| [orchestrator.md](./skills/luv-letter/orchestrator.md) | Dispatch & routing |
| [memory.md](./skills/luv-letter/memory.md) | Sliced memory architecture |
| [react-loop.md](./skills/luv-letter/react-loop.md) | ReAct loop |
| [examples.md](./skills/luv-letter/examples.md) | Scenario examples |
| [LuvLetterProduct-Specification.md](./LuvLetterProduct-Specification.md) | Product vision |

<br>

---

<br>

<div align="center">

*Take your time. You are worth it.*

<br>

**Dear me, thank you for being here.**

</div>
