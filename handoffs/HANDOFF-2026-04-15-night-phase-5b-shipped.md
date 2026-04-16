# Handoff — April 15, 2026 (late night, Phase 5b shipped)

## Session result

**Phase 5b Option A shipped.** Real Add to CRM + Watchlist persistence is live. Two new Supabase tables (`crm_added`, `watchlist`) with RLS policies, plus 4 surgical frontend changes to `index.html`. Persistence confirmed: add a prospect on one device, hard refresh, still there. Cross-device sync works via Supabase.

Three commits on `main` in the Apex-App repo this session:
1. `d26851a` — Phase 5b: real Add to CRM + Watchlist persistence (Supabase) — initial 4-function rewrite
2. `4537a41` — Fix: pass user_id explicitly in CRM/Watchlist POST (RLS compat) — `DEFAULT auth.uid()` didn't populate via PostgREST; explicit `user_id:UID` in POST body fixed it
3. (Phase 5a commit from earlier session also pushed in same terminal session)

## What shipped

### Backend (Supabase — apex-intel-db)

**`crm_added` table:**
- `user_id uuid NOT NULL DEFAULT auth.uid()` — FK to `auth.users(id)` ON DELETE CASCADE
- `prospect_id uuid NOT NULL` — FK to `prospect_shortlist(id)` ON DELETE CASCADE
- `school_id uuid` — FK to `schools(id)`, nullable
- `added_at timestamptz NOT NULL DEFAULT now()`
- PRIMARY KEY: `(user_id, prospect_id)`
- RLS enabled, 3 policies: SELECT/INSERT/DELETE scoped to `auth.uid() = user_id`
- Grants: ALL to `authenticated` and `anon`

**`watchlist` table:**
- Identical schema and RLS to `crm_added`

### Frontend (Apex-App repo, `index.html`)

4 functions modified:

1. **`loadData()`** — Promise.all extended from 12 → 14 endpoints:
   - `r[12]`: `api("/rest/v1/crm_added?select=prospect_id")` with `.catch(→[])`
   - `r[13]`: `api("/rest/v1/watchlist?select=prospect_id")` with `.catch(→[])`
   - In `.then()`: `ADDED_TO_CRM` and `WATCHLIST` Sets populated from server rows
   - RLS auto-scopes to current user via bearer token

2. **`toggleAddCrm(prospectId)`** — optimistic update pattern:
   - Flip `ADDED_TO_CRM` Set + `render()` immediately
   - POST `{user_id:UID, prospect_id, school_id:CURRENT_SCHOOL_ID}` to `/rest/v1/crm_added`
   - On DELETE: filter by `?prospect_id=eq.X`
   - On failure: rollback Set + re-render + "Sync failed" toast

3. **`toggleWatch(prospectId)`** — same optimistic pattern against `/rest/v1/watchlist`

4. **`signOut()`** — clears both Sets: `ADDED_TO_CRM=new Set();WATCHLIST=new Set();`

### File stats
- `index.html`: 3,403 lines (up from 3,379 in Phase 5a)
- Diff: +32 insertions, -10 deletions across both commits
- JS validates clean via `node --check`

## Gotchas discovered (carry forward)

### Supabase new-table checklist (burned in for all future tables)
When creating tables via SQL editor (not Supabase dashboard UI), you MUST do all of these or PostgREST will return 403:

1. `CREATE TABLE ...`
2. `ALTER TABLE ... ENABLE ROW LEVEL SECURITY;`
3. `CREATE POLICY ... FOR SELECT ...`
4. `CREATE POLICY ... FOR INSERT ...`
5. `CREATE POLICY ... FOR DELETE ...`
6. **`GRANT ALL ON {table} TO authenticated;`** ← easy to forget
7. **`GRANT ALL ON {table} TO anon;`** ← easy to forget
8. **`NOTIFY pgrst, 'reload schema';`** ← forces PostgREST to see new tables immediately

Steps 6-8 are handled automatically when creating tables through the Supabase dashboard UI, but NOT when using raw SQL. Missing any of them causes silent 403s.

### `DEFAULT auth.uid()` does NOT work via PostgREST
Column defaults using `auth.uid()` don't populate when inserting via the PostgREST API. Always pass `user_id` explicitly from the frontend: `{user_id: UID, ...}` in the POST body. The RLS `WITH CHECK (auth.uid() = user_id)` policy still validates the value, so it's secure.

### Supabase SQL editor single-statement rule
Still applies. Run DDL one statement at a time. `DROP TABLE ... CASCADE` deletes all policies on the table — if you recreate, you must recreate all policies too.

### File naming convention (new)
Deliverable builds are now named `index-{phase}.html` (e.g., `index-5b.html`) to prevent Downloads folder collisions. The file gets renamed to `index.html` when moved into `~/code/Apex-App/`.

## Database state (updated)

- 22 tables total in Supabase (was 20): added `crm_added` + `watchlist`
- 7,526 total players in `school_rosters` (unchanged)
- 14 Supabase endpoints feeding `loadData()` (was 12): added `crm_added` + `watchlist`
- `crm_added` and `watchlist` each have test data (Juke Harris in CRM, Flory Bidunga on watchlist)

## Repository state

### Apex-App (frontend)
- 3 new commits on `main` this session
- Latest commit: `4537a41` — Fix: pass user_id explicitly in CRM/Watchlist POST (RLS compat)
- Live at: [apexver1.github.io/Apex-App/?v=5b2](https://apexver1.github.io/Apex-App/?v=5b2)

### apex-intel (backend)
- No new commits this session (SQL executed via dashboard)

## Phase 5b+ candidates (pick ONE per session)

### Option B: Real player photos backfill
- Add `photo_url text` to `school_rosters` and `prospect_shortlist`
- ESPN headshot URL pattern: `https://a.espncdn.com/i/headshots/mens-college-basketball/players/full/{espn_player_id}.png`
- Requires `espn_player_id` on roster rows (may need scrape pass first)
- Frontend: pass `p.photo_url` to existing `photoAv()` calls

### Option C: `prospect_contacts` schema generalization
- Replace `dummyContactsFor()` with real `prospect_contacts` table
- Migrate `prospect_family` data
- Wire contact profile modal edit/save

### Option D: `prospect_visits` table
- Replace `VISITS_SEED` with real Supabase table
- New visit creation flow from the app

### Option E: Action buttons (Call / Text / Email)
- Wire contact profile modal's Call/Text/Email tiles to `tel:` / `sms:` / `mailto:` links
- Log the touchpoint to `crm_log` automatically

## Working agreements (carry forward — all still active)

1. Full copy-paste-ready code blocks. Never find-and-replace.
2. Every URL as clickable markdown link.
3. Mac-native instructions (Terminal, Cmd shortcuts, local git).
4. Maximally specific: full paths, exact paste locations, every command.
5. Numbered step-by-step lists.
6. Never suggest stopping.
7. Read CLAUDE.md + latest handoff at session start.
8. Generate handoff at session end.
9. Map file ONCE up front, work from memory.
10. Larger contiguous str_replace blocks.
11. Validate at midpoint AND end with `node --check`.
12. Skip exploratory view calls during execution.
13. ONE shippable thing per session.
14. Validate as LAST step before present_files.
15. **NEW:** Name deliverable builds `index-{phase}.html` (e.g., `index-5b.html`).
16. **NEW:** When creating Supabase tables via SQL, always run GRANT + NOTIFY after RLS policies.
17. **NEW:** Always pass `user_id: UID` explicitly in POST bodies — never rely on `DEFAULT auth.uid()`.

## Reference constants (carry forward)

### Repos + URLs
- Frontend repo: [github.com/Apexver1/Apex-App](https://github.com/Apexver1/Apex-App)
- Backend repo: [github.com/Apexver1/apex-intel](https://github.com/Apexver1/apex-intel)
- Live URL: [apexver1.github.io/Apex-App/](https://apexver1.github.io/Apex-App/)
- Phase 5b cache-bust: [apexver1.github.io/Apex-App/?v=5b2](https://apexver1.github.io/Apex-App/?v=5b2)

### Local paths (Mac)
- Frontend: `~/code/Apex-App/`
- Backend: `~/code/apex-intel/`
- Python venv: `~/code/apex-intel/.venv` (Python 3.12.13 via Homebrew)

### Auth
- Auto-login email: `apex@test.com`
- Auto-login password: `ApexTest123`
- School picker password: `stewart2026`
- Override flags: `?nologin=1` (bypass auto-login), `?splash=1` (force splash)

### Supabase
- Project: [midyxvjfoggchbugxzkd.supabase.co](https://midyxvjfoggchbugxzkd.supabase.co)
- SQL editor: one statement at a time
- Default API row cap: 1000 (paginate)
- New table checklist: CREATE → RLS → policies → GRANT authenticated → GRANT anon → NOTIFY pgrst

### Oregon
- Supabase UUID: `d4dd73c7-2ec6-49bf-b63b-5c4a333a2bf7`
- ESPN ID: `2483` / CBBD ID: `223` / BallDontLie ID: `123` / sports247 ID: `24090`
- NCAA slug: `oregon`

### Long-script paste pattern
```bash
cat > path/to/file.py <<'APEX_EOF'
[paste full script]
APEX_EOF
```

### Fuzzy school name matching
0.94 threshold, 0.04 margin — or Ohio State → Chico State.

## Session starter for next chat

```
Loading project context for Phase 5c — Apex Intel next feature.

Please fetch:
https://github.com/Apexver1/apex-intel/raw/main/CLAUDE.md
https://github.com/Apexver1/Apex-App/raw/main/handoffs/HANDOFF-2026-04-15-night-phase-5b-shipped.md

If GitHub fetch fails, I'll paste both files directly.

Pick one of Options B/C/D/E from the handoff, or I'll specify what I want to build.

Working agreements (critical):
1. Map index.html ONCE at the start.
2. Larger contiguous str_replace blocks.
3. node --check at midpoint AND end.
4. Deliver as index-{phase}.html via present_files.
5. Numbered step-by-step lists.
6. Every URL as a clickable markdown link.
7. For new Supabase tables: CREATE → RLS → policies → GRANT auth → GRANT anon → NOTIFY pgrst.
8. Always pass user_id:UID explicitly in POST bodies.
```

Then drag in your production `~/code/Apex-App/index.html`.
