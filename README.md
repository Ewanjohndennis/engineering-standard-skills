# engineering-standards-skills

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Claude Skills](https://img.shields.io/badge/Claude-Skills-blueviolet)](https://docs.claude.com/en/docs/agents-and-tools/agent-skills)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](#contributing)

Two [Claude Skills](https://docs.claude.com/en/docs/agents-and-tools/agent-skills) that make Claude hold code to a rigorous, production-grade bar — automatically, without being asked each time.

> Stop re-explaining "write clean code, handle errors, add tests" in every prompt. Drop these in once, and Claude applies the standard silently from then on.

## Table of contents

- [What's in here](#whats-in-here)
- [Why this exists](#why-this-exists)
- [Install](#install)
- [What's covered](#whats-covered)
- [Example](#example)
- [FAQ](#faq)
- [Attribution](#attribution)
- [Contributing](#contributing)
- [License](#license)

## What's in here

| Skill | Folder | Triggers on |
|---|---|---|
| **Code generation** | [`code-generation/`](./code-generation) | Writing, refactoring, or debugging code |
| **Code review** | [`code-review/`](./code-review) | Reviewing existing code, a diff, or a PR you share |

They're split because writing fresh code and reviewing someone else's diff need different framing — one is "produce this to a bar," the other is "evaluate this against a bar and prioritize what to say." If a single request does both ("write this, then review it for issues"), both skills load together automatically: Claude matches each skill's `description` against the task and applies every skill that's relevant.

## Why this exists

Most "write clean code" system prompts are vague enough that models ignore them under task pressure. These are opinionated and concrete — a hard rule on commented-out code, a heuristic for when a function is too long, explicit severity tiers for review feedback — so they actually change output instead of sitting there as a platitude.

They're language- and framework-agnostic by design: both skills explicitly defer to a project's existing conventions before falling back to their own defaults, so they don't fight the idioms of whatever language or codebase they're applied to.

## Install

Drop each folder into your skills directory:

- **Claude Code / CLI**: `~/.claude/skills/<name>/SKILL.md` (user-level) or `.claude/skills/<name>/SKILL.md` (project-level)
- **claude.ai / Claude Cowork**: upload each folder as a custom skill via Settings → Capabilities → Skills

No extra prompting needed afterward — each skill triggers automatically based on what you're asking for.

## What's covered

**`code-generation`** — design, complexity, naming (codebase-aware), comments, tests, error handling, security, concurrency/resource safety, code health, dependency hygiene.

**`code-review`** — a priority-ordered checklist (correctness → security → backward compatibility → concurrency → design → readability → naming → tests → error handling → code health), severity tiers (blocking / should-fix / nit), and guidance on tone so reviews stay specific and non-preachy.

## Example

**Without this skill**, asking Claude to write a quick helper often gets you something that works but skips the boring-but-important parts — no input validation, swallowed exceptions, no tests.

**With `code-generation` installed**, the same request silently comes back with explicit error types, validated input, and a docstring — no extra prompting, no "please also add tests" follow-up needed.

**With `code-review` installed**, sharing a PR gets you triaged feedback instead of a wall of text:

```
Blocking:
- Line 42: SQL built via string concatenation — injection risk. Use a parameterized query.

Should fix:
- No test for the empty-list case; `.split()` returns `['']` not `[]`, so this will silently misbehave.

Nits:
- `tmp` on line 8 could be named for what it holds.
```

## FAQ

**Do I need both skills, or can I use just one?**
Either works standalone. Install just `code-generation` if you only want better output when Claude writes code for you; install just `code-review` if you mainly paste in code and ask for feedback.

**Will this conflict with my own `CLAUDE.md` or project conventions?**
No — both skills explicitly check for and defer to existing codebase conventions before applying their own defaults (see "Match the existing codebase first" in `code-generation/SKILL.md`).

**Can I edit these for my own team's standards?**
Yes, that's the point — fork it, edit the Markdown, done. No code, no build step.

## Attribution

Both skills are inspired by the ideas in Google's publicly released [Engineering Practices documentation](https://google.github.io/eng-practices/) (the Code Reviewer's Guide and Change Author's Guide), which Google makes available under a [CC-BY 3.0 license](https://github.com/google/eng-practices/blob/master/LICENSE), plus general production-engineering norms (security, concurrency, backward compatibility) not covered in that source. No text from Google's documentation is reproduced here — this is an original rewrite of the underlying principles, adapted and extended for use as LLM skills.

**This project is not affiliated with, endorsed by, or sponsored by Google.** "Google" is referenced only to describe the source of inspiration for some of these engineering principles.

## Contributing

Issues and PRs welcome — especially real-world examples of where a skill under- or over-triggered, or language-specific conventions worth calling out explicitly. Keep additions concrete (a rule + a one-line reason), not just more general advice.

## License

MIT — see [LICENSE](./LICENSE). Use it, fork it, adapt it for your own team's standards.
