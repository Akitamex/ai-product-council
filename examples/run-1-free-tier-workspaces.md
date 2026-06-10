# Council run: team workspaces on the free tier

A complete, lightly edited transcript of a council run. The product is fictional: **Ledgerline**, a bootstrapped invoicing SaaS (solo founder, ~900 paying users, free tier capped at 5 invoices/month).

**Invocation:**

```
/product-council should we add team workspaces to the free tier? paid teams
start at $19/seat, support keeps getting "can my accountant see my invoices"
emails, maybe free workspaces grow word-of-mouth
```

---

## Mediator: scope + selection

**Scope: Medium.** New feature on a revenue-bearing boundary (free vs paid), touches auth (membership model) and a public pricing surface.

**Spawning:** council-product (worth/scope), council-engineering (membership model touches auth), council-risk (data sharing between accounts, pricing boundary), council-voice (pricing-page copy will change). **Skipping:** council-design (no new surface yet at decision stage; UI comes after the model is ruled).

---

## Persona reviews

### Product & Scope

**Verdict (worth building?):** yes-if. The support signal is real (access-sharing requests recur), but "workspaces" is the maximal reading of it. The minimal reading is read-only invite for one external viewer, which serves the accountant use case without building team infrastructure on a free plan.

**Smallest honest version:** one read-only guest per free account, full workspaces stay paid. Measures the demand (guest-invite acceptance rate) before any multi-seat build.

**Concerns:** free workspaces cannibalize the $19 seat: the accountant who joins free never becomes a seat. Word-of-mouth growth via free teams is a hypothesis with no number behind it; seat revenue is measured and pays the founder's rent.

**Asks (must-have to ship):** ship the guest-viewer version first; define the success metric before launch (guest invites sent per 100 free accounts); a 90-day checkpoint that either upgrades the bet or kills it.

**Non-blocking suggestions:** the support emails are themselves the funnel: reply with the guest-invite feature and track conversion from those threads.

### Engineering

**Feasibility:** sound-with-changes. The current auth model is single-owner per account; any sharing requires a membership table and permission checks on every invoice read. That work is identical whether one guest or full workspaces ship, but the blast radius differs.

**Real cost:** membership model + permission middleware (every read path), invite flow with token expiry, ~7 modules touched, one migration. Read-only guest avoids seat-billing changes; full workspaces add billing-proration logic, the single hairiest module in the codebase.

**Invariants at risk:** "an account's data is visible only to its owner" is assumed in three export paths and the email-notification job. Those paths need explicit guest-awareness or they leak invoices to guests after revocation.

**Asks (must-have to ship):** permission checks centralized in one module, not sprinkled per-route; revocation kills active sessions, not only future logins; the export and notification paths covered by tests with a revoked-guest case.

**Non-blocking suggestions:** the membership table should anticipate roles (`viewer` now, `editor` later) so paid workspaces reuse it.

### Risk

**Veto status:** none.

**Data flows created or changed:** invoices (client names, amounts, tax IDs) become visible to a second identity. Tax IDs are regulated data in several markets: the guest must see them only if the owner explicitly enables it.

**Irreversible steps + guards:** invite emails cannot be unsent: token expiry (≤72h) and single-use are mandatory. Revocation must be immediate (session-level, matching Engineering's ask).

**Asks (must-have to ship):** tax-ID field masked for guests by default; invite tokens single-use with expiry; an audit line on the account ("guest X viewed invoices") so the owner can see access; guest emails excluded from marketing lists.

**Non-blocking suggestions:** a one-page data note for the pricing page ("what guests can and cannot see") preempts the support tickets this feature will otherwise generate.

### Voice & Copy

**Register verdict:** consistent risk is low today, but the proposed framing drifts. "Team workspaces" promises Slack-grade collaboration; the actual feature under discussion is invoice viewing. Naming it "workspaces" on the free tier sets up the disappointment the support inbox will inherit.

**Claim honesty:** "invite your accountant" is the honest claim and matches the demand signal verbatim. Use the user's own words.

**Concerns:** pricing-page hierarchy: if the free tier gains a sharing feature, the $19 seat needs a visibly stronger story ("edit, send, and reconcile together") or the upgrade path blurs.

**Asks (must-have to ship):** call the free feature "guest access" or "invite your accountant", never "workspaces"; rewrite the seat tier's pricing copy in the same release.

**Suggested lines (where copy was missing):** invite email subject: "{owner} shared their Ledgerline invoices with you (view-only)".

---

## Mediator: synthesis

**Alignment:** all four personas independently landed on the same reduction: the demand signal is "let my accountant see", and a read-only guest serves it. Nobody defended free multi-seat workspaces as v1. Engineering and Risk converge on revocation-kills-sessions; Product and Voice converge on measuring before expanding.

**Conflicts:**

1. **Product vs the original ask.** The invocation proposed workspaces for word-of-mouth growth; Product argues the growth story has no number and the cannibalization story has one ($19/seat). This is a strategy ruling, not a fact dispute: it goes to the human.
2. **Engineering sequencing vs Product checkpoint.** Engineering wants the membership table built role-ready (extra day now, cheaper paid-workspaces later); Product wants the absolute minimum until the 90-day checkpoint passes. Trade-off: one day of speculative schema vs a possible rebuild.

**Veto status:** none. Risk's asks are binding but satisfiable.

**Open questions for the human:**

1. Accept the reduction from "free workspaces" to "free read-only guest"? (Product's case: serves the signal, protects the seat. The growth hypothesis stays testable later.)
2. Role-ready membership table now (Engineering) or minimal schema until the checkpoint (Product)?

---

## Human rulings (recorded verbatim)

> 1. yes, guest-only. workspaces stay paid, revisit after the checkpoint
> 2. role-ready table, one day is cheap insurance

## Final plan (excerpt from the decision record)

1. Membership table with `role` column (`viewer` only exposed), single guest per free account.
2. Centralized permission module; export + notification paths gain revoked-guest tests.
3. Invite: single-use token, 72h expiry; revocation kills sessions immediately.
4. Tax-ID masked for guests by default; per-account audit line.
5. Copy: "Invite your accountant" on free tier; seat tier rewritten ("edit, send, and reconcile together"); invite email per Voice's line.
6. Metric: guest invites per 100 free accounts; 90-day checkpoint 2026-09-15, expand-or-kill.

Persisted to `_decisions/2026-06-15-free-guest-access.md`.

---

*What to notice: the council did not average four opinions into mush. It reduced the feature to the demand signal, surfaced a strategy conflict the model had no right to settle, and turned "risky" into four checkable guards. Total wall-clock: about six minutes.*
