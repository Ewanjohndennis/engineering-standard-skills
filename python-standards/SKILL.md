---
name: python-standards
description: Apply Python-specific engineering standards when writing, reviewing, refactoring, or debugging Python code. Use alongside code-generation-standards and code-review-standards — this skill adds Python idioms, ecosystem norms, and language-specific pitfalls that the generic skills can't cover. Triggers on any Python file, function, class, or snippet.
---

# Python Standards

> Language-specific standards for Python. Apply on top of the generic code-generation-standards and code-review-standards skills — this narrows them to Python idioms and common Python pitfalls.

---

## Type hints

Add type hints to all function signatures — parameters and return types. Use `from __future__ import annotations` at the top of files that need forward references. Prefer `X | None` over `Optional[X]` (Python 3.10+); use `Optional` only when the project targets older versions.

Use `TypedDict` for structured dictionaries that cross function boundaries. Use `Protocol` for structural subtyping instead of ABC where duck typing is the intent. Use `Final` for constants. Don't use `Any` unless there is genuinely no better option — if you do, leave a comment explaining why.

For complex types used more than once, define a type alias rather than repeating the full annotation.

---

## Pythonic patterns

Use Python's idioms — don't write Java or C in Python syntax:

- Prefer list/dict/set comprehensions over `map()`/`filter()` with lambdas when the comprehension is readable in one line. If it needs two lines, use an explicit loop.
- Use `enumerate()` instead of `range(len(...))`. Use `zip()` to iterate pairs. Use `itertools` for anything more complex than a single comprehension.
- Use unpacking: `first, *rest = items`, `a, b = b, a`. Don't index into tuples when unpacking is available.
- Use `collections.defaultdict`, `collections.Counter`, and `collections.deque` when they're the right tool — don't reimplement them.
- Use `dataclasses` for plain data containers. Use `__slots__` on classes that will be instantiated many times. Use `NamedTuple` when immutability is wanted and tuple unpacking is useful.
- Prefer `pathlib.Path` over `os.path` string manipulation for file system operations.
- Use walrus operator (`:=`) sparingly and only when it genuinely reduces repetition — not just because it's available.

---

## Context managers and resource safety

Always use `with` for anything that has a `__enter__`/`__exit__`: files, locks, database connections, HTTP sessions, subprocesses. Never rely on `__del__` or garbage collection to release resources.

For classes that manage resources, implement `__enter__` and `__exit__` (or use `contextlib.contextmanager` for simpler cases). If a class owns a resource but isn't used as a context manager, document clearly who is responsible for cleanup.

---

## Exception handling

Never use bare `except:` — it catches `SystemExit`, `KeyboardInterrupt`, and `GeneratorExit`, which is almost never intended. At minimum use `except Exception:`.

Catch the most specific exception type possible. Catching `Exception` broadly is acceptable at top-level boundaries (e.g. a request handler that must not crash the server), but nowhere else without a comment explaining why.

When re-raising with added context, use `raise NewException("context") from original_exception` — this preserves the original traceback and makes debugging significantly easier. Don't use `raise NewException(...) from None` unless you have a deliberate reason to suppress the chain.

Define custom exception classes for errors that callers need to handle programmatically — don't use `ValueError` or `RuntimeError` as a catch-all. Put custom exceptions in a dedicated `exceptions.py` module or near the code that raises them.

---

## Imports

Follow the standard import order: stdlib → third-party → local, with a blank line between each group. Use `isort` or `ruff` conventions if the project already uses them.

Never use wildcard imports (`from module import *`) except in `__init__.py` for deliberately re-exporting a public API, and even then define `__all__` explicitly.

Don't import inside functions unless it's to avoid a circular import or to defer an expensive import. If deferring for performance, leave a comment saying so.

---

## Module and package structure

Define `__all__` in any module that is imported by other modules — it makes the public API explicit and prevents accidental re-export of implementation details.

Keep `__init__.py` files thin. They should re-export the public API of the package, not contain logic.

Circular imports are a design smell. If you hit one, the fix is usually to extract the shared dependency into a third module, not to work around it with import-inside-function tricks.

---

## Async Python

Don't mix sync and async code carelessly. A sync function that calls `asyncio.run()` inside a running event loop will deadlock. An async function that calls a blocking I/O operation (file read, `requests.get`, `time.sleep`) blocks the entire event loop.

Use `asyncio.sleep(0)` to yield control in long-running async loops. Use `asyncio.gather()` for concurrent tasks, not sequential `await` in a loop when parallelism is the intent. Use `anyio` or `trio` primitives if the project uses them — don't mix concurrency frameworks.

For blocking I/O that can't be made async, use `asyncio.to_thread()` (Python 3.9+) or `loop.run_in_executor()` to run it in a thread pool.

---

## Testing

Use `pytest` unless the project uses something else. Write fixtures instead of setUp/tearDown. Use `pytest.mark.parametrize` for table-driven tests rather than looping inside a test function.

Use `unittest.mock.patch` carefully — patching at the wrong level (patching where the object is defined rather than where it's used) is a common mistake that makes tests pass even when the code is broken.

For async tests, use `pytest-asyncio` with `@pytest.mark.asyncio`. Don't call `asyncio.run()` inside tests.

Test files mirror the source tree: `src/module/foo.py` → `tests/module/test_foo.py`.

---

## Common pitfalls

- **Mutable default arguments**: `def f(x=[])` shares the list across all calls. Use `def f(x=None): x = x or []` instead.
- **Late binding closures**: variables in a lambda or inner function capture by reference, not by value. If you're creating closures in a loop, use `default argument binding`: `lambda i=i: ...`.
- **`is` vs `==`**: use `is` only for identity checks (typically `None`, `True`, `False`). Use `==` for value equality.
- **String concatenation in loops**: use `"".join(parts)` not `result += s` in a loop.
- **`datetime` without timezone**: always use timezone-aware datetimes (`datetime.now(tz=timezone.utc)`) in any code that crosses a system boundary or is stored/compared.
- **Float equality**: never use `==` to compare floats. Use `math.isclose()` or compare within a tolerance.
