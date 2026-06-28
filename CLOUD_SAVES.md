# Cloud Saves (cross-device sync)

Dungeon Loot can optionally link a player's **save slots** to an account so
progress carries across devices. It's built on the **same Supabase project** as
the [global leaderboard](LEADERBOARD.md) — it reuses the existing
`LB_SUPABASE_URL` / `LB_SUPABASE_KEY` values in `index.html`, talking directly to
Supabase Auth and a `saves` table over the publishable (anon) key. No SDK and no
build step: the game stays a single self-contained `index.html`.

Signing in is **entirely optional**. When a player is logged out (or no backend
is configured), the game saves only to `localStorage` exactly as before. When
logged in, `localStorage` stays the fast local cache and every save is also
mirrored to the cloud; on boot and on login the two are reconciled per slot by
**last-write-wins** using each save's own timestamp.

## How it works

- **Login** is email + password via Supabase Auth (the GoTrue REST API:
  `/auth/v1/signup` and `/auth/v1/token`). The session (access + refresh tokens)
  is kept in `localStorage` so a login survives reloads, and the access token is
  refreshed automatically.
- **Saves** live in a `saves` table, one row per `(user_id, slot)`, with the
  whole save JSON in a `jsonb` column. A row-level-security policy scopes every
  row to its owner (`auth.uid()`), so the public key can only ever read or write
  the signed-in player's own saves.
- The account UI is reachable from the **landing page** ("☁️ Cloud Save" / the
  "Sign in to carry your save across devices" callout) and from the in-game
  **⚙️ Settings → CLOUD** button, so players can sign in before or during play.

## One-time Supabase setup

In the Supabase **SQL Editor**, run this once to create the saves table and its
access policies:

```sql
create table if not exists public.saves (
  user_id    uuid        not null references auth.users(id) on delete cascade,
  slot       int         not null,
  data       jsonb       not null,
  updated_at timestamptz not null default now(),
  primary key (user_id, slot)
);

alter table public.saves enable row level security;

create policy "own saves select" on public.saves
  for select using (auth.uid() = user_id);
create policy "own saves insert" on public.saves
  for insert with check (auth.uid() = user_id);
create policy "own saves update" on public.saves
  for update using (auth.uid() = user_id) with check (auth.uid() = user_id);
create policy "own saves delete" on public.saves
  for delete using (auth.uid() = user_id);
```

Then, in **Authentication → Providers → Email**, make sure **Email** sign-ups
are enabled. For the smoothest experience, turn **"Confirm email" off** so new
accounts can sign in immediately. If you leave confirmation **on**, sign-up still
works — the game tells the player to confirm via the emailed link and then log
in. (Email auth needs an SMTP sender configured for confirmation mails; the
built-in Supabase mailer is rate-limited and meant for testing only.)

That's all the backend needs. The same project URL and publishable key already
used by the leaderboard cover cloud saves too.

## Notes

- **No backend configured?** If `LB_SUPABASE_URL` / `LB_SUPABASE_KEY` are blank,
  the Cloud Save panel says so and the game just uses local saves.
- **Conflicts.** Sync is last-write-wins per slot by save timestamp, which fits
  the "play on one device at a time" case. Playing the same slot on two devices
  at once can let the later save overwrite the earlier one.
- **Privacy.** Only the player's own save JSON is stored, and RLS prevents anyone
  else's key from reading it. Passwords are handled entirely by Supabase Auth —
  the game never stores them.
</content>
</invoke>
