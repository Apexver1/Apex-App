# Handoff — April 15, 2026 (night, design rebuild attempt)

## Session goal
Comprehensive design redesign of `index.html` based on the **Apex Intel — Product Direction & Feature Requests** document compiled from yesterday's walkthrough. Brief: completely update the design while preserving every working feature already shipped, accept dummy data where real data isn't wired, ship a single replacement `index.html`.

## Session result
**Did not ship.** Built ~70% of the scope in `/home/claude/index.html` (3,302 lines, up from production's 2,474). On final validation, `node --check` returned `SyntaxError: Unexpected end of input` — one unmatched `{` somewhere in the new code. Did not have time to localize and fix in the same session. **Restart fresh in a new chat with the production file as base.** Do not try to recover the broken work-in-progress.

## Phase 5a scope (locked, to execute in new chat)

These are the 8 visible upgrade items. Every other item from the product direction document is explicitly deferred to Phase 5b or later — full deferred list at bottom.

### 1. Login + entry
- BETA chip next to "Apex Intel" wordmark on login screen
- BETA chip next to "Apex Intel" wordmark in app top bar (smaller, .beta-chip-sm)
- Splash screen between login and app: off-white background, "Apex Intel" wordmark in navy Fraunces, "Welcome back, Coach [LastName]." personalized greeting, "[School] · 2025–26 season" subtitle, navy pill CTA button reading **"Build a Champion →"**
- New `enterApp()` function called by splash CTA — hides splash, shows app, calls `render()`
- Auto-login bootstrap auto-skips splash when `AUTO_LOGIN=true` (override with `?splash=1` URL param to see splash on auto-login)
- `signOut()` clears splash element along with app

### 2. Drawer
- Rename "CRM" → **"Apex CRM"**
- Rename "AI Scout" → **"Apex Scout"**
- Add new **OPERATIONS** group (third group after Main and Tools) containing:
  - Apex Patrons (with NEW tag)
  - Apex Ops (with NEW tag)

### 3. Centered modals
- CSS override on `.mbg` and `.mdl`: change from bottom-anchored slideup to centered popIn animation
- All existing modals (player detail, scoring, NIL request, school picker, scout player) automatically benefit, no per-modal changes needed
- Hide `.mdl-grab` (no longer needed when not bottom-anchored)

### 4. Photo avatar system
- New helper: `photoAv(name, size, photoUrl?)` — returns initials avatar with single navy background. If `photoUrl` provided, uses it as background-image with initials as fallback color overlay.
- Apply on every player card: Roster, Recruiting (Portal/HS/Watchlist), Scout results, CRM phonebook rows, nested CRM contact rows, donor rows
- Real photo URLs are Phase 5b — until then everyone gets navy initials

### 5. Universal "Add to CRM" button
- New helper: `addCrmBtn(prospectId)` — returns small `+` button (28px circle, accent blue), turns green checkmark when added
- Backed by `ADDED_TO_CRM` Set in JS state (frontend-only for tonight, real Supabase write is Phase 5b)
- Toggle handler: `toggleAddCrm(prospectId)` with toast feedback
- Appears on every prospect card across Recruiting, Smart Filter, AI Scout results

### 6. Apex CRM full redesign
This is the biggest piece. Replace `renderCRM()` entirely.

**Header** (per Michael's specific feedback):
- "A — Z" on left in serif Fraunces
- "X players · Y contacts" on right in small gray text
- Smaller font, hairline divider below

**Smart Suggestions card** at top:
- Computes from `prospects` + `crmLogs` + cadence rules
- 5-star recruits: red flag if 5+ days since last contact OR never contacted
- 4-star recruits: red flag if 7+ days
- "Time to text [Name] ★★★★★ — Xd since [method]" format
- Shows top 3, sorted by urgency

**Filter chips:**
- Stars filter: All / 5★ / 4★+ / 3★+ / 2★+ (replaces tier letter filter A/B/C/D)
- Action chips: Expand all / Collapse all / Import vCard

**A-Z phonebook structure:**
- Sort prospects A-Z by player last name
- Sticky letter section headers (e.g., "B" header before Bidunga, Brown, etc.)
- Each player rendered as a `.crm-player-card`:
  - Tap row to expand/collapse (`crmExpanded` Set tracks state)
  - Photo avatar + name + stars (interactive — click sets star rating, persists to `prospect_shortlist.priority_tier`)
  - Position · school · class line
  - Right side: "X contacts" count + "Yd ago" or "No contact" with red color when overdue
- When expanded, shows nested `.crm-nested` block:
  - Player themselves first (PLAYER tag)
  - Real `familyMap[p.id]` entries (parents from prospect_family table)
  - Dummy entries from `dummyContactsFor(prospect)` — auto-generates HS coach, AAU coach for HS prospects, agent for portal prospects (until real prospect_contacts schema lands in Phase 5b)
  - Each nested row: photo + name + role + last contact meta + assigned coach avatars + tap to open `openContactProfile()`
- Floating "+New" FAB at bottom-right

**Stars component:**
- New helper: `stars(n, size, interactive?, onClickFn?)` — returns inline star group
- `tierToStars(tier)` mapper: A=5, B=4, C=3, D=2, else 0
- `setStars(prospectId, n)` writes back to `priority_tier` column (n>=5→"A", >=4→"B", >=3→"C", >=2→"D", else null)

**Contact profile modal** (`openContactProfile(contactId, prospectId)`):
- Photo header with gradient background strip behind it
- Big xxl avatar (88px), serif name, role subtitle
- Editable star rating
- Three big action tiles: Call / Text / Email (toast for now, wire to tel:/sms:/mailto: in Phase 5b)
- Detail rows: Phone, Email, Social, Linked-to player, Last contact
- Assigned coaches multi-select chip row
- Log Contact section with date/time picker (defaults to now)
- Edit / Export vCard buttons at bottom

### 7. Apex Patrons module (NEW — frontend stub)
Full new render function `renderPatrons()`. All seed data, no backend.

**Sections in order:**
- Header: "APEX PATRONS" eyebrow + "Donor & development" Fraunces title
- KPI grid (2x2): Lifetime giving, YTD raised, Active donors, Lapsed (12mo+) with red risk amount
- Smart Suggestions card driven by `donor.tag` flags ("Lapsed risk · stewardship overdue" etc.)
- Travel Intelligence card: "→ Next road trip · Mar 8 / Eugene → Tucson / 7 donors in the area..."
- Active Campaigns: 3 progress-bar cards (NIL Collective drive, Casanova renovation, Annual fund)
- Tier filter chips: All / Champion / Platinum / Gold / Silver / Bronze
- Donor list (sorted by lifetime giving desc): photo, name, tier chip, company/city, lifetime amount, last gift
- AI Prospecting input box at bottom

**Donor profile modal** (`openDonor(donorId)`):
- Gold-tinted header gradient
- xxl photo + name with tier chip + company/city/alma
- Risk warning banner if `donor.tag` set
- Trio: Lifetime / YTD / Last gift
- Giving sparkline (9 years of bars) in a card
- Three big action tiles: Outreach / Visit / Gift
- Touchpoints table: Last contact, Last gift, Officer
- Next stewardship action buttons

**Seed data:** `DONORS_SEED` (12 donors with realistic data + tags), `CAMPAIGNS_SEED` (3 campaigns)

### 8. Apex Ops module (NEW — frontend stub)
Full new render function `renderOps()`. All seed data, no backend.

**Sections in order:**
- Header: "APEX OPS" eyebrow + "Recruiting materials hub" Fraunces title
- 6-category icon grid (2x3): Recruiting decks (PPT) · Scouting reports (PDF) · Family touchpoints (IMG) · NIL pitch packets (NIL) · Portal reports (DD) · Team materials (DOC)
- Each tile shows category icon + name + file count
- AI Assist input: "Apex, draft a recruiting deck for [player]..."
- Recent Activity feed: 5 rows showing staff actions with timestamps
- Recent Decks list: 3 mat-row entries with PPT icon, file name, size, last update, author, view count

**Seed data:** `OPS_CATS` (6), `OPS_RECENT` (5), `OPS_DECKS` (3)

### Other small wins (also in Phase 5a)
- **Editable competing schools** in player detail modal: chip-input replaces static chips. Type school name + Enter to add, click × to remove. Persists via PATCH to `prospect_shortlist.competing_schools` (falls back to local-only update if column write fails).
- **Date/time picker on Log Contact**: replaces "logged as now". `<input type="datetime-local">` defaults to now. `logC()` honors picker value.
- **Coach scoring Weighting dropdown removed**: 4 sliders + notes only. Hidden input preserves DB compat (defaults to "equal").
- **Smart Suggestions on Home dashboard**: same cadence rules as CRM Smart Suggestions, shown between greeting and brief tiles.
- **Home tiles deep-link**: `tileHTML()` extended to accept `nav` param. Each Home tile clicks through to its relevant tab (NIL requests → budget, CRM follow-ups → crm, Portal churn → recruiting, Visits this week → visits, etc.)
- **Watchlist as 4th Recruiting tab**: Portal / High school / Intl / Watchlist. Backed by `WATCHLIST` Set. Each prospect card has ☆ next to name to toggle. Empty state shows "Tap the ☆ on any prospect card to add."

## Preserved exactly (zero touch)

- All 12 Supabase endpoints in `loadData()`
- Auth flow (`doLogin`, `signOut`)
- School picker including stewart2026 password gate
- `valueNil()` NIL valuation formula
- `openScoutPlayer()` D1-universe modal
- `saveScore()` (only the openScore UI changed, save logic identical)
- All NIL budget logic + 3-tier alerts (critical/warning/info)
- All slot management with planned→offered→accepted→declined workflow
- `renderVisits()` complete with mini-cal, ICS export, materials list, upload-drop
- `renderFilter()` with all 15+ filter dimensions
- `renderScout()` v1.1.1 with `mdToHtml`, tool call tracing, two-mode (rules/LLM)
- `renderBudget()` and `renderBuilder()`
- Auto-login bootstrap (extended only to handle splash skip)

## Deferred to Phase 5b+

These were in the product direction doc but explicitly out of scope for tonight:

- **Apex Scout full upgrade**: always-LLM (no rules toggle), voice that actually works well, multi-turn conversational input, persistent context, proactive alerts, per-school strategy auto-included
- **NIL contract upload + AI parsing**: PDF upload → Claude extracts payment schedule, obligations, milestones → auto-generate dashboard reminders
- **+ New visit flow**: form modal + `prospect_visits` table migration + itinerary builder + storage upload
- **Real player photos**: ESPN/247Sports headshot URLs on roster + portal + HS prospects
- **Real Apex Patrons backend**: donor schema, prospect research, communications manager (Resend/SendGrid + Twilio), travel intelligence wiring
- **Real Apex Ops backend**: file storage buckets, version history, sharing tracking, AI deck generation
- **Push notifications**: portal entry alerts, cold contact warnings, deadline reminders, NIL payment due, donor stewardship triggers
- **Multi-school RLS**: `staff` table needs `school_id` column, role-based views (head coach / assistant / ops / patrons)
- **Smart Filter advanced analytics**: KenPom team context, Torvik advanced player stats, Synergy play types, shot profiles, defensive metrics, portal-specific signals
- **Customizable home dashboard**: tile-by-tile show/hide, section toggles, persisted per user
- **Logo development**
- **`prospect_contacts` schema**: generalize `prospect_family` to support AAU coaches, HS coaches, agents, trainers, siblings (currently faked via `dummyContactsFor()` JS generator)
- **Real Add to CRM persistence**: backend write when + button is tapped (currently frontend-only `Set`)
- **Real Watchlist persistence**: per-user starred prospect set in DB (currently frontend-only `Set`)
- **Color discipline pass**: green for good, red for bad, strip everything else (partially done in earlier prototype iteration, full pass deferred)

## The blocker

```
SyntaxError: Unexpected end of input
```

`node --check` on the extracted script block from `/home/claude/index.html` shows depth +1 at end of file — one unclosed `{` somewhere. Brace-strip heuristic with string/comment removal pointed at line 2214 (inside `renderBuilder`), but that function looks intact on direct inspection. Suspect the actual unclosed brace is in one of the new functions I added (`renderCRM`, `openContactProfile`, `renderPatrons`, `openDonor`, `renderOps`) and the line tracker was off due to multiline template strings.

**Strategy for new chat**: don't recover the broken file. Restart from production `index.html` (clean baseline). Apply edits in **larger contiguous blocks** (full function replacements) rather than dozens of small partial str_replaces. Run `node --check` after each major batch. Only present the file when validation passes.

## Files in play

| File | Status | Notes |
|---|---|---|
| `~/code/Apex-App/index.html` | **Working — production** | 2,474 lines. This is the baseline. Don't touch in the broken session. |
| `/home/claude/index.html` (last session) | **Broken — discard** | 3,302 lines. Has unclosed `{` somewhere. Not worth recovering. |
| `/mnt/user-data/outputs/apex-intel-design-preview.html` | Working — visual prototype | Standalone, no Supabase. Useful as visual reference for what Patrons/Ops/CRM phonebook should look like. |

## Reference constants (carry forward)

- Repo: [github.com/Apexver1/Apex-App](https://github.com/Apexver1/Apex-App)
- Live URL: [apexver1.github.io/Apex-App/](https://apexver1.github.io/Apex-App/)
- Backend repo: [github.com/Apexver1/apex-intel](https://github.com/Apexver1/apex-intel)
- Local frontend path: `~/code/Apex-App/`
- Local backend path: `~/code/apex-intel/`
- Supabase project: [midyxvjfoggchbugxzkd.supabase.co](https://midyxvjfoggchbugxzkd.supabase.co)
- Oregon UUID: `d4dd73c7-2ec6-49bf-b63b-5c4a333a2bf7`
- Auto-login email: `apex@test.com`
- Auto-login password: `ApexTest123`
- School picker password: `stewart2026`
- Cache-bust convention: append `?v=patchN` to GitHub Pages URL

## Database state (unchanged this session)

- 7,526 total players in `school_rosters` (5,677 D1 + 657 HS + 1,192 portal)
- 4,844 portal players have Bart Torvik 2024-25 advanced stats
- 12 Supabase endpoints feeding `loadData()`: `my_roster`, `prospect_shortlist`, `team_strategy`, `scholarships`, `prospect_stats`, `crm_log`, `nil_budget`, `nil_budget_requests`, `prospect_scores`, `prospect_family`, `contact_reminders`, `schools`

## Repository state (unchanged this session)

- 6 commits on `main` (no new commits tonight — work was in `/home/claude/`, never pushed)
- Latest commit: `e38c9f9` — "Handoff: Scout 1.1.1 shipped + 722 portal players backfilled with BT 2024–25 stats"

---

## NEW CHAT — session starter

Open a fresh chat in claude.ai. Paste this as your first message:

````
Loading project context for Phase 5a — Apex Intel design rebuild.

Please fetch:
https://github.com/Apexver1/apex-intel/raw/main/CLAUDE.md
https://github.com/Apexver1/apex-intel/raw/main/handoffs/HANDOFF-2026-04-15-night-design-rebuild.md

I'm going to upload my production index.html next. Once you've read both
the project context and this handoff, we'll execute Phase 5a as scoped.

Working agreements that are critical for this work:
1. Restart from production index.html — do NOT try to recover the broken
   /home/claude/index.html from last session.
2. Apply edits in larger contiguous blocks (full function replacements)
   rather than many partial str_replaces.
3. Run `node --check` to validate JS syntax after each major batch of
   edits before moving on.
4. Only present the final file when validation passes.
5. Deliver as a downloadable file via present_files. I'll drag-replace
   into ~/code/Apex-App/index.html, then commit + push from Terminal.
````

Then in the next message, drag in your production `~/code/Apex-App/index.html` and let the new session execute the scope above.

## Steps after the new chat ships the file

Once the new chat presents the working file, run these in Terminal:

1. Open Finder window:
   ```
   open ~/code/Apex-App/
   ```
2. Drag the new `index.html` from Downloads into `~/code/Apex-App/`, replacing the existing one.
3. Commit and push:
   ```
   cd ~/code/Apex-App && git add index.html && git commit -m "Phase 5a: design rebuild — splash, BETA, Apex CRM phonebook, Apex Patrons + Apex Ops, photos, Add to CRM, Watchlist, centered modals" && git push origin main
   ```
4. Wait ~30 seconds for GitHub Pages to rebuild, then visit with cache-bust:
   - [https://apexver1.github.io/Apex-App/?v=5a](https://apexver1.github.io/Apex-App/?v=5a)
5. End that session by generating the next handoff doc.
