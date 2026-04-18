# Apex Intel — Claude Context File

**Purpose:** Basketball operations intelligence platform. School-agnostic architecture, piloting with Oregon men's basketball.

**Repos:**
- Frontend: [github.com/Apexver1/Apex-App](https://github.com/Apexver1/Apex-App) (private) — `index.html` single-file app, GitHub Pages deploy at [apexver1.github.io/Apex-App/](https://apexver1.github.io/Apex-App/)
- Backend: [github.com/Apexver1/apex-intel](https://github.com/Apexver1/apex-intel) (private) — Python scripts, schema, docs

**Database:** Supabase project `apex-intel-db` (`midyxvjfoggchbugxzkd`)

**Owner workflow:** Mac with Terminal + Safari, Python 3.12.13 venv at `~/code/apex-intel/.venv`. Frontend edits via surgical Python regex patches against `~/code/Apex-App/index.html` (minified, one-file, ~236K bytes). All SQL via Supabase dashboard SQL editor.

---

## Current Phase Status

- ✅ Phase 0 — Architecture interview, school-agnostic design locked
- ✅ Phase 1 — GitHub repo scaffolded, `.env.example` committed
- ✅ Phase 2 — Database schema built (22 tables, executed against live Supabase). `schools` loaded with 1,518 NCAA D1 records.
- ✅ Phase 3 — RLS policies live, auth configured, API accounts created
- ✅ Phase 4 — First real data load: Oregon roster + 5,677 D1 players + 657 HS prospects + 1,192 portal players + school-mapping for 334 programs
- ✅ Phase 5 — Frontend wired to live advanced-stats data (KenPom Sessions 3/4a/4b, Torvik Session 5)
- 🔄 Phase 6 — Continued UI polish + remaining surfacing (Scout polish Session 6, Apex Picks preset strip Session 7, NIL per-school Session 8, Patrons + NIL Vault stub Session 9, beta-2 dress rehearsal Session 10)
- ⬜ Phase 7 — Multi-user auth roles (BACKLOG — post-14-day-plan)

---

## Data Sources

| Source | Auth | Status | Use |
|---|---|---|---|
| BallDontLie | API key | ✅ Active (Phase 4) | Player/team/game data, not yet surfaced |
| CBBD (CollegeBasketballData.com) | API key | ✅ Active (Phase 4) | College-specific stats, not yet surfaced |
| **KenPom** | Bearer token ($95/yr) | ✅ **Wired Session 3 backend + 4a/4b frontend** | Team efficiency metrics on Home + recruiting cards + player modal; Scout deferred to Session 6 |
| **Torvik** | None (JSON endpoint) | ✅ **Wired Session 5 end-to-end** | Team efficiency metrics on Smart Filter; Scout deferred to Session 6; eFG/eFG-D/TOV columns NULL per S5-minor-1 |
| 247Sports Composite | Scrape | ✅ Active (Phase 4) | HS + portal rankings |
| ESPN | Keyless | ✅ Verified | Recruits, schedules, photos (5,136) |
| NCAA-API | Keyless | ✅ Verified | Official rosters, scores |

Keys stored in iPad Notes + `~/code/apex-intel/.env` (gitignored). SUPABASE_SERVICE_ROLE_KEY rotated Session 3 after accidental screenshot exposure; current key is `apex_backend` (`sb_secret_*` format).

---

## Database — 22 Tables (Supabase Postgres)

### Core entities
- `schools` — 1,518 records (NCAA universe), 365 D1 with non-null `conference`. Core columns `id (uuid)`, `ncaa_id`, `espn_id`, `sports247_id` for cross-source joins. **8 kenpom_* columns (Session 3), 9 torvik_* columns (Session 5).**
- `staff` — coaches and ops personnel per school
- `school_rosters` — 7,580 players (Sessions 4a+4b loaded), official rosters by school/season, 5,136 with ESPN photos
- `my_roster` — the user's own team roster (78 Oregon players)

### Recruiting / prospects
- `prospect_shortlist` — currently 100 rows (see B8 — documented as 1,849 but actual is 100, all school_id=Oregon)
- `prospect_stats` — statistical profiles
- `prospect_scores` — internal evaluation scores
- `prospect_film` — video links (YouTube, Hudl, Synergy, InStat)
- `prospect_family` — guardian / family / HS coach / AAU contacts

### CRM / workflow
- `crm_contacts` — all contactable people across prospects
- `crm_log` — every touchpoint
- `crm_added`, `watchlist` — user-action tracking
- `contact_reminders` — follow-up queue
- `team_strategy` — program-level recruiting strategy notes

### NIL / scholarships
- `nil_budget` — NIL dollars allocated (Oregon only until Session 8)
- `nil_budget_requests` — pending requests
- `scholarships` — scholarship count / allocation tracking

### System
- `agent_memory` — persistent memory for AI agents (used by Apex Scout)
- `audit_log` — every write operation logged
- `search_history` — user query history
- `source_tags` — tagging layer for data provenance

### Design principles
- School-agnostic: every operational table keyed by `school_id`, never hardcoded to Oregon
- External-ID mapping: `ncaa_id` + `espn_id` + `sports247_id` on `schools` enable joining across all data sources
- RLS enforced on every table
- Soft deletes via `deleted_at` timestamp
- Every mutation logged to `audit_log`
- Data provenance tracked via `source_tags`

---

## Frontend architecture — `~/code/Apex-App/index.html`

Single-file minified HTML+JS+CSS, ~236K bytes. Built-in state management, no framework. Key global maps populated in `loadData()`:

| Global map | Key | Value | Helper |
|---|---|---|---|
| `schoolEspnMap` | lowercase name | espn_id | `window.schoolLogo` |
| `schoolKenPomMap` | lowercase name | {rank, adjEm, adjO, adjD, tempo, season, name} | `window.kenpomFor(name)` |
| `schoolKenPomById` | UUID | same | `window.kenpomById(uuid)` |
| `schoolTorvikMap` | lowercase name | {rank, adjOe, adjDe, barthag, season, name} | `window.torvikFor(name)` |
| `schoolTorvikById` | UUID | same | `window.torvikById(uuid)` |

Additional frontend helper: `window.kenpomChipHTML(p, isPr)` — renders KenPom rank chip for prospect or roster player. No Torvik-chip helper yet (Smart Filter uses the raw helpers directly).

Filter state globals live as a `var` chain on line ~939. Key filter state vars: `fSrc`, `fSrcCy`, `fPos`, `fStatus`, `fTier` (priority tier), `fTopN` (On3 national rank), `fTTier` (Torvik rank tier, Session 5c), `fPPG`/`fRPG`/`fAPG`/`fSPG`/`fBPG`/`f3P`, `fMaxNil`, `fHt`, `fComp`.

Filter-apply logic in `portal.filter()` at ~line 2855. UI render in `renderFilter()` at ~line 2854. State setter: `window.uf(k,v)` at ~line 2207 (key-based dispatch).

---

## Working Agreements with Claude

1. **Always write full replacement code blocks.** Never say "find the function that…" — always paste the complete block and state the exact file path + which tag/line to paste above or below.
2. **Never ask the user to find-and-replace inside a file.** Always provide the complete updated file as one copy-paste-ready block. Workflow: delete all existing content, paste the new version. Exception: for very large files like `index.html`, use surgical Python patches with match-count asserts (pattern established Session 4a+, refined Session 4b+5b+5c).
3. **Always include every URL as a clickable markdown link.** GitHub editor links, Supabase dashboard links, docs links — formatted as `[label](https://...)`, never bare text.
4. **Always spell out Terminal/SQL commands in full.** No abbreviations.
5. **Start every session** by reading this file and the latest `/handoffs/HANDOFF-*.md` from Project Knowledge. Repos are private so raw GitHub fetches will fail — rely on Project Knowledge uploads.
6. **End every session** by generating four docs: updated `DATA-STATUS.md`, updated `BACKLOG.md`, updated `CLAUDE.md` (this file) if architecture/phase changed, and new `HANDOFF-YYYY-MM-DD-session{N}-{slug}.md` in `/handoffs/`. **Remind user to re-upload the three modified docs to Project Knowledge** so next session boots clean.
7. **Reset sessions** at each major milestone — don't run marathon chats.
8. **Heredoc discipline for long Python scripts:** use `cat > path/to/file.py <<'APEX_EOF'` pattern; closing `APEX_EOF` must be on its own line with no leading whitespace. Multi-line Python string concatenation is brittle inside heredocs — prefer single-line string literals (Session 5b lesson).
9. **Surgical-patch discipline for index.html:** safety backup → grep recon for exact byte-form of anchor strings → Python regex patch with match-count asserts → sanity grep → if needed, re-run with corrected anchors → Safari `file://` smoke test → commit & push.

---

## Constraints

- Mac + Terminal primary workflow (shifted from iPad Safari during Phase 4)
- All frontend edits via local `~/code/Apex-App/`, pushed to GitHub main, auto-deploys via GitHub Pages
- All SQL via Supabase dashboard SQL editor — editor only runs the first statement in multi-statement queries; run DDL one statement at a time (or use single `ALTER TABLE ... ADD COLUMN IF NOT EXISTS ..., ADD COLUMN ...` for multi-column in one statement)
- Supabase REST API caps at 1000 rows per call — always paginate `select()` over full tables
- Fuzzy school name matching: `rapidfuzz.fuzz.ratio` at 94 threshold + 4 runner-up margin. **Do NOT switch to `token_set_ratio`** — causes wrong-row writes on short names like "Miami OH" (S3-minor-2)
- Safari aggressively caches on GitHub Pages — `Cmd+Shift+R` hard-reload or clear history (last hour) when live changes don't appear

---

## Reference constants (stable across sessions)

- **Oregon:** UUID `d4dd73c7-2ec6-49bf-b63b-5c4a333a2bf7`, ESPN id 2483, CBBD id 223, BallDontLie 123, sports247_id 24090, NCAA slug `oregon`, KenPom #101 / AdjEM +7.08, Torvik #91 / Barthag 0.7068
- **Duke:** UUID `279abc4c-77dd-4a3a-8f46-5daa6f9d8da1`, KenPom #3 / AdjEM +37.37, Torvik #3 / Barthag 0.9768
- **Supabase project:** `midyxvjfoggchbugxzkd` — project URL [https://midyxvjfoggchbugxzkd.supabase.co](https://midyxvjfoggchbugxzkd.supabase.co)
- **Publishable key** (shipped in index.html, safe to expose): `sb_publishable_MizrCkmoqSNlQAF3ty_FUA_WkATU4Ai`
- **Test login:** `apex@test.com` / `ApexTest123`
- **Auto-login:** wired — splash lands directly on Home for dev convenience
