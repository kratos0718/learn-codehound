# CH004 — Deprecated `asyncio.get_event_loop()`

> *"asyncio.get_event_loop() is deprecated outside a running loop."*

## The bug

`get_event_loop()` behaves differently depending on context and Python version — sometimes
returning a loop, sometimes creating one, sometimes emitting a `DeprecationWarning`. That
ambiguity is the problem.

```python
loop = asyncio.get_event_loop()          # ❌ behaviour depends on context
loop.run_until_complete(main())

asyncio.run(main())                      # ✅ modern entry point
loop = asyncio.get_running_loop()        # ✅ inside async code
```

## The key property of `get_running_loop()`

It **raises `RuntimeError` unless called from the thread currently running the loop.** That
turns a vague situation into an explicit one you can handle:

```python
try:
    loop = asyncio.get_running_loop()
except RuntimeError:
    asyncio.run(coro())          # no loop -> run one
    return
loop.create_task(coro())         # loop exists, and we're on its thread
```

That guarantee has a useful consequence: reaching the line *after* `get_running_loop()` proves
you're on the loop thread — so mutating a shared set there needs no lock. (This exact argument
settled a reviewer's thread-safety question on the **agno** PR.)

## Where it was merged

**crewAI** and **agno**.

> Next: [CH005_resource_leak.md](CH005_resource_leak.md)
