<img src="https://yurtayev.com/icon.svg" width="36" alt="">

# AI Product Council

A multi-agent review workflow for product decisions, runnable with Claude Code. Five persona agents debate your decision in parallel, a mediator synthesizes where they agree and where they collide, and you rule on the conflicts before anything gets built.

I'm a Head of Product. This is the distilled version of the workflow I run before every non-trivial change on my own projects: one model giving one opinion is a yes-man, five personas with different stakes is a review.

## Why a council instead of one prompt

Ask an LLM "review my plan" and you get agreeable mush: a few compliments, a few generic risks, approval. The structure fixes the incentive. Each persona gets a narrow mandate and a reason to object (the product sentinel kills scope, the risk steward hunts leaks, the engineer defends invariants). The mediator's job is to surface their conflicts, never to average them. Disagreement is the product.

## What's in the box

- **[template/agents/](template/agents/)**: five persona definitions: `council-product` (scope and worth), `council-engineering` (feasibility and invariants), `council-design` (craft and coherence), `council-risk` (security, privacy, irreversibility: holds a veto), `council-voice` (copy and naming, when words ship to users).
- **[template/product-council.md](template/product-council.md)**: the mediator command. Classifies scope, picks the relevant persona subset, spawns them in parallel, synthesizes, asks you the unresolved questions, writes a decision record.
- **[template/claude-md-snippet.md](template/claude-md-snippet.md)**: three lines for your project's CLAUDE.md so the council triggers on non-trivial changes.
- **[examples/](examples/)**: two complete council runs on fictional products, including a conflict the mediator refused to resolve on its own and a veto. Read these first: the transcripts show why the structure earns its overhead.

## Quick start

```bash
git clone https://github.com/Akitamex/ai-product-council
cd your-project
mkdir -p .claude/agents .claude/commands
cp ../ai-product-council/template/agents/*.md .claude/agents/
cp ../ai-product-council/template/product-council.md .claude/commands/
```

Then, in Claude Code, from your project:

```
/product-council should we add team workspaces to the free tier?
```

The mediator reads your project context, spawns the personas with a stake in the question, and returns a synthesis with the conflicts named. Decision records land in `_decisions/` so the "why" survives the chat session.

## Rules that make it work

1. **Scope gates ceremony.** Typo-level changes skip the council. The command classifies scope first and tells you when a council is overkill.
2. **Personas are read-only.** They investigate and argue; they never edit files. Deliberation and implementation stay separate.
3. **Conflicts reach the human.** When two personas genuinely collide, the mediator presents the trade-off and waits. A council that auto-resolves its own disagreements is one prompt with extra steps.
4. **The risk persona can veto.** A veto blocks approval until you either fix the plan or explicitly override it on the record.
5. **Decision records persist.** Every run writes a dated markdown doc: the next session, or the next person, inherits the reasoning instead of re-litigating it.

## When NOT to convene

Taste calls (colors, naming you already like), reversible one-file tweaks, anything you've already decided. The council sharpens decisions you haven't made; it is theater for decisions you have.

## License

[CC BY 4.0](LICENSE). Use it, adapt the personas to your domain, keep the attribution.

---

Built by [Nikita Yurtayev](https://yurtayev.com) · [LinkedIn](https://linkedin.com/in/yurtayev) · sibling playbook: [product-metric-breakdown](https://github.com/Akitamex/product-metric-breakdown) · status at [yurtayev.com/now](https://yurtayev.com/now).
