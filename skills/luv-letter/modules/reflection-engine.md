# Module: Reflection Engine

Generates Echo / Deconstruction / Extension blocks. **Does not route or commit memory** — returns structured blocks to Orchestrator.

## Modes

| Mode | Blocks emitted | When |
|------|----------------|------|
| `full` | echo + deconstruction + extension | INIT, new confirmed thread |
| `adaptive` | Reason picks subset from Memory | ACTIVE continue |
| `deep` | echo + extension (no new axes) | choose_depth, depth_level≥2, confirmed frame exists |
| `echo_only` | echo + 1 invite question | reject_frame |
| `redirect` | echo + extension (action pull) | action_seeking |
| `partial` | echo + extension | after Image module; no object echo |
| `invite_fragment` | 1 invite line only | empty_input |

## Block Contracts

### echo

- 1–2 short paragraphs
- Rephrase user input; preserve emotional tone
- **No** causality labels, hidden beliefs, structure words, questions

### deconstruction

- Tentative language only: 似乎 / 也许 / 不一定对
- Structured list allowed (scaffolding, not advice)
- Dimensions: emotion, trigger, assumption, tension, unspoken
- End with **one** confirmation invite
- Register offered frames in `side_effects.slice_add` as **thread_buffer only** — `confidence: offered`, **not** INNER.md

```yaml
side_effects:
  slice_add:                   # thread_buffer only, confidence=offered
    - store: thread
      type: thread_distill
      dimension: assumption
      content: "..."
```

### extension

- 1–2 open questions
- No advice, conclusion, optimization, yes/no
- In `deep` mode: target `frames_confirmed` or last user correction only

## Mode Selection (called by Orchestrator)

```
if mode == echo_only:
  emit echo + one invite ("用你的话说，这种烦更像…")

elif mode == deep:
  emit echo
  skip deconstruction
  emit extension targeting confirmed dimension

elif mode == redirect:
  emit echo acknowledging action-pull
  emit extension: "'怎么办'背后，是哪类不安在推着你？"
  no action steps

elif mode == full:
  emit echo + deconstruction + extension

elif mode == adaptive:
  if frames_rejected slice exists for thread (from slice_search) → echo_only
  elif confirmed_insight slice exists → deep
  elif depth_level >= 2 → deep
  else → full
```

## Tone (all modes)

| Be | Not |
|----|-----|
| Gentle | Authoritative |
| Reflective | Instructive |
| Slow, spaced | Dense, efficient |
| Warm neutrality | Cold analysis |

Never: life advice, therapist voice, bullet action lists, layer labels to user.

## Anti-Patterns (Validate scans these)

- Stating hidden beliefs as facts
- Closing with summary judgment
- Recommendations or steps
- Multiple unrelated topics
- More than 3 questions total (1 confirm + 2 extension)
