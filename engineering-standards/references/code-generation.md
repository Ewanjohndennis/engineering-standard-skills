
# Code Generation Standards

> Inspired by Google's [Engineering Practices documentation](https://google.github.io/eng-practices/) (CC-BY 3.0) and general production engineering norms. Not affiliated with or endorsed by Google. See README for details.

When writing or modifying any code, internalize and apply the following standards automatically. Don't narrate that you're doing this — just produce code that meets this bar.

---

## Priority order

These standards can pull in different directions — "match the existing codebase" and "use modern idioms" aren't always compatible; neither are "don't over-engineer" and "design for the future." When two principles in this file genuinely conflict, resolve it in this order:

1. **Correctness** — the code must do what it's supposed to do.
2. **Security** — never trade safety for elegance or speed.
3. **Existing codebase conventions** — consistency beats a theoretically better pattern.
4. **Maintainability** — will this be easy to change safely later?
5. **Readability** — will another engineer understand this quickly?
6. **Performance** — fast, but only after the above are satisfied.
7. **Style preferences** — the least important axis; never worth a real tradeoff elsewhere.

Never sacrifice a higher-priority concern to satisfy a lower-priority one. If a request seems to require that tradeoff, say so explicitly rather than silently picking one side.

---

## Match the existing codebase first

Before applying any naming or style rule below: look at the surrounding code. If the project already has a convention (casing style, file layout, error-handling pattern, test framework), follow that convention even if it differs from the defaults here. These defaults are a fallback for greenfield code or when no convention exists yet — they are not meant to override an established codebase's style.

---

## Confidence

Do not invent APIs, framework features, library methods, or version-specific behavior. If you're not certain a method, parameter, or behavior actually exists in the library/version being used, say so explicitly rather than presenting a guess as fact. A confident-sounding hallucinated import is worse than an honest "I'm not certain this exists in this version — worth double-checking the docs."

When uncertain:
- State the uncertainty plainly rather than hedging vaguely.
- Ask for the specific library/framework version if it would resolve the ambiguity.
- Prefer documented, stable APIs over obscure ones you're recalling from memory with low confidence.

This matters more than it sounds: code that compiles but calls a method that doesn't exist, or that quietly relies on behavior that changed between versions, is more dangerous than code that's missing a feature, because it fails later and less obviously.

---

## Design

Write code that fits cleanly into the existing architecture. Every function, class, or module should have one clear responsibility. Avoid mixing concerns — a function that fetches data should not also format it for display.

Don't over-engineer. Only implement what is needed right now. If the user might need something later, note it briefly in a comment but don't build it yet.

When designing APIs or interfaces, ask: would another developer be able to understand and use this correctly without reading the implementation?

Consider blast radius: prefer the smallest change that correctly solves the problem, especially when touching shared/public interfaces. If a change could break existing callers, call that out explicitly rather than silently changing a signature or behavior.

---

## Keep changes self-contained and small

Each piece of code generated should do one thing. If the user asks for a large feature, think about the natural seams — what's a working, shippable increment versus what's the whole thing at once? When it makes sense, say so: "I'll implement the data model and its tests first; the API layer can come next so each piece is reviewable on its own."

Don't fold refactoring into a feature change silently. If you're also cleaning up something unrelated while implementing what was asked, call it out clearly — either as a separate block, or with a comment marking it as a distinct cleanup — so the user can see what's a behavior change and what's housekeeping.

When producing tests, include them in the same output as the code they cover, not as an afterthought. A function and its tests belong together.

### When not to refactor

Don't refactor code solely because it could be marginally cleaner. Refactor when you're fixing a bug, adding a feature, removing real duplication, or directly improving maintainability of code you're already touching for another reason. Avoid large, unrelated rewrites — if a file could be improved but nothing about the current task requires it, mention it briefly and move on rather than rewriting it unprompted.

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

## Documentation

If the code being written or changed alters a public API, a CLI interface, configuration options, or the way something is built, tested, or deployed: update the relevant documentation too. This means README sections, docstrings, inline usage examples, or whatever the project uses. Don't generate a behavior change and leave the docs describing the old behavior. If there's no existing documentation to update but the new code warrants it, add a minimal docstring or usage note.

---

## Commit and PR descriptions

When asked to write a commit message or PR/CL description alongside code, apply the same rigor:

- **First line**: a short, specific summary of *what* the change does, written as an imperative sentence ("Add retry logic to the payment client", not "Added retry logic" or "Retry logic"). It should stand alone — someone reading git log should understand the change without opening the diff.
- **Body**: explain *why* the change is being made. What problem does it solve? What did you decide and why? Are there any tradeoffs or known shortcomings? Include relevant context (bug numbers, links to design docs, benchmark results) when that context will help a future reader.
- Avoid vague descriptions like "Fix bug", "Add patch", "Refactor", or "Phase 1" — these communicate nothing useful to someone searching history later.

---

## Tests

Write testable code by default. This means:
- Avoid hidden state and global variables
- Prefer pure functions where possible
- Accept dependencies as parameters (makes them injectable/mockable)

When writing test code, each test should verify one behavior. Test names should describe the scenario being tested (`test_returns_empty_list_when_no_users_found`, not `test_users`).

Prioritize test coverage in this order when you can't cover everything equally:
1. Critical business logic — the code where a bug actually costs something.
2. Edge cases — empty input, zero, None/null, boundary values.
3. Error paths — does it fail the way it's supposed to?
4. Integration points — boundaries with external systems, other modules.
5. Happy paths — the straightforward case, last, since it's also the one most likely to already work.

Don't write tests just to hit a coverage number — write tests that would catch real bugs. Avoid snapshot-heavy tests that just freeze current output without asserting anything meaningful, and avoid mocking everything in a unit test to the point that the test only verifies the mocks were called.

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

## Performance

Don't optimize prematurely. Before recommending or applying an optimization:
- Identify the actual bottleneck — don't guess.
- Estimate the real impact of fixing it.
- Weigh the added complexity against that impact.

Prefer algorithmic improvements (better data structure, fewer passes over the data) over micro-optimizations (loop unrolling, premature caching) — the former tends to matter more and cost less in readability.

Default to writing code that's reasonably efficient without obsessing over it: avoid obviously wasteful patterns (quadratic behavior on data that can grow large, repeated work that could be cached or hoisted out of a loop, unnecessary copies of large data), but don't trade away readability for a speedup nobody asked for and nothing measured.

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
