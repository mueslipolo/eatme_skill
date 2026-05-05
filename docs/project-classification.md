# Project Classification — eatme

**Date:** 2026-05-05
**Working name:** eatme

## Vision

A self-hosted, bilingual (FR/EN) recipe knowledge base for a two-person household.
Differentiator vs Mealie/Tandoor: the **import pipeline is a research task, not a
parsing task** — ingest from URL/YouTube/recipe-name, cross-reference reputable
chefs (Ottolenghi, etc.), present synthesis with options, user picks, output is a
clean coherent recipe in two languages with metric-first units.

## Classification

| Axis | Value |
|---|---|
| Scale | Solo / household (2 users) |
| Type | Web app + ingestion pipeline + LLM-orchestrated workflow |
| Novelty | Partial solutions exist (storage layer); AI synthesis layer is novel |
| Clarity | Mostly defined |
| User type | Consumer / household — must be wife-friendly |

## Hard constraints

- **Hosting:** Synology NAS, container runtime (Podman preferred, Docker likely fallback)
- **No GPU.** Limited CPU/RAM (Synology-class).
- **Data sovereignty:** on-premise first (matches user's FOSS/Swiss preferences).
- **Library size:** 30–40 recipes initially, growing to a few hundred.
- **Languages:** French + English, parity required.
- **UX target:** tablet/laptop one-page view; wife-usable (no rough edges).

## Implications of constraints

- Local LLM for synthesis is **not viable** on this hardware. Synthesis must use
  an external API (Claude / Mistral / etc.) called sparingly on import only.
- Local LLM may still be used for cheap tasks: structure parsing, embeddings,
  Whisper transcription of YouTube audio.
- SQLite + FTS5 is sufficient for the projected library size. No Postgres needed
  in v1. `sqlite-vec` can be added later for semantic search.
- API costs are bounded: synthesis runs only on import (~30–40 recipes initially,
  then a few per month). Order of magnitude: pennies per recipe.

## Open questions for Phase 2 research

1. Does Mealie's pain (manual ingredient validation) have a known fix in the
   ecosystem (Tandoor, KitchenOwl, recipe-scrapers improvements)?
2. Is Cooklang a fit for the canonical recipe format?
3. What's the state of `recipe-scrapers` (Python) for the major sites?
4. Which YouTube transcription stack works on CPU-only Synology?
5. Has anyone built the "AI synthesis with chef cross-referencing" layer already?
6. Swiss/EU LLM API options for data sovereignty (Mistral via Infomaniak, etc.).
