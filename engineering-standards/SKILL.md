---
name: engineering-standards
description: Apply rigorous engineering standards when writing, refactoring, debugging, or reviewing Python, Java, or C code. Loads calibration and language-specific guidance automatically.
---

# Engineering Standards

> Inspired by Google's [Engineering Practices documentation](https://google.github.io/eng-practices/) (CC-BY 3.0) and general production engineering norms. Not affiliated with or endorsed by Google. See README for attribution details.

This skill bundles several layers of engineering guidance. Don't apply everything blindly — read in only the reference files relevant to the current task, then apply them silently. Never narrate which files you're reading or that you're "loading a skill."

---

## How to route

**Step 1 — Is this worth calibrating?**

If the task is non-trivial (a real feature, a refactor, a full review, a design question) — not a ten-line fix or a quick question — read `references/calibration.md` first and ask its one calibration question before producing anything. Skip this for small tasks or when the user has already given context (e.g. "this is a quick prototype," "this is going to prod").

**Step 2 — Generation or review?**

- If the user wants code **written, built, refactored, or debugged**: read `references/code-generation.md` and apply it.
- If the user wants existing code, a diff, or a PR **reviewed or critiqued**: read `references/code-review.md` and apply it.
- If the task involves both (e.g. "write this, then review it"), read both.

**Step 3 — Which language?**

Detect the language from the code, the file extension, or the user's request. Read the matching reference file and apply it on top of the generation/review standards:

- Python → `references/python-standards.md`
- Java → `references/java-standards.md`
- C → `references/c-standards.md`

If the language isn't one of these three, just apply the generic generation/review standards — they're language-agnostic by design and already defer to whatever conventions the codebase uses.

If a single task spans multiple languages (e.g. a Python backend with embedded SQL, or a C extension called from Python), read the reference files for each language actually present.

---
 
## Priority order across all layers

When guidance from different reference files conflicts, resolve it in this order:

1. **Correctness**
2. **Security**
3. **Existing codebase conventions**
4. **Maintainability**
5. **Readability**
6. **Performance**
7. **Style preferences**

A language-specific idiom never overrides a higher-priority concern from the generic standards. Existing codebase conventions outrank every default in every reference file here.

---

## Output behavior

Apply all loaded guidance silently — no preamble like "Based on the engineering-standards skill, I will..." Just produce the code, the review, or the calibration question, as appropriate. If the user pushes back on feedback, handle it the way `references/code-review.md` describes.
