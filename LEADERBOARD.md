# Global Leaderboards

Dungeon Loot has a global leaderboard shared across **everyone who plays** —
ranking heroes by **furthest floor**, **highest level**, and **most gold**.

The game is still a single self-contained `index.html`. The leaderboard talks
directly to a free [Supabase](https://supabase.com) project over its REST API.
Supabase's public **anon / publishable key** is *designed* to be embedded in
client code: a row-level-security (RLS) policy decides what that key may do, so
shipping it in the page is safe.

If the two config values are left blank (or the network is down), the game falls
back to a **device-local** leaderboard stored in `localStorage`, so the feature
always works.

## Where the config lives

In `index.html`, in the `GLOBAL LEADERBOARD` section:

```js
const LB_SUPABASE_URL = 'https://YOURPROJECT.supabase.co'; // base URL, no trailing path
const LB_SUPABASE_KEY = 'sb_publishable_...';              // anon / publishable key
```

Use the **base** project URL (no `/rest/v1/`) — the code appends the REST path
itself.

## One-time Supabase setup

### 1. Create the table + access policies

In the Supabase **SQL Editor**, run:

```sql
create table if not exists public.leaderboard (
  name         text primary key,
  player_class text,
  max_floor    int  not null default 1,
  level        int  not null default 1,
  gold         int  not null default 0,
  updated_at   timestamptz not null default now()
);

alter table public.leaderboard enable row level security;

create policy "public read"   on public.leaderboard for select using (true);
create policy "public insert" on public.leaderboard for insert with check (true);
create policy "public update" on public.leaderboard for update using (true) with check (true);
```

There is intentionally **no delete policy**, so the public key can never delete
rows.

### 2. Server-side anti-cheat (recommended)

Because the whole game runs in the player's browser, anyone *can* edit their
local state. These database-side **CHECK constraints** run on Supabase, so a
tampered client still can't write impossible or implausible scores:

```sql
-- Remove the connection-test row created while wiring things up.
delete from public.leaderboard where name = '__connection_test__';

alter table public.leaderboard
  add constraint lb_sane check (
    char_length(name) between 1 and 16
    and max_floor between 1 and 1000
    and level     between 1 and 999
    and gold      between 0 and 100000000
    and max_floor <= level * 5 + 20   -- can't reach deep floors at a trivial level
  );
```

The bounds are deliberately generous so a legitimate run never trips them; the
client clamps to the same numbers (`LB_MAX_FLOOR` / `LB_MAX_LEVEL` /
`LB_MAX_GOLD`) so it never submits something the database would reject. Tune the
`max_floor <= level * 5 + 20` rule if you want it looser or stricter.

## How far anti-cheat can go

This is the honest part: a leaderboard for a **client-side** game can be made
*hard to cheat*, not *impossible*. All gameplay happens in the browser, so a
determined cheater can craft an API call with any numbers that satisfy the rules
above. The layers we ship raise the bar against casual tampering:

1. **No delete policy** — nobody can wipe the board with the public key.
2. **Server-side CHECK constraints** — absurd values (negative, billions, floor
   9999 at level 1) are rejected by the database, not the client.
3. **Client clamping** — the game itself never submits out-of-range values.

True, unbeatable anti-cheat would require moving gameplay onto a server that
validates every action — a much larger project and at odds with this game's
"single shareable HTML file" design.

### Optional hardening

- **Block score-lowering / griefing:** add a `BEFORE UPDATE` trigger that
  rejects an update whose `max_floor`, `level`, or `gold` is lower than the row's
  current value, so a row can only ever go up.
- **Rotate the key:** if the publishable key is ever abused, rotate it in the
  Supabase dashboard and paste the new one into `index.html`.
</content>
