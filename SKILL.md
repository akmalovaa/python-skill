---
name: python-skill
description: Use when writing or reviewing Python code, starting a Python project, editing pyproject.toml, configuring linting/typing/testing, choosing Python libraries, handling errors or exceptions, or naming functions, variables, and classes.
license: MIT
---

# Python Stack

Default stack for new Python code. Existing projects: keep their tools, layout, and conventions (poetry, requests, mypy, unittest, …) — don't migrate or flag it unless asked. Tool, layout, config, and command guidance below is greenfield-only; naming, fail-loud, exception/async hygiene, and testing discipline apply to any Python code, using the project's existing stack.

## Stack

| Task | Tool |
|---|---|
| Packages, venv, Python versions | **uv** |
| Lint + format | **ruff** |
| Type checking | **pyright** |
| Models / validation at boundaries | **pydantic v2** |
| Config from env / `.env` | **pydantic-settings** |
| HTTP client | **httpx2** ([Pydantic's continuation of httpx](https://github.com/pydantic/httpx2) — real package, not a typo; API continues httpx 0.x) |
| Logging | **structlog** |
| Tests | **pytest** + pytest-asyncio, pytest-httpx2, time-machine |

Add when needed: **FastAPI** + uvicorn (API), **SQLAlchemy 2.0** + alembic + psycopg 3 (DB), **typer** (CLI), **tenacity** (retries).

uv, ruff, pyright, pytest — every project. The rest only when the project has that concern: pydantic when there's boundary data, structlog when there's logging, pydantic-settings when there's config. A tiny CLI doesn't need them.

## Don't use → use instead

| Don't | Use |
|---|---|
| pip, poetry, pipenv, pyenv | uv |
| requirements.txt, setup.py | pyproject.toml + uv.lock |
| black, isort, flake8, pylint | ruff |
| mypy (greenfield) | pyright |
| requests, aiohttp, httpx | httpx2 |
| python-dotenv, scattered os.environ | pydantic-settings |
| print for logging | structlog |
| freezegun | time-machine |
| unittest | pytest |

## Layout & commands

Always `src/` layout: `pyproject.toml`, `uv.lock`, `src/<package>/`, `tests/`.

```bash
uv init --package            # new project with src/ layout (--lib for a library)
                             # note: doesn't create tests/ — mkdir it yourself
uv add <pkg>                 # dependency (--dev for dev)
uv add --dev ruff pyright pytest   # tools as dev deps, so `uv run` finds them
                                   # + pytest-asyncio / pytest-httpx2 / time-machine when the project needs them
uv sync                      # install

# local
uv run ruff check --fix . && uv run ruff format .
uv run pyright && uv run pytest

# CI — read-only, never --fix
uv sync --locked
uv run --locked ruff check . && uv run --locked ruff format --check .
uv run --locked pyright && uv run --locked pytest
uv audit --locked            # audit dependencies for known vulnerabilities
```

`ruff check --fix` + `ruff format` don't fix everything — E501 (long lines) needs a manual wrap.

Ruff + pyright baseline — `N` enforces naming case, `PTH` forces pathlib, `ARG` catches unused arguments:

```toml
[tool.ruff.lint]
select = ["E", "F", "I", "B", "UP", "SIM", "N", "PTH", "ARG", "RUF"]

[tool.ruff.lint.per-file-ignores]
"tests/**" = ["ARG"]  # fixtures look like unused args

[tool.pyright]
typeCheckingMode = "strict"  # enforces "type hints everywhere"; drop to "basic" if strict is too noisy
```

One-off scripts: `uv run --script` with PEP 723 inline metadata — no full project needed.

## Naming

Linters check case only — semantics is on you. Applies to code you write or modify; when reviewing existing code, flag naming only on changed lines (full-file renames only when asked):

- Functions/methods = verb + noun: `load_user_profile()`, `send_report()` — never a bare noun (`user_profile()`). Doesn't apply where the name is set by a protocol or API: dunders, framework hooks, `@property` accessors (nouns), and established conventional names (`main()`, `close()`, `commit()`).
- Booleans = predicate: `is_active`, `has_permission`, `can_retry`.
- Classes = noun naming one responsibility: `InvoiceGenerator`, not `InvoiceManager`.
- One verb per concept per project: don't mix `get` / `fetch` / `retrieve` for the same thing.
- Banned name words: `data`, `info`, `utils`, `helper`, `manager`, `process`, `handle`, `tmp` — unless the word *is* the domain term (an OS `process`, a file `handle`).
- Modules and packages: short snake_case domain nouns.
- Every name says what it holds or does, in domain terms — no single-letter names, including loop counters and comprehension variables. Only exceptions: `e` in `except ... as e`, `_` for discards.
- No magic numbers — named `UPPER_SNAKE_CASE` constants.

## Rules

- Type hints everywhere, including return types. Modern syntax: `list[str]`, `X | None` — not `typing.List`, `Optional` (match the project's `requires-python`).
- Validate data at boundaries (HTTP, env, external APIs) with pydantic; don't pass raw dicts around.
- async when the calling stack is async or concurrent I/O is needed; plain scripts stay sync. No blocking calls (`time.sleep`, sync clients) inside async — wrap unavoidable ones in `asyncio.to_thread()`.
- `pathlib.Path`, not string paths.
- No mutable default args (`def f(x, items=[])`); `is None`, not `== None`; `isinstance(x, T)`, not `type(x) == T`; no `from module import *`.
- Specific exceptions, chained with `raise X(...) from e`; no bare `except:`; no `except Exception` outside entry-point boundaries (`main()`, request handler, worker loop).

## HTTP (httpx2)

- Reuse one long-lived `Client` / `AsyncClient` per configuration (context manager or app lifecycle) — never a new client per request.
- Set explicit timeouts; call `raise_for_status()` unless non-2xx is part of the protocol.
- Retry only idempotent operations — bounded attempts with backoff + jitter (tenacity).
- "Don't use httpx" means don't import it; it may still show up as a transitive dep (pytest-httpx2 pulls it in) — that's fine.

## Fail loud

- Don't collapse exceptions into strings (`return {"error": str(e)}`) — traceback and type are lost.
- Never substitute fake/sample data when a fetch or parse fails — fail instead.
- Batch loops: collect per-item errors and report them; no silent `continue`.
- Optional scalars: guard with `is None`, not truthiness — `if timeout:` silently breaks a legit `0`. (`if items:` for collections is fine and idiomatic.)
- At entry-point boundaries (the same ones where `except Exception` is allowed): translate to a typed domain error (for APIs — an error response), log the traceback once, never leak it to the client.

## Tests

- Mock the boundary (HTTP via pytest-httpx2 or `httpx2.MockTransport`, subprocess), never the function under test.
- pytest-httpx2's fixture is `httpx2_mock` — a respx-style router: `httpx2_mock.get(url).respond(json={...})`.
- async tests: pytest-asyncio with `asyncio_mode = "auto"` in pytest config.
- No conditional assertions; assert preconditions first — a test that can't fail is worse than none.
- Bugfix order: failing regression test first, then the fix. If the fix came first — revert it, confirm the test fails for the right reason, restore.
