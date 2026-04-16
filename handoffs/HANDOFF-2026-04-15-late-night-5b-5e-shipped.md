# Handoff — April 15, 2026 (late night, Phases 5b–5e shipped)

## Session result

**Massive ship night.** Four phases deployed in one session: 5b (persistence), 5c (photos), 5d (action links + scout cards), 5e (logos + badges + comparison). The app went from "frontend-only prototype with initials everywhere" to "real data-persisted tool with ESPN headshots, school logos, and working Call/Text/Email."

**Commits on `main` in Apex-App repo this session (8 total):**
1. `d26851a` — Phase 5b: real Add to CRM + Watchlist persistence (Supabase)
2. `4537a41` — Fix: pass user_id explicitly in CRM/Watchlist POST (RLS compat)
3. `dea797e` — Phase 5c: wire photo_url to all photoAv calls
4. `88dbb96` — Phase 5c: hero-size (120px) headshots in player modals
5. `899374a` — Phase 5c: larger card photos (lg/44px), roster photo propagation
6. `f0afc49` — Phase 5d: Call/Text/Email action links, hero modal photos, scout card photos
7. `aff24b6` — Phase 5e: school logos, drawer badges, player comparison, Call/Text/Email links
8. Final polish — bigger logos (24px cards, 28px modals), subtitle font bump

---

## Everything that shipped tonight

### Phase 5b — CRM + Watchlist persistence
- **`crm_added` table** in Supabase: `(user_id, prospect_id)` PK, `school_id`, `added_at`
- **`watchlist` table** in Supabase: identical schema
- Both have RLS (SELECT/INSERT/DELETE scoped to `auth.uid() = user_id`)
- Both have GRANT ALL to `authenticated` + `anon`
- Frontend `loadData()` extended from 12 → 14 endpoints (r[12] = crm_added, r[13] = watchlist)
- `toggleAddCrm()` and `toggleWatch()` use optimistic update pattern: flip Set + render immediately, POST/DELETE to Supabase, rollback on failure
- `signOut()` clears both Sets
- **User_id is passed explicitly** in POST body (`{user_id:UID, prospect_id, school_id}`) — `DEFAULT auth.uid()` does NOT work via PostgREST

### Phase 5c — ESPN headshot photos
- **`photo_url text`** column added to `school_rosters`, `prospect_shortlist`, and `my_roster`
- **`espn_player_id text`** column added to `school_rosters`
- **Python script** `scripts/espn_photos.py` fetched ESPN roster JSONs for all 365 D1 schools
  - 3,168 name matches → 2,343 new photo_url rows written to school_rosters
  - ESPN headshot URL pattern: `https://a.espncdn.com/i/headshots/mens-college-basketball/players/full/{espn_player_id}.png`
  - Rate limited at 0.35s per school (~3 min total runtime)
- **SQL propagation** copied photos from school_rosters → prospect_shortlist (by player name) and school_rosters → my_roster (by player name)
- **69 of 78 Oregon my_roster players** have photos. 9 missing are ESPN data gaps (no headshot on file), not code bugs.
- **Photo coverage across DB:**
  - school_rosters: ~5,136 of 7,580 rows have photo_url (2,793 already had + 2,343 new)
  - prospect_shortlist: propagated from school_rosters by name match
  - my_roster: 69 of 78 Oregon players
- Frontend `photoAv()` calls updated in 6 locations to pass `p.photo_url`:
  - `renderRoster` (lg/44px)
  - `_recCard` (lg/44px)
  - `renderCRM` player card (lg/44px)
  - `renderCRM` nested player row (sm)
  - `renderFilter` result cards (sm)
  - `renderScout` rules-mode result cards (sm)
- **New CSS class `.av-hero`** (120px) for player detail modals
- Player detail modal (`openP`) and scout player modal (`openScoutPlayer`) both use `hero` size

### Phase 5d — Action links + scout card photos
- **Call/Text/Email tiles** in contact profile modal (`openContactProfile`) wired to real `tel:` / `sms:` / `mailto:` links
  - If phone number exists → `<a href="tel:...">` (opens dialer on phone)
  - If email exists → `<a href="mailto:...">` (opens mail app)
  - If missing → toast "No phone/email on file"
- **LLM scout result cards** now have photo avatars (were missing entirely)

### Phase 5e — Logos + badges + comparison
- **School logos** via ESPN CDN: `https://a.espncdn.com/i/teamlogos/ncaa/500/{espn_id}.png`
  - New global `schoolEspnMap` built in `loadData()` from `schools.espn_id`
  - New helper `schoolLogo(name, size)` returns `<img>` tag with onerror fallback
  - `espn_id` added to schools fetch SELECT
  - Wired into: `_recCard` (24px), player modal subtitle (28px)
- **Drawer badges**
  - `recBadge` on Recruiting: count of portal prospects being watched
  - `homeBadge` on Home: pending reminders count
  - `crmBadge` on Apex CRM: overdue follow-ups (existing, unchanged)
  - All updated via `updateCrmBadge()` called after `loadData()`
- **Player comparison mode**
  - New global `compareTarget` (null or `{id, kind}`)
  - "Compare ↔" button in player detail modal header
  - Flow: tap Compare → toast "Now tap another player to compare" → tap second player → `showComparison(a, b)` opens side-by-side modal
  - Comparison rows: PPG, RPG, APG, 3PT%, Height, NIL Value
  - Green highlight on the better stat per row
  - "Done" button clears compareTarget and closes

---

## Known issues / polish items for next session

1. **Logo + subtitle font sizing** — user wants to play with sizes more. Currently 24px on cards, 28px in modals, 13px subtitle font. May want different sizing.
2. **9 missing Oregon roster photos** — ESPN data gap, not code bug. Players: Darren Harris, Dezdrick Lindsay, Ege Demir, Kwame Evans Jr., Nikolas Khamenia, Sven Djopmo, Travelle Bryson, Tyler Lundblade, Win Miller. Could manually set photo_url from other sources.
3. **~2,500 unmatched school_rosters players** — ESPN roster JSON didn't return these players (transfers out, graduated, name format differences). Could improve with fuzzy matching.
4. **Drawer badges show dot not number for Home** — CSS `.dot` class was replaced with `.badge` but the styling may differ; verify on mobile.
5. **Compare button styling** — currently a small text button; could be more prominent.

---

## Database state (updated)

### Tables: 22 total
Original 20 + `crm_added` + `watchlist`

### New columns added tonight
- `school_rosters.photo_url text`
- `school_rosters.espn_player_id text`
- `prospect_shortlist.photo_url text`
- `my_roster.photo_url text`

### Row counts
- `school_rosters`: 7,580 total (5,677 D1 + 657 HS + 1,192 portal + some extras)
- `prospect_shortlist`: 50 tracked prospects for Oregon
- `my_roster`: 78 Oregon players (69 with photos)
- `crm_added`: test data (Juke Harris)
- `watchlist`: test data (Flory Bidunga)
- `schools`: 1,518 total, 365 with espn_id + conference

### Supabase endpoints in loadData(): 14
Original 12 + `crm_added` + `watchlist`

---

## Supabase new-table checklist (CRITICAL — carry forward)

When creating tables via SQL editor (not dashboard UI), MUST do ALL of these:

1. `CREATE TABLE ...`
2. `ALTER TABLE ... ENABLE ROW LEVEL SECURITY;`
3. `CREATE POLICY ... FOR SELECT ...`
4. `CREATE POLICY ... FOR INSERT ...`
5. `CREATE POLICY ... FOR DELETE ...`
6. `GRANT ALL ON {table} TO authenticated;`
7. `GRANT ALL ON {table} TO anon;`
8. `NOTIFY pgrst, 'reload schema';`

Missing any of 6-8 causes silent 403s. Missing policies causes 403 on specific operations.

**Also:** Always pass `user_id: UID` explicitly in POST bodies — never rely on `DEFAULT auth.uid()` via PostgREST.

---

## Live links (working RIGHT NOW on any device)

| Link | Purpose |
|---|---|
| [apexver1.github.io/Apex-App/?v=5e4](https://apexver1.github.io/Apex-App/?v=5e4) | Production (auto-login) |
| [apexver1.github.io/Apex-App/?v=5e4&splash=1](https://apexver1.github.io/Apex-App/?v=5e4&splash=1) | Production with splash screen |
| [apexver1.github.io/Apex-App/?nologin=1](https://apexver1.github.io/Apex-App/?nologin=1) | Manual login (skip auto) |

### Credentials
- **Auto-login email:** `apex@test.com`
- **Auto-login password:** `ApexTest123`
- **School picker password:** `stewart2026`

---

## Repos + local paths

| What | Where |
|---|---|
| Frontend repo | [github.com/Apexver1/Apex-App](https://github.com/Apexver1/Apex-App) |
| Backend repo | [github.com/Apexver1/apex-intel](https://github.com/Apexver1/apex-intel) |
| Frontend local | `~/code/Apex-App/` |
| Backend local | `~/code/apex-intel/` |
| Python venv | `~/code/apex-intel/.venv` (Python 3.12.13) |
| Photo script | `~/code/apex-intel/scripts/espn_photos.py` |

### Oregon reference constants
- Supabase UUID: `d4dd73c7-2ec6-49bf-b63b-5c4a333a2bf7`
- ESPN ID: `2483` / CBBD ID: `223` / BallDontLie ID: `123` / sports247 ID: `24090`
- NCAA slug: `oregon`
- Supabase project: [midyxvjfoggchbugxzkd.supabase.co](https://midyxvjfoggchbugxzkd.supabase.co)

---

## What to work on next (priority order)

### High impact / small scope
1. **Logo + subtitle polish** — play with sizes, possibly add logos to more screens (school picker, CRM, filter results)
2. **Roster page photos for Oregon** — 69/78 working, 9 need manual photo_url or alternate source
3. **Compare mode polish** — bigger button, maybe add to recruiting card directly, more stat rows

### Medium impact / medium scope
4. **Option C: `prospect_contacts` schema** — replaces `dummyContactsFor()` with real editable contacts. Unblocks the "Edit contact" button in contact profile modal.
5. **Option D: `prospect_visits` table** — replaces `VISITS_SEED` with real Supabase table. New visit creation from the app.
6. **Bottom nav bar** — the app currently uses drawer navigation only. A fixed bottom nav (Home / Recruiting / CRM / More) would be a major mobile UX upgrade.

### High impact / larger scope
7. **Real-time roster builder** — the slot management system writes to Supabase but the UI could be smoother
8. **Push notifications** — alert staff when portal player status changes
9. **Multi-user auth roles** — head coach / assistant / ops see different data

---

## Working agreements (carry forward — ALL still active)

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
13. ONE shippable thing per session (we shipped 4 tonight — ambitious but it worked).
14. Validate as LAST step before present_files.
15. Name deliverable builds `index-{phase}.html`.
16. When creating Supabase tables via SQL: CREATE → RLS → policies → GRANT auth → GRANT anon → NOTIFY pgrst.
17. Always pass `user_id: UID` explicitly in POST bodies.
18. For Python scripts > 30 lines, use heredoc paste pattern.

---

## File stats

- `index.html`: ~3,479 lines (up from 3,379 at session start)
- 8 commits on `main` this session
- Latest commit: final 5e polish (bigger logos + subtitle font)

---

## Session starter for tomorrow

Open a fresh chat in claude.ai. Paste this as your first message:

````
Loading project context — Apex Intel next session.

Please fetch:
https://github.com/Apexver1/apex-intel/raw/main/CLAUDE.md
https://github.com/Apexver1/Apex-App/raw/main/handoffs/HANDOFF-2026-04-15-late-night-5b-5e-shipped.md

If GitHub fetch fails, I'll paste both files directly.

# Where we are
Phases 5b through 5e all shipped last night:
- 5b: CRM + Watchlist Supabase persistence (crm_added + watchlist tables)
- 5c: ESPN headshot photos for 5,000+ players, hero-size modals
- 5d: Call/Text/Email action links, scout card photos
- 5e: School logos on cards + modals, drawer badges, player comparison mode

Live right now: https://apexver1.github.io/Apex-App/?v=5e4
Login: apex@test.com / ApexTest123

# Environment (Mac)
cd ~/code/apex-intel && source .venv/bin/activate

# What I want to work on
[fill in — or pick from the priority list in the handoff]

Working agreements (critical):
1. Map index.html ONCE at the start.
2. Larger contiguous str_replace blocks.
3. node --check at midpoint AND end.
4. Deliver as index-{phase}.html via present_files.
5. Numbered step-by-step lists.
6. Every URL as a clickable markdown link.
7. For new Supabase tables: CREATE → RLS → policies → GRANT auth → GRANT anon → NOTIFY pgrst.
8. Always pass user_id:UID explicitly in POST bodies.
````

Then drag in your production `~/code/Apex-App/index.html`.
