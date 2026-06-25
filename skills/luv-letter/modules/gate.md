# Module: Gate

Manages depth pacing — **not** a content layer. Appended after Reflection when policy triggers.

## Modes

| Mode | When |
|------|------|
| `offer` | `round_count >= 4` and phase transitions ACTIVE → GATED |
| `await` | phase already GATED — parse user choice |
| `clarify` | Reason uncertain about user gate response |

## offer Output

Append after Reflection blocks:

```yaml
blocks:
  - type: gate
    content: "你想继续往深处走，还是先在这里停一停？"
side_effects:
  gate_pending: true
  set_phase: GATED
```

## await — Parsing User Choice

| Signal | Route intent |
|--------|--------------|
| 继续 / 深入 / go deeper / yes (in gate context) | `user_choose_depth` |
| 停 / 休息 / pause / 先这样 / rest here | `user_choose_pause` |
| New substantive content ignoring gate | implicit `user_choose_depth` |
| Ambiguous | `gate_clarify` |

## clarify Output

One short line only:

> 我想确认一下——你是想继续往深处走，还是先在这里停一停？

No Reflection content in clarify turn. Do not increment depth.

## Gate Rules

- Never offer gate before round 4
- Never offer gate in same turn as `pause.close`
- Gate question is the **only** question allowed in a gated offer turn (Extension questions still allowed in Reflection portion — max total 3 questions per turn including gate)
