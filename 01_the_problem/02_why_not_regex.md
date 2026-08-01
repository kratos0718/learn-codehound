# 2 · Why not just search the text?

The obvious first idea is to grep for the bad thing:

```bash
grep -rn "time.sleep" .
```

## Everything that breaks

**1. It can't see scope.** The rule isn't "`time.sleep` is bad" — it's "`time.sleep` **inside an
`async def`** is bad". In a normal function it's completely correct. Text search has no idea
which function a line belongs to.

```python
def cleanup():
    time.sleep(1)          # ✅ fine, not async

async def poll():
    time.sleep(1)          # ❌ bug
```

Both lines are byte-identical. Only their *position in the tree* differs.

**2. It matches things that aren't code.**

```python
# don't use time.sleep here          ← a comment
"""Avoid time.sleep in async code""" ← a docstring
msg = "call time.sleep to wait"      ← a string
```

Three false positives, none of them code.

**3. It misses real calls that don't match the pattern.**

```python
from time import sleep
sleep(1)                    # grep for "time.sleep" finds nothing
```

**4. It can't tell whether the call is awaited.**

```python
await client.get(url)       # ✅ async client, fine
requests.get(url)           # ❌ blocking
```

## What the AST gives you instead

Parsing turns the source into a **tree that Python itself understands**. Now you can ask
structural questions:

- *Is this node a function call?*
- *Is the thing being called an attribute access of the form `time.sleep`?*
- *Walking up the tree, is the nearest enclosing function an `AsyncFunctionDef`?*
- *Is the direct parent of this call an `Await`?*

Comments and strings simply **aren't in the tree**, so they can't produce false positives.
That's the whole argument for the AST in one sentence:

> **Regex sees characters. The AST sees structure — and the bug is structural.**

> Next: [../02_the_ast/01_what_is_an_ast.md](../02_the_ast/01_what_is_an_ast.md)
