# engineering-standards-skills

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Claude Skills](https://img.shields.io/badge/Claude-Skills-blueviolet)](https://docs.claude.com/en/docs/agents-and-tools/agent-skills)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](#contributing)

A collection of [Claude Skills](https://docs.claude.com/en/docs/agents-and-tools/agent-skills) that make Claude hold code to a rigorous, production-grade bar — automatically, without being asked each time.

> Stop re-explaining "write clean code, handle errors, add tests" in every prompt. Drop these in once, and Claude applies the standard silently from then on.

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

### Core skills — install these for everyone

| Skill | Folder | Triggers on |
|---|---|---|
| **Code generation** | [`code-generation/`](./code-generation) | Writing, refactoring, or debugging code |
| **Code review** | [`code-review/`](./code-review) | Reviewing existing code, a diff, or a PR |
| **Calibration** | [`calibration/`](./calibration) | Before any non-trivial review or generation task |

### Language skills — install the ones you use

| Skill | Folder | Triggers on |
|---|---|---|
| **Python standards** | [`python-standards/`](./python-standards) | Any Python file, function, or snippet |
| **Java standards** | [`java-standards/`](./java-standards) | Any Java file, class, method, or snippet |
| **C standards** | [`c-standards/`](./c-standards) | Any C file, function, struct, or snippet |

### Project config — one per repo

| File | Purpose |
|---|---|
| **CLAUDE.md** | Project-specific context: conventions, constraints, gotchas. Copy to your project root and fill it in. |

---

## How it works

Skills are Markdown files Claude reads at the start of a session. Each has a `description` field that tells Claude when to apply it — Claude matches the description against what you're asking and loads every relevant skill automatically. If you ask Claude to write a Python function, it loads `code-generation`, `calibration`, and `python-standards` together without you doing anything.

Skills stack in layers:

1. **`calibration`** fires first — asks one question to understand context (prototype vs production, hotfix vs normal timeline) before producing anything
2. **`code-generation` or `code-review`** sets the general engineering bar
3. **Language skill** narrows it to idioms and pitfalls specific to the language in use
4. **`CLAUDE.md`** narrows it further to your specific project's conventions and constraints

Each layer adds specificity without overriding the ones below it. The whole thing is silent — nothing to invoke manually.

---

## Install

Each skill is a folder containing a single `SKILL.md`. Place them under your skills directory and Claude picks them up automatically.

**Claude Code / CLI — user level** (applies across all your projects):

```sh
mkdir -p ~/.claude/skills

# Core skills
cp -r code-generation ~/.claude/skills/
cp -r code-review ~/.claude/skills/
cp -r calibration ~/.claude/skills/

# Language skills — add the ones you use
cp -r python-standards ~/.claude/skills/
cp -r java-standards ~/.claude/skills/
cp -r c-standards ~/.claude/skills/
```

**Claude Code / CLI — project level** (applies only in one repo):

```sh
mkdir -p .claude/skills
cp -r code-generation .claude/skills/
cp -r code-review .claude/skills/
# etc.
```

**claude.ai / Claude Cowork**: upload each folder as a custom skill via Settings → Capabilities → Skills.

**CLAUDE.md**: copy it to your project root and fill in the sections. Delete any section that doesn't apply — an absent section is better than a wrong one.

> **Recommendation:** install core and language skills at user level so they apply everywhere. Keep `CLAUDE.md` at the project level since it's project-specific.

---

## What's covered

**`code-generation`** — design, complexity, naming, comments, tests, error handling, security, concurrency/resource safety, code health, dependency hygiene, commit/PR description quality, keeping changes self-contained.

**`code-review`** — the "net-improving is good enough" principle, priority-ordered checklist (correctness → security → backward compatibility → concurrency → design → readability → naming → tests → error handling → docs), severity tiers (blocking / should-fix / nit), reviewing in context not just the diff, handling pushback on review comments.

**`calibration`** — asks one targeted question before non-trivial work and uses the answer to pitch output at the right level of rigor.

**`python-standards`** — type hints, pythonic patterns, context managers, exception chaining, async pitfalls, import discipline, module structure, common gotchas (mutable defaults, late-binding closures, datetime without timezone, float equality).

**`java-standards`** — modern Java idioms (records, sealed classes, `var`, pattern matching), nullability discipline, immutability, exception handling with cause chains, try-with-resources, streams, concurrency, common pitfalls (Integer cache, Calendar vs java.time, array equality).

**`c-standards`** — memory management, buffer/string safety, undefined behavior avoidance, error handling patterns, struct and header discipline, resource/fd safety, pthreads concurrency, `goto cleanup` pattern, common pitfalls (realloc failure, char signedness, format string mismatches).

**`CLAUDE.md`** — template covering: project overview, language/runtime, dependencies (use/avoid), project structure, conventions (naming, errors, logging, testing), gotchas, no-touch zones, high-risk areas, and commit conventions.

---

## Example

**Without these skills**, asking Claude to write a Python function often gets you something that works but skips the important parts — no type hints, bare `except`, mutable default arguments, no tests.

**With `code-generation` + `python-standards` installed**, the same request comes back with proper type annotations, explicit exception types with cause chaining, no mutable defaults, and a `pytest` test covering the happy path and edge cases. No extra prompting.

**With `calibration` installed**, before diving into a review Claude asks: *"Is this going to production, or is it a prototype? That'll shape how much I focus on hardening vs. overall approach."* A throwaway script doesn't get blocked on missing test coverage; a payment handler does.

**With `code-review` + `java-standards` installed**, sharing a PR gets you triaged feedback:

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

**Do I need all the skills, or can I pick and choose?**
Pick and choose. Core skills work standalone. `calibration` is a small addition that improves both. Language skills only matter for the languages you use. `CLAUDE.md` is worth filling in for any project you work on regularly.

**Can I install language skills at user level even if I don't use that language in every project?**
Yes — the `description` field in each skill means Claude only loads it when the code is actually in that language. Installing extras costs nothing when you're working in a different language.

**Will this conflict with my own `CLAUDE.md` or project conventions?**
No — the core skills explicitly defer to existing codebase conventions before applying their own defaults. `CLAUDE.md` is the highest-priority layer and overrides everything else.

**Can I edit these for my own team's standards?**
Yes, that's the point — fork it, edit the Markdown, done. No code, no build step. The language skills are designed to be extended: add your preferred libraries, add project-specific gotchas.

---

## Attribution

The core skills are inspired by Google's publicly released [Engineering Practices documentation](https://google.github.io/eng-practices/) (CC-BY 3.0). The language skills draw on general production engineering norms for each language. No text from Google's documentation is reproduced here — this is an original rewrite of the underlying principles adapted for use as LLM skills.

**This project is not affiliated with, endorsed by, or sponsored by Google.**

---

## Contributing

Issues and PRs welcome — especially real-world examples of where a skill under- or over-triggered, language-specific conventions worth calling out, and additional language skills (TypeScript, Rust, Go, etc.). Keep additions concrete (a rule + a one-line reason), not just more general advice.

---

## License

MIT — see [LICENSE](./LICENSE). Use it, fork it, adapt it for your own team's standards.