# 3 · Finding files, and choosing what to ignore

## The skip list is a design decision, not a detail

```python
DEFAULT_SKIP_DIRS = frozenset({
    ".git", "__pycache__", ".mypy_cache", ".pytest_cache", ".ruff_cache",
    "node_modules", "dist", "build", ".venv", "venv", ".tox", ".nox",
    "vendor", "site-packages",
    "tests", "test", "testing", "examples", "example", "cookbook", "docs",
})
```

Three groups, three different reasons:

| Group | Why skipped |
|---|---|
| `.git`, `__pycache__`, caches | not source code |
| `node_modules`, `vendor`, `site-packages`, `.venv` | **third-party — not yours to fix** |
| `tests`, `examples`, `cookbook`, `docs` | ⚠️ **deliberately contain "bad" patterns** |

That last group is the insight. A test file *should* contain `def f(x=[])` — it's testing that
the bug is detected. An examples folder demonstrates anti-patterns on purpose. Scanning them
generates noise that makes the tool feel broken.

**A linter that cries wolf gets uninstalled.** Choosing what *not* to look at is as much of the
design as the rules themselves.

## Pruning during the walk

```python
for dirpath, dirnames, filenames in os.walk(root):
    dirnames[:] = [d for d in dirnames if d not in skip_dirs]
```

The `dirnames[:] = ...` slice assignment is the important bit. `os.walk` reads that list to
decide where to descend next, so mutating it **in place** prunes those subtrees entirely —
it never even enters `node_modules`. Rebinding with `dirnames = [...]` would silently do
nothing.

## Handling broken files

Any real repository contains a file that won't parse — Python 2 leftovers, a template with
placeholders, a truncated file. Those must be skipped quietly rather than crashing the scan.
**Robustness is a feature:** a tool that dies on file 400 of 2,000 is useless against the
codebases you most want to scan.

> Next: [../04_the_checks/CH001_blocking_async.md](../04_the_checks/CH001_blocking_async.md)
