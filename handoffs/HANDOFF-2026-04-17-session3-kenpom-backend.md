# Handoff — April 17, 2026 (Session 3: KenPom backend ingestion)

## Session result

Session 3 target hit: **365 of 365 D1 schools in Supabase now have KenPom rank, AdjEM, AdjO, AdjD, Tempo, Season, TeamID, and last-sync timestamp.** Backend-only — frontend wiring is Session 4's job per the 14-day plan.

Session ran ~3 hours end-to-end (with a ~30 min security detour for SUPABASE_SERVICE_ROLE_KEY rotation after accidental screenshot exposure). Actual KenPom-specific work was ~90 min once keys were loaded.

---

## What shipped

### 1. Backend repo cleanup (`apex-intel`) — 3 commits pushed to main
- `c24990e` — `chore: ignore .env.backup, *.bak, *.backup, and /data dumps`
- `ff73966` — `feat(apex-scout): Scout 1.2 — switch to Claude tool-use with find_players, get_player_profile, diagnose_team_gaps, get_metric_distribution; ground prompt in actual data inventory (5677 D1 / 1192 portal / 657 HS / 334 schools); defer KenPom + NIL reasoning until those ship`
- `edb30c2` — `feat(scripts): add load_bt_player_stats.py — Bart Torvik 2025-26 advanced stats ingestion with 0.94/0.04 fuzzy-match into school_rosters (22 stat columns)`

### 2. Security incident + remediation
- Accidental screenshot of `.env` exposed SUPABASE_SERVICE_ROLE_KEY
- Old key disabled in Supabase
- New key `apex_backend` (prefix `sb_secret_8lRzu...`) rotated into `.env`

### 3. Schema migration on `schools` — 8 new columns
`kenpom_rank` (integer), `kenpom_adj_em`, `kenpom_adj_o`, `kenpom_adj_d`, `kenpom_tempo` (all numeric(6,2)), `kenpom_season` (text), `kenpom_team_id` (integer), `kenpom_last_sync` (timestamptz).

### 4. `scripts/kenpom_sync.py` shipped in `apex-intel`
171 lines. 3-pass resolver:

1. **Pass 1 — Hand-coded overrides (62 entries):** Exact KenPom TeamName → schools.name map
2. **Pass 2 — Normalize-exact:** St. → State, strip hyphens, collapse whitespace
3. **Pass 3 — Fuzzy fallback (`fuzz.ratio`, 94/4 threshold+margin):** Unused in final run. Uses `fuzz.ratio` not `fuzz.token_set_ratio` — latter caused 6 wrong-row writes mid-session.

Pagination handles Supabase 1000-row API cap. `conference IS NOT NULL` filter used (not `schools.division`, which is unreliable per CLAUDE.md).

### 5. Sync result
```
Matched: 365  (override: 62, normalize-exact: 303, fuzzy: 0)
Unmatched: 0
Update errors: 0
DataThrough: Monday, April 6
```

Mid-session `UPDATE schools SET kenpom_* = NULL` pass on all 365 D1 rows before the final sync, to wipe stale data from earlier imperfect runs.

### 6. Spot-check passed

All verify correct, no wrong-row bugs:
- Oregon: rank 101, AdjEM 7.08 (matches API probe baseline)
- Miami (FL): rank 30 / Miami (OH): rank 90
- Penn State: rank 139 / Pennsylvania: rank 156
- Tennessee: rank 14 / UT Martin: rank 224
- Illinois: rank 5 / UIC: rank 110
- USC: rank 77 / South Carolina Upstate: rank 302

DB-wide: 365 total, 365 synced, 0 unsynced, distinct ranks = 365 (no duplicates).

---

## Mid-session failure modes and recoveries

1. **Fuzzy-match false positives with token_set_ratio.** First 2 runs used `fuzz.token_set_ratio` which treats a short name as a 100% match against any name containing it as a token. Caused "Miami OH" → "Miami" (FL), "Penn" → "Penn State", "Illinois Chicago" → "Illinois", "USC Upstate" → "USC", etc. Caught via post-run distinct-ranks check. **Fix: use fuzz.ratio (character-level) for fallback.**

2. **Stale contamination between iterative runs.** Each sync only UPDATEs matched rows. If run N mismatched school A → row B, then run N+1 correctly matched A → A, row B was left with stale wrong data. **Fix: NULL all kenpom_* on all D1 schools before the final authoritative run.**

---

## Files changed

| File | Status |
|---|---|
| `apex-intel/scripts/kenpom_sync.py` | NEW (171 lines, not yet committed) |
| `apex-intel/.env` | MODIFIED (local only) |
| `apex-intel/.gitignore` | COMMITTED `c24990e` |
| `apex-intel/supabase/functions/apex-scout/index.ts` | COMMITTED `ff73966` |
| `apex-intel/scripts/load_bt_player_stats.py` | COMMITTED `edb30c2` |
| `Apex-App/DATA-STATUS.md` | MODIFIED |
| `Apex-App/BACKLOG.md` | MODIFIED |
| supabase schools table | MODIFIED (schema + data) |

**Key note:** `kenpom_sync.py` was NOT committed to git during the session. Next session opener must commit it.

---

## Still open

### Immediate (handle at start of Session 4)
- Commit `scripts/kenpom_sync.py` to `apex-intel` main
- Verify frontend authenticates against Supabase with rotated key

### Session 4 — KenPom frontend surfacing
- `schoolKenPomMap` in `loadData()` (mirrors `schoolEspnMap` pattern)
- `kenpomFor(schoolName)` helper
- Wire into: Home Next Up, Roster Pulse, recruiting cards, player modal, scout cards

---

## Process notes (carry forward)

- **Standing session opener worked:** The 2-min uncommitted-clutter opener caught the `.env.backup` security risk and cleaned up 4 stranded working-tree items.
- **Greenlight confirmations must block.** User ran Step 1 before confirming three greenlight questions. For future sessions, Claude should not advance without explicit confirmation.
- **Security detour cost matters.** Key rotation was the right call but cost ~30 min due to iterative gotchas (Supabase has two API key tabs, partial-paste issues, etc).
- **Don't run DB queries for `Hawaii` — look for `Hawai'i`** (with the ʻokina). General principle: when a DB probe returns "not found," try Unicode variants before declaring a gap.

---

## Reference constants (unchanged)

- Oregon: UUID `d4dd73c7-2ec6-49bf-b63b-5c4a333a2bf7`, KenPom rank 101, KenPom TeamID 221
- Duke: UUID `279abc4c-77dd-4a3a-8f46-5daa6f9d8da1`
- Supabase project: `midyxvjfoggchbugxzkd`
- KenPom base URL: `https://kenpom.com/api.php`
- KenPom auth: `Authorization: Bearer <KENPOM_API_KEY>`
- KenPom endpoints used: `?endpoint=ratings&y=2026`, `?endpoint=teams&y=2026`
- KenPom API expires: 2027-04-10

---

## Session starter for next chat

Paste this as the first message in Session 4:

```
Loading project context — Apex Intel, Session 4 of 14-day plan (KenPom frontend surfacing).

Please fetch all five:
https://github.com/Apexver1/apex-intel/raw/main/CLAUDE.md
https://github.com/Apexver1/Apex-App/raw/main/PLAN-2026-04-16-to-2026-04-30.md
https://github.com/Apexver1/Apex-App/raw/main/DATA-STATUS.md
https://github.com/Apexver1/Apex-App/raw/main/BACKLOG.md
https://github.com/Apexver1/Apex-App/raw/main/handoffs/HANDOFF-2026-04-17-session3-kenpom-backend.md

Session 4 target (one sentence):
Wire KenPom data (365 schools, 8 kenpom_* cols) into Home Next Up, Roster Pulse, recruiting cards, player modal, scout cards.

Carryover from Session 3 — handle in session opener:
1. Commit scripts/kenpom_sync.py to apex-intel main (exists locally only)
2. Verify frontend authenticates against Supabase with the new key (rotated 2026-04-17)
```
