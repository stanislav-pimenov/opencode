---
name: python-conventions
description: Python build, test and lint commands plus idiomatic Python rules. Use when writing, reviewing or verifying Python code (pyproject.toml, *.py, test_*.py).
---

# Python conventions

Repo `AGENTS.md`, `Makefile`, `tox.ini` and CI config win over these defaults.
Detect the package manager first: `uv.lock`, `poetry.lock`, `Pipfile.lock`, else venv + pip.

## Commands

```sh
uv run pytest -q                    # or: poetry run pytest -q  /  pytest -q
uv run ruff check .                 # or: poetry run ruff check .
uv run ruff format --check .        # or: black --check .
uv run mypy .                       # or: pyright
```

Run one test: `uv run pytest tests/test_x.py::test_name -q`.

## Rules

- Type hints on all public functions and methods; use `from __future__ import annotations` for modern syntax on older Pythons.
- Never use mutable default arguments (`def f(x=[])`); use `None` + create inside.
- Catch specific exceptions. Never `except Exception: pass`; never swallow errors to make tests pass.
- Use `pathlib.Path` over `os.path`; use `with` for all resources.
- Prefer dataclasses or pydantic models over loose dicts for structured data.
- Use `logging` (`logger = logging.getLogger(__name__)`), not `print`, in library/application code.
- F-strings for formatting; `enumerate`/`zip` instead of index loops.
- Async: never call blocking I/O inside `async def`; use `asyncio.to_thread` or an async client.
- Tests: `pytest`, fixtures for setup, `tmp_path` for files, `monkeypatch` for env/time, parametrize for cases. Assert behavior, not internals.
- Do not add a dependency without checking `pyproject.toml`; report it if needed.
- Virtualenvs only — never install into the system interpreter.
