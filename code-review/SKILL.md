---
name: code-review-standards
description: Apply rigorous, Google-style review standards when the user shares existing code, a diff, or a pull request and asks for feedback, a review, or "what's wrong with this." Use for explicit review requests — not for code Claude is generating fresh (see the code-generation-standards skill for that case). Flag issues with a short rationale, prioritized by severity, in a constructive tone.
---

# Code Review Standards

> Inspired by Google's [Engineering Practices documentation](https://google.github.io/eng-practices/) (CC-BY 3.0) and general production engineering norms. Not affiliated with or endorsed by Google. See README for details.

When reviewing code the user shares, evaluate it against the standards below. Flag issues with a short explanation of *why* it matters, not just *what* is wrong. Prioritize substance over style.

---

## What to check, roughly in priority order

1. **Correctness** — does the code do what it claims to do? Are there edge cases (empty input, null/None, zero, concurrent access) that would break it?
2. **Security** — untrusted input handled safely (no injection via raw string concatenation into queries/commands/paths), no hardcoded secrets, no secrets in logs.
3. **Backward compatibility / blast radius** — does this change a public interface, shared module, or existing behavior in a way that could break other callers? Flag this even if the user didn't ask about it.
4. **Resource and concurrency safety** — are files/connections/locks released deterministically? Any shared mutable state that could race?
5. **Design** — single responsibility per function/class; no mixing of unrelated concerns; no premature abstraction.
6. **Readability** — would another engineer understand this without the author explaining it? Overly clever code is a readability bug even if it's correct.
7. **Naming** — does naming match the language's own convention and the rest of the codebase, and is it unambiguous?
8. **Tests** — are the important cases (happy path, edge cases, error cases) covered? Do tests check behavior, not implementation details?
9. **Error handling** — are errors handled explicitly, with specific types, and meaningful messages? Anything swallowed silently?
10. **Code health** — duplication, unnecessary dependencies, leftover commented-out code, anything noticeably worse than the surrounding codebase.

---

## How to flag issues

Distinguish severity so the user can triage:

- **Blocking** — correctness bugs, security issues, breaking changes to public interfaces. These should be fixed before the code ships.
- **Should fix** — design issues, missing test coverage for real risk, error handling gaps.
- **Nit** — naming, minor style, comment wording. Worth mentioning but not worth blocking on. Label these explicitly as nits so they don't get conflated with real problems.

For each issue, briefly state what's wrong and why it matters — not just a rule citation. "This will throw on empty input because `.split()` never returns an empty list" is more useful than "handle edge cases."

Don't review against rules the codebase doesn't follow. If the surrounding code consistently uses a convention that differs from the defaults in this skill, defer to the codebase's convention and don't flag it as an issue.

---

## Tone

Be direct and specific, not exhaustive for its own sake. A review that nitpicks ten trivial things and misses one real bug is a bad review. Lead with anything blocking; group nits together at the end rather than interleaving them with substantive issues.

Don't be preachy or moralize about best practices in the abstract — point at the specific line/behavior and explain the concrete consequence.

If the code is genuinely solid, say so plainly rather than manufacturing nitpicks to seem thorough.
