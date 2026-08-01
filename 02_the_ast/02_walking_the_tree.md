# 2 · Walking the tree

## `ast.walk` — visit every node

```python
import ast

tree = ast.parse(open("some_file.py").read())

for node in ast.walk(tree):
    if isinstance(node, ast.Call):
        print("a function call on line", node.lineno)
```

`ast.walk` yields **every node in the tree**, in no particular order. It's the workhorse — every
codehound check starts with `for node in ast.walk(tree)`.

## Finding a specific call

To match `time.sleep(...)` you need two levels:

```python
for node in ast.walk(tree):
    if not isinstance(node, ast.Call):
        continue
    func = node.func                       # what is being called
    if isinstance(func, ast.Attribute):    # it's  something.something()
        if isinstance(func.value, ast.Name):
            module = func.value.id         # "time"
            attr   = func.attr             # "sleep"
            if (module, attr) == ("time", "sleep"):
                print("found it, line", node.lineno)
```

For `time.sleep(1)`:
- `node` is the `Call`
- `node.func` is an `Attribute`
- `node.func.value` is `Name(id='time')`
- `node.func.attr` is `'sleep'`

CodeHound wraps exactly this in a helper called `attr_call_parts()`.

## ⚠️ The problem: nodes don't know their parents

This is the key limitation you hit immediately.

```python
node.lineno     # ✅ works
node.parent     # ❌ AttributeError — does not exist
```

Python's `ast` records **children, never parents**. But every interesting question codehound
asks is an *upward* question:

- "Is this call inside an `async def`?"
- "Is this `open()` inside a `with`?"
- "Is this call the operand of an `await`?"

All of those require walking **up**, and there is no `.parent` to walk up with.

The next file solves it.

> Next: [03_the_parent_problem.md](03_the_parent_problem.md)
