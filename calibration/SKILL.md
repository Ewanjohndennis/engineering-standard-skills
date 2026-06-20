---
name: calibration
description: Ask one targeted calibration question before starting a code review or beginning a significant code generation task, to tailor output to the actual situation rather than a generic bar. Use at the start of any session where the user shares code for review, asks Claude to build a feature, or requests a refactor — before producing the review or the code. Do not use for quick one-off questions, bug fixes under ten lines, or when the user has already provided explicit context about the situation.
---

# Calibration

> Before reviewing or generating non-trivial code, ask one question to understand the context that most affects how to pitch the output. Don't ask multiple questions. Don't ask what can be inferred from the code itself. One question, then proceed.

---

## Why calibrate

The same code warrants completely different feedback depending on context. A prototype that's about to be thrown away needs different treatment than a public API that a hundred services will call. A hotfix deployed in the next hour needs different treatment than a feature going through normal review. Without knowing this, a review risks being either too strict (blocking progress on something that doesn't need it) or too lenient (waving through something that's going into production).

One question, asked upfront, resolves most of this ambiguity. It should not feel like a form — it should feel like the first thing a good teammate would ask before diving in.

---

## The calibration question

Pick the single question most relevant to what the user has shared. Only ask one. Examples by situation:

**For code review:**
- "Before I dig in — is this going to production, or is it a prototype / work in progress? That'll shape how much I focus on hardening vs. overall approach."
- "Is there a specific area you're most concerned about, or do you want a full pass?"
- "Is this a hotfix that needs to ship quickly, or is timeline relaxed?"

**For code generation:**
- "Is this for a production service or a script / prototype? Helps me calibrate how much to invest in error handling and edge cases."
- "Any constraints I should know about — target language version, libraries to use or avoid, performance requirements?"
- "Is this going to be maintained long-term by a team, or is it a one-off tool?"

**When the code suggests a specific concern** (e.g. it touches auth, payments, or external I/O):
- "This looks like it handles [auth / payments / external data] — should I treat security hardening as a priority, or is this an internal tool where that's less critical?"

---

## How to use the answer

**Production / long-lived / team-maintained:** apply all standards at full rigor. Flag should-fixes, not just blocking issues. Call out missing tests, missing docs, error handling gaps. Don't wave through things with "you can clean this up later."

**Prototype / throwaway / personal script:** focus on correctness and anything that would make the code actively misleading or dangerous. Skip nits. Don't push for full test coverage on a script that runs once. Still flag security issues if real credentials or untrusted data are involved — prototypes have a way of becoming production.

**Hotfix / time-sensitive:** focus on correctness and the specific bug. Flag anything that looks like it could make things worse. Note — but don't block on — anything that should be cleaned up afterward. Explicitly say what's being deferred and why.

**Unclear or mixed:** default to the higher bar, note the assumption, and offer to adjust if the context is different.

---

## What not to ask

- Don't ask about things visible in the code (language, framework, what the code does).
- Don't ask more than one question — it feels like a questionnaire, not a conversation.
- Don't ask if the user has already provided the context ("this is a quick prototype" / "this is going to prod next week" tells you everything you need).
- Don't ask on very small tasks (a ten-line function, a single bug fix) where the answer won't meaningfully change the output.

---

## Output behavior

Ask the calibration question naturally, as a single sentence, before the review or code. Don't explain that you're "calibrating" or reference this skill. Then wait for the answer before proceeding. If the user says "just go ahead" or doesn't answer the question, default to the production bar.
