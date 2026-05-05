# recipe-forge skill

A Claude skill that synthesises a canonical recipe **in French** from multiple
trusted sources and outputs Mealie-importable JSON. English fallbacks are
preserved in parentheses inside the ingredient note when translation is
ambiguous (meat cuts, specific flour types, US-specific ingredients).

## Install on Claude.ai (web)

1. **claude.ai → Settings → Capabilities → Skills**
2. Click **Create skill** (or **Upload skill**)
3. Upload the `SKILL.md` file from this directory
4. Invoke from any Claude.ai conversation with `/recipe-forge` followed by a dish name or URL

## Install for Claude Code (local)

```bash
mkdir -p ~/.claude/skills/recipe-forge
ln -s "$PWD/SKILL.md" ~/.claude/skills/recipe-forge/SKILL.md
```

Restart Claude Code. The skill becomes available as `/recipe-forge`.

## Usage

```
/recipe-forge fesenjān
/recipe-forge https://www.ottolenghi.co.uk/recipes/chicken-with-caramelised-onion
/recipe-forge youtube.com/watch?v=...
```

No arguments — the skill always outputs French recipes with metric units, uses the source's serving size (defaults to 4–6 if none given), and keeps English fallbacks in parens for ambiguous technical terms (`paleron de bœuf (chuck roast)`, `farine T55 (all-purpose flour)`, etc.).

The skill outputs three sections:
1. **SOURCES table** — what was fetched, what failed, which is the spine
2. **DECISIONS** — points where sources disagreed; you confirm or override
3. **Mealie JSON** — paste this into Mealie → Create Recipe → Create from JSON

## Iterating on the prompt

Edit `SKILL.md` directly. On Claude.ai web, re-upload after changes
(skills don't hot-reload). For Claude Code, edits take effect on next invocation.

Bump `extras.skill_version` in the JSON output when making non-trivial changes —
recipes in Mealie carry a record of which prompt version produced them, useful
when iterating.

## Mealie setup needed

Recipes are imported via **Recipes → Create Recipe → Create from JSON** in the
Mealie UI. The skill does not push directly to Mealie's API (that would
require exposing the Synology to claude.ai's outbound — currently keeping
the NAS private).
