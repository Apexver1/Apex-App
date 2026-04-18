# Apex Intel — Claude Context File

**Purpose:** Basketball operations intelligence platform. School-agnostic architecture, piloting with Oregon men's basketball.

**Repos:**
- Frontend: [github.com/Apexver1/Apex-App](https://github.com/Apexver1/Apex-App) (private) — `index.html` single-file app, GitHub Pages deploy at [apexver1.github.io/Apex-App/](https://apexver1.github.io/Apex-App/)
- Backend: [github.com/Apexver1/apex-intel](https://github.com/Apexver1/apex-intel) (private) — Python scripts, schema, docs

**Database:** Supabase project `apex-intel-db` (`midyxvjfoggchbugxzkd`)

**Owner workflow:** Mac with Terminal + Safari, Python 3.12.13 venv at `~/code/apex-intel/.venv`. Frontend edits via surgical Python regex patches against `~/code/Apex-App/index.html` (minified, one-file, ~247K bytes). All SQL via Supabase dashboard SQL editor.

---

## Current Phase Status

- ✅ Phase 0 — Architecture interview, school-agnostic design locked
- ✅ Phase 1 — GitHub repo scaffolded, `.env.example` committed
- ✅ Phase 2 — Database schema built (22 tables, executed against live Supabase). `schools` loaded with 1,518 NCAA D1 records.
- ✅ Phase 3 — RLS policies live, auth configured, API accounts created
- ✅ Phase 4 — First real data load: Oregon roster + 5,677 D1 players + 657 HS prospects + 1,192 portal players + school-mapping for 334 programs
- ✅ Phase 5 — Frontend wired to live advanced-stats data (KenPom Sessions 3/4a/4b, Torvik Session 5)
- ✅ Phase 6 — Scout v2 hero + metric tooltips (Session 6 Commits 1 + 2a)
- 🔄 Phase 6.5 — Drawer logo swap + role label (Session 7 Commit 2b, deferred from Session 6 — needs Safari DevTools recon)
- 🔄 Phase 6.5 — Remaining UI polish + Sessions 7-10 per original plan (NIL Budget per-school, Patrons seeding, NIL Vault stub, beta dry run)
- ⬜ Phase 7 — Multi-user auth roles (BACKLOG — post-14-day-plan)

---

## Data Sources

| Source | Auth | Status | Use |
|---|---|---|---|
| BallDontLie | API key | ✅ Active (Phase 4) | Player/team/game data, not yet surfaced |
| CBBD (CollegeBasketballData.com) | API key | ✅ Active (Phase 4) | College-specific stats, not yet surfaced |
| **KenPom** | Bearer token ($95/yr) | ✅ **Wired Sessions 3/4a/4b, tooltips Session 6b** | Team efficiency metrics on Home + recruiting + player modal + Scout; tappable ⓘ popovers |
| **Torvik** | None (JSON endpoint) | ✅ **Wired Session 5; player-level in prospect_stats** | Team metrics on Smart Filter + Scout presets; player-level AdjOE on player-modal stat-box with tooltip |
| 247Sports Composite | Scrape | ✅ Active (Phase 4) | HS + portal rankings |
| ESPN | Keyless | ✅ Verified | Recruits, schedules, photos (5,136) |
| NCAA-API | Keyless | ✅ Verified | Official rosters, scores |

Keys stored in iPad Notes + `~/code/apex-intel/.env` (gitignored). SUPABASE_SERVICE_ROLE_KEY rotated Session 3.

---

## Database — 22 Tables (Supabase Postgres)

### Core entities
- `schools` — 1,518 records, 365 D1 with non-null `conference`. **8 kenpom_* columns (Session 3), 9 torvik_* columns (Session 5).**
- `staff` — coaches and ops personnel per school
- `school_rosters` — 7,580 players, 5,136 with ESPN photos
- `my_roster` — the user's own team roster (78 Oregon players)

### Recruiting / prospects
- `prospect_shortlist` — 100 rows actual (B8, documented as 1,849)
- `prospect_stats` — statistical profiles **including player-level Torvik AdjOE/AdjDE (surfaced in player modal with tooltip as of Session 6)**
- `prospect_scores` — internal evaluation scores
- `prospect_film` — video links
- `prospect_family` — guardian / family / HS coach / AAU contacts

### CRM / workflow
- `crm_contacts`, `crm_log`, `crm_added`, `watchlist`, `contact_reminders`, `team_strategy`

### NIL / scholarships
- `nil_budget`, `nil_budget_requests`, `scholarships`

### System
- `agent_memory`, `audit_log`, `search_history`, `source_tags`

### Design principles
- School-agnostic: every operational table keyed by `school_id`
- External-ID mapping: `ncaa_id` + `espn_id` + `sports247_id` on `schools`
- RLS enforced on every table
- Soft deletes via `deleted_at`
- Every mutation logged to `audit_log`
- Data provenance tracked via `source_tags`

---

## Frontend architecture — `~/code/Apex-App/index.html`

Single-file minified HTML+JS+CSS, ~247K bytes (grew from ~236K after Session 6 tooltip infrastructure). No framework.

### Key global maps populated in `loadData()`

| Global map | Key | Value | Helper |
|---|---|---|---|
| `schoolEspnMap` | lowercase name | espn_id | `window.schoolLogo` |
| `schoolKenPomMap` | lowercase name | {rank, adjEm, adjO, adjD, tempo, season, name} | `window.kenpomFor(name)` |
| `schoolKenPomById` | UUID | same | `window.kenpomById(uuid)` |
| `schoolTorvikMap` | lowercase name | {rank, adjOe, adjDe, barthag, season, name} | `window.torvikFor(name)` |
| `schoolTorvikById` | UUID | same | `window.torvikById(uuid)` |

### Tooltip infrastructure — `window.apxMetricInfo` + `window.apxShowInfo` (NEW Session 6b)

Six metric definitions live in `window.apxMetricInfo` (object with keys: `team_strength`, `team_offense`, `team_defense`, `team_pace`, `team_winprob`, `player_offense`). Each has `ttl` (plain-English title), `acr` (monospace acronym), `body` (1-2 sentence explanation).

`window.apxShowInfo(ev, key, extraVal)` is the popover renderer — creates a floating `.apx-pop` div, positions it near the trigger, includes a close button, and sets up tap-outside-to-close.

Usage: any clickable element gets `onclick="window.apxShowInfo(event,'team_strength','Wake Forest: +10.58 · #80')"` and automatically gets the full popover.

### Frontend helpers

- `window.kenpomChipHTML(p, isPr)` — renders a tappable KenPom chip with trailing ⓘ icon. One function call propagates to Home Next Up, Recruiting (both renderers), Player modal, Scout results (both modes).
- `window.schoolLogo(name, size)` — available at line 1086, renders the school logo. Will be used in Session 7 Commit 2b for the drawer logo swap.

### Scout hero (Session 6 Commit 1)

`window.renderScout(c)` rewritten. Hero block: serif "Build your roster." + dynamic subhead + pill input + inline mic icon + 2×2 preset grid per section (smart + static) + below-fold instructions. Results rendering (LLM mode + rules mode) unchanged.

### Filter state

Filter state globals live as a `var` chain on line ~939. Key filter state vars: `fSrc`, `fSrcCy`, `fPos`, `fStatus`, `fTier`, `fTopN`, `fTTier` (Session 5c), `fPPG`/`fRPG`/`fAPG`/`fSPG`/`fBPG`/`f3P`, `fMaxNil`, `fHt`, `fComp`.

### `aiMode` state

Used by Scout's `aiRun()` to branch between rules and LLM engines. Currently **never declared with `var`** — defaults to `undefined`, which branches to rules mode internally. Attempted in Session 6 Commit 2c to declare `var aiMode="llm"` and switch default to LLM — caused JS parse error (the adjacent `setAiMode=function` declaration is not itself prefixed with `var` or `window.`, so injecting a `var` broke statement context). Reverted. **Pending: find correct context for this declaration.** See BACKLOG → "Scout engine auto-routing (LLM default)".

---

## Working Agreements with Claude

1. **Always write full replacement code blocks.** Never say "find the function that…" — always paste the complete block and state the exact file path + which tag/line to paste above or below.

2. **Never ask the user to find-and-replace inside a file.** Always provide the complete updated file as one copy-paste-ready block. Workflow: delete all existing content, paste the new version. Exception: for very large files like `index.html`, use surgical Python patches with match-count asserts.

3. **Always include every URL as a clickable markdown link.** GitHub editor links, Supabase dashboard links, docs links — formatted as `[label](https://...)`, never bare text.

4. **Always spell out Terminal/SQL commands in full.** No abbreviations.

5. **Start every session** by reading this file and the latest `/handoffs/HANDOFF-*.md` from Project Knowledge. Repos are private so raw GitHub fetches will fail — rely on Project Knowledge uploads.

6. **End every session** by generating four docs: updated `DATA-STATUS.md`, updated `BACKLOG.md`, updated `CLAUDE.md` (this file) if architecture/phase changed, and new `HANDOFF-YYYY-MM-DD-session{N}-{slug}.md` in `/handoffs/`. **Remind user to re-upload the three modified docs to Project Knowledge** so next session boots clean.

7. **Reset sessions** at each major milestone — don't run marathon chats.

8. **Heredoc discipline for long Python scripts:** use `cat > path/to/file.py <<'APEX_EOF'` pattern; closing `APEX_EOF` must be on its own line with no leading whitespace. Multi-line Python string concatenation is brittle inside heredocs — prefer single-line string literals (Session 5b lesson).

9. **Surgical-patch discipline for index.html:** safety backup → grep recon for exact byte-form of anchor strings → Python regex patch with match-count asserts → sanity grep → if needed, re-run with corrected anchors → Safari `file://` smoke test → commit & push.

10. **Byte-extraction for anchors (NEW Session 6 — critical lesson).** For any patch anchor that contains nested quotes (single inside double, or vice versa), escape sequences, unicode literals (`\u00b7`), or any JS-in-a-Python-string gnarliness, **do NOT type the anchor as a Python string literal**. Instead:

    a. Use `python3` inline to locate the target in the file via `.find()`
    b. Extract the substring by byte position
    c. Save to `/tmp/anchor_name.txt`
    d. In the actual patch script, `pathlib.Path("/tmp/anchor_name.txt").read_text()` to load the anchor
    e. The anchor is now guaranteed to match the real file bytes — no escape-hell

    This pattern burned Session 5 once (fTier colon/equals) and Session 6 **four times** (chip return, pulse label, stat-box, valline). Always faster to do byte extraction upfront than debug a count=0 mismatch.

    Additionally: when a diagnostic is needed to understand *why* a byte-literal anchor didn't match, run `repr()` on the real file substring and `repr()` on the attempted anchor, compare character-by-character in Python to find the first mismatch. Takes 30 seconds, prevents an hour of guesswork.

11. **DevTools recon for dynamic-render targets (NEW Session 6 — for Session 7 drawer work).** `grep` cannot find strings that are built via JS concatenation from variables at runtime. If a user-visible string like "Dana Altman" or "Logged in as" doesn't grep, the render is dynamic — Safari DevTools → Inspect Element → examine the actual DOM → trace parent classes or React-like data attributes back to the render function. Four rounds of `grep` recon in Session 6 couldn't find the drawer render function for these reasons. Don't repeat.

---

## Constraints

- Mac + Terminal primary workflow
- All frontend edits via local `~/code/Apex-App/`, pushed to GitHub main, auto-deploys via GitHub Pages
- All SQL via Supabase dashboard SQL editor — editor only runs the first statement in multi-statement queries; run DDL one statement at a time
- Supabase REST API caps at 1000 rows per call — always paginate `select()` over full tables
- Fuzzy school name matching: `rapidfuzz.fuzz.ratio` at 94 threshold + 4 runner-up margin. **Do NOT switch to `token_set_ratio`** (S3-minor-2)
- Safari aggressively caches on GitHub Pages — `Cmd+Shift+R` hard-reload or `Safari → History → Clear History → Last hour`

---

## Reference constants (stable across sessions)

- **Oregon:** UUID `d4dd73c7-2ec6-49bf-b63b-5c4a333a2bf7`, ESPN id 2483, CBBD id 223, BallDontLie 123, sports247_id 24090, NCAA slug `oregon`, KenPom #101 / AdjEM +7.08, Torvik #91 / Barthag 0.7068
- **Duke:** UUID `279abc4c-77dd-4a3a-8f46-5daa6f9d8da1`, KenPom #3 / AdjEM +37.37, Torvik #3 / Barthag 0.9768
- **Supabase project:** `midyxvjfoggchbugxzkd` — project URL [https://midyxvjfoggchbugxzkd.supabase.co](https://midyxvjfoggchbugxzkd.supabase.co)
- **Publishable key** (shipped in index.html, safe to expose): `sb_publishable_MizrCkmoqSNlQAF3ty_FUA_WkATU4Ai`
- **Test login:** `apex@test.com` / `ApexTest123`
- **Auto-login:** wired — splash lands directly on Home for dev convenience
- **Latest commits (end Session 6):**
  - `7ee3241` — feat(tooltips): Session 6b (Commit 2a) — plain-English metric translation
  - `d30a3ca` — feat(scout): Session 6a — Scout v2 hero redesign
  - `5cad16a` — docs(session-5): Torvik end-to-end shipped + Sammy pitch backlog
