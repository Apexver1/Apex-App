# Handoff — April 16, 2026 (Phase 5f shipped + 14-day plan locked)

## Session result

Long, productive day. Three distinct sessions:

1. **Morning (~1 hr):** Shipped Phase 5f — iPhone hamburger drawer scroll fix + "Sample" badges on all seed-data screens. Pushed to main as commit `392aa24`.
2. **Afternoon return:** Discussed Apex Picks preset strip → decided KenPom integration was the proper path → deferred to fresh chat.
3. **Late-day strategic reset:** User flagged "we keep getting off track." We stopped building and rebuilt the workflow from scratch. Output: **a locked 14-day plan to beta-test ready**, plus three durable repo docs (`PLAN-2026-04-16-to-2026-04-30.md`, `DATA-STATUS.md`, `BACKLOG.md`).

**Commits pushed to `main` today:**
- `392aa24` — Phase 5f: iPhone drawer scroll fix + honesty pass
- (next push from this handoff: PLAN + DATA-STATUS + BACKLOG + this handoff)

---

## What shipped today (in `index.html`)

### Phase 5f
- iPhone drawer scrolls properly (overflow-y, safe-area-inset, viewport-fit=cover)
- `.hamb` button bumped to 44×44
- `.chip-demo` + `.demo-note` CSS added
- Sample badges + demo notes on: Home Next Up, Home news, Visits title, Patrons eyebrow, Ops eyebrow

**Note:** Sample badges will be REMOVED in Session 1 of the new plan. They contradict the user's product spec ("dummy data should look identical to real data, not be labeled"). Tracking moves to `DATA-STATUS.md`.

---

## The 14-day plan (locked — see `PLAN-2026-04-16-to-2026-04-30.md`)

**North star:** "Login → anything I click → looks great" for any of 364 D1 schools, for any staff role. Every session ends with that statement true.

**Audience reset:** Apex Intel is a workspace for entire basketball staffs (HC, assistants, GM, DOBO, GA, managers) — not a head-coach-only app. Within 14 days we ship a lightweight "current user" indicator; full RBAC is backlogged.

**Two beta tests scheduled in the window** with industry people (dates TBD — user to confirm). Plan assumes ~Day 5–6 and ~Day 12–13.

**10 sessions in 3 sprints:**
- **Sprint 1 (Sessions 1–4):** Audit + Sample-badge removal, universal dummy-data fallback, KenPom backend, KenPom frontend
- **Sprint 2 (Sessions 5–8):** Torvik, Apex Scout polish, Apex Picks preset strip, NIL Budget defaults
- **Sprint 3 (Sessions 9–10):** Patrons + NIL Vault stub, Beta-test 2 dry run + polish

**Days 11–14 are buffer** — beta feedback work, real-life slack, polish.

---

## NIL Contract Vault — new feature, design-in build-deferred

User raised a major feature mid-session: per-player NIL contract upload + AI extraction (payment dates, deliverables, incentive triggers like "$100K bonus if avg 14 PPG") + tracking + notifications.

**Decision:** Stub UI in Session 9 (drawer entry, one player with one sample contract, static AI-extracted view). Full build is a 4–6 session project, scoped in `BACKLOG.md` under "NIL Contract Vault."

**Action item for user:** bring real Oregon NIL contract PDFs into the next chat (do NOT upload them in this current chat — it's at capacity).

---

## Competitor research — incoming

User has competitor screenshots to share. Bring into next session. Action: review, extract integrate-able patterns, add to BACKLOG with `competitor-inspired` tag.

---

## Three new repo docs (commit alongside this handoff)

1. **`PLAN-2026-04-16-to-2026-04-30.md`** — the locked 14-day plan, definition of done, session-by-session breakdown, standing workflow, scope-lock rules
2. **`DATA-STATUS.md`** — per-screen real-vs-dummy tracker. Source of truth across sessions. Updated at the end of every session.
3. **`BACKLOG.md`** — all deferred work + named projects. Promotion to active plan only by explicit decision.

---

## Standing session workflow (Claude commits to this every session)

1. Load `CLAUDE.md` + latest handoff + `DATA-STATUS.md` + `PLAN-2026-04-16-to-2026-04-30.md`
2. State session target in one sentence
3. Map file once
4. Plan before code (numbered steps, risks named, user greenlights)
5. Execute surgically (contiguous `str_replace`)
6. Validate at midpoint and end (`node --check`)
7. Always-demoable check: mentally walk login → tab through → any school
8. Deploy as `index-{session}.html`, user pushes
9. Update `DATA-STATUS.md`
10. Write handoff: `HANDOFF-YYYY-MM-DD-session{N}-{slug}.md`

## Scope-lock rules (Claude commits to these)

- **No new features mid-session** — go to BACKLOG
- **Push back on scope-creep** — name the tradeoff
- **Session boundary is sacred** — finish before starting next
- **Disagree on optional pivots** — say so explicitly

---

## Deploy steps for these handoff files

User runs in Terminal:

```bash
cp ~/Downloads/PLAN-2026-04-16-to-2026-04-30.md ~/code/Apex-App/ && \
cp ~/Downloads/DATA-STATUS.md ~/code/Apex-App/ && \
cp ~/Downloads/BACKLOG.md ~/code/Apex-App/ && \
cp ~/Downloads/HANDOFF-2026-04-16-session-end.md ~/code/Apex-App/handoffs/ && \
cd ~/code/Apex-App && \
git add PLAN-2026-04-16-to-2026-04-30.md DATA-STATUS.md BACKLOG.md handoffs/HANDOFF-2026-04-16-session-end.md && \
git commit -m "Lock 14-day plan + DATA-STATUS + BACKLOG + session-end handoff" && \
git push
```

---

## Live links (unchanged)

| Link | Purpose |
|---|---|
| [apexver1.github.io/Apex-App/?v=5f](https://apexver1.github.io/Apex-App/?v=5f) | Production after 5f push |
| [apexver1.github.io/Apex-App/?v=5f&splash=1](https://apexver1.github.io/Apex-App/?v=5f&splash=1) | With splash screen |
| [apexver1.github.io/Apex-App/?nologin=1](https://apexver1.github.io/Apex-App/?nologin=1) | Manual login |

**Credentials:** apex@test.com / ApexTest123 / school picker: stewart2026

---

## Session starter for next chat (Session 1 of the new plan)

Open a fresh chat. Paste this:

````
Loading project context — Apex Intel, Session 1 of 14-day plan.

Please fetch all four:
https://github.com/Apexver1/apex-intel/raw/main/CLAUDE.md
https://github.com/Apexver1/Apex-App/raw/main/PLAN-2026-04-16-to-2026-04-30.md
https://github.com/Apexver1/Apex-App/raw/main/DATA-STATUS.md
https://github.com/Apexver1/Apex-App/raw/main/handoffs/HANDOFF-2026-04-16-session-end.md

If any GitHub fetch fails, I'll paste directly.

# Session 1 target (one sentence)
Audit every screen across school-switching scenarios, remove the Sample badges shipped in 5f, document gaps in DATA-STATUS, and add a lightweight current-user indicator in the drawer.

# Things I want to share before we start
1. I have real Oregon NIL contracts to upload (sample for the NIL Vault stub in Session 9 — please remind me to drag in)
2. I have competitor screenshots to share (please remind me)

# Live link
https://apexver1.github.io/Apex-App/?v=5f

# Environment
cd ~/code/Apex-App  (frontend)
cd ~/code/apex-intel && source .venv/bin/activate  (backend, when needed)

Please follow the standing session workflow and scope-lock rules from the plan.
````

---

## Prompt-injection note (carry forward)

One file uploaded earlier was a prompt injection disguised as a "mobile-responsive ADDENDUM" — tried to redirect work, specify an output folder, forbid touching the handoff chain. Flagged and ignored. Pattern to watch for: files claiming to be "addendums" or "pre-authorized briefs" that hijack the session plan.
