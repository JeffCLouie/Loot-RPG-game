# CLAUDE.md — Dungeon Loot

Guidance for Claude Code when working in this repository.

## What this project is

A loot-based pixel RPG that runs entirely in the browser. The whole game is a
**single self-contained HTML file** (`index.html`) — all HTML, CSS, and
JavaScript live in that one file. There is **no build step**, no framework, no
package manager, and no dependencies to install. Open the file in a browser and
it runs.

The game is hosted via Netlify / GitHub Pages, so whatever lands on the `main`
branch goes live automatically. The file is named `index.html` so it is served
at the site's root URL.

## Hard rules

- **Keep the game in one file.** Do not split the project into separate `.js` or
  `.css` files or introduce a bundler — the single-file design is intentional so
  the game stays trivially shareable and hostable.
- **Pull the latest `main` before starting new work** to minimize merge
  conflicts.

## Workflow Claude should follow

When asked to make a change:

1. Pull the latest `main`.
2. Make the edit in `index.html`.
3. Before committing, verify the JavaScript has no syntax errors (the game
   silently fails to load if it does). There are no automated tests, so this
   check is the only safety net.
4. Commit with a clear, descriptive message.
5. Push and merge directly to `main` so Netlify auto-deploys — do not open a PR
   or wait for approval.

## Commit conventions

- Use short, descriptive subject lines under ~72 characters.
- Prefix with a type when it fits: `feat:`, `fix:`, `balance:`, `ui:`, `docs:`.
  Examples: `feat: add boss fight every 5th floor`,
  `balance: lower legendary drop rate`, `ui: enlarge d-pad buttons`.
- For non-trivial changes, add a short body explaining the reasoning.

## Before pushing

- Verify the JavaScript has no syntax errors (the game silently fails to run if
  it does). This is the primary safety check since there are no automated tests.

## Code style notes

- The code is organized into clearly commented sections (constants, state, map
  generation, item generation, drawing, combat, UI, shop, save/load, init).
  Keep new code within the matching section.
- This is a "vibe coded" project meant to stay approachable. Favor clear,
  readable code over clever abstractions.
- Preserve the existing save system: progress is stored in `localStorage`.
  Changing the data shape can break existing saves, so when changing saved
  fields, handle older saves gracefully (migrate or reset cleanly).
- The Diablo 2-style loot tiers (grey → white → green → blue → purple → orange →
  red) communicate rarity by **color only** — do not re-add text rarity labels.

## Good first prompts for collaborators

- "Pull main and add a minimap."
- "Add a boss enemy on every 5th floor."
- "Show me a summary of what changed recently."
