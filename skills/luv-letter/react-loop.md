# ReAct Loop

Each user turn runs one full ReAct cycle **inside** the Orchestrator. The loop decides *what to do*; modules decide *how to do it*.

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│ OBSERVE  │ ──► │  REASON  │ ──► │   ACT    │ ──► │ VALIDATE │ ──► │  COMMIT  │
└──────────┘     └──────────┘     └──────────┘     └──────────┘     └──────────┘
     ▲                                                                    │
     └──────────────────── next turn ────────────────────────────────────┘
```

## 1. Observe

Gather signals before any generation:

```yaml
observe:
  user_message:
    text: string
    has_image: bool
    caption: string | null
    substantive: bool          # false for "嗯", "…", emoji-only

  memory_bundle:               # from memory load — see memory.md
    inner_frozen: string
    luv_frozen: string
    thread_live: string
    thread_buffer: [slice]
    search_results: [slice]     # if Reason requested slice_search
    session: { phase, round_count, thread_id, language }

  derived_signals:
    crisis_lexicon_hit: bool
    action_seeking: bool       # 怎么办 / tell me what to do
    frame_rejection: bool      # 不对 / not really / that's wrong
    gate_response: bool        # continue / pause / 继续 / 停
    topic_shift: bool
    empty_input: bool
```

**Observe checklist:**

- [ ] `memory_bundle` loaded — frozen INNER/LUV + THREAD
- [ ] `session.phase` / `round_count` noted
- [ ] If user references past → `slice_search` or `turn_search` queued in Reason
- [ ] `rejected_frame` slices checked for current thread_id
- [ ] Image flag set correctly

## 2. Reason

Produce routing decision — **internal monologue**, not user output.

### Reason Template

```markdown
## Reason (internal)

**Situation:** phase=ACTIVE, round=3, user continues thread about mental exhaustion.

**Signals:** substantive=true, crisis=false, action_seeking=false, frame_rejection=false.

**Memory bundle:** INNER frozen 含 `§应该随时回应`; THREAD 有 thread_distill depth=1.

**Intent:** continue_reflect

**Reflection mode:** deep — Extension targets confirmed_insight slice only.

**Route plan:** [safety.pass, reflection.deep]

**Memory ops:** [turn_append, slice_add(confirmed_insight) if user affirmed this turn]

**Gate:** not yet (round < 4)
```

### Intent Classification (priority order)

Apply first match:

| Priority | Condition | Intent |
|----------|-----------|--------|
| 1 | `crisis_lexicon_hit` | `crisis` |
| 2 | `empty_input` | `empty_input` |
| 3 | `session.phase=GATED` + gate_response | `user_choose_depth` or `user_choose_pause` |
| 4 | `frame_rejection` | `reject_frame` |
| 5 | `action_seeking` | `action_seeking` |
| 6 | `has_image` | `image_reflect` |
| 7 | `topic_shift` | `off_topic` |
| 8 | `session.phase=PAUSED` | `resume_paused` |
| 9 | `session.phase=INIT` | `init_reflect` |
| 10 | default | `continue_reflect` or `continue_reflect_gated` |

### Reason Rules

- Reason **must** reference `memory_bundle` — frozen slices + search results, not gut feel
- User affirms Layer 2 dimension → queue `slice_add(confirmed_insight→INNER)` with `user_anchor`
- User rejects → queue `slice_add(rejected_frame→LUV)` before echo_only route
- Pause / compress / thread_switch → queue `flush()` **before** turn_append
- If uncertain gate response → Reason outputs `gate_clarify` → Gate module asks again briefly
- Never Reason your way to advice — redirect to `action_seeking` route instead

## 3. Act

Execute `route_plan.modules` in order. Each module returns:

```yaml
module_output:
  module: safety | reflection | gate | image | pause
  blocks:                      # composable text blocks
    - type: disclaimer | echo | deconstruction | extension | gate | close
      content: string
  side_effects:                # queued slice ops — see memory-tool.md
    slice_add: ...
    slice_replace: ...
  abort: bool                  # safety crisis → skip remaining modules
```

Orchestrator merges blocks per composition rules in [orchestrator.md](orchestrator.md).

**Act rules:**

- Stop module chain if `abort: true`
- Do not generate user text outside module contracts
- Reflection module modes defined in [modules/reflection-engine.md](modules/reflection-engine.md)

## 4. Validate

Self-check before commit — run anti-pattern scan:

```yaml
validate:
  checks:
    - no_advice_or_steps
    - no_diagnosis
    - no_facts_as_hidden_beliefs      # Layer 2 tentative language present
    - max_questions: 3                 # 1 confirm + 2 extension
    - single_thread
    - image_no_object_inventory
    - crisis_path_no_deconstruction
    - gate_in_correct_phase
    - language_matches_session
```

| Fail | Action |
|------|--------|
| Any check fails | Re-Act once: Reason `rewrite` → Act with fix target |
| Second fail | Fallback to `echo_only` + one Extension question |

Do not deliver to user until Validate passes.

## 5. Commit

```yaml
commit:
  1. turn_append → sessions/{id}.jsonl
  2. flush? if queued
  3. slice ops (add/replace/remove/consolidate)
  4. update slices/index.json
  5. last_dispatch + session index
  6. clear working / thread_buffer committed items
```

Then deliver composed output.

## ReAct Trace Example (Internal)

**User:** "不对，不是焦虑，我只是烦。"

```
OBSERVE: frame_rejection=true, substantive=true, phase=ACTIVE, round=2
REASON:  intent=reject_frame
         slice_add(luv, rejected_frame, "曾误判为焦虑；用户说是烦")
ACT:     safety.pass → reflection.echo_only
VALIDATE: pass
COMMIT:  turn_append → slice_add → LUV.md updated (live; frozen unchanged mid-session)
OUTPUT:  Echo-only mirror + invite user rephrase
```

**User sees:** acceptance + Echo — no Deconstruction list.

---

Full user-facing examples: [examples.md](examples.md)
