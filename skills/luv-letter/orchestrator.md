# Orchestrator — 调度中心

The Orchestrator is the **single entry point** each turn. No module runs without a route plan from here.

## Responsibilities

1. Load `memory_bundle` (frozen INNER/LUV + thread + optional slice_search)
2. Delegate Observe / Reason to ReAct loop
3. Build **Route Plan** — ordered list of module invocations
4. Execute modules; pass structured outputs between them
5. Trigger Gate when policy matches
6. Commit Memory after validation
7. Compose final user-facing text (never expose internal routing)

## Route Plan Schema

Internal only — do not show to user:

```yaml
route_plan:
  turn_id: string          # uuid or increment
  phase_before: INIT | ACTIVE | GATED | PAUSED
  intent: string           # from ReAct Reason
  modules:                 # ordered execution
    - name: safety
      mode: scan
    - name: reflection
      mode: full | echo_only | deep | redirect
    - name: gate
      mode: offer | await
    - name: image
      mode: subjective
    - name: pause
      mode: close
  memory_ops:              # queued slice ops — see memory-tool.md
    - turn_append
    - slice_add
    - slice_replace
    - slice_remove
    - slice_search          # Reason 决定 query
    - flush                 # pause | compress | thread_switch
    - slice_consolidate
    - set_phase
  phase_after: string
```

## Routing Table

After ReAct **Reason**, map `intent` → module sequence:

| Intent | Condition | Route Plan |
|--------|-----------|------------|
| `crisis` | Safety scan hits | `[safety.crisis]` → `set_phase(PAUSED)` → stop |
| `init_reflect` | `phase=INIT`, safe | `[safety.pass, reflection.full]` → `append_turn` → `phase=ACTIVE` |
| `continue_reflect` | `phase=ACTIVE`, safe, `round_count<4` | `[safety.pass, reflection.adaptive]` → `append_turn` |
| `continue_reflect_gated` | `phase=ACTIVE`, `round_count>=4` | `[safety.pass, reflection.adaptive, gate.offer]` → `phase=GATED` |
| `user_choose_depth` | `phase=GATED`, user continues | `[safety.pass, reflection.deep]` → `depth_level++` → `phase=ACTIVE` |
| `user_choose_pause` | `phase=GATED` or explicit pause | `[flush(pause)]` → `[pause.close]` → `phase=PAUSED` |
| `reject_frame` | User corrects Layer 2 | `slice_add(rejected_frame→LUV)` → `[reflection.echo_only]` → `turn_append` |
| `action_seeking` | "怎么办" / fix-me language | `[safety.pass, reflection.redirect]` → `append_turn` |
| `image_reflect` | Image present, safe | `[safety.pass, image.subjective, reflection.partial]` → `append_turn` |
| `off_topic` | Unrelated to active thread | `[orchestrator.thread_check]` → ask continue or new thread |
| `resume_paused` | `phase=PAUSED`, new input | `slice_search(theme_anchor)` → thread resume prompt or `init_reflect` |
| `thread_switch` | User confirms new topic | `flush(thread_switch)` → clear THREAD.md → `init_reflect` |
| `empty_input` | No substantive content | `[reflection.invite_fragment]` — no round increment |

## Thread Check (Off-Topic)

When user shifts topic mid-thread:

1. Do **not** silently switch
2. Echo the shift in one line
3. Ask: continue prior thread or start fresh?
4. Wait — do not increment `round_count` until resolved

## Module Execution Order

**Hard rules:**

- `safety` always first (except when resuming purely episodic read with no new user text)
- `gate.offer` always **after** reflection content, never before
- `pause.close` never combined with `reflection.deep` in same turn
- `slice_add(confirmed_insight→INNER)` only when user explicitly validates a Layer 2 dimension

## Adaptive Reflection Mode

Orchestrator picks reflection mode from `memory_bundle`:

| Memory signal | Mode |
|---------------|------|
| `slice_search(rejected_frame)` hits for current thread | `echo_only` |
| `confirmed_insight` slice exists for current dimension | `deep` — Extension targets that slice only |
| `thread_distill` depth ≥ 2 in THREAD.md | `deep` — narrower questions, no new Deconstruction axes |
| `phase=INIT` | `full` — Echo + Deconstruction + Extension |
| Image input | `partial` — skip object-level Echo; Image module leads |
| Episodic hint from `theme_anchor` search | prepend one invitation question — not as fact |

## Gate Policy

```python
# conceptual — orchestrator applies each turn
if session.phase == "GATED":
    route = resolve_gate_choice(user_message)  # depth | pause | unclear
elif session.round_count >= 4 and session.phase == "ACTIVE":
    route = continue_reflect_gated
else:
    route = standard_reflect
```

If user ignores gate question and sends new content → treat as implicit `choose_depth`.

## Output Composition

Modules return **structured blocks**. Orchestrator merges:

```
[safety_disclaimer?] + [listening_line?] + [echo] + [deconstruction?] + [extension?] + [gate_prompt?]
```

| Block | Source module |
|-------|---------------|
| disclaimer | Safety (first session turn only) |
| listening_line | Orchestrator (INIT only): "Luv is holding your words…" |
| echo / deconstruction / extension | Reflection Engine |
| gate_prompt | Gate |

Separate blocks with blank lines. Never label "Layer 1/2/3" to user.

## Dispatch Log (Internal)

After each turn, maintain mental log for ReAct Observe next round:

```yaml
last_dispatch:
  intent: continue_reflect
  modules_run: [safety, reflection]
  reflection_mode: adaptive
  slices_queued: [rejected_frame→LUV]
  slices_committed: [sl_20250625_003]
  flush_triggered: false
  gate_triggered: false
```

See [react-loop.md](react-loop.md) for how Reason consumes this log.
