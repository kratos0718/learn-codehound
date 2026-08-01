# CH006 — Floating (fire-and-forget) task

> *"Result of create_task()/ensure_future() discarded; task may be GC'd before completion."*

**The subtlest bug in the tool — and the one that impresses people.**

## The bug

```python
async def handler():
    asyncio.create_task(send_analytics())     # ❌ result thrown away
```

Two separate failures:

**1. Exceptions vanish.** If `send_analytics()` raises, nothing is awaiting the result, so the
exception is never retrieved. Best case a "Task exception was never retrieved" warning; worst
case, silent data loss.

**2. ⚠️ The task can be garbage-collected mid-run.** The event loop keeps only a **weak
reference** to tasks. Discard the returned handle and there may be no strong reference anywhere
— so the garbage collector is free to destroy the task *while it's still running*.

The result is a task that works fine in testing and randomly stops under load. This is
documented behaviour in the Python docs, and almost nobody knows it.

```python
_background_tasks = set()

def spawn(coro):
    task = asyncio.create_task(coro)
    _background_tasks.add(task)                        # strong reference
    task.add_done_callback(_background_tasks.discard)  # clean up when done
    return task
```

## What it looks for

Calls to `asyncio.create_task()` / `asyncio.ensure_future()` / `loop.create_task()` whose
return value is discarded — i.e. the call is a bare `ast.Expr` statement rather than being
assigned, awaited, or passed onward.

## Where it was found

**agno** (trace exporter), **vLLM**, **Microsoft autogen**, **litellm**, **Future AGI**.

## 🎤 Explaining it

> "`create_task` schedules a coroutine but returns a handle the caller usually discards. Two
> things go wrong: exceptions are never surfaced because nothing awaits the result, and since
> the loop keeps only a weak reference, the task can be garbage-collected mid-flight. The fix
> is a strong reference in a module-level set plus a done-callback for cleanup and error
> logging."

> Next: [../05_build_your_own/01_write_a_check.md](../05_build_your_own/01_write_a_check.md)
