# CH003 — Deprecated `datetime.utcnow()`

> *"datetime.utcnow()/utcfromtimestamp() are deprecated and return naive datetimes."*

## The bug

```python
datetime.utcnow()                    # ❌ naive — no timezone attached
datetime.now(timezone.utc)           # ✅ timezone-aware
```

`utcnow()` returns a datetime holding UTC time but with **no `tzinfo`**. Python has no way to
know it's UTC. So:

```python
naive = datetime.utcnow()
aware = datetime.now(timezone.utc)
naive - aware        # TypeError: can't subtract offset-naive and offset-aware
```

Or worse — no error, just arithmetic that's silently wrong by your timezone offset. A bug that
only manifests for users in other countries is the expensive kind.

Deprecated in **Python 3.12**.

## What it looks for

Calls matching `("datetime", "utcnow")` and `("datetime", "utcfromtimestamp")`.

## Where it was merged

**crewAI** — replaced across the memory subsystem. Note the fix used
`datetime.now(timezone.utc).replace(tzinfo=None)` rather than a bare
`datetime.now(timezone.utc)`, because the stored values were compared against other *naive*
datetimes. **Changing behaviour while fixing a deprecation is how you break someone's code** —
the correct fix preserved the existing semantics exactly.

> Next: [CH004_get_event_loop.md](CH004_get_event_loop.md)
