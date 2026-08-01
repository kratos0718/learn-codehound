# 1 · What is an AST?

**AST = Abstract Syntax Tree.** It's the structure Python builds from your source code before
running it.

When you read *"the cat sat on the mat"*, you don't just see letters — you know "the cat" is the
subject and "sat" is the verb. An AST is that, for code.

## See it yourself

```python
import ast
print(ast.dump(ast.parse("x = 1 + 2"), indent=2))
```

```
Module(
  body=[
    Assign(
      targets=[Name(id='x', ctx=Store())],
      value=BinOp(
        left=Constant(value=1),
        op=Add(),
        right=Constant(value=2)))])
```

Read it as a tree:

```
Module
└── Assign
    ├── target: Name(x)
    └── value: BinOp
                ├── left:  Constant(1)
                ├── op:    Add
                └── right: Constant(2)
```

Every construct becomes a **node**. Nodes contain other nodes. That's the whole idea.

## The node types codehound cares about

| Node | Source it represents |
|---|---|
| `ast.Module` | the whole file |
| `ast.FunctionDef` | `def foo():` |
| `ast.AsyncFunctionDef` | `async def foo():` ← **different node type!** |
| `ast.Call` | `foo()` |
| `ast.Attribute` | `time.sleep` (the `.sleep` part) |
| `ast.Name` | a bare identifier like `time` or `x` |
| `ast.Await` | `await something()` |
| `ast.With` / `ast.AsyncWith` | `with open(...)` |
| `ast.arguments` | a function's parameter list, including defaults |

**The single most important fact for codehound:** `def` and `async def` are *different node
types*. That one distinction is what makes CH001 possible at all — the entire "is this blocking
call inside async code?" question reduces to "is the enclosing node an `AsyncFunctionDef`?"

## Every node knows where it came from

```python
node.lineno      # line number
node.col_offset  # column
```

That's how a finding can say `app.py:42:8` and point you at the exact character.

> Next: [02_walking_the_tree.md](02_walking_the_tree.md)
