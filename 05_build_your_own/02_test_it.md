# 2 · Test it

Because `run()` is a pure function of `(tree, parents, path)`, testing needs no filesystem, no
fixtures and no subprocess. Parse a string, run the check, assert.

```python
import ast
from codehound.core import build_parents
from codehound.checks.bare_except import BareExcept


def run_check(src: str):
    tree = ast.parse(src)
    return BareExcept().run(tree, build_parents(tree), "test.py")


def test_flags_bare_except():
    findings = run_check("try:\n    f()\nexcept:\n    pass\n")
    assert len(findings) == 1
    assert findings[0].code == "CH007"
    assert findings[0].line == 3


def test_allows_specific_exception():
    findings = run_check("try:\n    f()\nexcept ValueError:\n    pass\n")
    assert findings == []
```

## Always write both tests

A check that reports the bug is only half the job. **The negative test — proving it stays quiet
on correct code — is what makes it trustworthy.** A rule with false positives gets disabled,
and a disabled rule catches nothing.

This is the same discipline as a regression test on a pull request: it must **fail before the
fix and pass after**. Here it's *fire on the bug, silent on the good code.*

## Run it

```bash
python -m pytest tests/ -q
PYTHONPATH=src python3 -m codehound.cli scan . --select CH007
```

> Next: [../06_defending_it/01_questions.md](../06_defending_it/01_questions.md)
