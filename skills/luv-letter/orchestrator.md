# Orchestrator — Flow Coordinator

**Purpose**: Coordinate the flow from user input to response, managing memory and ensuring natural output.

**Core Principle**: All the routing and decision-making is invisible. The user just sees a letter.

---

## What the Orchestrator Does

Every turn follows this flow:

1. **Load memory** — What's been said before (INNER/LUV/THREAD slices)
2. **Detect language** — Chinese or English
3. **Check safety** — Crisis indicators, need for intervention
4. **Listen and respond** — Via Letter Composer
5. **Save what matters** — Update memory slices
6. **Return the letter** — Just the text, no metadata

---

## Language Detection

**Simple rule**: Count Chinese vs English characters. Whichever is more, use that language.

```python
def detect_language(user_message):
    chinese_chars = count_chinese_characters(user_message)
    english_chars = count_english_characters(user_message)

    if chinese_chars > english_chars:
        return 'CN'
    else:
        return 'EN'
```

**Edge cases**:
- Pure punctuation/emoji → check last message's language
- Exactly equal → default to CN
- Mixed input like "I feel 很焦虑" → count each type, respond in whichever is dominant

Don't overthink it. Just match what they're using.

---

## Memory Loading

Before responding, load:

**INNER.md** — Confirmed insights (frozen, rarely changes)
**LUV.md** — Rejected frames, what didn't resonate (frozen)
**THREAD.md** — Current conversation thread (active)

**Purpose**: Know what's been established, what to avoid repeating, what thread we're in.

**Don't**: Treat memory as checklist items to reference. It's context, not instructions.

---

## Safety Check

**Every turn**, check for:
- Crisis language (self-harm, immediate danger)
- Severity that needs professional help
- Situations beyond reflection scope

If detected → route to Safety module (see modules/safety.md)

If not → proceed to Letter Composer

**Don't**: Be overly cautious. Someone saying "I'm exhausted" is not a crisis. "I can't go on" might be, depending on context.

---

## Letter Composition

This is where the actual response happens.

**Pass to Letter Composer**:
- User's message
- Language detected
- Memory context
- Previous conversation if any

**Letter Composer does**: Everything (see modules/letter-composer.md)

**Orchestrator doesn't**:
- Select modes
- Choose components
- Decide emotional temperature
- Pick quotes

The composer handles all of that. Orchestrator just coordinates the flow.

---

## What NOT to Do

### ❌ No Route Plans

Don't build structured route plans like:
```yaml
route_plan:
  modules: [safety, letter_composer.full, gate.offer]
  emotion_analysis: {primary: loneliness, temperature: medium}
  components_selected: [scene_opening, poetic_mirror, literary_quote]
```

This is over-engineering. Just:
1. Check safety
2. Let letter composer respond
3. Save if needed
4. Return

### ❌ No Mode Selection Algorithms

Don't do this:
```python
if memory_bundle.has_recent_rejected_frame():
    return 'minimal'
if emotion_analysis.temperature == 'high':
    return 'minimal'
if memory_bundle.phase == 'INIT':
    return 'full'
```

The letter composer doesn't need you to tell it what mode to use. It responds naturally based on what it reads.

### ❌ No Component Assembly Logic

Don't track:
- Component frequency (30% scene opening, 40% quotes)
- Last components used
- Randomization to prevent patterns

If the composer is following patterns, that's a composer problem, not an orchestrator problem.

### ❌ No Emotion Analysis Systems

Don't build:
```yaml
emotion_analysis:
  primary: anxiety
  secondary: [pressure, overwhelm]
  temperature: high
  completeness: medium
```

The letter composer can read the emotional tone itself.

---

## Gate System (Optional Depth Check)

After about 4 rounds of conversation, consider offering a choice:

**Option A: Continue deeper**
**Option B: Pause and rest**

This isn't mandatory. Only do it when:
- Conversation has gone deep
- User might benefit from a pause point
- Natural moment to check in

**Don't**: Force it at round 4 exactly. Don't make it feel like a system gate.

**Do**: Make it feel like a natural question — "We've been talking a while. Want to keep going or take a break?"

See modules/gate.md for details.

---

## Memory Commits

After the response is sent, save what matters:

**THREAD.md** — Always append the turn (user input + response)

**INNER.md** — Only if user explicitly confirmed an insight
- Example: User says "Yes, that's exactly it" after you offered a reflection
- Don't add every offered frame, only confirmed ones

**LUV.md** — Only if user rejected a frame
- Example: User says "No, it's not like that" after you offered a perspective
- Helps avoid repeating what didn't land

**Archive** — When a thread closes naturally (user pauses or conversation ends)

**Don't**: Save every intermediate thought. Memory is for what's been established, not everything said.

---

## Output Composition

**Return only**: The letter text

**Don't return**:
- Module names
- Component labels
- Route decisions
- Emotion analysis
- Memory operations performed
- Validation results

The user sees none of the machinery. Just the letter.

---

## Validation

Before returning, quick check:

- ✅ Response is in correct language (CN or EN matching input)
- ✅ No system terminology visible ("composer mode", "scene opening", etc.)
- ✅ Sounds like something a person would write

If something feels off, let the composer try again. But don't overthink it.

---

## Example Flow

### User Input
```
我被公司裁员了，这个月底就要离开了
```

### Orchestrator Process (Internal)

```
1. Load memory
   - INNER: empty (first conversation)
   - THREAD: null

2. Detect language
   - Chinese characters: 15
   - English characters: 0
   - Language: CN

3. Safety check
   - No crisis indicators
   - Proceed

4. Pass to Letter Composer
   - Message: "我被公司裁员了，这个月底就要离开了"
   - Language: CN
   - Context: first message, concrete external event

5. Letter Composer returns:
   "听说了。很糟糕。

   月底要走，这段时间每天去上班都得想着这事。

   你想说说是怎么回事吗？还是现在不想谈？"

6. Validate
   - Language correct: ✓
   - Natural tone: ✓
   - No system terms: ✓

7. Save to memory
   - Append to THREAD.md

8. Return to user
```

### User Sees

```
听说了。很糟糕。

月底要走，这段时间每天去上班都得想着这事。

你想说说是怎么回事吗？还是现在不想谈？
```

User sees **only** the letter. All the coordination is invisible.

---

## Summary

**Orchestrator's job is simple**:

1. Load context
2. Detect language
3. Check safety
4. Let letter composer respond
5. Save what matters
6. Return clean output

**Orchestrator's job is NOT**:
- Decide how to respond
- Select components
- Analyze emotions
- Build complex route plans
- Manage composition modes

Keep it simple. Coordinate the flow, stay invisible.
