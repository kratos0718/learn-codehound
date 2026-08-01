# 1 · The two core abstractions

The whole engine rests on two small pieces: what a *result* looks like, and what a *rule* looks
like. Get these right and adding rules becomes trivial.

## `Finding` — one result

```python
@dataclass(frozen=True)
class Finding:
    """A single rule violation at a specific source location."""
    path: str
    line: int
    col: int
    code: str
    message: str

    def as_text(self) -> str:
        return f"{self.path}:{self.line}:{self.col}: {self.code} {self.message}"

    def as_dict(self) -> dict: ...
```

**Why `frozen=True`?** Findings are immutable — once a rule reports something, nothing
downstream can quietly alter it. Cheap guarantee, one keyword.

**Why both `as_text` and `as_dict`?** `as_text` produces the classic
`path:line:col: CODE message` format every editor and CI system already knows how to parse.
`as_dict` feeds `--format json` for tooling. The rules don't know or care which one is used.

## `Check` — one rule

```python
class Check:
    """Base class for a single static-analysis rule."""
    code: str = ""
    name: str = ""
    description: str = ""

    def run(self, tree: ast.AST, parents: dict, path: str) -> list[Finding]:
        raise NotImplementedError
```

That's the entire contract. **Every rule receives the same three things and returns a list.**

| Argument | Why it's passed in |
|---|---|
| `tree` | the parsed AST — the rule never opens files itself |
| `parents` | the precomputed parent map, built **once per file** and shared |
| `path` | so findings can name the file |

## Why this design matters

Because `run()` is a pure function of its inputs, each check is:

- **independently testable** — hand it a parsed snippet, assert on the findings. No filesystem,
  no subprocess, no fixtures
- **cheap to add** — a new rule is one class in one file
- **isolated** — a buggy rule can't corrupt another rule's results

The `parents` map being passed *in* rather than built inside each rule is the one real
performance decision in the codebase: six checks on one file share a single map instead of
building six identical ones.

> Next: [02_helpers.md](02_helpers.md)
