# Module: Pause

Closes an active thread when user chooses rest or crisis aborts session.

## Modes

| Mode | When |
|------|------|
| `close` | user_choose_pause, crisis abort, explicit "先停" |

## Pre-Close: flush(pause)

**Before** emitting close block, Orchestrator runs `flush(trigger=pause)`:

1. Distill `confirmed_insight` → INNER.md (with user_anchor)
2. Write `pause_snapshot` + `theme_anchor` → archive
3. Write final `thread_distill` → THREAD.md or archive
4. Clear thread_buffer

See [memory-tool.md](../memory-tool.md) flush spec.

## close Output

Echo-only mirror — **no** Deconstruction, **no** new questions.

```yaml
blocks:
  - type: close
    content: |
      [2–3 sentences mirroring where thread arrived]
      [Optional: "当你准备好，可以再来。"]
side_effects:
  flush: { trigger: pause }     # if not already run
  set_phase: PAUSED
```

## Snapshot Rules

Episodic continuity = **archive slices**, not YAML entries:

| Written on pause | store | type |
|------------------|-------|------|
| User-confirmed insight | inner + archive | `confirmed_insight` |
| Thread closing note | archive | `pause_snapshot` |
| Recurring theme | inner + archive | `theme_anchor` |

Do **not** snapshot offered-but-rejected hypotheses into INNER.

## After Pause

New session:

1. Load frozen INNER/LUV
2. `slice_search(type=theme_anchor, limit=1)` — one invitation, not assertion
3. Ask: continue theme or new fragment?
