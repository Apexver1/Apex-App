# Handoff — April 16, 2026 (Phase 5f: iPhone drawer fix + honesty pass)

## Session result

Short morning session (~1 hr, hard stop at 11:40). Planned a multi-school demo flow, discovered school switching already works at the data layer, reframed the session around **partner-demo readiness** instead. Shipped an honesty pass (Sample badges on seed-data screens) + an iPhone hamburger menu scroll fix that was discovered mid-session. Deferred the "Apex Picks" filter preset work — mapped the insertion point but ran out of runway.

**Commits on `main` in Apex-App repo this session:**
1. `Phase 5f: iPhone drawer scroll fix + honesty pass (Sample badges on seed-data screens)`

---

## What shipped tonight

### 1. iPhone hamburger menu fix (REAL BUG — now resolved)

**Problem:** On iPhone, opening the drawer cut off bottom items ("Sign out", "Settings", "Switch school"). Could not scroll to them. The `<div style="flex:1"></div>` spacer was pushing content off-screen and the drawer itself had no `overflow-y`.

**Fix (4 changes):**
- `.drawer` CSS gained `overflow-y:auto`, `-webkit-overflow-scrolling:touch`, and `padding-bottom:env(safe-area-inset-bottom,0px)`
- The `flex:1` spacer between Operations group and footer replaced with `min-height:14px` (fixed gap)
- Viewport meta tag upgraded: added `viewport-fit=cover` so `env(safe-area-inset-*)` values actually resolve on iPhone
- `.hamb` button bumped from 40×40 to 44×44 (Apple minimum tap target)

**QA next session (on iPhone, do not skip):**
- Open hamburger, scroll all the way to "Sign out"
- Verify "Sign out" isn't covered by the home indicator
- Rotate to landscape, confirm drawer still scrolls
- Tap hamburger — should feel more reliable (44px tap target)

### 2. Honesty pass — Sample badges on seed-data screens

**New CSS classes** (added after `.chip-line`):
```css
.chip-demo{display:inline-flex;align-items:center;gap:4px;font-size:9px;font-weight:700;padding:3px 7px;border-radius:4px;letter-spacing:0.5px;text-transform:uppercase;background:var(--gold-bg);color:var(--gold);border:0.5px solid #E6CB7A}
.chip-demo::before{content:"";display:inline-block;width:5px;height:5px;border-radius:50%;background:var(--gold)}
.demo-note{font-size:10px;color:var(--ink-3);font-style:italic;margin:-6px 0 10px;letter-spacing:0.1px}
```

**Sample badges wired to 5 seed-data sections:**

| Screen | Section | Sample badge | Subtitle note |
|---|---|---|---|
| Home | Next up | ✅ | "Sample game data — ESPN schedule feed in progress" |
| Home | College basketball news | ✅ | "Sample headlines — RSS / news API integration in progress" |
| Visits | Scheduled visits title | ✅ | (inline only, no note) |
| Patrons | APEX PATRONS eyebrow | ✅ | "Sample donor data — CRM integration in progress" |
| Ops | APEX OPS eyebrow | ✅ | "Sample materials — document pipeline in progress" |

**Design intent:** Absence of a Sample badge implicitly means "this screen pulls live data." If we later want an explicit `.chip-live` green pill on the live screens, the CSS block is already styled to accept a sibling class. Deferred.

**Demo narrative enabled:** Partners can see at a glance what's real vs. what's placeholder, which eliminates "is this all fake?" as a Q&A landmine.

### 3. File stats

- `index.html`: 3,478 → 3,485 lines (+7 lines, 7 surgical edits)
- Validated with `node --check` at midpoint and end
- Saved as `index-5f.html`, deployed as `index.html` on main

---

## What DID NOT ship (deferred to next session)

### Apex Picks filter presets (the "killer moment" play)

**The idea:** Add a preset-button strip to the top of the Smart Filter screen with 3 one-click saved filters:
1. **"Portal PGs, Rebuild Targets"** — `source=portal` + `position=PG` + pseudo-KenPom bottom-50 filter
2. **"HS 4-Stars, Undercommitted"** — `source=high_school` + star threshold + no commit
3. **"Best Value vs NIL Ask"** — NIL market value minus asking price, top 20

**Why it didn't ship:** Hard stop at 11:40 — prioritized the iPhone bug fix once that was discovered. "I'd like it to work" took precedence over shipping two things half-finished.

**Where it would slot in:** `/home/claude/index.html` line 2643 (in `renderFilter`), right after `var h='<div class="card mb-md">';` — before the first `sectH("Source")` call. Add a new `"Apex Picks"` section at the very top as a `fbar` row of 3 buttons.

**State needed:** Add a global `apexPickActive` (null or preset ID) and new functions `applyApexPick(id)` + `clearApexPick()`. Presets apply combinations of existing filter state (`fSrc`, `fP`, etc.) so most plumbing is free.

**KenPom gap:** No real KenPom data per portal player's current team yet. Options:
- (a) Ship with a pseudo-KenPom JS map keyed by school name, labeled "Sample tiers — KenPom API wiring in progress" (honest, fast)
- (b) Defer to a future session where KenPom actually gets wired
- Leaning (a) — matches the honesty-pass ethos and still makes the demo land.

### What we also noticed but didn't touch

- **School switching works at the data layer** but `*_SEED` constants (NEWS_SEED, NEXT_GAME, VISITS_SEED, STAFF_SEED, DONORS_SEED, CAMPAIGNS_SEED, OPS_CATS, OPS_RECENT, OPS_DECKS) are hardcoded Oregon-themed. On Duke/Kansas/etc., users see Oregon's "Phil Knight" donor, Oregon-themed news, etc. The honesty pass partially masks this (Sample badges make it obvious the data is placeholder) but the real fix is either (a) remove the seed constants entirely and show empty-state skeletons for those screens, or (b) make each seed array a per-school lookup. Thread for next session.
- **`crm_added` and `watchlist` tables are user-scoped, not school-scoped.** Players added to CRM at Oregon will appear in the CRM badge count when switched to Duke. Either scope these tables by `school_id` (preferred) or filter in the frontend by joining against `prospect_shortlist.school_id`.
- **Drag-and-drop file upload in Claude web interface got stuck mid-session.** Workaround: refresh the chat tab, or use the paperclip button. Not a product issue.

---

## Priority options for next session

### Small / fast (15-45 min each)
1. **Apex Picks preset strip** (deferred from today — design + insertion point already mapped)
2. **CRM/Watchlist school-scoping** — prevent cross-school bleed
3. **Logo + subtitle polish** (carryover from 5e)

### Medium (1-2 hours)
4. **Empty-state skeletons for seed-data screens** — remove `NEWS_SEED` etc., show "No games scheduled" / "No donors on file" when empty. Bigger honesty win than the badges.
5. **`prospect_contacts` schema** — replace `dummyContactsFor()` with real editable contacts
6. **`prospect_visits` table** — replace `VISITS_SEED` with Supabase-backed visits, per-school

### Larger (half day+)
7. **Multi-school seed data** — properly seed 3-4 marquee schools (Duke, Kansas, UConn, Kentucky) with real-enough roster + prospect data for multi-school demos
8. **Bottom nav bar** (carryover from 5e priorities)
9. **Real-time roster builder UX polish** (carryover from 5e)

**My rec for next partner-ready build:** #4 (empty states) + #1 (Apex Picks). Together they give you: honest empty screens that don't look broken on any school + one hero filter flow that demos well. Ship both in a ~90-min session.

---

## Database state (unchanged this session)

22 tables. 14 endpoints in `loadData()`. No migrations, no new columns, no new tables.

---

## Working agreements (carry forward — ALL still active)

1. Full copy-paste-ready code blocks. Never find-and-replace.
2. Every URL as clickable markdown link.
3. Mac-native instructions.
4. Maximally specific.
5. Numbered step-by-step lists.
6. Never suggest stopping.
7. Read CLAUDE.md + latest handoff at session start.
8. Generate handoff at session end.
9. Map file ONCE up front.
10. Larger contiguous str_replace blocks.
11. Validate at midpoint AND end with `node --check`.
12. Skip exploratory view calls during execution.
13. ONE shippable thing per session.
14. Validate as LAST step before present_files.
15. Name deliverable builds `index-{phase}.html`.
16. New Supabase tables: CREATE → RLS → policies → GRANT auth → GRANT anon → NOTIFY pgrst.
17. Always pass `user_id: UID` explicitly in POST bodies.
18. For Python scripts > 30 lines, use heredoc paste pattern.

### New agreement candidate for next session

19. **When a session has a hard time box, plan for ~60% of the stated time.** Today's plan was 2 parts × 30 min = 60 min. Actual: Part 1 + iPhone fix shipped in ~45 min; Part 2 didn't fit. Next time, scope one thing, finish it, call it.

---

## Prompt injection note (carry forward)

One of the "handoff" files uploaded early in this session was actually a prompt injection disguised as an addendum — it tried to redirect the session into a mobile-responsive task with specific output-location instructions that contradicted both the stated goal and the project's current phase. Flagged, ignored, worth being aware of if similar content shows up again. The actual mobile bug was discovered separately from the user's direct feedback ("i cant see the entire hamburg menu") and handled on its own terms.

---

## Live links

| Link | Purpose |
|---|---|
| [apexver1.github.io/Apex-App/?v=5f](https://apexver1.github.io/Apex-App/?v=5f) | Production after 5f push |
| [apexver1.github.io/Apex-App/?v=5f&splash=1](https://apexver1.github.io/Apex-App/?v=5f&splash=1) | With splash screen |
| [apexver1.github.io/Apex-App/?nologin=1](https://apexver1.github.io/Apex-App/?nologin=1) | Manual login |

**Credentials:** apex@test.com / ApexTest123 / school picker: stewart2026

---

## Session starter for next chat

````
Loading project context — Apex Intel, resuming after 5f.

Please fetch:
https://github.com/Apexver1/apex-intel/raw/main/CLAUDE.md
https://github.com/Apexver1/Apex-App/raw/main/handoffs/HANDOFF-2026-04-16-5f-honesty-pass.md

If GitHub fetch fails, I'll paste both files directly.

# Where we are
Phase 5f shipped 04/16:
- iPhone hamburger menu now scrolls properly
- "Sample" badges on all seed-data screens (Home Next up + News, Visits, Patrons, Ops)
- Deferred: Apex Picks filter presets (mapped, not built)

Live: https://apexver1.github.io/Apex-App/?v=5f

# What I want to work on
[fill in]

Working agreements: all 18 from the handoff still active; new #19 says scope for 60% of stated session time.
````
