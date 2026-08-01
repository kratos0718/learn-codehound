# 3 · The parent problem — and codehound's solution

## Build the map yourself

Since `ast` gives you children, you can invert that into a parent lookup in one pass. This is
`build_parents()` in `core.py`:

```python
def build_parents(tree: ast.AST) -> dict:
    """Map id(child) -> parent_node for the whole tree."""
    parents: dict = {}
    for parent in ast.walk(tree):
        for child in ast.iter_child_nodes(parent):
            parents[id(child)] = parent
    return parents
```

**How it works:** walk every node; for each one, look at its direct children; record
"this child's parent is me."

**Why `id(child)` as the key?** AST nodes aren't hashable in a useful way, and two distinct
nodes can compare equal. `id()` gives the object's unique memory identity, so every node maps
to exactly one entry. It's built **once per file** and handed to every check — so the cost is
paid a single time, not per rule.

## Now upward questions become easy

```python
def enclosing_function(node, parents):
    """Nearest enclosing FunctionDef/AsyncFunctionDef, or None."""
    cur = node
    while cur is not None:
        p = parents.get(id(cur))
        if p is None:
            return None
        if isinstance(p, (ast.FunctionDef, ast.AsyncFunctionDef)):
            return p
        cur = p
```

Climb until you hit a function definition. Then CH001 is simply:

```python
fn = enclosing_function(call_node, parents)
if isinstance(fn, ast.AsyncFunctionDef):
    # blocking call inside async -> report it
```

## Stopping at the right boundary

`inside_with_statement()` shows a subtlety — it must **stop climbing** at a function, class or
module boundary:

```python
if isinstance(p, (ast.With, ast.AsyncWith)):
    return True
if isinstance(p, (ast.FunctionDef, ast.AsyncFunctionDef, ast.ClassDef, ast.Module)):
    return False
```

Without that, an `open()` in a plain function nested inside some outer `with` block would look
"protected" when it isn't. **Knowing where to stop is as important as knowing where to look.**

> Next: [../03_the_engine/01_finding_and_check.md](../03_the_engine/01_finding_and_check.md)
