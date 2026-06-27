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

## Pull requests: merge and conflicts automatically

When work happens on a PR instead of straight on `main` (for example, when a
task pins development to a feature branch), drive it to a merged state without
waiting for manual review:

- **Auto-merge new PRs.** Once a PR you opened is pushed and green (or has no CI,
  which is the normal state for this single-file project), merge it into `main`
  yourself — do not wait for approval. Prefer enabling auto-merge so it lands as
  soon as any checks pass; if checks are absent, merge immediately.
- **Resolve merge conflicts automatically.** If a PR (or a `git pull`/rebase)
  conflicts with `main`, resolve it yourself:
  1. Pull/fetch the latest `main` and rebase or merge it into the branch.
  2. Resolve every conflict in `index.html` by hand, keeping **both** sides'
     intent — combine the changes rather than blindly taking one side. The whole
     game is one file, so conflicts are almost always mergeable.
  3. After resolving, re-verify the JavaScript has no syntax errors (this is the
     only safety net), then continue the merge and push.
- **Always leave `main` deployable.** Because Netlify auto-deploys `main`, never
  merge a state with a syntax error. Run the syntax check after any conflict
  resolution before completing the merge.
- Only stop and ask the user when a conflict is genuinely ambiguous (two changes
  that can't both be true at once) or when the user explicitly asked to hold the
  PR open.

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
