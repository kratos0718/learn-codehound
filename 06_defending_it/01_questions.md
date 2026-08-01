# Defending codehound — the questions people actually ask

## "What is an AST, and why not regex?"
> "Regex sees characters; the AST sees structure — and these bugs are structural. To flag
> `time.sleep` I need to know it's a real call and that its enclosing function is an
> `async def`. Regex can't express containment or scope, and it would match the same text in a
> comment, a docstring or a variable name. The AST is the same tree Python itself uses, so the
> check is precise."

## "Why zero dependencies?"
> "`ast` is in the standard library, so it installs and runs anywhere with no version conflicts.
> That matters for a linter because it runs in *other people's* environments and CI. Adding
> dependencies to a static-analysis tool creates exactly the friction that stops people using
> it."

## "How do you avoid false positives?"
> "Mostly by asking structural questions rather than textual ones, plus explicit guards. The
> clearest example is `is_awaited()` — `await client.get(url)` looks like a blocking HTTP call
> by name, but if the call is the direct operand of an `await` it isn't blocking the loop, so
> it's skipped. I also skip `tests/`, `examples/` and vendored code, which deliberately contain
> the patterns I'm looking for."

## "Static vs dynamic analysis?"
> "Static inspects code without running it — fast, safe, covers every path, but it can't know
> runtime values so it produces some false positives. Dynamic observes real execution — precise
> about what actually happened, but only for the paths exercised."

## "How did a finding become a merged PR?"
> "Take the Weaviate one. codehound flagged `time.sleep(1)` inside an `async def
> wait_for_weaviate` retry loop. I confirmed it against the live repository, changed it to
> `await asyncio.sleep(1)`, and explained in the PR *why* it mattered — asyncio is
> single-threaded and cooperative, so that call stalls every concurrent operation in the user's
> application, not just this one. A maintainer reviewed and merged it."

## "Isn't this just reimplementing ruff?"
> "Partly, and I'd use ruff in production. codehound exists because every rule in it came from
> a bug I actually found and got merged upstream — it's the distillation of six real bug
> classes rather than a general-purpose linter. It's also small enough to read end to end,
> which was the point."

## "What would you add next?"
> "Multi-file awareness. Right now every check sees one file at a time, so it can't follow a
> helper that wraps `time.sleep` in another module. Cross-file call-graph analysis would catch
> indirect blocking calls — and it's the main reason a real tool like ruff is harder to build
> than it looks."

---

## The five facts to have ready
1. **~754 lines**, zero dependencies, stdlib `ast` only
2. **Six checks**, CH001–CH006, each from a real merged bug
3. **`build_parents`** exists because Python's AST records children, not parents
4. **`is_awaited`** is the guard that keeps CH001 usable
5. Findings merged into **unsloth, agno, mem0, pydantic-ai, Weaviate, xorbitsai** and more
