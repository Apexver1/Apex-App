# HANDOFF — Session 7 (2026-04-18)

**Slug:** scout-llm-default-emergency
**Primary commits:** `9c0d90e` (Scout P0 fix) + `d1806d2` (repo cleanup) + `f68fb4f` (Donor Development copy)
**Entering state:** Session 6 closed at commit `edbae04` — Scout v2 hero live, metric tooltips live, aiMode defaulting to rules (broken), Commit 2b drawer work deferred.
**Exiting state:** Scout 1.1.1 is live on https://apexver1.github.io/Apex-App/ with LLM-default routing + fixed mic visual state. Commit 2b drawer still deferred. Ready for Session 8 to pick up drawer work OR to start Scout Edge Function context sync (new #1 project).

---

## Session target (one sentence)

**Planned:** Ship Commit 2b — 72px centered school logo in the drawer + role label rewrite + Howard NULL head-coach fallback.

**What actually happened:** Emergency pivot at ~11:30 AM when user reported "scout no longer works" + mic turning into "weird red listening button" three hours before a 3:00 PM mentor demo. Session 7 target changed to: *get Scout reliable enough to demo by 2:00 PM hard stop, clean docs, clean handoff.*

---

## What shipped

### Commit 9c0d90e — Scout P0 fix (one atomic commit, four patches + the real fix)

**P1 — aiRun one-liner inverted.** Original code at byte 149492:

    aiRun=function(){if(aiMode==="llm")aiLLM();else aiSearch();}

Changed to:

    aiRun=function(){if(aiMode==="rules")aiSearch();else aiLLM();}

Rationale: Invert branches so the default (anything other than "rules") routes to LLM. This alone wasn't enough — see P5.

**P2 — aiVoice function rewritten (byte 145616).** Old version destroyed the SVG mic icon by setting `btn.textContent="Listening…"` and turned the button red via `btn.style.background="var(--neg)"`. That is the "weird red listening button" the user reported. New version:
- Adds `btn.classList.add("listening")` to trigger CSS-driven pulse animation
- Stores active rec on `window._apxRec`
- On click-while-listening, calls `rec.stop()` and clears class (tap-to-cancel)
- On result / error / end, cleanly removes `.listening` and clears `window._apxRec`
- Preserves the aiInput auto-fill + aiRun() auto-fire on transcript

**P3 — Mic button HTML got apx-mic class.** At byte 216365, added `class="apx-mic"` to the `<button id="micBtn">` in Scout hero rendering.

**P4 — New CSS injected after .apx-pop block (~byte 18341):** added `.apx-mic`, `.apx-mic.listening`, and `@keyframes apxMicPulse` rules so the listening state shows a subtle navy pulse ring instead of destroying the icon.

**P5 — THE REAL FIX: aiMode state init flipped (byte 64474).** After P1–P4 shipped, Scout still returned Juke Harris on every query. Diagnosis revealed a state initializer line:

    sortBy="rank",aiQuery="",aiResults=null,aiMode="rules",aiLoading=false,...

This was setting aiMode explicitly to "rules" on page load — so P1's `if(aiMode==="rules")` test always matched. One-character fix: change "rules" to "llm". **This is what Session 6 Commit 2c missed** — we were chasing the `var aiMode` declaration when the real bug was the state initializer value.

### Commit d1806d2 — repo cleanup

Commit 9c0d90e accidentally committed 3 untracked local backup files (11,384 lines of HTML backup got tracked). This commit:
- `git rm --cached` on all three (keeps on disk, untracks from repo)
- Adds `index.html.*-precommit-*` pattern to `.gitignore`

Net delta: -11,370 lines. Repo back to healthy size.

### Commit f68fb4f — Apex Patrons header copy fix

"Donor & development" → "Donor Development" (capital D, no ampersand) in Apex Patrons page header. Cosmetic. User-requested as a before-we-wrap polish.

---

## The real root cause (post-mortem)

Session 6's Commit 2c theory was: "aiMode is undefined by default, so we need to `var aiMode="llm"` somewhere." That was wrong.

Actual truth: aiMode was ALREADY being initialized, explicitly, to "rules" — in a comma-chained state block at byte 64474. This is why the Session 6 `var aiMode="llm"` attempt broke JS parsing — we were trying to add a new var declaration adjacent to state code that was already doing a single-statement multi-variable assignment.

Session 7's diagnostic loop went:
1. P1 patched aiRun — still broken (because aiMode==="rules" was true)
2. Thought it was Safari caching file:// fetches — opened DevTools Console, saw zero errors, zero logs
3. Confirmed P1 was on disk
4. Found aiMode="rules" at byte 64474 via grep for aiMode= occurrences
5. Flipped it — Scout immediately worked, `[Scout 1.1.1] raw response:` began appearing in Console

**Lesson for future sessions:** When debugging a feature toggle that doesn't toggle, search for ALL assignment sites to the variable, not just declarations. A regex like `aiMode\s*=\s*` catches both `var aiMode=...` AND inline state initializers.

---

## Scout 1.1.1 QA pass (10 queries, deployed app)

Ran after all three commits shipped. Full matrix:

| # | Query | Grade | Tool calls | Iterations | Notes |
|---|---|---|---|---|---|
| 1 | point guards in the portal averaging 15+ ppg | GREEN | 1 | 2 | 17 real PGs, all 15+ ppg |
| 2 | elite shooters hitting 40%+ from three | GREEN | 1 | 2 | 25 real shooters all 40%+, Darius Acuff on shortlist surfaced |
| 3 | rim protectors with 2+ blocks per game | RED | 2 | 3 | FAILED — returned guards, not rim protectors. Skip in demo. |
| 4 | defensive-minded bigs who can protect the paint | GREEN | 4 | 2 | 49 centers with real rebounding — Oregon's Nate Bittle surfaced as contextual pick |
| 5 | upperclassmen from mid-majors who could start at a P5 school | YELLOW | 11 | 5 | Impressive reasoning (11 tool calls) but missed mid-major filter — returned P5 players |
| 6 | guards who can score and defend | YELLOW | 2 | 2 | Claude correct; card-display shows BPG (near-zero for guards) not SPG. UI gap, not Scout gap. |
| 7 | portal transfers under $500K averaging 12+ ppg | GREEN | 1 | 1 | 0 players but EXCELLENT anti-hallucination — Claude said "NIL valuation data is not yet available" and offered refinement paths. Demo gold. |
| 8 | shooters from top-50 KenPom defenses | YELLOW | 1 | 1 | Claude said "KenPom team efficiency data is not yet integrated" — WRONG (KenPom ingested Session 4) but CORRECT at payload level (not in LLM prompt). See BACKLOG #1. |
| 9 | 6'8+ forwards shooting 35%+ from three | GREEN | 2 | 2 | 34 SFs all 35%+ — can't verify heights on card (UI gap) |
| 10 | players like Cameron Boozer | DOUBLE GREEN | 9 | 6 | Similarity reasoning — JT Toppin / Shelton Williams-Dryden / Graham Ike / Caleb Wilson (on shortlist) all real PF comps. Demo centerpiece. |

**Demo-ready sequence (in order):** Q10 → Q4 → Q7 → Q2 or Q9 → Q6 with narrative caveat.

**Demo-safe skip list:** Q3 (bad results), Q5 (partial filter), Q8 (exposes Edge Function gap directly).

---

## Gaps surfaced by the QA pass

### B10 (new) — Player cards show BPG on guards
Guards almost always have 0.0–0.5 bpg. Shown on cards, this misrepresents defensive value. Coach looks at "0.1 bpg" on Markus Burton and reads "no defense." Scout may have surfaced him as a two-way guard but the card can't show that. Fix: position-aware stat display — SPG for guards, BPG for forwards/centers. Logged in BACKLOG.

### B11 (new) — Scout Edge Function missing advanced-metric payload
Confirmed via Q7 + Q8: Scout's Edge Function prompt/payload does NOT include:
- KenPom team context (AdjEM, AdjO, AdjD, Rank, Tempo) — wired since Session 4 on the data layer
- Torvik team context (AdjO, AdjD, eFG%, TO%) — wired since Session 5
- Player-level Torvik (prospect_stats.adjoe, adjde, usg_pct, min_pct) — available in Supabase

Claude reasons over counting stats (ppg/rpg/apg/3pct) + apex_model score only. When asked about KenPom defenses, Claude honestly reports it doesn't have that data — honest but product-visible gap.

**This is the new #1 named project in BACKLOG.** Scope: extend aiLLM payload in index.html to pull these fields from schoolKenPomMap / schoolTorvikMap / prospect_stats, update Edge Function system prompt + tool definitions. Estimated 2–4 sessions (frontend-first path is lower risk).

### Card display doesn't surface height or class
Q9 returned "6'8+ forwards" but we can't see heights on cards to verify. Same issue with class year. Small S7-minor logged.

### Edge Function system prompt outdated
Scout Analysis responses reference "Scout 1.2" and "Scout 1.3" roadmap labels that don't match current DATA-STATUS (KenPom is live, NIL is partial). Update prompt once we have Edge Function edit access. Logged as S7-minor-2.

---

## Scope decisions / deferrals

- **Commit 2b drawer work:** still deferred. Original Session 7 target. Now Session 8.
- **"Team ·" prefix on popovers (1a):** still deferred. Trivial when attempted solo.
- **Scout engine auto-routing (2c):** RESOLVED this session via the correct root-cause fix.
- **NIL data backfill:** not in scope for any near-term session; Q7 showed Scout's honesty about the gap is actually a demo asset.

---

## Mid-session lessons

### Check all assignment sites, not just declarations
Session 6 chased a phantom `var aiMode` problem for multiple hours. Session 7 solved it in 10 minutes after grepping aiMode= in all its forms. **Lesson:** when a variable seems to default to the wrong value, `grep -E 'varname\s*=\s*'` finds BOTH declarations AND inline initializers. State-initializer patterns (a=null,b="x",c=false,...) don't grep cleanly on the var keyword.

### Don't trust Safari file:// for feature testing
Early in the session I almost recommended LLM-mode work before realizing file:// probably blocks fetches to external HTTPS endpoints. Always test at the deployed URL for anything touching external APIs. Scout testing should default to https://apexver1.github.io/Apex-App/, not the local filesystem.

### Console is the fastest diagnostic
The Safari console showing 100% zero output after a query was the single clearest signal that aiLLM wasn't firing. Always open DevTools Console before clicking anything in a debug session.

### Emergency pivots are worth it
The planned Commit 2b would have been polish. Shipping working Scout IS the pitch. Session 7 pivot was correct.

---

## Files changed this session

| File | Status | Commit |
|---|---|---|
| index.html | Patched 4× for Scout; 1× for Donor Development copy | 9c0d90e, f68fb4f |
| .gitignore | Added index.html.*-precommit-* pattern | d1806d2 |
| 3 backup files | Removed from repo (kept on disk) | d1806d2 |
| index.html.s7-cosmetic-20260418-130320 | New local backup before Donor fix | Not tracked (gitignored) |
| DATA-STATUS.md | Full rewrite — added Session 7 entry, B10 + B11, Edge Function context gap | (this commit) |
| BACKLOG.md | Full rewrite — added Scout Edge Function context sync as #1, B10 logged, Session 7 minors | (this commit) |
| handoffs/HANDOFF-2026-04-18-session7.md | New | (this commit) |

---

## Current commit state (end of Session 7)

    [next commit]  docs(session-7): Scout LLM-default shipped + QA pass + Edge Function gap logged
    f68fb4f        fix(patrons): capitalize 'Donor Development' header
    d1806d2        chore: gitignore backup files (accidentally tracked in 9c0d90e)
    9c0d90e        fix(scout): Session 7 P0 — LLM default routing + mic state fix
    edbae04        docs(session-6): Scout v2 + metric tooltips shipped + 2b deferred
    7ee3241        feat(tooltips): Session 6b (Commit 2a) — plain-English metric translation
    d30a3ca        feat(scout): Session 6a — Scout v2 hero redesign

---

## Session 8 pickup — what to start with

Priority call pending 3:00 PM mentor feedback. Two paths:

### Path A — Scout Edge Function context sync (new #1 project)

See BACKLOG. Multi-session. Frontend-first approach minimizes risk. Pilot:
1. Modify aiLLM payload in index.html to enrich prospect objects with KenPom team fields (from schoolKenPomMap) and Torvik team fields (from schoolTorvikMap)
2. Modify loadData() prospects SELECT to JOIN prospect_stats for player-level AdjOE/AdjDE/usg_pct
3. Ship as frontend-only commit first — Edge Function will ignore unknown fields; Claude may start using new data simply because it's in the JSON
4. Re-run Q7 + Q8 from this session's QA pass — expect them to go from YELLOW to GREEN
5. Follow-up session: update Edge Function system prompt to explicitly document the new fields

### Path B — Commit 2b drawer work (original Session 7 target)

Deferred twice now. DevTools recon still needed per Session 6 handoff:
1. Safari DevTools → Inspect Element on DA initials circle
2. Trace CSS classes back to render function
3. Byte-extract anchor using S6 pattern (Python .find() + save to /tmp/ + read back)
4. Four-patch atomic commit: 72px logo swap + "Oregon Ducks" subhead + "Coach Stewart · Assistant Coach" label (muted gray) + Howard null fallback copy

### Default if no mentor input

Path A. Scout Edge Function context sync is the highest-leverage work — it makes every query significantly sharper AND addresses Sammy's core "stop paying for the ranking number, pay for the player" critique.

---

## Post-session mentor meeting (2026-04-18, ~1pm)

Happened separately from the 3pm demo — a loose strategy conversation. Key themes captured:

1. Claude Code vs chat workflow — mentor advocates switching for direct repo/db access. User decision: DEFER until MacBook upgrade (~1 week). Stay in chat-based workflow for now.
2. Code-first vs design-first — mentor favors data-structure-first thinking. Noted but not actionable this cycle.
3. Derived data layer — secondary signals built from primary data (fit score, undervaluation score, transfer risk, defensive ceiling). Directly reinforces the B11 / Scout Edge Function context sync project. High-value concept.
4. Vectorization for Scout — embed prospects semantically, kNN search before Claude. Multi-session project.
5. Product stories as structured data — promote BACKLOG from markdown to queryable schema.
6. Apps vs services business model — strategic question, not technical. Deferred.
7. Proprietary data moat — explicit definition of what data only Apex Intel has.
8. Code hygiene — periodic scrape of dead code.

User direction 2026-04-18: Stick to the original 14-day plan. Re-evaluate which mentor ideas to promote into the plan AFTER 14-day window ships (around 2026-04-30). Do not slip mentor concepts into active sessions in the meantime — discipline matters more than the ideas themselves right now.

Implication for Session 8: Default to Path A (Scout Edge Function context sync) OR Path B (Commit 2b drawer) per the 14-day plan. Mentor concepts stay in BACKLOG as ideas, NOT promoted to active work.

---

## Post-session mentor meeting (2026-04-18, ~1pm)

Happened separately from the 3pm demo — a loose strategy conversation. Key themes captured:

1. **Claude Code vs chat workflow** — mentor advocates switching for direct repo/db access. User decision: DEFER until MacBook upgrade (~1 week). Stay in chat-based workflow for now.
2. **Code-first vs design-first** — mentor favors data-structure-first thinking. Noted but not actionable this cycle.
3. **Derived data layer** — secondary signals built from primary data (fit score, undervaluation score, transfer risk, defensive ceiling). Directly reinforces the B11 / Scout Edge Function context sync project. High-value concept.
4. **Vectorization for Scout** — embed prospects semantically, kNN search before Claude. Multi-session project.
5. **Product stories as structured data** — promote BACKLOG from markdown to queryable schema.
6. **Apps vs services business model** — strategic question, not technical. Deferred.
7. **Proprietary data moat** — explicit definition of what data only Apex Intel has (KenPom + Torvik + derived signals + per-school context combinations).
8. **Code hygiene** — periodic scrape of dead code.

**User direction 2026-04-18:** Stick to the original 14-day plan. Re-evaluate which mentor ideas to promote into the plan AFTER 14-day window ships (around 2026-04-30). Do not slip mentor concepts into active sessions in the meantime — discipline matters more than the ideas themselves right now.

**Implication for Session 8:** Default to Path A (Scout Edge Function context sync) OR Path B (Commit 2b drawer) per the 14-day plan. Mentor concepts stay in BACKLOG as ideas, NOT promoted to active work.

---

## Reference constants (unchanged)

- Oregon: UUID d4dd73c7-2ec6-49bf-b63b-5c4a333a2bf7, KenPom #101 / +7.08, Torvik #91 / 0.7068
- Duke: UUID 279abc4c-77dd-4a3a-8f46-5daa6f9d8da1, KenPom #3 / +37.37, Torvik #3 / 0.9768
- Supabase project: midyxvjfoggchbugxzkd
- Publishable key (in index.html, safe): sb_publishable_MizrCkmoqSNlQAF3ty_FUA_WkATU4Ai
- Test login: apex@test.com / ApexTest123 (auto-login wired)
- Edge Function URL: https://midyxvjfoggchbugxzkd.supabase.co/functions/v1/apex-scout

---

## Session starter for Session 8

Paste this as the first message in Session 8's chat:

    Loading project context — Apex Intel, Session 8.

    Please read from Project Knowledge:
    - CLAUDE.md
    - handoffs/HANDOFF-2026-04-18-session7.md  (latest)
    - DATA-STATUS.md
    - BACKLOG.md
    - PLAN-2026-04-16-to-2026-04-30.md

    Session 7 shipped Scout LLM-default routing + mic fix.
    Scout 1.1.1 live and working on https://apexver1.github.io/Apex-App/.
    Demo at 3pm went [FILL IN FEEDBACK].

    Session 8 target: [PATH A — Scout Edge Function context sync / PATH B — Commit 2b drawer / OTHER based on mentor feedback]

    On Mac, Terminal ready. Let's plan before code.

---

## Reminder — upload updated docs to Project Knowledge

Three docs changed this session and need re-upload before Session 8:
- DATA-STATUS.md
- BACKLOG.md
- handoffs/HANDOFF-2026-04-18-session7.md (this file, new)

No CLAUDE.md changes this session (architecture didn't shift; the Scout fix was within existing patterns).

Without re-upload, Session 8's project_knowledge_search returns stale Session 6 data.
