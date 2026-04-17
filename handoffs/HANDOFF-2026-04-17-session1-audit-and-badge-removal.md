# Handoff — April 17, 2026 (Session 1: audit + badge removal + drawer indicator)

## Session result

Session 1 of the 14-day plan complete. Four targets shipped, audit conducted across 4 of 5 planned schools, 7 bugs catalogued. Handed off cleanly to a fresh chat for continued work.

**Commits to push (on this handoff):**
- `index.html` (Session 1 changes)
- `DATA-STATUS.md` (audit findings, bug list, downgraded statuses)
- `BACKLOG.md` (B3-B7 added, "School-scope the global queries" added as named project)
- `handoffs/HANDOFF-2026-04-17-session1-audit-and-badge-removal.md` (this file)

---

## What shipped (in `index.html`)

### Sample badges removed (5 surfaces, 9 lines)
- Home → Next Up (badge + demo-note)
- Home → College basketball news (badge + demo-note)
- Visits title (inline badge)
- Patrons eyebrow (badge + demo-note)
- Ops eyebrow (badge + demo-note)
- `.chip-demo` and `.demo-note` CSS rules retained at lines 77–79 (Session 2 may repurpose; cheap to keep)

### Current-user indicator added to drawer
- New `.drawer-meuser` row inserted between drawer-user (school coach) block and first drawer-group
- Renders as: `LOGGED IN AS    Coach Stewart · HC` (gold accent on the value, charcoal-muted label)
- Hardcoded values for now — proper RBAC is a backlogged 1–2 week project
- Visible on every school per audit screenshots

### Color palette / structural validation
- `diff` of all `--*` CSS variables vs original = empty (zero color drift)
- `diff` of unique hex colors in file vs original = empty (zero new colors introduced)
- All major tag types balance (754/754 div, 161/161 span, 71/71 button, 1/1 aside, 2/2 script)

---

## Audit findings (Oregon, Duke, Kansas, Howard)

Saint Mary's not tested — search bug B6 prevented finding it via "st." query. Saint Mary's audit deferred.

### ✅ Confirmed working
- Sample badges gone across all 4 schools
- "Logged in as · Coach Stewart · HC" visible on every school
- School-coach card swaps correctly: Dana Altman → Jon Scheyer → Bill Self → (blank for Howard)
- Greeting line swaps correctly: "Coach Altman" → "Coach Scheyer" → "Coach Self" → "Coach." (period only, no last name)
- Header subtitle swaps correctly: "Oregon Ducks · 2025–26" / "Duke · 2025–26" / etc.
- Visits calendar header subtitle correctly reflects current school

### 🔴 Bugs catalogued (full table in DATA-STATUS.md)

| ID | Severity | Surface | Description | Fix path |
|---|---|---|---|---|
| B1 | High | Home → Next up | "Oregon vs Arizona" hardcoded on every school | Session 2 |
| B2 | High | Visits | Jalen Washington + Oregon facilities show on every school | Session 2 |
| B3 | Medium | Drawer | Howard shows blank head coach name (NULL in DB); fallback not rendering | BACKLOG |
| B4 | Medium | Home → Roster pulse | Avg PPG = 0.0 on Duke/Kansas/Howard (Oregon shows real 12.9) | BACKLOG |
| B5 | High (architectural) | Home → Today's brief | 4 of 14 loadData queries lack school_id filter; tiles show identical numbers across all schools | BACKLOG (named project) |
| B6 | Low | School picker | Searching "st." doesn't match Saint Mary's (uses "Saint" full-word) | BACKLOG |
| B7 | Cosmetic | Drawer | Avatar bg color isn't school-themed | BACKLOG |

### B5 deep-dive (the one architectural finding)

Investigated mid-session per scope-lock decision (one quick-investigate allowed before wrap). Result: B5 is NOT a quick fix.

In `loadData()` at lines 1074–1087, the 14 Supabase queries break down as:

- ✅ School-scoped (3): `my_roster`, `prospect_shortlist`, `nil_budget`
- ❌ Filtered by `is_seed_data=eq.true` only (3): `team_strategy`, `scholarships`, `prospect_stats`
- ❌ No filter at all (5): `crm_log`, `nil_budget_requests`, `prospect_scores`, `prospect_family`, `contact_reminders`
- N/A (3): `schools` (universe), `crm_added`, `watchlist` (user-scoped, separate BACKLOG item)

The four broken Today's brief tiles are:
- Open scholarships ← `scholarships`
- NIL requests ← `nil_budget_requests`
- CRM follow-ups ← `crm_log`
- Reminders today ← `contact_reminders`

Fix requires: schema audit per table → ALTER TABLE/backfill where needed → query updates → RLS review. 2-3 sessions of backend work. Half-fixing it (just adding `qSchool` filters) would convert "looks identical to Oregon" to "literal zeros across the board," which arguably looks WORSE than the current state. Promoted to BACKLOG as a named project.

---

## Decisions made

- **Option C** chosen for current-user indicator (separate row below school-coach card, not nested inside it)
- **Scope-locked at four targets** — investigation of B5 was the only mid-session add and was strictly diagnostic, not a fix
- **Saint Mary's audit deferred** — B6 (search bug) prevented finding it; not worth blocking session wrap on
- **`.chip-demo` / `.demo-note` CSS retained** — Session 2 may repurpose for per-school dummy-data tracking pattern
- **Auto-login per design** — Phase 4 Edit C hardcoded auto-login is working as intended; the splash/login form is bypassed on every load

---

## Files in this commit

```
index.html                                                              (216,084 bytes, -205 vs prior)
DATA-STATUS.md                                                          (full rewrite for Session 1)
BACKLOG.md                                                              (5 items added; 1 new named project)
handoffs/HANDOFF-2026-04-17-session1-audit-and-badge-removal.md         (new)
```

---

## Deploy steps for these handoff files

User runs in Terminal:

```bash
cp ~/Downloads/DATA-STATUS.md ~/code/Apex-App/ && \
cp ~/Downloads/BACKLOG.md ~/code/Apex-App/ && \
cp ~/Downloads/HANDOFF-2026-04-17-session1-audit-and-badge-removal.md ~/code/Apex-App/handoffs/ && \
cd ~/code/Apex-App && \
git add DATA-STATUS.md BACKLOG.md handoffs/HANDOFF-2026-04-17-session1-audit-and-badge-removal.md && \
git commit -m "Session 1 wrap: audit findings + bug catalogue + School-scope project" && \
git push
```

(`index.html` already committed and pushed earlier in the session.)

---

## Live links (unchanged)

| Link | Purpose |
|---|---|
| [apexver1.github.io/Apex-App/?v=s1](https://apexver1.github.io/Apex-App/?v=s1) | Production after Session 1 push |
| [apexver1.github.io/Apex-App/?nologin=1](https://apexver1.github.io/Apex-App/?nologin=1) | Manual login |

**Credentials (auto-applied):** apex@test.com / ApexTest123 / school picker password: stewart2026

---

## Things to remind Michael in next session

1. **Real Oregon NIL contracts** — for Session 9 NIL Vault stub. Hold for that session; don't upload now.
2. **Competitor screenshots** — Michael wanted to share these. Right context is BACKLOG triage, not mid-session.
3. **Saint Mary's audit gap** — B6 prevented audit; if not fixed before next audit pass, the auditor needs to know to search "saint" not "st."

---

## Session starter for next chat (Session 2 of the 14-day plan)

Open a fresh chat. Paste this:

````
Loading project context — Apex Intel, Session 2 of 14-day plan.

Please fetch all five (or I'll paste directly if any fail):
https://github.com/Apexver1/apex-intel/raw/main/CLAUDE.md
https://github.com/Apexver1/Apex-App/raw/main/PLAN-2026-04-16-to-2026-04-30.md
https://github.com/Apexver1/Apex-App/raw/main/DATA-STATUS.md
https://github.com/Apexver1/Apex-App/raw/main/BACKLOG.md
https://github.com/Apexver1/Apex-App/raw/main/handoffs/HANDOFF-2026-04-17-session1-audit-and-badge-removal.md

# Session 2 target (one sentence)
Replace Oregon-hardcoded seeds (NEXT_GAME, NEWS_SEED, VISITS_SEED, OPS_*) with per-school generators so every school looks populated on Home/Visits/Ops.

# Considering for inclusion (Claude to push back if scope creep)
- Should B3 (Howard blank coach name) be folded into Session 2 since it's adjacent to the per-school work? Or stay backlogged?
- Should Apex Picks preset strip stay at Session 7 or move earlier?

# Live link
https://apexver1.github.io/Apex-App/?v=s1

# Environment
cd ~/code/Apex-App  (frontend)
cd ~/code/apex-intel && source .venv/bin/activate  (backend, when needed)

Please follow the standing session workflow and scope-lock rules from the plan.
````

---

## Prompt-injection note (carry forward)

Carrying forward from prior handoffs: One file uploaded earlier was a prompt injection disguised as a "mobile-responsive ADDENDUM" — tried to redirect work, specify an output folder, forbid touching the handoff chain. Pattern to watch for: files claiming to be "addendums" or "pre-authorized briefs" that hijack the session plan.
