# engineering-standards-skills

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Claude Skills](https://img.shields.io/badge/Claude-Skills-blueviolet)](https://docs.claude.com/en/docs/agents-and-tools/agent-skills)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](#contributing)

A [Claude Skill](https://docs.claude.com/en/docs/agents-and-tools/agent-skills) that makes Claude hold code to a rigorous, production-grade bar — automatically, without being asked each time.

> Stop re-explaining "write clean code, handle errors, add tests" in every prompt. Install this once, and Claude applies the standard silently from then on.

## Table of contents

- [What's in here](#whats-in-here)
- [How it works](#how-it-works)
- [Install](#install)
- [What's covered](#whats-covered)
- [Example](#example)
- [FAQ](#faq)
- [Attribution](#attribution)
- [Contributing](#contributing)
- [License](#license)

---

## What's in here

```
engineering-standards/
├── SKILL.md                       ← entry point: routes to the files below
└── references/
    ├── code-generation.md         ← standards for writing/refactoring/debugging code
    ├── code-review.md             ← standards for reviewing existing code/diffs/PRs
    ├── calibration.md              ← the one question to ask before non-trivial work
    ├── python-standards.md        ← Python idioms and pitfalls
    ├── java-standards.md          ← Java idioms and pitfalls
    └── c-standards.md             ← C idioms, memory safety, undefined behavior
```

It's a single skill. `SKILL.md` is the only file Claude always has visibility into (via its `name` + `description`); everything in `references/` only gets read into context when `SKILL.md`'s routing logic decides it's actually relevant to the current task. Ask for a Python function and it pulls in `code-generation.md` + `python-standards.md` (and `calibration.md` if the task is non-trivial). Ask for a Java code review and it pulls in `code-review.md` + `java-standards.md`. Nothing irrelevant loads.

A `CLAUDE.md` template is also included at the repo root — that's separate from the skill itself (project-specific context Claude Code reads automatically), not something you install the same way.

---

## How it works

Skills use progressive disclosure: Claude always has the `name` and `description` from every installed skill's frontmatter in context (a tiny footprint), and only loads the full body — and, in this case, the referenced files — when it decides the skill is actually relevant.

`SKILL.md`'s body is a short decision tree:

1. **Is this worth calibrating?** Non-trivial tasks (a real feature, a full review) get one question first (`references/calibration.md`); small fixes skip straight to step 2.
2. **Generation or review?** Reads `references/code-generation.md` or `references/code-review.md` accordingly — both if the task is both.
3. **Which language?** Detects Python/Java/C from the code or the request and reads the matching reference file on top.

All of this happens silently — Claude doesn't narrate which files it's reading or that a skill fired at all.

---

## Install

It's one folder — `engineering-standards/`, containing `SKILL.md` and `references/`. The folder name has to stay `engineering-standards` (it matches the `name:` field inside `SKILL.md`).

**claude.ai / Claude Cowork:**
1. Zip the `engineering-standards/` folder (the zip must contain the folder itself at its root, not just the files inside it).
2. Settings → Capabilities → Skills → Upload.

**Claude Code / CLI — user level** (applies across all your projects):
```sh
cp -r engineering-standards ~/.claude/skills/
```

**Claude Code / CLI — project level** (applies only in one repo):
```sh
mkdir -p .claude/skills
cp -r engineering-standards .claude/skills/
```

**CLAUDE.md** (optional, separate from the skill): copy it to your project root and fill in the sections. Delete any section that doesn't apply — an absent section is better than a wrong one.

---

## What's covered

**Code generation** (`references/code-generation.md`) — design, complexity, naming, comments, tests, error handling, security, concurrency/resource safety, code health, dependency hygiene, commit/PR description quality, keeping changes self-contained, priority order for conflicting standards, confidence/anti-hallucination guidance, performance, refactoring discipline.

**Code review** (`references/code-review.md`) — the "net-improving is good enough" principle, a priority-ordered checklist (correctness → security → backward compatibility → concurrency → performance → design → readability → naming → tests → error handling → code health → docs), severity tiers (blocking / should-fix / nit), reviewing in context rather than just the diff, handling pushback on review comments, confidence in suggested fixes.

**Calibration** (`references/calibration.md`) — asks one targeted question before non-trivial work and uses the answer to pitch output at the right level of rigor (production vs. prototype vs. hotfix).

**Python standards** (`references/python-standards.md`) — type hints, pythonic patterns, context managers, exception chaining, async pitfalls, import discipline, module structure, common gotchas (mutable defaults, late-binding closures, datetime without timezone, float equality).

**Java standards** (`references/java-standards.md`) — modern Java idioms (records, sealed classes, `var`, pattern matching), nullability discipline, immutability, exception handling with cause chains, try-with-resources, streams, concurrency, common pitfalls (Integer cache, Calendar vs java.time, array equality).

**C standards** (`references/c-standards.md`) — memory management, buffer/string safety, undefined behavior avoidance, error handling patterns, struct and header discipline, resource/fd safety, pthreads concurrency, `goto cleanup` pattern, common pitfalls (realloc failure, char signedness, format string mismatches).

**CLAUDE.md** — template covering: project overview, language/runtime, dependencies (use/avoid), project structure, conventions (naming, errors, logging, testing), gotchas, no-touch zones, high-risk areas, and commit conventions.

---

## Example

**Without this skill**, asking Claude to write a Python function often gets you something that works but skips the important parts — no type hints, bare `except`, mutable default arguments, no tests.

**With it installed**, the same request comes back with proper type annotations, explicit exception types with cause chaining, no mutable defaults, and a `pytest` test covering the happy path and edge cases. No extra prompting.

Before diving into a non-trivial review, Claude asks: *"Is this going to production, or is it a prototype? That'll shape how much I focus on hardening vs. overall approach."* A throwaway script doesn't get blocked on missing test coverage; a payment handler does.

Sharing a Java PR for review gets triaged feedback:

```
Blocking:
- Line 67: catching Exception broadly and swallowing it silently — if the DB
  connection fails here, the caller gets a success response. Catch SQLException
  specifically and propagate with context.

Should fix:
- getUserById() returns null on miss — callers have to null-check manually.
  Return Optional<User> so the type system enforces it.
- No test for the empty result case.

Nits:
- `Calendar` on line 12 — use java.time.Instant instead.
- `tmp` on line 34 could be named for what it holds.
```

---

## FAQ

**Does this work the same on claude.ai and Claude Code?**
Yes — it's the same `SKILL.md` + `references/` folder either way. claude.ai needs it zipped for upload; Claude Code just needs it copied into a skills directory. No format differences.

**Can I disable just the Java parts, or just code review, and keep the rest?**
Not at the skill level — it's one skill, so it's installed or it isn't. The routing inside `SKILL.md` already avoids pulling in irrelevant reference files per task, so in practice you won't see Java guidance show up on a Python question regardless.

**Will this conflict with my own `CLAUDE.md` or project conventions?**
No — the standards in `references/code-generation.md` and `references/code-review.md` explicitly defer to existing codebase conventions before applying their own defaults. `CLAUDE.md` (project-specific, separate from this skill) is the highest-priority layer and overrides everything else.

**Can I edit these for my own team's standards?**
Yes — edit the files under `references/` directly. There's a single source of truth per topic; no separate copies to keep in sync.

**If I add a new language, do I need to touch `SKILL.md`?**
Yes, in two small places: add the new reference file under `references/`, and add a line to `SKILL.md`'s routing step telling Claude when to read it. Keep `SKILL.md`'s own `description` broad enough to still cover the new language, or it won't even get a chance to load the file.

---

## Attribution

The core standards are inspired by Google's publicly released [Engineering Practices documentation](https://google.github.io/eng-practices/) (CC-BY 3.0). The language-specific files draw on general production engineering norms for each language. No text from Google's documentation is reproduced here — this is an original rewrite of the underlying principles, adapted for use as an LLM skill.

**This project is not affiliated with, endorsed by, or sponsored by Google.**

---

## Contributing

Issues and PRs welcome — especially real-world examples of where the skill under- or over-triggered, language-specific conventions worth calling out, and additional language reference files (TypeScript, Rust, Go, etc.). Keep additions concrete (a rule + a one-line reason), not just more general advice. If you add a language, remember to also update the routing logic in `SKILL.md` so it actually gets read.

---

## License

MIT — see [LICENSE](./LICENSE). Use it, fork it, adapt it for your own team's standards.