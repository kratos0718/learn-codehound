# 1 · Write your own check

The best way to prove you understand codehound is to add a rule. Here's a complete one.

## Goal: CH007 — bare `except:`

```python
try:
    risky()
except:            # ❌ swallows KeyboardInterrupt and SystemExit too
    pass

except Exception:  # ✅
    ...
```

A bare `except:` catches **everything**, including `KeyboardInterrupt` and `SystemExit` — so
Ctrl-C stops working and your program becomes unkillable in a loop.

## The AST shape

```python
import ast
print(ast.dump(ast.parse("try:\n  f()\nexcept:\n  pass"), indent=2))
```

The handler is an `ast.ExceptHandler`, and for a bare except its **`type` attribute is `None`**.
That's the entire test.

## The implementation

`src/codehound/checks/bare_except.py`:

```python
"""CH007 - Bare except swallows KeyboardInterrupt and SystemExit."""

from __future__ import annotations

import ast

from codehound.core import Check, Finding


class BareExcept(Check):
    code = "CH007"
    name = "bare-except"
    description = "Bare 'except:' also catches KeyboardInterrupt and SystemExit."

    def run(self, tree: ast.AST, parents: dict, path: str) -> list[Finding]:
        findings: list[Finding] = []
        for node in ast.walk(tree):
            if isinstance(node, ast.ExceptHandler) and node.type is None:
                findings.append(
                    Finding(
                        path=path,
                        line=node.lineno,
                        col=node.col_offset,
                        code=self.code,
                        message="bare 'except:' — catch Exception instead",
                    )
                )
        return findings
```

Then register it in `checks/__init__.py` alongside the others.

Notice it ignores `parents` entirely — this question is answerable without climbing the tree.
Not every rule needs the parent map; CH001 and CH005 do, this one doesn't.

> Next: [02_test_it.md](02_test_it.md)
