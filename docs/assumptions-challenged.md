# Assumptions Challenged — eatme

**Date:** 2026-05-05
**Reviewer:** assumption-challenger agent

## Strongest concerns (could derail the project)

### 1. Synthesis quality is the entire value proposition — and unproven
LLMs hallucinate quantities. Averaging across recipes is *mathematically wrong* —
recipes are coupled systems (hydration, salt:flour, fat:flour), not independent
parameters. A "mean" recipe can be worse than any source. An experienced cook
already trusts Ottolenghi; what does synthesis add over importing Ottolenghi
verbatim and normalising units?

**Test before building:** run 5 already-mastered recipes through synthesis
manually in Claude. If user edits >30% of output, the skill is a manual editor
with extra steps.

### 2. The skill abstraction may be the wrong tool for production work
Skills are designed for ad-hoc reasoning. This use case needs idempotency,
retries, logging, schema validation, partial-failure handling, rate limiting —
none of which a skill handles well. A 200-line Python CLI with the Anthropic
SDK has logs, retries, JSON schema validation via Pydantic, pytest for prompt
regressions.

**Counter-info from user:** Claude skills now run on Claude.ai web. This
materially improves wife-accessibility (any device, any browser) and weakens
the "skill is wrong abstraction" argument for *this* use case. Logging /
retries / schema validation can still be embedded in the skill itself via the
prompt.

### 3. Bilingual parity is aspiration, not requirement
Mealie has no native bilingual field. All architectures are bad: 2 records
duplicates and drifts; FR-primary + EN-in-notes makes one user second-class;
Cooklang-as-source-of-truth defeats "Mealie as-is."

**Honest path:** one language per recipe (origin language wins), add an
on-demand "translate this recipe" action for rare cases. Bilingual-by-default
is over-engineering for two readers who almost certainly understand both.

## Weak concerns (probably fine)

- **YouTube fallback ladder** — 90/8/2 split with yt-dlp + whisper is reasonable
- **Backup** — Mealie SQLite + data dir → Hyper Backup nightly to B2 or Infomaniak Swiss Backup. One job, done.
- **Wife as consumer** — fine if she's not the curator type. If she finds a recipe, she sends URL; user runs skill. That's collaboration, not friction.
- **Mealie API changes** — pin the image SHA, low risk.

## Reframed alternatives

1. **Minimum-viable validation (recommended start):** prompt template in a
   markdown file + Claude.ai web + Mealie "Create from JSON" UI. Zero code.
   Test the synthesis premise on 5–10 recipes for two weeks before committing.
2. **If tooling is justified:** Python CLI (or web skill — see user note above)
   with structured `{quantity, unit, food, note}` output → keeps Mealie's
   shopping-list / meal-planning features rather than bypassing them.
3. **Drop bilingual parity** — one language per recipe + on-demand translation.

## Recommendation

**Step back. Do not build the skill yet.**

Two-week experiment:
- Write the synthesis prompt
- Run 5–10 recipes manually via Claude.ai web
- Paste output into Mealie's JSON import
- Cook 3 of them

Then decide based on evidence:

| Outcome | Path forward |
|---|---|
| Synthesis quality is good, manual loop annoys | Build the skill (web-accessible) with structured ingredients, drop bilingual default |
| Synthesis quality is mediocre | Abandon synthesis. Use LLM only for translation + unit normalisation on a single trusted source |
| Manual loop is fine | Ship nothing. The pain was tedium, not capability |

**Biggest risk: committing to architecture before validating the core premise.**
