# CLAUDE.md: ai-product-council

Public workflow repo by Nikita Yurtayev (Akitamex). Audience: product managers and solo builders running Claude Code.

## Hard rules (project isolation)

- This repo is PUBLIC. Never mention the owner's employer, its industry, product names, metrics, or colleagues. Example products in `examples/` are fictional; keep them fictional.
- No real user data. No `.env` files. gitleaks runs pre-commit.
- Prose: no em-dashes (use , . :). Active voice. No invented adoption numbers or testimonials.

## Structure

- `README.md`: what a council is, when to convene one, quick start.
- `template/agents/*.md`: five persona definitions, drop into your project's `.claude/agents/`.
- `template/product-council.md`: the mediator command, drop into `.claude/commands/`.
- `template/claude-md-snippet.md`: lines to add to your project's CLAUDE.md.
- `examples/`: two full council runs on fictional products, transcript-style.

## Conventions

License CC BY 4.0. Persona files follow Claude Code agent frontmatter (`name`, `description`, `tools`). Keep personas read-only (no Edit/Write in their tools line): councils deliberate, implementers implement.
