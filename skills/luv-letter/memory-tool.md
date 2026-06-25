# Memory Tool — Slice API

Reflection 模块不直接碰磁盘。所有持久化经 **Memory Tool**，由 Orchestrator 在 ReAct Act / Commit 阶段调用。

---

## slice_add

```yaml
slice_add:
  store: inner | luv | thread | archive
  type: confirmed_insight | rejected_frame | theme_anchor | thread_distill | pause_snapshot | preference | image_felt
  content: string              # ≤ 280 chars，密集
  user_anchor: string | null
  meta:
    thread_id: string
    session_id: string
    turn_id: int
    dimension: emotion | trigger | assumption | tension | unspoken | null
    confidence: confirmed | rejected | offered | distilled
    tags: [string]
```

**Returns:**

```yaml
success: true | false
slice_id: sl_20250625_003
usage: "1,080/1,500"           # 仅 hot store
error: "INNER at capacity. Consolidate first." | null
```

**Rules:**

- 精确 duplicate → `success: true, duplicate: true`，不重复写
- 超 cap → error + `current_entries` 列表，同 turn 内 consolidate 后 retry
- `confirmed_insight` 无 `user_anchor` 且无 confirm 信号 → reject

---

## slice_replace

Hermes substring matching — 不需要 slice id。

```yaml
slice_replace:
  store: inner | luv | thread
  old_text: string             # 唯一匹配子串
  content: string              # 新 content，仍受 store cap 约束
```

| 匹配数 | 行为 |
|--------|------|
| 0 | error: not found |
| 1 | replace，旧 slice `superseded_by` 指向新 id |
| >1 | error: ambiguous，要求更 specific 的 old_text |

---

## slice_remove

```yaml
slice_remove:
  store: inner | luv | thread
  old_text: string
```

---

## slice_search

冷存储 + archive 检索；**不**搜 frozen 内容以外的 live 全文。

```yaml
slice_search:
  query: string | null         # keyword / 用户原话片段
  type: confirmed_insight | theme_anchor | pause_snapshot | rejected_frame | null
  thread_id: string | null
  tags: [string] | null
  limit: int                   # default 5, max 10
  since: ISO8601 | null
```

**Returns:**

```yaml
slices:
  - id: sl_...
    type: theme_anchor
    content: "..."
    user_anchor: "..."
    score: float               # keyword relevance
    created_at: ISO8601
```

**Use cases:**

- 新会话：「有哪些重复主题？」→ `type=theme_anchor, limit=3`
- 拒框后：「这个 thread 曾拒什么？」→ `type=rejected_frame, thread_id=…`
- 用户：「上次谈过回应焦虑」→ `query=回应`

---

## slice_consolidate

Hot store 超 80% 或 add 失败时调用。

```yaml
slice_consolidate:
  store: inner | luv | thread
  strategy: merge_tags | merge_dimension | drop_stale
```

**Behavior:**

- 同 tag / 同 dimension 的多条 merge 成一条更短 slice
- `offered` 未确认条目优先 drop
- 被 merge 的 slice → archive 留档，`superseded_by` 链

**Example merge:**

```
Before (3 entries, 420 chars):
  §不回消息就焦虑
  §消息来就要回
  §随时在线的压力

After (1 entry, 95 chars):
  §回应压力：不回消息就焦虑；「应该随时在线」（用户原话）
```

---

## flush

Pause / 压缩 / 换题前调用。内部专用，不对用户展示 flush 指令。

```yaml
flush:
  trigger: pause | compress | thread_switch | gate_depth
  session_id: string
  thread_id: string
  context_window:            # 待蒸馏源
    user_messages: [string]
    assistant_messages: [string]
  thread_buffer: [slice]
```

**Output — queued ops:**

```yaml
flush_result:
  slices_written:
    - { store: inner, type: confirmed_insight, content: "..." }
    - { store: archive, type: pause_snapshot, content: "..." }
  thread_cleared: bool
  inner_usage: "1,200/1,500"
```

**Flush 优先级（写入顺序）:**

1. `rejected_frame` → LUV.md（用户校正过的）
2. `confirmed_insight` → INNER.md
3. `theme_anchor` → INNER.md（重复 ≥2 或 pause 时）
4. `thread_distill` → THREAD.md 或 archive
5. `pause_snapshot` → archive

---

## turn_append

L4 回合归档 — 全文存，**不注入 prompt**。

```yaml
turn_append:
  session_id: string
  turn_id: int
  user: string
  assistant: string
  meta:
    intent: string
    reflection_mode: string
    slices_written: [slice_id]
```

---

## turn_search

类似 Hermes `session_search` — episodic 召回。

```yaml
turn_search:
  query: string
  session_id: string | null    # null = 跨会话
  limit: int                   # default 3 windows
```

**Returns:** 匹配窗口 ± 上下文，max 500 chars/window — **不** LLM 摘要，原文截断。

---

## write_approval（可选）

```yaml
memory:
  write_approval: false        # default：confirmed_insight 用户确认即可写
  inner_char_limit: 1500
  luv_char_limit: 800
  thread_char_limit: 600
```

`write_approval: true` 时，`confirmed_insight` 写入 INNER 需用户显式「可以记住」。

---

## Security Scan（写入前）

Block before disk:

- 危机细节持久化（自伤计划等）
- Prompt injection patterns
- 诊断标签（「你有焦虑症」）

Return error; slice 不落盘。

---

## Orchestrator 调用时序

```
Turn Start:  load memory_bundle (frozen + thread + optional search)
Turn Act:     modules → side_effects.queue_slice_ops
Turn Commit:  turn_append → slice ops → flush? → consolidate?
```

Queued ops 示例：

```yaml
memory_ops:
  - slice_add: { store: luv, type: rejected_frame, ... }
  - slice_replace: { store: thread, old_text: "耗竭", content: "..." }
  - flush: { trigger: pause }
```
