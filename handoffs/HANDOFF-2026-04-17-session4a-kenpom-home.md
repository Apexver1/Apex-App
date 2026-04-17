# Handoff — April 17, 2026 (Session 4a: KenPom frontend — Home surfaces)

## Session result

Session 4a target hit: **KenPom data (365 D1 schools, 8 kenpom_* cols) is live on Home → Next Up and Home → Roster Pulse for every D1 school.** Session 3's backend ingestion is now visible to end users.

Remaining three surfaces from the original Session 4 plan — **player modal, recruiting cards, scout cards** — deferred to Session 4b/4c rather than rushed. Scope-lock rule #3 in practice: ship smaller when you can't keep it working.

Session ran ~90 min. Shipped commit `d010c6c` to `Apex-App/main`.

---

## What shipped

### Commit `d010c6c` — Session 4a: wire KenPom into Next Up + Roster Pulse
1 file changed, 9 insertions, 7 deletions on `index.html` (7 targeted patches applied via regex swap; 233,075 → 234,068 bytes).

**Patch 1 — Schools SELECT expanded.** `fetch(SU+"/rest/v1/schools?select=...")` at line 1339 now includes all 8 kenpom_* columns: `kenpom_rank, kenpom_adj_em, kenpom_adj_o, kenpom_adj_d, kenpom_tempo, kenpom_season, kenpom_team_id, kenpom_last_sync`.

**Patch 2 — Declaration.** `var schoolKenPomMap={};` added next to `var schoolEspnMap={};` at line 947. Mirrors the Espn pattern.

**Patch 3 — Population.** In `loadData()` right after `schoolEspnMap` populates (~line 1348), `schoolKenPomMap` is built lazily from the `schools` array, keyed by lowercase name, values are `{rank, adjEm, adjO, adjD, tempo, season}` objects.

**Patch 4 — Helper.** `window.kenpomFor(name)` — takes a school name, returns `schoolKenPomMap[name.toLowerCase()] || {}`. Mirrors `window.schoolLogo` pattern.

**Patch 5 — s2nextGame rewire.** Non-Oregon path in `s2nextGame(sid)` (~line 729) now reads real KenPom via `kenpomFor(s2short(sid))` / `kenpomFor(s2opponent(sid))`, with random-fallback (20+rng*60, 15+rng*60) preserved if lookup misses. Every non-Oregon school previously showed random kpom values 15–79 — now shows real.

**Patch 6 — NEXT_GAME lazy patch.** Oregon's NEXT_GAME constant at line 880–885 has hardcoded `kpom:18` (home) and `kpom:9` (away). Rather than edit the const, we overwrite the values inside `loadData()` the moment `schoolKenPomMap` populates. Wrapped in `try/catch` for defensive safety. Result: Oregon shows #101 (not #18), Arizona shows #2 (not #9) — matches live KenPom truth.

**Patch 7 — Roster Pulse 4th tile.** New tile added to the Roster Pulse strip at line 2340, positioned after Depth Score. Shows `+X.XX AdjEM` as the label and `#rank` as the value. Derives the current school from `CURRENT_SCHOOL_ID` via `schools.filter` (not `CURRENT_SCHOOL_NAME` — that global carries the `"Oregon Ducks"` form, which doesn't key into `schoolKenPomMap` directly; the `schools` array lookup gives us the canonical `schools.name` form).

### Visual verification on live production (apexver1.github.io)
| School | Next Up home rank | Next Up away | Roster Pulse chip |
|---|---|---|---|
| Oregon | #101 (was #18) | Arizona #2 (was #9) | +7.08 AdjEM · #101 |
| Duke | #3 | Miami #30 | +37.37 AdjEM · #3 |
| Kansas | #21 | Utah #128 | +24.15 AdjEM · #21 |
| Howard | #196 | Delaware State #362 | -2.64 AdjEM · #196 |

**Negative AdjEM renders correctly** (Howard -2.64) — was a failure mode I watched for explicitly.

---

## Mid-session failure modes and recoveries

### 1. Failed first ship — `SCHOOL_NAME` placeholder bug
First patch build used `SCHOOL_NAME` as a placeholder variable name in the Roster Pulse chip. When I opened the local file in Safari, red error on the splash screen: `Can't find variable: SCHOOL_NAME`. Auto-login blocked. `index.html` was restored from backup (`index.html.s4a-backup-20260417-043201`). Rewrote Patch 7 to derive school via `schools.filter(s=>s.id===CURRENT_SCHOOL_ID)[0].name`. Rebuilt, reshipped. Root cause: I used a placeholder identifier instead of finding the real variable-in-scope before building.

**Lesson:** when I tell the user "I'm guessing here," the grep to verify the identifier needs to happen BEFORE the patch build, not after.

### 2. Node not installed — lost `node --check` safety net
`node --check /tmp/index-s4a.html` errored with `-bash: node: command not found`. Fell back to a Python regex-based brace/paren balance check against extracted `<script>` bodies. Both pre-patch and post-patch files showed identical 0/0 balance — strong signal patches didn't break structure. Safari render confirmed as ground truth.

**Added to BACKLOG** as a minor tooling item: install Node.js on Mac for future sessions, OR switch to a different JS syntax validator (e.g., `npx esprima-cli` via existing Node-less path, or in-browser headless Chrome via puppeteer if the setup is worth it).

### 3. Safari cache masking the deployed fix
After `git push`, the live URL apexver1.github.io still showed the old #18/#9 values even though `origin/main` had the patched file. Diagnosed via `git log --oneline -5` (commit present) + Safari's "Clear History → Last hour" (cache-bust). Noted in CLAUDE.md's existing gotcha: Safari is aggressive about caching Supabase REST responses, and the same aggression applies to GitHub Pages HTML.

---

## Files changed

| File | Status |
|---|---|
| `Apex-App/index.html` | COMMITTED `d010c6c` — 7 patches (SELECT + map decl + populate + helper + s2nextGame + NEXT_GAME lazy patch + Roster Pulse tile) |
| `Apex-App/index.html.s4a-backup-20260417-043201` | LOCAL ONLY, untracked — pre-patch safety backup. Should be gitignored per Session 3 cleanup pattern. |
| `Apex-App/DATA-STATUS.md` | MODIFIED (this session) |
| `Apex-App/BACKLOG.md` | MODIFIED (this session) |
| `Apex-App/handoffs/HANDOFF-2026-04-17-session4a-kenpom-home.md` | NEW (this file) |

---

## Still open (Session 4 rollover)

Session 4 target was 5 surfaces (Home Next Up, Roster Pulse, recruiting cards, player modal, scout cards). Session 4a shipped 2 of 5 — the two Home surfaces. Remaining 3 deferred to **Session 4b** (fresh chat).

### Session 4b target (one sentence)
Wire `kenpomFor()` into player modal (via `openP` entry point), recruiting cards (via `renderRecruiting` at line 2428), and Apex Scout cards (via `renderScout` at line 3271).

### Pre-located entry points for 4b (no re-recon needed)
- **`renderRecruiting`** at line 2428 — function signature `(c, portal, hs)`. Card renders include `p.current_school` and `p.current_conference` already in DOM — KenPom chip is a drop-in next to those.
- **`renderScout`** at line 3271 — two branches: `aiResults.mode==="llm"` (Scout 1.2 structured tool-use, newest code) and `else if(aiResults)` legacy rules-mode regex parser around line 3460. Both branches render a card with `p.current_school`. **Caution:** Scout is slated for Session 6 polish ("Apex Scout: the showpiece"). Consider whether KenPom wiring should wait for Session 6 rather than ship in 4b — tradeoff: 4b is tighter scope, Session 6 already has a polish budget that can absorb the wiring.
- **Player modal** — entry point is `openP(id, 'p')` (seen at line 3470 and 1574 onclick handlers). The render function for the detail view was NOT located in Session 4a reconnaissance; Session 4b opener needs one targeted grep: `grep -nE "openP=|function openP|navTo.*player|hash.*player"`.

### Open-ended scope consideration for 4b
Scout cards have two render branches (LLM and rules-mode). If wiring both branches gets complex, **defer scout-cards to Session 6** (Apex Scout polish) and ship 4b as just (player modal + recruiting cards). That's a cleaner decomposition than trying to touch a mid-refactor Scout render from 4b.

### Still-open Session 4a items (low priority)
- `index.html.s4a-backup-20260417-043201` is untracked. Add to `~/code/Apex-App/.gitignore`: `*.s4a-backup-*` pattern, or broader `*-backup-*`. Repo-hygiene work item already on BACKLOG.

---

## Session log observations (for future sessions)

- **~1 hour of reconnaissance before first code** was the right call for this session because PATCH 7 depended on finding 3 separate identifiers (`schools` array scope, `CURRENT_SCHOOL_ID`, `CURRENT_SCHOOL_NAME`). Speeding past reconnaissance would have cost more than the rebuild did.
- **The patch-build flow (cat heredoc → Python with match-count asserts → balance-check → shasum verify → copy-in-place) was clean and repeatable.** Use as the standard pattern going forward.
- **Screenshot-based smoke test of 5 schools** surfaces more bugs than CLI verification. Oregon + Duke + Kansas + Howard + Saint Mary's picker is the right default browse-pattern for multi-school UI work.
- **Observation (not actionable yet):** Oregon Roster Pulse shows `On roster: 17` but DATA-STATUS says Oregon has 78 players in `my_roster`. Likely a starter/active filter (maybe `depth_chart_position IS NOT NULL`). Logged as observation only; may be related to B4.
- **Observation:** Howard drawer shows "Good morning, Coach." with no name (B3, already in BACKLOG). Confirmed still present — nothing made it worse.
- **Observation:** School picker search for "saint" returned 4 matching programs including Saint Mary's. B6 was "st." (period-abbreviation) failing to match — "saint" seems to work. Bug may be narrower than originally specced; worth a 5-min probe in a future session to confirm whether only "st." is broken, not all saint-family searches.

---

## Reference constants (unchanged)

- Oregon: UUID `d4dd73c7-2ec6-49bf-b63b-5c4a333a2bf7`, KenPom rank 101, AdjEM +7.08
- Duke: UUID `279abc4c-77dd-4a3a-8f46-5daa6f9d8da1`, KenPom rank 3, AdjEM +37.37
- Supabase project: `midyxvjfoggchbugxzkd`
- Publishable key (in `index.html`): `sb_publishable_MizrCkmoqSNlQAF3ty_FUA_WkATU4Ai` — unchanged by 2026-04-17 service-role rotation
- Commit shipped: `d010c6c` on `origin/main`

---

## Session starter for next chat (Session 4b)

Paste this as your first message in Session 4b:

```
Loading project context — Apex Intel, Session 4b of 14-day plan
(finish KenPom frontend — remaining 3 surfaces).

Please read from project knowledge:
- CLAUDE.md
- handoffs/HANDOFF-2026-04-17-session4a-kenpom-home.md  ← latest
- DATA-STATUS.md
- BACKLOG.md
- PLAN-2026-04-16-to-2026-04-30.md

Session 4b target (one sentence):
Wire kenpomFor() into player modal (openP entry point), recruiting
cards (renderRecruiting at line 2428), and Apex Scout cards
(renderScout at line 3271).

Open decision at session opener:
Whether scout-cards KenPom wiring ships in 4b or waits for Session 6
Apex Scout polish (the showpiece session). Tradeoff: 4b simpler scope
without Scout, Session 6 already budgeted for Scout polish.

No carryover tasks. Jump straight to work.
```
