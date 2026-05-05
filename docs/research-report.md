# Research Report — eatme

**Date:** 2026-05-05
**Researcher:** research-synthesizer agent
**Confidence:** HIGH (ecosystem landscape, costs) / MEDIUM (Synology CPU inference)

## Headline finding

**No existing FOSS tool solves the user's core requirement** (multi-source
synthesis on import with chef cross-referencing and canonical bilingual
normalisation). The closest commercial product (Samsung Food / Whisk) does
ingredient personalisation, not synthesis. This is a genuine gap.

The strategic question is therefore not *which app does this* but *which
existing app to use as the storage/UI layer while building the synthesis
pipeline on top*.

## Top three FOSS options as base layer

| Solution | Strengths | Weaknesses for our need |
|---|---|---|
| **Mealie** v3.16 (FastAPI/Vue, AGPL-3.0) | Best tablet UX & cook mode, complete REST API, ~9K stars, ~30 langs incl. FR | Manual ingredient validation step on import (the documented pain point) |
| **Tandoor** v2.6 (Django/Vue, AGPL-3.0+CC) | Best search (trigram + ingredient-level), more relaxed import | Less polished tablet UX, no cook mode |
| **KitchenOwl** (Flutter/Flask) | Best mobile feel, real-time household sync | Weakest import, limited FR localisation |

**Recipe storage formats:** Cooklang is the best portable canonical format
(plain-text, machine-parseable, scaling support). schema.org/JSON-LD is the
de-facto *ingestion* format (covered by `recipe-scrapers`, 638 sites).

**Ingestion building blocks** (all FOSS, all CPU-friendly):
- `recipe-scrapers` (Python, MIT) — 638 site scrapers + schema.org wild_mode
- `ingredient-parser-nlp` — NER for English ingredient strings (gap: no FR)
- `yt-dlp` + `faster-whisper` (int8 quantised) — tiny.en or base.en is the
  CPU sweet spot for cooking-content transcription
- `trafilatura` — fallback main-content extraction for unstructured sites

**LLM strategy on no-GPU Synology:**
- Local synthesis is **not viable** (Mistral 7B on J4125 ≈ 5-10 min per recipe)
- Local *is* viable for: translation (Llama 3.2 3B), ingredient parsing
- **Hybrid is the recommended pattern**: hosted API for synthesis, local for cheap tasks

**Cost estimate for 100-recipe one-time synthesis:**
- Mistral Small (EU): **~$4 total** (cheapest sovereign option)
- Mistral Medium: ~$22 total
- Claude Haiku: ~$43 total
- Infomaniak AI Tools (Geneva): pricing opaque pre-signup, but Swiss sovereignty

## Recommendation: HYBRID — Mealie + standalone ingestion microservice

**Don't fork Mealie.** Its codebase is 50K+ lines of Python/Vue and the
synthesis logic is unrelated to its data model.

**Do this instead:**

1. **Deploy Mealie** on Synology for storage / search / tablet UX / meal planning
2. **Build a small Python service** (~500–800 lines) that:
   - Accepts URL, YouTube URL, or recipe name
   - Calls `recipe-scrapers` for 3-5 sources (search alt versions via SerpAPI/Brave)
   - Calls `yt-dlp` + `faster-whisper` for YouTube
   - Calls hosted LLM API for synthesis with structured prompt → canonical JSON
   - Bilingual output handled in same call (lower cost, consistent style)
   - POSTs the result to Mealie's REST API → bypasses Mealie's manual validation step entirely
3. **Optional thin web UI** for the "pick your preferred synthesis" step
   (otherwise terminal prompt)

**Why this works:** Mealie's pain point is the manual validation step. By
producing fully-structured, normalised recipes via LLM and POSTing through
the API, we *bypass* that step entirely. Mealie sees clean, complete recipes
and never asks for clarification.

**Effort:** Weekend-scale for a Python-comfortable engineer.

## Open questions surfaced by research

1. **Bilingual storage in Mealie** — Mealie's i18n is UI-only. To store the
   same recipe in FR + EN, we either (a) create two linked Mealie records,
   (b) embed second-language content in Mealie's `notes` field, or (c)
   maintain a separate canonical store (Cooklang files) and use Mealie as
   read-only display. This is an architectural decision for Phase 5.

2. **Source discovery for "fesenjān"-style queries** — given just a name,
   we need a way to find 3-5 trusted sources. Options: SerpAPI/Brave Search
   API → filter to known-good domains; or curated allow-list of chef sites
   (Ottolenghi, Smitten Kitchen, Serious Eats, NYT Cooking, BBC Good Food).

3. **French-language ingredient parsing** — `ingredient-parser-nlp` is
   English-only. We may need to rely on the LLM for FR ingredient
   structuring, accepting higher per-call cost.

## Sources

See agent output (preserved in conversation history). Key references:
GitHub repos for Mealie, Tandoor, recipe-scrapers, faster-whisper, Cooklang;
Mistral AI pricing page; Infomaniak AI Tools; Cooklang blog comparison post.
