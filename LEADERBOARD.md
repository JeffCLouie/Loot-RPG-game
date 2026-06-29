# Global Leaderboards

Dungeon Loot has a global leaderboard shared across **everyone who plays** —
ranking heroes by **furthest floor**, **highest level**, **most gold**, and
**highest Power**.

The game is still a single self-contained `index.html`. The leaderboard talks
directly to a free [Supabase](https://supabase.com) project over its REST API.
Supabase's public **anon / publishable key** is *designed* to be embedded in
client code: a row-level-security (RLS) policy decides what that key may do, so
shipping it in the page is safe.

If the two config values are left blank (or the network is down), the game falls
back to a **device-local** leaderboard stored in `localStorage`, so the feature
always works.

Every submitted score is taken at face value — there is no server-side
validation of the numbers. Any value a player reaches is treated as real.

## Where the config lives

In `index.html`, in the `GLOBAL LEADERBOARD` section:

```js
const LB_SUPABASE_URL = 'https://YOURPROJECT.supabase.co'; // base URL, no trailing path
const LB_SUPABASE_KEY = 'sb_publishable_...';              // anon / publishable key
```

Use the **base** project URL (no `/rest/v1/`) — the code appends the REST path
itself.

## One-time Supabase setup

In the Supabase **SQL Editor**, run this once to create the table and its access
policies:

```sql
create table if not exists public.leaderboard (
  name         text primary key,
  player_class text,
  max_floor    int  not null default 1,
  level        int  not null default 1,
  gold         int  not null default 0,
  power        int  not null default 1,
  updated_at   timestamptz not null default now()
);

-- If you created the table before the Power board existed, add the column:
alter table public.leaderboard add column if not exists power int not null default 1;

alter table public.leaderboard enable row level security;

create policy "public read"   on public.leaderboard for select using (true);
create policy "public insert" on public.leaderboard for insert with check (true);
create policy "public update" on public.leaderboard for update using (true) with check (true);
```

That's all the backend needs. Each character upserts a single row keyed by its
name; the four boards are just that table sorted by `max_floor`, `level`,
`gold`, or `power`.

## Rotating the key

If the publishable key ever needs to change, rotate it in the Supabase dashboard
and paste the new value into `LB_SUPABASE_KEY` in `index.html`.
</content>
