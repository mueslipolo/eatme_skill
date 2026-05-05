# recipe-forge skill

A Claude skill that synthesises a canonical recipe **in French** from multiple
trusted sources and outputs **schema.org/Recipe JSON-LD** ready to paste into
Mealie's "Create Recipe → Create from JSON" form.

English fallbacks are preserved in parentheses inside the ingredient strings
when translation is ambiguous (meat cuts, specific flour types, US-specific
ingredients).

## Install on Claude.ai (web)

1. **claude.ai → Settings → Capabilities → Skills**
2. Click **Create skill** (or **Upload skill**)
3. Upload the `SKILL.md` file from this directory
4. Invoke from any Claude.ai conversation with `/recipe-forge` followed by a dish name or URL

## Install for Claude Code (local)

```bash
mkdir -p ~/.claude/skills/recipe-forge
ln -sfn "$PWD" ~/.claude/skills/recipe-forge
```

Restart Claude Code. The skill becomes available as `/recipe-forge`.

## Usage

```
/recipe-forge fesenjān
/recipe-forge https://www.ottolenghi.co.uk/recipes/chicken-with-caramelised-onion
/recipe-forge youtube.com/watch?v=...
```

No arguments — the skill always outputs French recipes with metric units, uses
the source's serving size (defaults to 4–6 if none given), and keeps English
fallbacks in parens for ambiguous technical terms (`paleron de bœuf
(chuck roast)`, `farine T55 (all-purpose flour)`, etc.).

The skill outputs three sections:
1. **SOURCES table** — what was fetched, what failed, which is the spine
2. **DECISIONS** — points where sources disagreed; you confirm or override
3. **schema.org Recipe JSON-LD** — paste into Mealie → Create Recipe → Create from JSON

## Pasting into Mealie — read this

Mealie's `/recipes/create/html-or-json` endpoint detects JSON via
`req.data.startswith("{")`. The very first character pasted **must be
`{`** — no leading whitespace, no markdown fence, no BOM. Otherwise
the endpoint returns `BAD_RECIPE_DATA`.

Safest paste workflow:
- Click the **copy** button on the JSON code block (bypasses fence/whitespace), or
- Save to a file and pipe through clipboard:
  ```bash
  xclip -selection clipboard < recipe.json   # X11
  wl-copy < recipe.json                      # Wayland
  pbcopy < recipe.json                       # macOS
  ```

## What schema.org JSON-LD does NOT preserve

Trade-off vs Mealie's authenticated API path:
- **Structured ingredients** — JSON-LD ingredients are strings only.
  Mealie stores them as raw `note` lines. Cross-recipe shopping-list
  aggregation and meal-planner integrations are weaker than with
  structured `quantity/unit/food`.
- **Custom `notes` blocks with title** — workaround: chef's note and
  sources are appended as trailing `HowToStep` items with `name` set
  as a heading. They render as the last steps of the recipe.
- **Skill version provenance** — written into the Sources HowToStep
  text, not a structured `extras` field.

If you ever need richer fidelity (structured ingredients, true notes,
extras), there's a path via `POST /recipes` with a Mealie API token.
That requires authentication and exposing Mealie to the skill —
deferred for now.

## Iterating on the prompt

Edit `SKILL.md` directly. On Claude.ai web, re-upload after changes
(skills don't hot-reload). For Claude Code, edits take effect on next invocation.

The skill writes the synthesis date and `recipe-forge vX` into the Sources
HowToStep. Bump the version string when making non-trivial prompt changes —
recipes in Mealie carry a record of which prompt version produced them.
