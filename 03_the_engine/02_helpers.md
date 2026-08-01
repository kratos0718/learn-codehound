# 2 · The shared helpers

These four functions in `core.py` are where the actual cleverness lives. Every check is built
out of them.

## `enclosing_function(node, parents)`
Climbs to the nearest `FunctionDef` / `AsyncFunctionDef`. Returns `None` at module level.
→ **This is what makes CH001 possible.**

## `is_awaited(node, parents)`

```python
def is_awaited(node, parents) -> bool:
    parent = parents.get(id(node))
    return isinstance(parent, ast.Await)
```

Two lines, and it prevents a whole class of false positive:

```python
await client.get(url)     # ✅ an async client — must NOT be flagged
requests.get(url)         # ❌ genuinely blocking
```

A local variable can be named anything. If the call is the direct operand of an `await`, it
isn't blocking the loop, whatever it's called. **A false positive costs you a user's trust —
this guard is why the tool is usable.**

## `inside_with_statement(node, parents)`
Climbs looking for `With`/`AsyncWith`, **stopping** at a function/class/module boundary.
→ Used by CH005 to tell `with open(...)` from a bare `open(...)`.

## `attr_call_parts(node)`
Turns a call node into a `(prefix, attribute)` pair:

| Source | Returns |
|---|---|
| `time.sleep()` | `("time", "sleep")` |
| `urllib.request.urlopen()` | `("urllib.request", "urlopen")` |
| `sleep()` | `(None, "sleep")` |
| `x + 1` | `(None, None)` |

It handles dotted prefixes by walking down nested `Attribute` nodes and reversing the parts.
Everything the checks need to match a call reduces to comparing this tuple against a set.

> Next: [03_scanning_files.md](03_scanning_files.md)
