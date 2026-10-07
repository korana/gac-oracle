---
pattern: "Always post-process MyMemory Thai output: MyMemory drops English sentence-ending punctuation (. ? !) when translating to Thai, causing sentences to run together. Repair with Thai full stop ๏ before clause-initial words (เนื่องจาก, คุณ after แล้ว)."
date: 2026-10-07
source: rrr: korana/gac-oracle
concepts: [translation, mymemory, thai, post-processing, mixed-script]
---
# MyMemory Thai Punctuation Fix

## The Problem

MyMemory's MT engine drops English sentence-ending punctuation (`.`, `?`, `!`) when translating English → Thai. Thai sentences merge into one long run-on text with no sentence breaks.

**Example input:**
```
...incorrect. Because...
```

**Raw MyMemory output:**
```
...不正确เนื่องจาก...
```

The period is gone — `不正确` (incorrect) and `เนื่องจาก` (because) are glued together.

## Root Cause

MyMemory normalizes the text before translation and strips ASCII punctuation when the target is Thai. The Thai equivalent of a period (๏) is not inserted.

## The Fix

Post-process MyMemory Thai output with regex:

```python
import re

def mymemory_thai_fix(text):
    # 1. Space between adjacent Latin/ASCII and Thai (mixed-script guard)
    text = re.sub(r'([A-Za-z0-9])([\u0E00-\u0E7F])', r'\1 \2', text)
    text = re.sub(r'([\u0E00-\u0E7F])([A-Za-z0-9])', r'\1 \2', text)

    # 2. Insert Thai full stop ๏ before clause-initial words
    text = re.sub(r'(\S)(เนื่องจาก)', r'\1 ๏ \2', text)  # because/due to
    text = re.sub(r'(แล้ว)(คุณ)', r'\1 ๏ \2', text)       # you after already/done

    # 3. Clean
    text = re.sub(r'  +', ' ', text)
    return text.strip()
```

## Why These Specific Patterns

| Pattern | Thai meaning | Why it signals a sentence break |
|---------|-------------|--------------------------------|
| `เนื่องจาก` | because/due to | Starts a new clause after a dropped period |
| `แล้ว` + `คุณ` | already/done + you | `แล้ว` ends the previous clause; `คุณ` starts the next |

## Do Not Over-Generalize

Only insert ๏ at these specific boundaries. A general "insert ๏ after any Thai verb phrase" risks over-segmenting valid Thai sentence structure. Conservative targeting at known English-period drop points is safer.

## Reference

- Skill: `/home/gacai/.hermes/skills/productivity/translate/SKILL.md` (v1.2.0)
- MyMemory API: `api.mymemory.translated.net/get`
