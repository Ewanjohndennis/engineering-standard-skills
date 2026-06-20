---
name: code-generation-standards
description: Apply rigorous engineering standards when writing, generating, refactoring, or debugging code. Use whenever the user asks Claude to write a function, build a feature, refactor existing code, fix a bug, or add tests. Even for casual requests ("write me a function that..."), apply these standards silently — don't announce it, just produce code that meets this bar.
---

# Code Generation Standards

> Inspired by Google's [Engineering Practices documentation](https://google.github.io/eng-practices/) (CC-BY 3.0) and general production engineering norms. Not affiliated with or endorsed by Google. See README for details.

When writing or modifying any code, internalize and apply the following standards automatically. Don't narrate that you're doing this — just produce code that meets this bar.

---

## Match the existing codebase first

Before applying any naming or style rule below: look at the surrounding code. If the project already has a convention (casing style, file layout, error-handling pattern, test framework), follow that convention even if it differs from the defaults here. These defaults are a fallback for greenfield code or when no convention exists yet — they are not meant to override an established codebase's style.

---

## Design

Write code that fits cleanly into the existing architecture. Every function, class, or module should have one clear responsibility. Avoid mixing concerns — a function that fetches data should not also format it for display.

Don't over-engineer. Only implement what is needed right now. If the user might need something later, note it briefly in a comment but don't build it yet.

When designing APIs or interfaces, ask: would another developer be able to understand and use this correctly without reading the implementation?

Consider blast radius: prefer the smallest change that correctly solves the problem, especially when touching shared/public interfaces. If a change could break existing callers, call that out explicitly rather than silently changing a signature or behavior.

---

## Complexity

Simpler is better. If two approaches solve the problem, choose the one that is easier to read.

Avoid clever tricks. Code is read far more often than it is written. A future developer (or the user themselves, 3 months from now) should be able to understand any given function in under a minute.

Keep functions short. If a function is doing too many things, split it. A good heuristic: if you need to scroll to see the whole function, it's probably too long.

Don't add unnecessary layers of abstraction. Each layer of indirection should earn its place by genuinely simplifying something.

---

## Naming

Names should be descriptive and unambiguous. Avoid single-letter variables except for conventional loop indices (`i`, `j`) or well-understood math.

Apply the casing convention idiomatic to the language and project (e.g. `snake_case` in Python/Rust, `camelCase` in JavaScript/Java, `PascalCase` for types in most C-family languages) rather than forcing one style across all languages:

- Functions: verb phrases that describe what they do (`fetch_user_data` / `fetchUserData`)
- Booleans: prefix with a form of `is`/`has`/`can`/`should` (`is_authenticated` / `isAuthenticated`)
- Types/classes: nouns that describe what they represent (`UserRepository`, `PaymentProcessor`)
- Constants: follow the language's convention for immutable values (e.g. `ALL_CAPS_SNAKE_CASE` in Python/C, `PascalCase` or `const` bindings elsewhere)
- Avoid abbreviations unless they are universally understood in the domain (`url`, `id`, `http` are fine; `usrNm` is not)

---

## Comments

Comments explain **why**, not **what**. The code itself explains what it does. Comments explain decisions that aren't obvious from reading the code.

Good comment: `// Using exponential backoff here to avoid thundering herd on retry`
Bad comment: `// Increment i by 1`

Don't leave commented-out code blocks. Either keep the code or delete it — version control preserves history.

Document public functions, classes, and modules using the idiom of the language (docstrings, Javadoc, doc comments, etc). Describe what it does, its parameters, its return value, and any edge cases worth flagging.

---

## Tests

Write testable code by default. This means:
- Avoid hidden state and global variables
- Prefer pure functions where possible
- Accept dependencies as parameters (makes them injectable/mockable)

When writing test code, each test should verify one behavior. Test names should describe the scenario being tested (`test_returns_empty_list_when_no_users_found`, not `test_users`).

Cover the important cases: happy path, edge cases (empty input, zero, None/null), and error cases. Don't write tests just to hit a coverage number — write tests that would catch real bugs.

Don't test implementation details. Test observable behavior. If you can refactor the internals without changing the tests, the tests are well-designed.

---

## Error handling

Handle errors explicitly. Don't swallow exceptions silently. If you catch an exception, either recover from it, log it with context, or re-raise/propagate it with additional information.

Use specific exception/error types, not bare `except Exception` or `catch (e)`. This makes it easier to reason about what can go wrong.

Return meaningful error messages. If a function can fail, the caller should be able to understand why without digging into the implementation.

---

## Security

Treat all external input (user input, API responses, file contents, environment variables) as untrusted until validated.

- Never construct queries, shell commands, or file paths via raw string concatenation of untrusted input — use parameterized queries, safe APIs, or proper escaping.
- Never hardcode secrets, tokens, or credentials. Read them from environment variables or a secrets manager, and never log them.
- Validate and sanitize input at trust boundaries (where external data enters the system), not just deep inside business logic.

---

## Concurrency and resource safety

When code involves threads, async tasks, or shared state, be explicit about what is and isn't safe to access concurrently. Avoid introducing race conditions or deadlocks through shared mutable state.

Always release resources (files, connections, locks, sockets) deterministically — use the language's idiomatic mechanism for this (`with`/context managers, `try/finally`, `defer`, RAII, `using`, etc.) rather than relying on manual cleanup that can be skipped on an early return or exception.

---

## Code health

Don't leave the codebase worse than you found it. If you're touching a file and notice a small unrelated issue (a poorly named variable, a missing type hint), it's fine to fix it — but keep that fix visibly separate from the main change (a distinct commit, or called out clearly) rather than folding it invisibly into the same diff.

Avoid duplication. If the same logic appears twice, it belongs in a shared function. Three or more times, it definitely does.

Dependencies: prefer the standard library or an already-used dependency over introducing a new one for the same job. Each new dependency is a maintenance and supply-chain burden — only add what's genuinely necessary.

---

## Output behavior

Apply all of the above silently when producing code. The user should not see a preamble like "Following these engineering practices, I will...". Just write good code.

If the user's request would result in code that violates one of these principles in a significant way (e.g. they ask for a function that does six unrelated things), it's fine to gently suggest a better structure — but keep it brief and constructive, not preachy.
