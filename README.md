# eatme — recipe-forge skill

A Claude skill that synthesises a canonical recipe **in French** from multiple
trusted sources and outputs **schema.org/Recipe JSON-LD** ready to paste
into [Mealie](https://mealie.io)'s "Create Recipe → Create from JSON" form.

Designed for an experienced French-speaking home cook who wants the *synthesis*
of several versions of a dish (Ottolenghi vs. Persian-Mama vs. NYT Cooking)
rather than just a one-source import. Sources are cited; disagreements are
surfaced as decisions; quantities are never invented.

French is the canonical language because it is the language of cooking.
English fallbacks are preserved in parentheses inside the ingredient note
when translation would lose precision — meat cuts (`paleron de bœuf
(chuck roast)`), flour types (`farine T55 (all-purpose flour)`), and
US-specific ingredients without a clean French equivalent.

## What it does

Given a dish name, a recipe URL, or a YouTube link, the skill:

1. Discovers (or fetches) 2–5 reputable sources
2. Cross-references them; picks a coherent *spine* recipe and flags real disagreements as decisions
3. Outputs Mealie-compatible JSON with metric-first units, short technique-aware instructions, structured ingredients, and explicit source citations
4. All output is in French; English technical terms preserved in parens where translation is ambiguous

## Repository layout

```
.
├── README.md                          # this file
└── skills/
    └── recipe-forge/
        ├── SKILL.md                   # the skill itself
        └── README.md                  # install + usage details
```

## Install

### Claude.ai (web)

1. **Settings → Capabilities → Skills → Upload**
2. Pick `skills/recipe-forge/SKILL.md`
3. Invoke from any conversation:
   ```
   /recipe-forge fesenjān
   /recipe-forge https://www.ottolenghi.co.uk/recipes/...
   /recipe-forge https://youtube.com/watch?v=...
   ```

### Claude Code (local)

```bash
git clone git@github.com:mueslipolo/eatme_skill.git
ln -s "$PWD/eatme_skill/skills/recipe-forge" ~/.claude/skills/recipe-forge
```

Restart Claude Code; `/recipe-forge` becomes available.

## Usage

The skill takes no arguments beyond the dish/URL. Servings are inferred
from the source (defaults to 4–6 if absent). Output is always bilingual.

It produces three sections:

1. **SOURCES table** — what was fetched, what failed, which is the spine
2. **DECISIONS** — significant disagreements between sources for you to confirm/override
3. **Mealie JSON** — paste into Mealie → Create Recipe → Create from JSON

## Mealie integration

The skill does not push to Mealie's API directly — recipes are imported
manually via Mealie's "Create from JSON" form. This keeps the Mealie
instance private (no inbound exposure required).

The JSON shape uses Mealie's structured ingredient model
(`quantity` + `unit` + `food` + `note`) so shopping list and meal-planning
features remain available.

## Iterating on the prompt

Edit `skills/recipe-forge/SKILL.md` directly. Bump `extras.skill_version`
in the output schema when making non-trivial changes — recipes in Mealie
carry a record of which prompt version produced them, useful for tracking
regressions.

On claude.ai web, re-upload the skill after edits (no hot reload). For
Claude Code, edits take effect on next invocation.
