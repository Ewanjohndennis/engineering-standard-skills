# engineering-standards-skills

Two [Claude Skills](https://docs.claude.com/en/docs/agents-and-tools/agent-skills) that make Claude hold code to a rigorous, production-grade bar — automatically, without being asked each time.

- **`code-generation/`** — applied whenever Claude writes, refactors, or debugs code.
- **`code-review/`** — applied whenever you share existing code, a diff, or a PR and ask for feedback.

They're split because they trigger on different things: writing fresh code and reviewing someone else's diff call for different framing (one is "produce this to a bar," the other is "evaluate this against a bar and prioritize what to say"). If a single request does both ("write this, then review it for issues"), both skills load together automatically — Claude matches each skill's `description` against the task and applies every skill that's relevant.

## Why

Most "write clean code" system prompts are vague enough that models ignore them under task pressure. These are opinionated and concrete (a hard rule on commented-out code, a heuristic for when a function is too long, explicit severity tiers for review feedback) so they actually change output instead of sitting there as a platitude.

They're language- and framework-agnostic by design: both skills explicitly defer to a project's existing conventions before falling back to their own defaults, so they don't fight the idioms of whatever language or codebase they're applied to.

## Install

Drop each folder into your skills directory:

- **Claude Code / CLI**: `~/.claude/skills/<name>/SKILL.md` (user-level) or `.claude/skills/<name>/SKILL.md` (project-level)
- **claude.ai / Claude Cowork**: upload each folder as a custom skill via Settings → Capabilities → Skills

No extra prompting needed afterward — each skill triggers automatically based on what you're asking for.

## What's covered

**code-generation**: design, complexity, naming (codebase-aware), comments, tests, error handling, security, concurrency/resource safety, code health, dependency hygiene.

**code-review**: a priority-ordered checklist (correctness → security → backward compatibility → concurrency → design → readability → naming → tests → error handling → code health), severity tiers (blocking / should-fix / nit), and guidance on tone so reviews stay specific and non-preachy.

## Attribution

Both skills are inspired by the ideas in Google's publicly released [Engineering Practices documentation](https://google.github.io/eng-practices/) (the Code Reviewer's Guide and Change Author's Guide), which Google makes available under a [CC-BY 3.0 license](https://github.com/google/eng-practices/blob/master/LICENSE), plus general production-engineering norms (security, concurrency, backward compatibility) not covered in that source. No text from Google's documentation is reproduced here — this is an original rewrite of the underlying principles, adapted and extended for use as LLM skills.

**This project is not affiliated with, endorsed by, or sponsored by Google.** "Google" is referenced only to describe the source of inspiration for some of these engineering principles.

## License

MIT — see [LICENSE](./LICENSE). Use it, fork it, adapt it for your own team's standards.
