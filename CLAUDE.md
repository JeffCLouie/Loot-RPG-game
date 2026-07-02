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
- **Work on a branch and open a PR — never commit straight to `main`.** Don't ask
  for confirmation, present options, or wait for approval on the *work itself*:
  make reasonable decisions and implement them. But land every change through a
  feature branch and a pull request into `main` (never a direct push to `main`),
  so GitHub gates the merge and any conflict surfaces to be resolved deliberately.
  Only stop to ask if a request is genuinely impossible or self-contradictory.
- **All on-screen art is real pixel art — never an emoji as the thing itself.**
  Every on-screen game asset — heroes, enemies, bosses, NPCs, summoned
  minions/allies, items, pickups, projectiles, status icons, and world objects —
  must be actual pixel-art imagery, both on the canvas and in the menus/HUD,
  never a raw emoji or hand-drawn shape standing in for the asset. Emoji are
  acceptable only as plain section-header punctuation, never as the icon that
  represents a game thing. Keep an emoji `drawEmoji`/`dlIcon` fallback only for
  the brief moment before an image has loaded. **Procedural terrain** (walls,
  floors, water, lava and similar background tiles) stays deliberately
  procedural.
  - **We generate our own art now — no longer restricted to DawnLike.** The
    existing DawnLike atlas art (DragonDePlatino & DawnBringer, CC-BY 4.0) is
    still in the game and fine to keep using and extending, but new art should be
    our own bespoke pixel art rather than pulled from DawnLike. When you add
    original art, make sure we have the right to ship it.
  - **Two ways art gets into the game.** (1) Simple single tiles live in the
    sprite atlas — a single base64 PNG assigned to `spriteSheet.src`, indexed by
    `SPRITE_IDX` (`name → tile index`, 16 tiles per row) — and render via
    `drawSpriteC` (canvas) or `dlIcon(spriteKey, px)` (DOM panels). To add one,
    composite the tile into a free atlas slot with a Chromium/canvas script
    (load the current atlas, `drawImage` into the next free index after the
    current maximum, grow the PNG by a 16px row if needed, `toDataURL`), swap the
    `spriteSheet.src` string, and add the `SPRITE_IDX` entries. (2) Multi-frame
    or larger art (e.g. the hero walk-cycle sheets) loads as its own embedded
    base64 `Image` with a ready flag and is drawn frame-by-frame with
    `ctx.drawImage` — see `heroWalkSheet` / `drawHeroWalk` for the pattern.
  - **Menus and the HUD count too.** Consumables, ingredients, food, currencies,
    buff/status indicators and any other item imagery shown in DOM panels (the
    bag, the town shops, the HUD bar, the log) must use real sprite imagery — an
    atlas tile via `dlIcon(spriteKey, px)` (it returns an inline
    `<span class="dl-ic">` backed by the atlas) or a bespoke sprite — **not** an
    emoji as the icon that represents a game thing.

## Workflow Claude should follow

When asked to make a change:

1. Fetch and branch off the latest `origin/main` (e.g. `claude/<short-topic>`) —
   never work directly on `main`.
2. Make the edit in `index.html`.
3. If the change adds or alters a gameplay mechanic, update `gameState()` and
   `gameGuide()` to match (see "Keep the AI-play API in sync" below).
4. Before committing, verify the JavaScript has no syntax errors (the game
   silently fails to load if it does). There are no automated tests, so this
   check is the only safety net.
5. Commit with a clear, descriptive message and push the branch.
6. Open a pull request into `main` with `gh`. Once it's green / mergeable (or has
   no CI — the normal state here), merge it yourself so Netlify auto-deploys — no
   need to wait for manual approval. If GitHub reports the PR is not mergeable
   (conflicts), resolve them on the branch first (see below), then merge.

## Pull requests: the standard flow (merge & resolve conflicts yourself)

**Every** change lands via a branch + PR — there is no direct-to-`main` path.
Drive each PR to a merged state without waiting for manual review, but let GitHub
gate the merge so conflicts are caught and resolved deliberately instead of
silently overwritten:

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

## Version history (changelog)

The in-game Version History popup is driven by the `CHANGELOG` array in
`index.html`. When adding or editing entries:

- **Never reference other games.** Don't describe a feature as "Diablo-2-style",
  "Golden Sun palette", "roguelike", or by comparison to any other title.
  Describe **what the change does** in plain terms instead — e.g. write
  "Rebalanced loot drops" or "Independent per-tier drop rolls", not
  "Diablo-2-style loot". This is player-facing copy and should stand on its own
  without leaning on another game for meaning.
- Keep the same shape as existing entries (`date`, `size`, `v`, `by`, `notes`)
  and add new releases at the top of the array (newest-first).
- **Add an entry for every user-facing change you ship**, in the same commit, so
  the Version tab never falls behind.
- **Be maximally concise — notes are for skimming, not reading.** Cut every
  non-essential word: drop leading articles/possessives ("The achievement badges
  moved…" → "Achievement badges move…"), use present-tense fragments over full
  sentences, and drop hedges/filler. Keep each note to roughly one line.

## Keep the AI-play API in sync (`gameState()` / `gameGuide()`)

`index.html` exposes two console functions that let an external agent play the
game without reading pixels — keep BOTH truthful on every gameplay change:

- **`gameState(radius)`** — a live snapshot of WHAT is happening right now: the
  ASCII map + glyph `legend`, the hero's stats/buffs, the skill hotbar, enemies,
  loot, hazards, shrines/teleporters, NPCs, and the menu/overlay state.
- **`gameGuide(topic)`** — the how-to-play reference explaining HOW the game
  works: the rules, formulas and habits an agent needs (overview, controls,
  movement, combat, skills, autocast, loot, autoloot, hazards, enemies,
  progression, town, driving, tips).

**Treat these two as first-class output of every gameplay change, not an
afterthought.** An agent relies on them being correct, so stale copy makes it
misplay. Whenever you add or change a mechanic:

- **New or changed live state → update `gameState()`.** If a change introduces
  something an agent must see to play (a new hazard, status effect, resource,
  enemy flag, NPC, ground pickup, menu field, overlay/`mode`, …), surface it in
  the returned object — and in the ASCII overlay + `legend` if it gets a glyph.
- **New or changed rules → update `gameGuide()`.** If a change alters how
  anything works (a reworked system, new control/keybind, retuned formula,
  changed default, new skill mechanic, …), edit the matching topic so the text
  matches the code. Add a new topic (and `alias` entries) for a whole new system.
- **Fix stale references, don't just append.** Correct anything the change made
  wrong — renamed/removed console helpers, changed key counts, reworked
  behaviours. Both functions must describe the game as it is NOW.
- **Verify after editing.** `gameGuide()` reads key bindings live and
  `gameState()` reads live game objects, so confirm the field names, helper
  names and keybind ids you reference still exist.

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
- **Typography: use the type scale, respect the floor.** No DOM text below
  `1.1rem` and no canvas text below `12px` — smaller is unreadable on phones.
  Floor dynamic canvas font sizes with `Math.max(12, …)`. The DOM scale is
  deliberate — pick the tier that matches the role, don't invent in-between
  sizes: `1.1rem` fine print (hotkey tags, badges) · `1.2rem` secondary
  (stat lines, chips, meta) · `1.3rem` body (descriptions, buttons, lists) ·
  `1.4rem` emphasized body · `1.5rem` small headings · `1.6–2rem` section
  headings/titles · `2.2rem+` display.

## Layout: no "loners" (orphans & widows)

The user strongly dislikes stranded single elements — a lone tile sitting by
itself in the corner of an otherwise-full grid, a single word wrapping onto its
own line, one item alone in a row, etc. It looks unfinished. Whenever you add or
change UI, check the result for these and fix them:

- **Grids:** if a row would hold a single trailing tile, centre it (or add
  another genuinely useful tile to balance the row — never filler). The title
  settings grid already does this via
  `#title-settings .title-set-btn:last-child:nth-child(3n+1) { grid-column: 2; }`;
  reuse that pattern for other fixed-column grids.
- **Text:** avoid one-word last lines in headings/labels/blurbs — tighten the
  wording, or use a non-breaking space (`&nbsp;`) between the last two words so
  they wrap together.
- Apply this proactively on every UI change, not just when asked.

## Good first prompts for collaborators

- "Pull main and add a minimap."
- "Add a boss enemy on every 5th floor."
- "Show me a summary of what changed recently."
