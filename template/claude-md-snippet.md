# Add to your project's CLAUDE.md

Append this block so the council convenes by default on non-trivial work:

```markdown
## Decision review

Before implementing any Medium or Large change (new feature, new route, new
dependency, public-surface change, anything touching auth/data/money), run
`/product-council <decision>` and wait for the human's rulings on its open
questions. Decision records live in `_decisions/`; read the relevant ones
before re-opening a settled question.
```

Two notes:

- Add `_decisions/` to the repo, committed: the records are documentation, not scratch.
- If your project has invariants the personas must defend (a palette, an isolation rule, a data boundary), state them in CLAUDE.md explicitly. The council is exactly as sharp as the invariants it can read.
