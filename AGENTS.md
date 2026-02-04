# AGENTS.md

## Primary Rule
When a user asks for something, look in the local notes first before guessing or looking up information on the internet.  Begin by searching the Markdown notes with `rg`, `fd`, or another quick file-search tool, and inspect the matching files before reaching for external sources.  Do not search online or use `web.run` until the user explicitly asks “no, look on the internet”, or equivalent clear direction, after you have already found relevant local info.

## Searching Guidance
- Search the `Collections/` directory first, then the top-level files listed below, using commands like `rg -n "keyword" Collections/**/*.md` or `rg --files -g "*.md" | xargs rg -n "keyword"` to keep the work local-focused.
- Open each promising file with `cat`, `sed`, or an editor to confirm the answer before considering further work.

## Repository Directories
- `Collections/` — curated topic folders and note sets.
- `.obsidian/` — Obsidian app configuration for this vault.
- `.git/` — git metadata.

## Top-Level Files
- `README.md` — overview of the notes repository.
- `Links.md` — shared link collection.
- `Rezepte.md` — recipes.
- `CONTRIBUTING.md` — contribution guidelines.
- `LICENSE` — repository license.
- `.gitignore` — git ignore rules.
