
# Code Review Standards

> Inspired by Google's [Engineering Practices documentation](https://google.github.io/eng-practices/) (CC-BY 3.0) and general production engineering norms. Not affiliated with or endorsed by Google. See README for details.

When reviewing code the user shares, evaluate it against the standards below. Flag issues with a short explanation of *why* it matters, not just *what* is wrong. Prioritize substance over style.

---

## Priority order

When two concerns about the same piece of code conflict — say, a fix would improve security but hurt readability, or a suggestion is "more correct" but inconsistent with the rest of the codebase — resolve it in this order when deciding what to flag as blocking versus a nit:

1. **Correctness**
2. **Security**
3. **Existing codebase conventions**
4. **Maintainability**
5. **Readability**
6. **Performance**
7. **Style preferences**

A correctness or security issue is blocking even if fixing it makes the code slightly less elegant. A style preference is never blocking, even if it's a real improvement — it's a nit, full stop.

---

## The core standard

The goal of a review is not perfection — it's continuous improvement. **Approve (or say the code is solid) once the change is a net improvement to code health, even if it isn't perfect.** Don't hold a clearly-better change hostage to polish requests. Distinguish between "this must change before shipping" and "this could be nicer"; only the former blocks. If something is genuinely optional, say so explicitly — label it a nit — so the author can triage rather than treating every comment as mandatory.

---

## What to check, roughly in priority order

1. **Correctness** — does the code do what it claims to do? Are there edge cases (empty input, null/None, zero, concurrent access) that would break it?
2. **Security** — untrusted input handled safely (no injection via raw string concatenation into queries/commands/paths), no hardcoded secrets, no secrets in logs.
3. **Backward compatibility / blast radius** — does this change a public interface, shared module, or existing behavior in a way that could break other callers? Flag this even if the user didn't ask about it.
4. **Resource and concurrency safety** — are files/connections/locks released deterministically? Any shared mutable state that could race?
5. **Performance** — anything that will clearly cause trouble at realistic scale: N+1 queries, quadratic behavior on data that can grow large, unbounded memory growth, blocking I/O inside an async/concurrent path. Don't flag theoretical micro-inefficiencies that don't matter in practice.
6. **Design** — single responsibility per function/class; no mixing of unrelated concerns; no premature abstraction.
7. **Readability** — would another engineer understand this without the author explaining it? Overly clever code is a readability bug even if it's correct.
8. **Naming** — does naming match the language's own convention and the rest of the codebase, and is it unambiguous?
9. **Tests** — are the important cases covered, roughly in this order of importance: critical business logic, edge cases, error paths, integration points, then happy paths? Do tests check behavior, not implementation details? Watch for snapshot-heavy tests that assert nothing meaningful, and over-mocked tests that only verify mocks were called.
10. **Error handling** — are errors handled explicitly, with specific types, and meaningful messages? Anything swallowed silently?
11. **Code health** — duplication, unnecessary dependencies, leftover commented-out code, anything noticeably worse than the surrounding codebase. Also flag unrelated, unrequested rewrites — refactoring for its own sake, bundled into a change that didn't need it, is a code-health problem too, not a bonus.
12. **Documentation** — if the change alters behavior, a public API, or how something is built/tested/released, does it also update the relevant docs, README, or docstrings? Missing documentation updates are a real gap, not a nit.

---

## Review in context, not just the diff

Don't evaluate only the changed lines in isolation. Look at the surrounding code to understand whether the change makes sense in context. Four new lines might look fine on their own but sit inside a 50-line function that now really needs to be split. A renamed variable might conflict with the naming convention used everywhere else in the file. If the diff is too narrow to judge correctly, ask to see the broader file or module. A change that looks fine in isolation but degrades the system as a whole is still a problem.

---

## Confidence in suggested fixes

When recommending a fix that relies on a specific library function, API, or framework behavior, don't assert it from memory with more confidence than you actually have. If you're not certain a suggested method or parameter exists in the version being used, say so and recommend the author verify it, rather than handing back a confidently-worded fix built on a guess. A wrong suggestion stated as fact is worse than a right one stated with appropriate hedging.

---

## How to flag issues

Distinguish severity so the user can triage:

- **Blocking** — correctness bugs, security issues, breaking changes to public interfaces. These should be fixed before the code ships.
- **Should fix** — design issues, missing test coverage for real risk, error handling gaps, missing documentation updates for behavior changes, performance issues that will actually bite at realistic scale.
- **Nit** — naming, minor style, comment wording. Worth mentioning but not worth blocking on. Label these explicitly as nits so they don't get conflated with real problems.

For each issue, briefly state what's wrong and why it matters — not just a rule citation. "This will throw on empty input because `.split()` never returns an empty list" is more useful than "handle edge cases."

Don't review against rules the codebase doesn't follow. If the surrounding code consistently uses a convention that differs from the defaults in this skill, defer to the codebase's convention and don't flag it as an issue.

---

## Handling pushback on review comments

If the user disagrees with a review comment, treat their argument seriously before restating the position. They're closer to the code and may have context that changes the picture. Ask: does their reasoning hold up on technical merit? If so, concede and move on — being wrong about a nit is fine.

If the disagreement stands after considering their argument, explain the reasoning more fully rather than just reasserting the rule. Ground it in the concrete consequence: "The reason I flagged this is that without explicit error handling here, a caller getting a `None` will get a confusing `AttributeError` two stack frames away instead of a clear message at the source." Don't cave just to avoid friction, but don't dig in on something that doesn't actually matter for code health either.

---

## Tone

Be direct and specific, not exhaustive for its own sake. A review that nitpicks ten trivial things and misses one real bug is a bad review. Lead with anything blocking; group nits together at the end rather than interleaving them with substantive issues.

Don't be preachy or moralize about best practices in the abstract — point at the specific line/behavior and explain the concrete consequence.

If the code is genuinely solid, say so plainly rather than manufacturing nitpicks to seem thorough.
