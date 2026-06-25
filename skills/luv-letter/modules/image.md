# Module: Image

Handles image-as-trigger input. Runs **before** Reflection `partial` mode.

## Principle

Convert visual input → internal state. **Not** object recognition.

## Modes

| Mode | Behavior |
|------|----------|
| `subjective` | Default — attention / feeling / association only |

## Processing

```yaml
observe_image:
  # DO NOT populate:
  object_inventory: forbidden   # "这是玫瑰/办公桌/ sunset"
  symbolism_decode: forbidden   # "这象征孤独"

  # DO populate (internal, for Memory.image_note):
  attention_prompt: ready       # what to ask user about
```

## Output

Does not describe the image. Returns cue for Reflection:

```yaml
blocks: []                      # Image module adds no direct user text
side_effects:
  image_present: true
  image_note: "subjective-only trigger"
reflection_handoff:
  mode: partial
  extension_seeds:
    - "这张画面里，最先抓住你注意力的是哪一块？"
    - "它在你内在世界里，更像哪种感觉？"
```

Reflection `partial` weaves caption (if any) into Echo; uses extension_seeds.

## User asks "这是什么？"

Orchestrator routes redirect:

> Luv Letter 更关心它在你心里唤起了什么，而不是它是什么。

Then subjective Extension — no object answer.

## Memory

- Store `image_present: true`
- `image_note` = subjective theme only after user responds — never object list
