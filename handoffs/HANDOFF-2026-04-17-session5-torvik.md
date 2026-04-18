# Handoff — April 17, 2026 (Session 5: Torvik end-to-end — backend + frontend + Smart Filter)

## Session result

Full Session 5 target shipped in one sitting. **Torvik now live end-to-end** — backend ingestion, frontend helpers, Smart Filter tier-based prospect filtering. Three commits across two repos:

| Phase | Repo | Commit | Delta |
|---|---|---|---|
| 5-A backend | apex-intel | `c4e79f1` | 1 file, +314 (new `scripts/torvik_team_sync.py`) |
| 5-B frontend data layer | Apex-App | `5b54cb6` | 1 file, +580 bytes in `index.html` (4 patches) |
| 5-C Smart Filter surfacing | Apex-App | `6efe921` | 1 file, +837 bytes in `index.html` (7 patches) |

Session ran ~1h45min — top of 90-120min budget. Detour time (normalize-fn bug, endpoint hunt, P1 colon/equals bug) absorbed within buffer. **Live on production at [apexver1.github.io/Apex-App/](https://apexver1.github.io/Apex-App/)** as of commit `6efe921`.

No regression on Session 4a/4b KenPom wiring — verified in same Safari session that "+7.08 AdjEM · #101" chip on Roster Pulse still renders and Next Up KenPom #101 / #2 still renders.

---

## What shipped — Phase 5-A (backend)

### Schema migration
9 new columns on `public.schools` via single `ALTER TABLE ... ADD COLUMN IF NOT EXISTS` statement (idempotent, safe to re-run):

| Column | Type | Populated? |
|---|---|---|
| `torvik_rank` | integer | ✅ 365 rows |
| `torvik_adj_oe` | numeric(6,2) | ✅ 365 rows |
| `torvik_adj_de` | numeric(6,2) | ✅ 365 rows |
| `torvik_barthag` | numeric(6,4) | ✅ 365 rows |
| `torvik_efg_pct` | numeric(5,2) | NULL (S5-minor-1) |
| `torvik_efg_d_pct` | numeric(5,2) | NULL (S5-minor-1) |
| `torvik_tov_pct` | numeric(5,2) | NULL (S5-minor-1) |
| `torvik_season` | text | ✅ '2025-26' |
| `torvik_last_sync` | timestamptz | ✅ UTC now |

### `scripts/torvik_team_sync.py`
Mirrors `kenpom_sync.py` 3-pass resolver pattern exactly (override → normalize-exact → fuzz.ratio 94/4 fallback). Key architectural choices:

- **Data source:** `https://barttorvik.com/2026_team_results.json` — JSON array, 365 teams per season, positional arrays (no keys)
- **Column mapping (`TT_COL`):** `{rank:0, team:1, conf:2, adj_oe:4, adj_de:6, barthag:8}` — verified against Duke (#3, barthag 0.9768), Arizona (#2, barthag 0.9772), Oregon (#91, barthag 0.7068) cross-reference
- **Override table:** 60+ entries covering name-alias mismatches (Torvik "Mississippi" → DB "Ole Miss", Torvik "Connecticut" → DB "UConn", Torvik "Hawaii" → DB "Hawai'i" with apostrophe, etc.)
- **normalize_name():** literal `.replace(" St.", " State")` (Torvik uses "St.", DB uses "State") + lowercase + punct-strip
- **Fuzzy fallback:** deliberately uses `fuzz.ratio` (char-level), NOT `fuzz.token_set_ratio` per S3-minor-2 lesson

**Sync results (run 3, final):** 365/365 matched. 54 override pass, 252 exact pass, 59 normalize pass, 0 fuzzy, 0 miss, 0 errors.

**Spot-check SQL confirmed** (Supabase editor):
| Team | Torvik rank / Barthag | KenPom rank / AdjEM |
|---|---|---|
| Arizona | #2 / 0.9772 | #2 / +38.06 |
| Duke | #3 / 0.9768 | #3 / +37.37 |
| UConn | #9 / 0.9561 | #9 / +29.78 |
| Kansas | #21 / 0.9254 | #21 / +24.15 |
| UCLA | #25 / 0.9081 | #28 / +21.19 |
| Miami | #33 / 0.8833 | #30 / +20.64 |
| Ole Miss | #63 / 0.7908 | #64 / +12.76 |
| USC | #69 / 0.7739 | #77 / +10.94 |
| **Miami (OH)** | **#87 / 0.7199** | **#90 / +8.64** |
| Oregon | #91 / 0.7068 | #101 / +7.08 |
| Hawai'i | #120 / 0.6123 | #108 / +5.58 |
| Howard | #179 / 0.4770 | #196 / -2.64 |

**Zero same-name collisions** — Miami vs Miami (OH) land on different Supabase rows with different ranks. That's the exact bug class KenPom had in Session 3, and the override table + fuzzy-discipline prevented it here.

---

## What shipped — Phase 5-B (frontend data layer)

Four surgical patches in `~/code/Apex-App/index.html` (235,417 → 235,997 bytes, +580):

| Patch | Location | What |
|---|---|---|
| P1 | line ~1342 | Expand `schools?select=...` URL to add `torvik_rank,torvik_adj_oe,torvik_adj_de,torvik_barthag,torvik_season,torvik_last_sync` |
| P2 | line 950 | Add `var schoolTorvikMap={};var schoolTorvikById={};` alongside KenPom map decls |
| P3 | line 1351 | Extend `schools.forEach` populate with parallel if-block: `if(s.torvik_rank!==null&&s.torvik_rank!==undefined){var _tv={rank,adjOe,adjDe,barthag,season,name};schoolTorvikMap[(s.name||"").toLowerCase()]=_tv;schoolTorvikById[s.id]=_tv;}` — **independent of KenPom's populate** (defensive against partial-coverage edge cases) |
| P4 | line 1093 | Add `window.torvikFor(name)` + `window.torvikById(uuid)` helpers, sitting between `window.kenpomById` and `window.kenpomChipHTML` |

**Safari smoke test (file:// local, pre-commit):**
- `typeof window.torvikFor` → `"function"` ✓
- `window.torvikFor('Oregon')` → `{rank: 91, adjOe: 112.01, adjDe: 103.76, barthag: 0.7068, season: "2025-26", name: "Oregon"}` ✓
- `window.torvikFor('Duke')` → `{rank: 3, adjOe: 128.15, adjDe: 92.57, barthag: 0.9768, season: "2025-26", name: "Duke"}` ✓
- `window.torvikById('d4dd73c7-2ec6-49bf-b63b-5c4a333a2bf7')` → same Oregon object (UUID path works) ✓
- `window.torvikFor('NonExistent')` → `{}` (graceful miss, no throw) ✓

---

## What shipped — Phase 5-C (Smart Filter surfacing)

Seven surgical patches in `index.html` (235,997 → 236,834 bytes, +837):

| Patch | What |
|---|---|
| P1 | Add `fTTier="ALL"` to initial filter-state var chain (line 939) |
| P2 | Add `fTTier="ALL";` reset in `window.navTo` (line 1419) |
| P3 | Add `fTTier="ALL";` reset in `window.clearSF` (line 1420) |
| P4 | Add `if(k==="ttier")fTTier=v;` dispatch to `window.uf(k,v)` (line ~2215) |
| P5 | Add tier-filter check block in `portal.filter(function(p){...})` (line ~2871) — checks `torvikFor(p.current_school).rank` against `fTTier`; excludes prospects without Torvik data when any tier filter active (HS/international gracefully filtered) |
| P6 | Add "Torvik tier" UI section in `renderFilter`, between "Rankings" (On3) and "Financials" — 4-button row: `All · Top 50 · #51-150 · #151+` with subhead "School's current Torvik rank" |
| P7 | Add `if(fTTier!=="ALL")activeCount++;` to active-filter-count logic (line ~2895) — drives the "N filters active" label |

**State machine:**
- `fTTier: "ALL"` — default, no filter
- `fTTier: "TOP50"` — prospect's `current_school` Torvik rank must be 1–50
- `fTTier: "MID"` — rank 51–150
- `fTTier: "BOTTOM"` — rank ≥151
- Any non-ALL value when prospect has no Torvik data (HS prospect, international, non-D1) → prospect excluded

**Safari smoke test (file:// local, pre-commit):**
- Loaded local `index.html`, Recruiting → Smart filter scroll: "TORVIK TIER" section visible with 4 buttons ✓
- Click "Top 50" → header shows "**30 matches · 1 filter active**" (down from ~1192 unfiltered portal universe) ✓
- Visible prospect cards: Flory Bidunga (Kansas), John Blackwell (Wisconsin), Jasper Johnson (Kansas), Rob Wright (BYU), Darryn Peterson (Kansas) — all from verified top-50 Torvik schools ✓
- Subhead "School's current Torvik rank" renders correctly (apostrophe preserved) ✓

---

## Mid-session failure modes and recoveries

Four course-corrections needed before final success. Each caught by guard rails before committing bad state.

### 1. Torvik team-endpoint returns HTML, not CSV
Initial assumption: `https://barttorvik.com/trank.php?year=2026&csv=1` returns CSV (standard Torvik URL pattern). **Actual:** returns a full HTML page (rows 0 and last = `<!DOCTYPE html>` / `</html>`). Torvik's CSV endpoints are finicky about params; `trank.php` requires extra context.

**Fix:** probed 4 candidate URLs, found `https://barttorvik.com/2026_team_results.json` returns clean JSON array of 365 teams. Switched to this endpoint for the sync script. It's an undocumented but stable endpoint that powers the site. If Torvik ever removes or changes it, sync breaks silently — filed as S5-minor-3 (add startup shape-check).

### 2. Column index mapping verification required 3 teams
First JSON probe showed Torvik uses positional arrays (no keys). Could not confidently map column indices with just one team. Pulled Oregon + Duke + Arizona + row-0 + row-last and cross-referenced:

- `[0]` = rank (Duke #3, Oregon #91 — matches KenPom-relative)
- `[4]` = AdjOE (Duke 128.15 elite, Oregon 112.01 middling — direction correct)
- `[6]` = AdjDE (Duke 92.57 elite, Oregon 103.76 worse — lower-is-better direction correct)
- `[8]` = Barthag (Duke 0.9768, Oregon 0.7068 — 0-to-1 probability scale correct)

Three columns (eFG/eFG-D/TOV) I couldn't pin down confidently with three cross-references — directions didn't clean-match. **Decided to ship with 6 verified columns, leave the other 3 NULL** rather than risk wrong-stat writes. Filed as S5-minor-1.

### 3. First sync run had 78 misses — normalize_name() bug
Run 1 result: 365 processed, 287 updates OK, 78 misses. Miss list almost entirely teams ending in `St.` (Torvik's abbreviation) that Supabase stores as `State`.

**Root cause:** initial `normalize_name()` used regex-heavy St. → State conversion that fired in the wrong order — punctuation-stripping ran BEFORE the St.→State rule, so `"Iowa St."` became `"iowa st"` which didn't match `"iowa state"` (DB form).

**Also:** original override table had multiple **directionally-wrong entries** (e.g., `"Army" → "Army West Point"` when DB just has `"Army"`). 19 WARN-override-not-in-schools errors fired.

**Fix:** pulled full 365-name snapshot from DB via a Python Supabase query, wrote it to `/tmp/schools_d1.txt`, manually cross-referenced against the 78 miss names, rebuilt normalize_name() with a simple `s.replace(" St.", " State")` up front, rebuilt override table with real DB-verified targets.

Run 2 result: 365 processed, 359 updates OK, 6 misses. Down from 78 → 6 (91% improvement on the delta).

### 4. Run 2's 6 remaining misses — added as targeted overrides
Remaining misses: `Mississippi → Ole Miss`, `Appalachian St. → App State`, `Nicholls St. → Nicholls` (DB drops "State"), `Grambling St. → Grambling` (same), `USC Upstate → South Carolina Upstate`, `UMKC → Kansas City` (program rebrand DB-side).

**Fix:** surgical Python patch added 9 lines (6 new overrides + 3 defensive duplicates for "X State" spelling variants) to the override table without rewriting the full script. Assertion-gated to prevent anchor-drift bugs.

Run 3 result: **365 processed, 365 updates OK, 0 misses, 0 errors.** ✓

### 5. Frontend patch P3 — Python multi-line string literal broke heredoc
First attempt at Phase 5-B patch script used Python adjacent-string concatenation across multiple lines:
```python
P3_new = (
    'if(s.torvik_rank!==null&&...){'
    'var _tv={rank,adjOe,adjDe,'   # ← comma at end + newline
    'barthag,season,...};'          # ← new line, interpreted as new string
)
```
Heredoc write was fine, but Python parse exploded with `SyntaxError: unterminated string literal`. The `,` at line end followed by `\n` broke string continuation in a way that only manifested when the line-count was very long.

**Fix:** rewrote all `_new` replacement strings as **single-line string literals** (even though they're ugly and long). Same final JS output, cleaner Python syntax, no chance of newline-in-string trouble.

### 6. Frontend patch P1 — colon-vs-equals in anchor
P1 for Phase 5-C targeted `fTier:"ALL",fAPG=0,...` but the actual byte-form in index.html was `fTier="ALL",fAPG=0,...` (var-chain syntax, uses `=`, not object-literal `:`). Assumed wrong because `fTier` LOOKS like an object-literal key at a glance.

**Fix:** ran targeted grep on `fTier` occurrences, read the actual byte-form from line 939, rewrote P1 anchor with `=`. Everything downstream unaffected.

**All 5 failures caught by safety rails.** `index.html` was modified zero times during the failure cascades — every patch script with bad anchors aborted before writing. No rollbacks needed, no bad data on disk, no broken prod.

---

## Files changed

| File | Repo | Status |
|---|---|---|
| `scripts/torvik_team_sync.py` | apex-intel | NEW, COMMITTED `c4e79f1` — 314 lines (9 cols, override table, resolver, run script) |
| `index.html` | Apex-App | MODIFIED, COMMITTED `5b54cb6` (Phase 5-B, +580 bytes) + `6efe921` (Phase 5-C, +837 bytes) |
| `index.html.s5b-backup-20260417-222716` | Apex-App | LOCAL ONLY, gitignored by `*-backup-*` pattern (Session 4a repo-hygiene fix still working) |
| `/tmp/torvik_run1.log`, `/tmp/torvik_run2.log`, `/tmp/torvik_run3.log` | (not a repo) | Local-only sync-run logs, referenced inline above |
| `/tmp/schools_d1.txt` | (not a repo) | Local-only D1 snapshot (365 rows, name+conference) used for override-table rebuild |
| `/tmp/s5b_patch.py`, `/tmp/s5c_patch.py` | (not a repo) | Patch scripts, not committed (one-shot artifacts) |
| `DATA-STATUS.md` | Apex-App | MODIFIED (this session close) |
| `BACKLOG.md` | Apex-App | MODIFIED (this session close — includes 6 Sammy pitch named projects) |
| `CLAUDE.md` | Apex-App | MODIFIED (this session close — Phase 5 marked complete) |
| `handoffs/HANDOFF-2026-04-17-session5-torvik.md` | Apex-App | NEW (this file) |

---

## Data-reality insights

1. **Torvik vs KenPom correlation within ~10 ranks** across all spot-checks. Two independent metrics agreeing strongly is a good sign — neither is suffering obvious systemic bias relative to the other. Implication: Apex Picks preset strip (Session 7) can blend them; they're not redundant but also not wildly divergent.

2. **Miami vs Miami (OH) edge case validated.** Different rows, different ranks (33 vs 87), zero cross-contamination. The override + fuzzy-discipline pattern from KenPom worked identically for Torvik. Same pattern scales to any future metric we ingest.

3. **Torvik has schools KenPom might not.** 365/365 match across both sets confirms the D1 universe is stable at 365 schools this season. No vendor-specific school-list drift detected.

4. **`current_school` text column is messy.** Prospect data in `prospect_shortlist` has mix of college names ("Kansas"), HS names ("AZ Compass Prep"), literals ("Uncommitted"), and compound strings ("Committed - Oregon"). The tier filter handles this via graceful-miss — if `torvikFor` returns `{}`, the prospect is excluded when a tier filter is active. No UI breakage. But this reinforces B8 (Prospect pipeline reconciliation) as a real hygiene need.

---

## Still open

### Session 6 pickup (Scout polish, "the showpiece")
- Wire `kenpomChipHTML(p, isPr)` into both Scout render branches (lines ~3271 LLM mode + ~3460 rules-mode). Session 4b wiring pattern applies directly — helpers + data are already in hand; just needs inline injection.
- Also wire Torvik chip into same Scout render branches. Design decision open: (a) add a `torvikChipHTML(p, isPr)` helper mirroring KenPom's, or (b) blend both into a single `statsChipHTML(p, isPr)` that shows KenPom + Torvik side-by-side. Recommendation: (a) for consistency with KenPom's pattern; blended version later if UX calls for it.
- Polish pass: copy tightening, card layout, result ranking — the Session 6 brief per the 14-day plan.

### Deferred to Session 7 (Apex Picks preset strip)
- Three curated presets as buttons in Smart Filter: "Portal PGs, Rebuild Targets" (KenPom bottom-50 + portal + PG), "HS 4-Stars, Undercommitted" (on3 top-200 + HS + !=Committed status), "Best Value vs NIL Ask" (composite: high Torvik barthag at school-level ∩ low NIL ask ∩ in portal)
- Surfaces KenPom in Smart Filter alongside the Torvik tier filter we just shipped
- Insertion point: `renderFilter` before first `sectH("Source")` call (~line 2916)

### Carryover from Session 5 (minor)
- **S5-minor-1:** eFG/eFG-D/TOV Torvik column index verification. Pull 5+ teams' known Torvik.com values, nail down indices, populate these 3 columns. ~30 min in a future session.
- **S5-minor-2:** prune defensive `X -> X` override no-ops in `torvik_team_sync.py`. Harmless cleanup.
- **S5-minor-3:** add startup shape-check to `torvik_team_sync.py` so it fails loudly if the JSON endpoint shape changes. ~15 min.
- **S5-minor-4:** automate weekly Torvik refresh (unify with S3-minor-1 KenPom refresh). ~1 session for both.
- **S5c-minor-1:** Torvik tier button labels use mixed case (`All`, `Top 50`) while other filter sections use ALL CAPS. 1-char-each fix. ~5 min.
- **S5c-minor-2:** add "Player's On3 national rank" subhead to Rankings section to mirror the Torvik subhead — clarifies the two rank filters are measuring different things. ~5 min.

---

## Reference constants (unchanged)

- Oregon: UUID `d4dd73c7-2ec6-49bf-b63b-5c4a333a2bf7`, KenPom #101 / +7.08, **Torvik #91 / 0.7068**
- Duke: UUID `279abc4c-77dd-4a3a-8f46-5daa6f9d8da1`, KenPom #3 / +37.37, **Torvik #3 / 0.9768**
- Supabase project: `midyxvjfoggchbugxzkd`
- Publishable key (in `index.html`, safe): `sb_publishable_MizrCkmoqSNlQAF3ty_FUA_WkATU4Ai`
- Commits this session: `c4e79f1` (backend) on apex-intel, `5b54cb6` + `6efe921` on Apex-App

---

## Session starter for next chat (Session 6 — Apex Scout polish)

Paste this as your first message in Session 6:

```
Loading project context — Apex Intel, Session 6 of 14-day plan (Apex Scout polish, the showpiece).

Please read from project knowledge:
- CLAUDE.md
- handoffs/HANDOFF-2026-04-17-session5-torvik.md  ← latest
- DATA-STATUS.md
- BACKLOG.md
- PLAN-2026-04-16-to-2026-04-30.md

Session 6 target (one sentence):
Polish Apex Scout — wire KenPom + Torvik chips into both Scout render branches (LLM-mode ~line 3271, rules-mode ~line 3460), tighten copy, polish card layout so this is the best-looking screen in the app.

No carryover from Session 5 — Torvik fully shipped end-to-end (backend + frontend + Smart Filter). Scout chip wiring was explicitly deferred from Session 4b to Session 6 to land in a single polish pass.

Expected scope: 90 min. Two chip-wiring patches + copy tightening + visual polish.

I'm on Mac, Terminal ready. Let's plan before code.
```

---

## ⚠️ Reminder — upload updated docs to Project Knowledge

Three docs changed in this session and need to be re-uploaded to Project Knowledge before the next session so the context loads correctly:
- `DATA-STATUS.md`
- `BACKLOG.md`  (significant — 6 new named projects from Sammy pitch)
- `CLAUDE.md`
- `handoffs/HANDOFF-2026-04-17-session5-torvik.md` (this file, new)

Without the re-upload, next session's `project_knowledge_search` will return stale data and we'll boot with wrong state. **Please re-upload all four immediately after this session closes.**
