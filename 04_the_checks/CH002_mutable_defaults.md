# CH002 — Mutable default argument

> *"Mutable default argument is shared across all calls to the function."*

## The bug

```python
def add_item(item, basket=[]):
    basket.append(item)
    return basket

add_item("apple")     # ['apple']
add_item("banana")    # ['apple', 'banana']   😱
```

## Why it happens

**Default arguments are evaluated once — when the `def` statement runs — not on each call.**
So that single list object is created at definition time and reused by every call that doesn't
pass its own. State leaks between completely unrelated callers.

Same trap applies to `{}`, `set()`, and any function call used as a default.

```python
def add_item(item, basket=None):     # ✅
    if basket is None:
        basket = []
    basket.append(item)
    return basket
```

## What it looks for

Function definitions whose `args.defaults` or `args.kw_defaults` contain a mutable literal —
`ast.List`, `ast.Dict`, `ast.Set` — or a call to a mutable constructor.

This one is a **downward** check: the information is inside the function's own `arguments` node,
so it doesn't need the parent map at all.

## Where it was merged

**mem0** (`Completions.create`, `BaseEmbedderConfig`) and **pydantic-ai** (a shared `deque` in
`process_tool_calls` leaking one run's state into the next).

`B006` is the flake8-bugbear code for the same rule — worth knowing, because interviewers who
use bugbear will recognise the number.

## 🎤 Explaining it

> "Python evaluates default arguments once at function-definition time, so a mutable default is
> shared across all calls. Mutations persist between calls and leak state — a classic
> heisenbug. The fix is defaulting to `None` and constructing inside the body."

> Next: [CH003_datetime_utcnow.md](CH003_datetime_utcnow.md)
