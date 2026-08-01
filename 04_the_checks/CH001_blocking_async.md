# CH001 — Blocking call inside an async function

> *"Synchronous blocking call inside an async function freezes the event loop."*

**This is the flagship check.** Most of codehound's merged upstream fixes came from it.

## The bug

```python
async def wait_for_service():
    while not ready():
        time.sleep(1)          # ❌ freezes EVERYTHING
```

The asyncio event loop is **single-threaded and cooperative**. Tasks only yield control at an
`await`. `time.sleep(1)` never yields — so for that whole second, *every* other coroutine in
the process is frozen. Not just this function. All of them.

```python
await asyncio.sleep(1)         # ✅ yields; other tasks run meanwhile
```

## Real bug this came from

`agno`'s Couchbase vector store had `time.sleep(1)` inside
`_async_create_collection_and_scope`. Same pattern later found and merged in **unsloth**,
**Weaviate** and **xorbitsai/inference**.

## What it looks for

```python
_BLOCKING_CALLS = {
    ("time", "sleep"),
    ("requests", "get"), ("requests", "post"), ("requests", "put"),
    ("requests", "delete"), ("requests", "patch"), ("requests", "head"),
    ("requests", "request"),
    ("subprocess", "run"), ("subprocess", "call"),
    ("subprocess", "check_call"), ("subprocess", "check_output"),
    ("os", "system"),
    ("urllib.request", "urlopen"),
}
```

## The logic

1. Walk the tree, keep only `ast.Call` nodes
2. `attr_call_parts()` → is it `(module, attr)` in `_BLOCKING_CALLS`?
3. `enclosing_function()` → is the nearest function an **`AsyncFunctionDef`**?
4. `is_awaited()` → **skip if it's awaited** (it's an async client, not blocking)
5. All conditions met → emit a `Finding`

Step 4 is what keeps the signal-to-noise ratio high.

## 🎤 Explaining it in an interview

> "The asyncio event loop is single-threaded and cooperative — tasks only yield at an `await`.
> A synchronous call like `time.sleep` never yields, so it stalls the entire loop, not just that
> coroutine. Every other pending task is starved for the duration. The fix is the async
> equivalent, or pushing the work to a thread pool with `run_in_executor`."

> Next: [CH002_mutable_defaults.md](CH002_mutable_defaults.md)
