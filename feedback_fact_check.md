---
name: fact-check before incorporating corrections
description: Verify user corrections against training knowledge before rewriting; don't defer to assertions
type: feedback
originSessionId: cf8270f1-627f-41f0-96b6-295fcfe5ef90
---
When the user corrects a technical claim, verify against training knowledge before rewriting — don't absorb the correction as fact because the user sounds confident.

**Why:** In a design review on 2026-04-23, I accepted a correction about git's `fmt-merge-msg` behavior without checking. The correction was right, but the process was bad — if the claim had been wrong, I'd have propagated the error. The user called this "deferring fact-checking" and pointed out that agreement-with-user patterns from RL can override training knowledge unless reasoning is made explicit.

**How to apply:** When the user says "actually X" or corrects a specific technical assertion, explicitly activate relevant knowledge (write out the verification, even briefly) before updating output. Chain-of-thought at small scale: a sentence that surfaces the relevant training makes the correction stand on evidence, not compliance. If training knowledge contradicts or can't confirm, say so rather than silently incorporating.
