# 1 · Why write a linter at all?

## The kind of bug that survives code review

```python
async def wait_for_service():
    while not ready():
        time.sleep(1)
```

Nothing here looks wrong. It reads fine. It passes tests. It works perfectly when one person
uses the app.

In production with 100 concurrent users, **all 100 freeze for a full second, repeatedly.**

That's the character of every bug codehound looks for:

- **locally invisible** — the line is fine in isolation, it's the *context* that makes it a bug
- **silent** — nothing raises, nothing logs, no test fails
- **only appears under concurrency or over time** — exactly when it's most expensive

A human reviewer scanning a 400-line diff will not catch this. A machine that knows
"is this call inside an `async def`?" catches it every time.

## Static vs dynamic analysis

| | Static analysis | Dynamic analysis |
|---|---|---|
| Runs your code? | ❌ no — reads the source | ✅ yes |
| Coverage | **every** line, every branch | only paths actually executed |
| Speed | milliseconds | as slow as the program |
| Knows runtime values? | ❌ no | ✅ yes |
| Downside | some false positives | misses untested paths |

CodeHound is **static**: it parses the source and reasons about structure. It never imports or
executes the code it scans — which is what makes it safe to point at any repository on the
internet.

## Why not just use existing linters?

You should — `ruff`, `flake8-bugbear` and `mypy` are excellent. CodeHound exists because:

1. It's a **learning artifact** — every rule maps to a bug that was actually merged upstream
2. It bundles a specific set of **async-safety** patterns that generic linters split across
   plugins or don't check at all
3. It's small enough to read end to end in an afternoon

> Next: [02_why_not_regex.md](02_why_not_regex.md)
