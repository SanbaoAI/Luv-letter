# Luv Letter — System Examples (Sliced Memory)

Each example: **ReAct trace** + **slice ops** + **user-facing output**.

---

## Example 1 — Init Turn

**User:** Send into Luv — "最近脑子停不下来，消息一来就要回，不回就焦虑。"

<details>
<summary>ReAct + Memory</summary>

```
OBSERVE: phase=INIT; memory_bundle.inner_frozen="" (new user)
REASON:  intent=init_reflect
ACT:     safety → disclaimer | reflection.full
COMMIT:  turn_append(sess_001, turn=1)
         slice_add(thread, thread_distill, offered, "谈回应压力与耗竭")
         # offered 留 thread_buffer，不进 INNER
```
</details>

**User sees:** *(standard full reflection — Echo + Deconstruction + Extension)*

---

## Example 2 — Reject Frame → rejected_frame slice

**User:** "不对，不是焦虑，我只是烦。"

<details>
<summary>ReAct + Memory</summary>

```
OBSERVE: frame_rejection=true; thread_buffer has offered "焦虑"
REASON:  intent=reject_frame
ACT:     reflection.echo_only
COMMIT:  slice_add(luv, rejected_frame, "§曾误判为焦虑；用户说是烦")
         turn_append(...)
         # LUV.md live 更新；INNER.frozen 不变
```
</details>

**User sees:**

好，我收回刚才的框架。烦——这个词本身就够了。

---

如果用你自己的话说，这种「烦」更像堵着、冲着，还是空着？

---

## Example 3 — Confirm → confirmed_insight slice

**User:** "应该是那种必须立刻有用的感觉。"

<details>
<summary>ReAct + Memory</summary>

```
OBSERVE: user affirms assumption axis
REASON:  intent=continue_reflect; mode=deep
COMMIT:  slice_add(inner, confirmed_insight,
           content="§必须立刻有用，暂停像浪费",
           user_anchor="必须立刻有用的感觉")
         turn_append(...)
```
</details>

**INNER.md live (after commit):**

```text
§必须立刻有用，暂停像浪费（用户原话：必须立刻有用的感觉）
```

**User sees:** deep mode Extension targeting this slice only.

---

## Example 4 — Pause → flush + archive

**User:** "先停在这里吧。"

<details>
<summary>ReAct + Memory</summary>

```
OBSERVE: gate_response=pause
REASON:  intent=user_choose_pause
ACT:     flush(pause) → pause.close
COMMIT:  slice_add(archive, pause_snapshot, "§触到必须有用；用户选择停下")
         slice_add(inner, theme_anchor, "§「必须有用」反复主题")
         THREAD.md cleared
         phase=PAUSED
```
</details>

**User sees:**

好。今天你已经触到了「必须立刻有用」这一层——它很重，不需要现在就有答案。

当你准备好，可以再来。

---

## Example 5 — New Session → frozen INNER + slice_search

**Next session start**

<details>
<summary>Memory load</summary>

```
load_bundle:
  inner_frozen: "§必须立刻有用…"   # from .snapshot/ — 上会话 flush 后下会话可见
  slice_search(type=theme_anchor, limit=1) → [sl_pause_001]
```
</details>

**User sees:**

欢迎回来。上次你曾触到「必须立刻有用」——今天想从那里继续，还是带一个新的片段进来？

---

## Example 6 — "上次谈过" → turn_search

**User:** "还记得上次说的回应压力吗？"

<details>
<summary>ReAct + Memory</summary>

```
OBSERVE: episodic reference signal
REASON:  queue slice_search(query="回应压力") + turn_search(query="回应")
ACT:     reflection.adaptive with search results as context
COMMIT:  turn_append only; no new inner slice unless user confirms again
```
</details>

---

## Example 7 — INNER at capacity → consolidate

**slice_add fails:**

```json
{
  "success": false,
  "error": "INNER at 1,420/1,500. Consolidate: merge §回应压力 + §随时在线 into one entry.",
  "current_entries": ["§...", "§..."]
}
```

**Same turn retry:**

```
slice_consolidate(inner, merge_tags=["回应"])
slice_add(inner, confirmed_insight, "§回应压力：不回就焦虑；应该随时在线")
→ success
```

---

## Example 8 — Compression flush (before context trim)

**Trigger:** context_window > policy limit at round 5

<details>
<summary>Internal flush</summary>

```
flush(trigger=compress):
  - confirmed_insight → inner (already there, skip)
  - drop offered thread_buffer entries
  - thread_distill → archive: "§第5轮仍在谈有用/回应"
  - trim context_window to last 3 user turns
```

</details>

---

## Storage Snapshot (after Example 4)

```
.luv-letter/memory/
├── INNER.md          # §必须立刻有用… §「必须有用」反复主题
├── LUV.md            # §曾误判为焦虑；用户说是烦
├── THREAD.md         # (empty — paused)
├── slices/
│   └── sl_pause_001.slice.json
└── sessions/
    └── sess_001.jsonl
```
