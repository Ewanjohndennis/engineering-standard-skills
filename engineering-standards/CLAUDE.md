# CLAUDE.md

> This file is read automatically by Claude Code and Claude-powered tools at the start of every session. It tells Claude what it needs to know about this specific project — conventions, constraints, and context that no generic skill can infer. Keep it up to date as the project evolves.
>
> **Instructions for setup**: fill in the sections below. Delete any section that genuinely doesn't apply. Don't leave placeholder text in place — an absent section is better than a wrong one.

---

## Project overview

<!-- One paragraph: what this project does, who uses it, and what "working correctly" means in plain terms. -->

**What it is:**

**Who uses it:**

**What correct looks like:**

---

## Language and runtime

<!-- Fill in what applies. -->

- **Primary language(s):**
- **Language version(s):** <!-- e.g. Python 3.11, Java 17, C11 -->
- **Runtime / platform:** <!-- e.g. Linux x86-64, embedded ARM Cortex-M4, browser -->
- **Build system:** <!-- e.g. Make, CMake, Gradle, Poetry, pip -->

---

## Key dependencies and libraries

<!-- List the libraries Claude should know about and prefer. Include any that are unusual or project-specific. -->

<!-- Examples:
- HTTP client: use `httpx` (not `requests`) — already a dependency
- ORM: SQLAlchemy 2.x (async style with `async_session`)
- Test framework: pytest with pytest-asyncio
- Logging: structlog (not stdlib logging)
-->

**Use:**

**Avoid:** <!-- libraries that are banned, deprecated in this project, or have known issues here -->

---

## Project structure

<!-- Where things live. Only include what's non-obvious. -->

```
<!-- paste a trimmed directory tree here, or describe in prose -->
```

Key locations:
- Source root:
- Tests:
- Configuration:
- Entry point(s):

---

## Conventions

### Naming
<!-- Any project-specific naming rules beyond language defaults. E.g. "all database models are suffixed with `Model`", "all public API functions are prefixed with the module name" -->

### Error handling
<!-- How errors are surfaced in this project. E.g. "all functions return a Result-style tuple (value, error)", "we use custom exceptions defined in exceptions.py", "HTTP errors map to status codes via the error_map in utils/errors.py" -->

### Logging
<!-- What logger to use, what to log, what not to log. E.g. "use `logger = structlog.get_logger(__name__)`, log all external API calls at DEBUG, never log PII" -->

### Testing
<!-- Test location, naming convention, how to run. E.g. "tests mirror src layout under tests/", "run with `pytest -x`", "integration tests require a running DB and are marked @pytest.mark.integration" -->

---

## What to be careful about

<!-- Things that have caused bugs or confusion before. Real project-specific gotchas. -->

<!-- Examples:
- The `user_id` field in the database is a string UUID, not an integer — don't assume int
- The config module loads at import time; don't import it in test fixtures without mocking env vars first
- `session.commit()` auto-closes the session in our ORM setup — don't reuse sessions across requests
-->

---

## What not to touch

<!-- Files, modules, or patterns that should not be changed without a specific reason. -->

<!-- Examples:
- `src/legacy/` — do not refactor; it's being replaced by a rewrite in progress
- `infra/` — managed by Terraform; don't edit manually
- `migrations/` — never edit existing migration files; only add new ones
-->

---

## High-risk areas

<!-- Parts of the codebase where bugs are especially expensive — security, payments, data integrity, external contracts. Claude should apply extra scrutiny here. -->

<!-- Examples:
- `src/auth/` — authentication and authorization; any change here needs careful review
- `src/billing/` — payment processing; never make changes without a matching test
- `src/api/v1/` — public API; backward compatibility must not break
-->

---

## Commit and PR conventions

<!-- How to write commit messages and PR descriptions for this project. -->

- **Commit message style:** <!-- e.g. Conventional Commits (`feat:`, `fix:`, `chore:`), or imperative plain English -->
- **PR description:** <!-- e.g. must reference a GitHub issue, must include a "Testing" section -->
- **Branch naming:** <!-- e.g. `feat/short-description`, `fix/issue-123` -->

---

## Running the project

<!-- Minimal instructions to build, run, and test. -->

```sh
# Install dependencies


# Run tests


# Run locally


```

---

## Things Claude should know that aren't obvious from the code

<!-- Anything else: architectural decisions, why something is the way it is, known tech debt, planned changes that would affect generated code. -->
