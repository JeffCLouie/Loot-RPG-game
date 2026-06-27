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

- **Never commit directly to `main`.** Always create a branch, commit there, and
  open a pull request. Two people work on this repo and both edit the same file,
  so working straight on `main` causes collisions.
- **Always pull the latest `main` before starting new work**, so you branch from
  the current version and minimize merge conflicts.
- **One change per branch.** Keep each branch focused on a single feature or fix
  (e.g. a new enemy type, a shop tweak) so pull requests stay small and easy to
  review and merge.
- **Keep the game in one file.** Do not split the project into separate `.js` or
  `.css` files or introduce a bundler — the single-file design is intentional so
  the game stays trivially shareable and hostable.

## Workflow Claude should follow

When asked to make a change:

1. Pull the latest `main`.
2. Create a descriptively named branch (e.g. `feature/boss-fights`,
   `fix/merchant-spawn`).
3. Make the edit in `index.html`.
4. Commit with a clear message describing what changed and why.
5. Push the branch and open a pull request summarizing the change.
6. Do **not** merge the PR automatically — leave it for a human to review and
   merge, so both collaborators stay aware of what's changing.

## Commit conventions

- Use short, descriptive subject lines under ~72 characters.
- Prefix with a type when it fits: `feat:`, `fix:`, `balance:`, `ui:`, `docs:`.
  Examples: `feat: add boss fight every 5th floor`,
  `balance: lower legendary drop rate`, `ui: enlarge d-pad buttons`.
- For non-trivial changes, add a short body explaining the reasoning.

## Before pushing

- Open `index.html` in a browser and confirm the game still loads and plays.
  There are no automated tests, so a quick manual check is the safety net.
- Sanity-check that the JavaScript has no syntax errors (the game silently fails
  to run if it does).
- Glance at the diff before merging — it's easy on mobile to merge without
  reviewing what actually changed.

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

- "Pull main, branch, and add a minimap, then open a PR."
- "Branch off main and add a boss enemy on every 5th floor, then PR it."
- "Show me a summary of what changed in this branch before I merge."
