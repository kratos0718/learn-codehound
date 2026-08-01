# CH005 — Unclosed file handle

> *"open() result stored without a context manager or matching close()."*

## The bug

```python
f = open("data.txt")      # ❌ if read() raises, close() never runs
data = f.read()
f.close()

with open("data.txt") as f:   # ✅ closed even on exception
    data = f.read()
```

## Why it's expensive

The OS gives each process a **limited number of file descriptors**. Leaked ones are never
returned. The symptom appears hours or days into uptime:

```
OSError: [Errno 24] Too many open files
```

By then the traceback points at whatever unlucky code asked for the *next* handle — not at the
leak. That distance between cause and symptom is exactly why static analysis pays off here.

## What it looks for

An `open()` call whose result is assigned to a variable, where:
- `inside_with_statement()` is **False** (no context manager protecting it), and
- there's no matching `.close()` in the same scope

Both conditions are needed. `with open(...)` is fine; so is a manual `open`/`close` pair.
Only the unprotected case is reported.

## Where it was merged

**agno** — a file handle left open in `OpenAITools.transcribe_audio`.

## 🎤 Explaining it

> "A context manager guarantees cleanup through `__enter__`/`__exit__`, even when an exception
> is raised. Without it, an exception between `open` and `close` leaks the descriptor, and the
> process eventually hits the OS limit."

> Next: [CH006_floating_task.md](CH006_floating_task.md)
