# Learn CodeHound — from zero to explaining every line

This repo teaches **[codehound](https://github.com/kratos0718/codehound)** from the ground up:
what problem it solves, how Python's AST works, how the engine is built, and how each of the
six checks decides that a piece of code is buggy.

It assumes **no prior knowledge** of static analysis or the `ast` module.

CodeHound is ~**754 lines of Python** with **zero dependencies**. Every rule in it came from a
real bug that was found and merged into a widely-used open-source project — so this is also a
tour of six bug classes that show up constantly in production Python.

---

## Read in this order

| # | Folder | What you'll understand |
|---|---|---|
| 1 | [01_the_problem](01_the_problem/) | Why a linter, and why the AST instead of regex |
| 2 | [02_the_ast](02_the_ast/) | Python's AST from scratch — nodes, walking, parents |
| 3 | [03_the_engine](03_the_engine/) | `Finding`, the `Check` contract, the helpers, file discovery |
| 4 | [04_the_checks](04_the_checks/) | All six rules, CH001–CH006, with the real bug behind each |
| 5 | [05_build_your_own](05_build_your_own/) | Write and test a new check yourself |
| 6 | [06_defending_it](06_defending_it/) | The questions people actually ask about it |

## Run it while you read

```bash
git clone https://github.com/kratos0718/codehound
cd codehound
PYTHONPATH=src python3 -m codehound.cli scan <path-to-any-python-project>
PYTHONPATH=src python3 -m codehound.cli scan . --select CH001
```

## The six checks at a glance

| Code | Name | Catches |
|---|---|---|
| **CH001** | `blocking-call-in-async` | `time.sleep` / `requests.get` inside `async def` — freezes the event loop |
| **CH002** | `mutable-default-argument` | `def f(x=[])` — the default is shared across all calls |
| **CH003** | `deprecated-datetime-utcnow` | `datetime.utcnow()` — returns a naive datetime, deprecated in 3.12 |
| **CH004** | `deprecated-get-event-loop` | `asyncio.get_event_loop()` outside a running loop |
| **CH005** | `unclosed-file-handle` | `open()` with no `with` and no matching `close()` |
| **CH006** | `floating-task` | `create_task()` result discarded — task can be garbage-collected mid-run |
