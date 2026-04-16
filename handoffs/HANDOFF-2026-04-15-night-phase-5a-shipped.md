# Handoff — April 15, 2026 (night, Phase 5a shipped)

## Session result

**Phase 5a shipped.** All 8 visible upgrades + 6 small wins from the locked Phase 5a scope are live in `~/code/Apex-App/index.html`. JS validates clean at 3,379 lines (up from production's 2,474). File was presented via `present_files` and is ready to drag-replace, commit, and push.

This was the third attempt at Phase 5a. First two sessions failed with `SyntaxError: Unexpected end of input` on final validation. Third session succeeded by:
1. Mapping the entire production file ONCE up front (74 `window.foo` functions, line ranges for CSS/body/script blocks)
2. Using larger contiguous `str_replace` blocks (full function replacements) instead of dozens of partial edits
3. Validating with `node --check` at the midpoint AND at the end
4. Skipping exploratory `view`/`grep` calls during execution — relied on the up-front map

## What shipped — full inventory

### Login + entry flow
- BETA chip on login wordmark
- BETA chip in topbar (smaller, `.beta-chip-sm`)
- BETA chip in drawer header
- Splash screen (`#splash`) between login and app
  - Off-white background, "Apex Intel" wordmark in navy Fraunces
  - Personalized "Welcome back, Coach [LastName]." greeting
  - "[School] · 2025–26 season" subtitle
  - Navy pill CTA: **"Build a Champion →"**
- `enterApp()` function — hides splash, shows app, calls `render()`
- `populateSplash()` — pulls coach name from `coachFor(CURRENT_SCHOOL_ID)`
- Auto-login bootstrap auto-skips splash via `window.SKIP_SPLASH`
- Override with `?splash=1` URL param to see splash on auto-login
- `signOut()` clears splash element along with app

### Drawer
- "CRM" → **"Apex CRM"**
- "AI Scout" → **"Apex Scout"**
- New **OPERATIONS** group (third group after Main and Tools):
  - Apex Patrons (with NEW tag — `.drawer-tag` gold pill)
  - Apex Ops (with NEW tag)

### Centered modals
- `.mbg`: centered flex (was bottom-anchored)
- `.mdl`: 18px border-radius all corners (was 22px top only), `popIn` cubic-bezier animation (was `slideup`)
- `.mdl-grab`: `display:none` globally
- `@media(max-width:520px)`: bottom slideup preserved on mobile via override
- All existing modals (player detail, scoring, NIL request, school picker, scout player) inherit this automatically

### Photo avatar system
- New helper: `photoAv(name, size, photoUrl?)`
  - Sizes: `sm` (22px), `md` (32px), `lg` (44px), `xl` (64px), `xxl` (88px)
  - With `photoUrl`: `background-image` + initials fallback
  - Without: navy background + initials only
- New helper: `initialsOf(name)` — returns `[F][L]` from "First Last"
- Wired into:
  - Roster cards (`renderRoster`)
  - Recruiting cards (Portal/HS/Watchlist via `_recCard` shared helper)
  - Smart Filter result cards (`renderFilter`)
  - Apex Scout result cards (`renderScout`)
  - CRM phonebook player rows + nested contact rows (`renderCRM`)
  - Donor rows + donor profile modal (`renderPatrons`/`openDonor`)
  - Contact profile modal (`openContactProfile`)
- Real photo URLs are Phase 5b — until then everyone gets navy initials

### Universal Add to CRM button
- New helper: `addCrmBtn(prospectId)` — 28px circle, accent-blue `+`, turns green ✓ when added
- Backed by `ADDED_TO_CRM` Set (frontend-only, frontend-only `Set` for now)
- `toggleAddCrm(prospectId)` with toast feedback
- Wired into recruiting cards, watchlist cards, Smart Filter cards, Apex Scout result cards
- Real Supabase persistence is Phase 5b

### Watchlist (4th Recruiting tab)
- `WATCHLIST` Set (frontend-only)
- `wlStarBtn(prospectId)` — ☆/★ toggle on every prospect card
- `toggleWatch(prospectId)` with toast feedback
- 4th Recruiting tab: Portal / High school / Intl / **Watchlist (N)**
- Empty state shows "Tap the ☆ on any prospect card to add."
- Reuses `_recCard()` for consistent rendering
- Real persistence is Phase 5b

### Apex CRM full redesign
- Header: "A — Z" Fraunces left, "X players · Y contacts" right, hairline divider
- Smart Suggestions card at top:
  - Computed via `computeSmartSuggestions()`
  - 5★: red flag if 5+ days since last contact OR never contacted
  - 4★: red flag if 7+ days
  - 3★: red flag if 14+ days
  - Top 3 sorted by stars desc, then urgency desc
- Star filter chips: All / 5★ / 4★+ / 3★+ / 2★+
- Action chips: Expand all / Collapse all / Import vCard
- A-Z phonebook with sticky letter section headers (sorted by last name)
- Each player rendered as `.crm-player-card`:
  - Tap row to expand/collapse (`crmExpanded` Set tracks state)
  - Photo avatar + name + interactive stars
  - Position · school · class meta
  - Right side: "X contacts" + "Yd ago" / "No contact" with red overdue color
- Expanded view: nested `.crm-nested` block
  - Player themselves first (PLAYER tag)
  - Real `familyMap[p.id]` entries (parents from `prospect_family` table)
  - Auto-generated dummies via `dummyContactsFor()` (HS coach + AAU coach for HS prospects, agent for portal prospects)
  - Each nested row: photo + name + role tag + meta + assigned coach avatars
- Floating "+" FAB at bottom-right (toast for now)
- Stars helpers:
  - `stars(n, size, interactive?, onClickFn?)` — returns inline star group
  - `tierToStars('A')` → 5, etc.
  - `starsToTier(5)` → 'A', etc.
  - `setStars(prospectId, n)` writes back to `prospect_shortlist.priority_tier`

### Contact profile modal
- `openContactProfile(contactId, prospectId)` — opens any nested contact
- Photo header with accent-blue gradient strip
- Big xxl avatar (88px), serif name, role subtitle
- Editable star rating (writes through to `setStars`)
- Three big action tiles: Call / Text / Email (toast for now, wire to `tel:` / `sms:` / `mailto:` in Phase 5b)
- Detail rows: Phone, Email, Social, Linked-to player, Last contact
- Assigned coaches multi-select chip row (visual toggle, not yet persisted)
- Log Contact section with `<input type="datetime-local">` defaulting to now
- Edit / Export vCard footer buttons

### Apex Patrons module
Full new render function `renderPatrons()`. All seed data, no backend.
- Header: "APEX PATRONS" eyebrow + "Donor & development" Fraunces title
- KPI grid (2x2): Lifetime giving · YTD raised · Active donors · Lapsed (12mo+)
- Smart Suggestions card driven by `donor.tag` flags
- Travel Intelligence card: "→ Next road trip · Mar 8 / Eugene → Tucson / 7 donors in the area..."
- 3 Active Campaigns with progress bars
- Tier filter chips: All / Champion / Platinum / Gold / Silver / Bronze
- Donor list (sorted by lifetime giving desc): photo, name, tier chip, company/city, lifetime amount, last gift
- AI Prospecting input box at bottom

`openDonor(donorId)` modal:
- Gold-tinted header gradient
- xxl photo + name + tier chip + company/city/alma
- Risk warning banner if `donor.tag` set
- Trio: Lifetime / YTD / Last gift
- 9-year giving sparkline with year labels
- Three big tiles: Outreach / Visit / Gift
- Touchpoints rows: Last contact, Last gift, Officer
- Add note / Create task footer

Seed data: `DONORS_SEED` (12 donors), `CAMPAIGNS_SEED` (3 campaigns)

### Apex Ops module
Full new render function `renderOps()`. All seed data, no backend.
- Header: "APEX OPS" eyebrow + "Recruiting materials hub" Fraunces title
- 6-category icon grid (2x3): Recruiting decks (PPT) · Scouting reports (PDF) · Family touchpoints (IMG) · NIL pitch packets (NIL) · Portal reports (DD) · Team materials (DOC)
- Each tile: category icon + name + file count
- AI Assist input
- Recent Activity feed (5 staff actions with timestamps)
- Recent Decks list (3 entries with PPT icon, file name, size, author, view count)

Seed data: `OPS_CATS` (6), `OPS_RECENT` (5), `OPS_DECKS` (3)

### Other small wins
- **Editable competing schools** in player detail modal — chip-input replaces static chips
  - Type school name + Enter to add, click × to remove
  - `addCompeting(pid, val)` / `removeCompeting(pid, school)` helpers
  - Persists via PATCH to `prospect_shortlist.competing_schools`, falls back to local-only if column write fails
- **Datetime-local picker on Log Contact** — replaces "logged as now"
  - `<input type="datetime-local" id="cd">` defaults to now
  - `logC()` honors picker value (`cd` element); falls back to `Date.now()` if absent
- **Coach scoring Weighting dropdown removed** — 4 sliders + notes only
  - `<input type="hidden" id="sc-preset" value="equal">` preserves DB compat
- **Smart Suggestions on Home dashboard** — same `computeSmartSuggestions()` rules as CRM, shown between greeting and brief tiles
- **Home tiles deep-link** — `tileHTML(label, value, sub, cls, nav)` extended with `nav` param
  - NIL requests → budget, CRM follow-ups → crm, Portal churn → recruiting, Visits → visits, Open scholarships → builder, Reminders → crm

### Preserved exactly (zero touch confirmed)
- All 12 Supabase endpoints in `loadData()`
- Auth flow (`doLogin`, `signOut`) — only added splash hooks, didn't touch auth logic
- School picker including stewart2026 password gate
- `valueNil()` NIL valuation formula
- `openScoutPlayer()` D1-universe modal
- `saveScore()` (only the openScore UI changed, save logic identical)
- All NIL budget logic + 3-tier alerts (critical/warning/info)
- All slot management with planned→offered→accepted→declined workflow
- `renderVisits()` complete with mini-cal, ICS export, materials list, upload-drop
- `renderFilter()` with all 15+ filter dimensions (only added photoAv + addCrmBtn to result cards)
- `renderScout()` v1.1.1 with `mdToHtml`, tool call tracing, two-mode (rules/LLM) (only added photoAv + addCrmBtn to result cards)
- `renderBudget()` and `renderBuilder()`

---

## Codebase map (carry forward)

### File structure
| Lines | Section |
|---|---|
| 1–10 | HTML head + font preconnect |
| 11–~530 | `<style>` block (CSS) |
| ~531–~400 | `<body>` with login, splash, app shell, drawer, modal container |
| ~400–~3340 | Main `<script>` (74+ `window.foo` functions) |
| ~3340–end | Auto-login bootstrap `<script>` |
| 3,379 | Total |

### CSS class additions (Phase 5a)
All new classes live in a clearly-marked block: `/* PHASE 5a — Design rebuild additions */` near end of `<style>`.

Categories:
- `.beta-chip`, `.beta-chip-sm`, `.drawer-tag`
- `.splash-wrap`, `.splash-brand`, `.splash-greet`, `.splash-sub`, `.splash-cta`, `.splash-foot`
- `.av-xl`, `.av-xxl`, `.av-photo` (photo avatar variants)
- `.crm-add-btn`, `.crm-add-btn.added`
- `.wl-star`, `.wl-star.on`, `.wl-empty`
- `.stars`, `.stars.interactive`, `.stars.sm/.md/.lg`, `.st`, `.st.off`
- `.smart-card`, `.smart-head`, `.smart-title`, `.smart-row`, `.sr-dot`, `.sr-text`, `.sr-meta`
- `.crm-az-head`, `.crm-az-title`, `.crm-az-count`, `.crm-letter`, `.crm-player-card`, `.crm-player-row`, `.crm-pmain`, `.crm-pname`, `.crm-pmeta`, `.crm-pright`, `.crm-pcount`, `.crm-plast`, `.crm-plast.overdue`, `.crm-nested`, `.crm-nested-row`, `.crm-nmain`, `.crm-nname`, `.crm-nrole`, `.crm-nmeta`, `.crm-navs`, `.crm-fab`
- `.cp-header`, `.cp-close`, `.cp-av-wrap`, `.cp-name`, `.cp-role`, `.cp-stars`, `.cp-actions`, `.cp-action`, `.cp-ai`, `.cp-row`, `.cp-l`, `.cp-v`, `.cp-section-h`, `.cp-coaches`, `.cp-coach-chip`, `.cp-foot-btns`
- `.dt-inp` (datetime-local styling)
- `.chip-input`, `.ci-chip`, `.ci-x` (chip-input for editable competing schools)
- `.patron-tier`, `.tier-champion/platinum/gold/silver/bronze`, `.donor-row`, `.dr-main/.dr-name/.dr-meta/.dr-right/.dr-amt/.dr-last`, `.camp-card`, `.camp-head`, `.camp-name`, `.camp-pct`, `.camp-bar`, `.camp-fill`, `.camp-fill.gold/.pos`, `.camp-meta`, `.travel-card`, `.travel-eyebrow`, `.travel-text`, `.donor-modal-head`, `.risk-banner`, `.spark`, `.spark-bar`, `.spark-bar.last`, `.spark-card`, `.spark-yrs`, `.ai-prosp-input`
- `.ops-grid`, `.ops-tile`, `.ops-tile-icon`, `.ops-tile-name`, `.ops-tile-count`, `.ic-ppt/.ic-pdf/.ic-img/.ic-nil/.ic-dd/.ic-doc`, `.act-feed`, `.act-row`, `.act-text`, `.act-time`, `.mat-row`, `.mat-icon`, `.mat-main`, `.mat-name`, `.mat-meta`, `.mat-views`

### New JS state (added in `var staff=STAFF_SEED.slice();` block)
```javascript
var ADDED_TO_CRM = new Set();
var WATCHLIST = new Set();
var crmExpanded = new Set();
var crmStarFilter = "ALL";   // ALL / 5 / 4 / 3 / 2
var patronTier = "ALL";      // ALL / champion / platinum / gold / silver / bronze
```

### New seed data
- `DONORS_SEED` — array of 12 donor objects with: id, name, tier, company, city, alma, lifetime, ytd, last_gift, last_gift_date, last_contact, officer (initials), tag (string|null), giving (9-year array)
- `CAMPAIGNS_SEED` — array of 3: id, name, goal, raised, kind ("" | "gold" | "pos")
- `OPS_CATS` — array of 6: id, name, cls, tag, count
- `OPS_RECENT` — array of 5: who (initials), what, item, time
- `OPS_DECKS` — array of 3: name, size, updated, author (initials), views, type ("ppt")

### Function map (74+ window.foo functions, alphabetical)

**Helpers (existing + Phase 5a additions)**
- `addCompeting(pid, val)` — *Phase 5a*
- `addCrmBtn(prospectId)` — *Phase 5a*
- `api(path, opt)`
- `chip(t, c)`, `pill(t, c)`
- `coachFor(schoolId)`
- `computeSmartSuggestions()` — *Phase 5a*
- `countOverdueFollowups()`, `countRemindersToday()`, `nextReminderTime()`
- `crmCollapseAll()`, `crmExpandAll()` — *Phase 5a*
- `dateLong(d)`
- `dummyContactsFor(p)` — *Phase 5a*
- `enterApp()` — *Phase 5a*
- `esc(s)`, `fmt$(n)`, `jp(v)`, `sc(s)`
- `greetingText()`
- `initialsOf(name)` — *Phase 5a*
- `mdToHtml(md)`
- `photoAv(name, size, photoUrl?)` — *Phase 5a*
- `populateSplash()` — *Phase 5a*
- `removeCompeting(pid, school)` — *Phase 5a*
- `setCrmStarFilter(v)` — *Phase 5a*
- `setStars(prospectId, n)`, `starsToTier(n)`, `stars(n, size, interactive?, onClickFn?)`, `tierToStars(tier)` — *Phase 5a*
- `toast(msg, kind)`
- `toggleAddCrm(prospectId)`, `toggleCrmExp(prospectId)`, `toggleWatch(prospectId)` — *Phase 5a*
- `wlStarBtn(prospectId)` — *Phase 5a*

**Auth**
- `doLogin()` — *splash-aware in Phase 5a*
- `signOut()` — *splash-aware in Phase 5a*

**Data**
- `loadData()`, `updateDrawerIdentity()`, `updateCrmBadge()`

**Drawer + nav**
- `openDrawer()`, `closeDrawer()`, `navTo(t)`, `clearSF()`, `go=navTo`

**School picker**
- `openSchoolPicker()`, `renderPicker()`, `renderPickerResults()`, `filterSchools(q)`, `setPickerConf(c)`, `setPickerSort(s)`, `selectSchool(sid, sname)`

**Player + contact modals**
- `openP(id, kind)` — *Phase 5a: editable competing schools + datetime-local Log Contact*
- `closeM()`
- `openScoutPlayer(idx)`
- `openContactProfile(contactId, prospectId)` — *Phase 5a NEW*
- `updS(pid, st)`, `logC(pid)` — *Phase 5a: honors `cd` datetime-local*

**NIL valuation**
- `valueNil(p, ps)`, `tier=valueNil`

**Coach score**
- `calcComposite(s, w)`, `getWeights(preset)`
- `openScore(pid)` — *Phase 5a: Weighting dropdown removed (hidden field)*
- `saveScore(pid)`

**NIL budget request**
- `openReqModal(prefillAmt, prefillReason)`, `reqIncrease()`, `decideReq(id, status)`

**Slot management**
- `openSlot(slotNum)`, `filterSlotList(q, slotNum)`, `commitP(pid, slot, nil)`, `openNilEdit(pid, slot, suggestedNil)`, `nilEditSync(val, otherCommitted, totBudget)`, `saveCommit(pid, slot)`, `changeSlotStatus(slot, newStatus)`, `markOffered(slot)`, `markAccepted(slot)`, `markDeclined(slot)`, `openNilEditExisting(slot)`, `removeSlot(slot)`

**Visits**
- `vSelDay(dateStr)`, `vClearSel()`, `vDateKey(d)`, `renderMiniCal(visitsList, monthOffset, selectedKey)`, `icsForVisit(v)`, `downloadIcs(vid)`, `openVisit(vid)`, `uploadMaterial(vid)`, `newVisit()`, `staffByInitials(i)`, `staffBlock(initials)`, `matIconClass(type)`, `matIconLabel(type)`

**AI Scout**
- `aiVoice()`, `aiLLM()`, `aiSearch()`, `setAiMode(m)`, `aiRun()`, `aiQuick(q)`

**Search + filter**
- `onSearch(v)`, `onPF(p)`, `uf(k, v)`, `filt(list)`, `posBtns()`, `setRecSrc(s)`

**Render**
- `render()` — *Phase 5a: switch extended with `patrons` + `ops`*
- `renderHome(c, portal, hs)` — *Phase 5a: Smart Suggestions block + tile deep-links*
- `tileHTML(label, value, sub, cls, nav)` — *Phase 5a: nav param added*
- `renderRoster(c)` — *Phase 5a: photo avatars*
- `renderRecruiting(c, portal, hs)` — *Phase 5a: Watchlist tab*
- `_recCard(p)` — *Phase 5a NEW: shared prospect card renderer*
- `renderCRM(c)` — *Phase 5a: full rewrite*
- `renderVisits(c)`
- `renderFilter(c, portal)` — *Phase 5a: photoAv + addCrmBtn on result cards*
- `renderBudget(c)`, `updateAllocLive(pos, val)`, `reqOverBudget()`, `saveAllocs()`
- `renderBuilder(c)`, `slotHTML(i, byPlayer, allP, small)`
- `renderScout(c)` — *Phase 5a: photoAv + addCrmBtn on result cards*
- `renderPatrons(c)` — *Phase 5a NEW*
- `openDonor(donorId)` — *Phase 5a NEW*
- `renderOps(c)` — *Phase 5a NEW*

---

## Working agreements (carry forward — burned in)

These are non-negotiable. Every session must follow them.

1. **Always provide full copy-paste-ready code blocks.** Never ask Michael to find-and-replace inside a file. Always provide the complete updated file as one block.
2. **Format every URL as a clickable markdown link.** Never bare text.
3. **Mac-native instructions.** Terminal, `Cmd+A/C/V`, local git, Claude Code workflows. No iPad/Safari unless he switches devices.
4. **Be maximally specific.** Full file paths, exact paste locations, every URL spelled out, every command in full.
5. **Numbered step-by-step lists.** Even for simple tasks. Each step on its own line with the exact action, the exact command, and any URL formatted as clickable markdown.
6. **Never suggest stopping or ask if he's tired.** Keep working until he says to stop.
7. **At session start:** read `CLAUDE.md` and the latest `/handoffs/HANDOFF-*.md` from project knowledge.
8. **At session end:** generate a new handoff doc.

### Phase 5a-specific patterns burned in
9. **Map the file ONCE up front, then work from memory.** Don't repeatedly grep for line numbers — they shift after every edit. Get the whole function map at session start.
10. **Use larger contiguous str_replace blocks.** Full function replacements > many small partial edits. Aim for ≤10 str_replaces per session for big rewrites.
11. **Validate at the midpoint and at the end with `node --check`.** Pattern:
    ```python
    python3 -c "
    import re
    with open('index.html') as f: html = f.read()
    scripts = re.findall(r'<script>(.*?)</script>', html, re.S)
    js = '\n;\n'.join(scripts)
    with open('/tmp/extracted.js','w') as o: o.write(js)
    "
    node --check /tmp/extracted.js
    ```
12. **Skip exploratory `view` calls during execution.** Burns budget, shifts line numbers, and risks running out before validation/ship. Only `view` when actually stuck.
13. **Scope to ONE shippable thing per session.** Phase 5a was 8+6 items and took 3 sessions. Don't bite off more than fits in a single chat's tool budget.
14. **Validate as the LAST step before `present_files`.** Never ship unvalidated.

---

## Reference constants (carry forward)

### Repos + URLs
- Frontend repo: [github.com/Apexver1/Apex-App](https://github.com/Apexver1/Apex-App)
- Backend repo: [github.com/Apexver1/apex-intel](https://github.com/Apexver1/apex-intel)
- Live URL: [apexver1.github.io/Apex-App/](https://apexver1.github.io/Apex-App/)
- Phase 5a cache-bust: [apexver1.github.io/Apex-App/?v=5a](https://apexver1.github.io/Apex-App/?v=5a)
- Splash override: [apexver1.github.io/Apex-App/?v=5a&splash=1](https://apexver1.github.io/Apex-App/?v=5a&splash=1)

### Local paths (Mac)
- Frontend: `~/code/Apex-App/`
- Backend: `~/code/apex-intel/`
- Python venv: `~/code/apex-intel/.venv` (Python 3.12.13 via Homebrew)

### Auth
- Auto-login email: `apex@test.com`
- Auto-login password: `ApexTest123`
- School picker password: `stewart2026`
- Override flags:
  - `?nologin=1` — bypass auto-login
  - `?splash=1` — force splash on auto-login

### Supabase
- Project: [midyxvjfoggchbugxzkd.supabase.co](https://midyxvjfoggchbugxzkd.supabase.co)
- SQL editor only runs the FIRST statement in multi-statement queries. Run DDL one statement at a time.
- Default API row cap is 1000. Always paginate `select()` over full tables.
- Local `.env` only — `SUPABASE_URL` + `SUPABASE_SERVICE_ROLE_KEY` (`sb_secret_` format)
- **Never paste service role key in chat.** If accidentally screenshot, rotate at Project Settings → API Keys → Secret keys.

### Oregon
- Supabase UUID: `d4dd73c7-2ec6-49bf-b63b-5c4a333a2bf7`
- ESPN ID: `2483`
- CBBD ID: `223`
- BallDontLie ID: `123`
- sports247 ID: `24090`
- NCAA slug: `oregon`

### Repository state
- 6 commits on `main` before this session (frontend repo)
- This session adds 1 new commit: Phase 5a design rebuild
- Latest commit before this session: `e38c9f9` — "Handoff: Scout 1.1.1 shipped + 722 portal players backfilled with BT 2024–25 stats" (backend repo)

### Database state (unchanged this session)
- 7,526 total players in `school_rosters` (5,677 D1 + 657 HS + 1,192 portal)
- 4,844 portal players have Bart Torvik 2024-25 advanced stats
- 12 Supabase endpoints feeding `loadData()`: `my_roster`, `prospect_shortlist`, `team_strategy`, `scholarships`, `prospect_stats`, `crm_log`, `nil_budget`, `nil_budget_requests`, `prospect_scores`, `prospect_family`, `contact_reminders`, `schools`

### Long-script paste pattern (burned in)
For Python scripts longer than ~30 lines, use heredoc:
```bash
cat > path/to/file.py <<'APEX_EOF'
[paste full script]
APEX_EOF
```
Critical: closing `APEX_EOF` must be on its own line with no leading whitespace.

### Fuzzy school name matching
Needs runner-up margin (0.94 threshold, 0.04 margin) or you get Ohio State → Chico State.

---

## Steps to deploy this Phase 5a build

1. Open Finder window:
   ```bash
   open ~/code/Apex-App/
   ```
2. Drag the new `index.html` from `~/Downloads/` into `~/code/Apex-App/`, replacing the existing one.
3. Commit and push:
   ```bash
   cd ~/code/Apex-App && git add index.html && git commit -m "Phase 5a: design rebuild — splash, BETA, Apex CRM phonebook, Apex Patrons + Apex Ops, photos, Add to CRM, Watchlist, centered modals" && git push origin main
   ```
4. Wait ~30 seconds for GitHub Pages to rebuild, then visit:
   - [apexver1.github.io/Apex-App/?v=5a](https://apexver1.github.io/Apex-App/?v=5a)
5. Verify splash on auto-login:
   - [apexver1.github.io/Apex-App/?v=5a&splash=1](https://apexver1.github.io/Apex-App/?v=5a&splash=1)
6. Smoke-test checklist:
   - [ ] Splash appears on `?splash=1`, "Build a Champion →" enters app
   - [ ] BETA chips visible on splash + topbar + drawer
   - [ ] Drawer shows OPERATIONS group with NEW tags on Patrons + Ops
   - [ ] Apex Patrons renders (donors, campaigns, KPIs)
   - [ ] Tap a donor → modal opens with sparkline
   - [ ] Apex Ops renders (category grid, recent activity)
   - [ ] Apex CRM renders A-Z phonebook with letter headers
   - [ ] Tap a player row → expands with nested contacts
   - [ ] Tap a nested contact → contact profile modal
   - [ ] Add to CRM `+` button on Recruiting cards toggles to ✓
   - [ ] ☆ on Recruiting card adds to Watchlist tab
   - [ ] Watchlist tab shows count and starred prospects
   - [ ] Player detail modal: competing schools chip-input works
   - [ ] Player detail modal: Log Contact has datetime-local picker
   - [ ] Coach score modal: no Weighting dropdown, slider+notes only
   - [ ] Home dashboard: Smart Suggestions card visible (if any prospects qualify)
   - [ ] Home tiles deep-link to their tabs

---

## Phase 5b — pick ONE for tonight's session

The Phase 5a deferred list (from prior handoff) is too large for a single session. Below are 4 candidate scopes, each sized to fit ONE chat with full validation budget. Pick the one that maps to highest impact for the demo / pilot story.

### Option A: Real Add to CRM + Watchlist persistence ⭐ recommended for tonight
**Why:** Smallest scope = highest ship probability. Immediately demoable ("add on phone, see on laptop"). Pattern reusable.

**Backend (apex-intel repo):**
1. Create `crm_added` table:
   ```sql
   CREATE TABLE crm_added (
     user_id uuid REFERENCES auth.users(id) ON DELETE CASCADE,
     prospect_id uuid REFERENCES prospect_shortlist(id) ON DELETE CASCADE,
     school_id uuid REFERENCES schools(id),
     added_at timestamptz DEFAULT now(),
     PRIMARY KEY (user_id, prospect_id)
   );
   ALTER TABLE crm_added ENABLE ROW LEVEL SECURITY;
   CREATE POLICY "users see own adds" ON crm_added FOR SELECT USING (auth.uid() = user_id);
   CREATE POLICY "users insert own adds" ON crm_added FOR INSERT WITH CHECK (auth.uid() = user_id);
   CREATE POLICY "users delete own adds" ON crm_added FOR DELETE USING (auth.uid() = user_id);
   ```
2. Create `watchlist` table — same shape as `crm_added`.

**Frontend (Apex-App repo, `index.html`):**
1. Add to `loadData()` Promise.all: `api("/rest/v1/crm_added?select=prospect_id&user_id=eq."+UID)` and same for watchlist
2. Populate `ADDED_TO_CRM` Set + `WATCHLIST` Set from results
3. Modify `toggleAddCrm`: POST to `/rest/v1/crm_added` (or DELETE) with `{prospect_id, school_id: CURRENT_SCHOOL_ID}`
4. Modify `toggleWatch`: same pattern for watchlist table

**Risk:** Low. Two small tables, two state migrations. Estimated 6-8 str_replaces total.

### Option B: Real player photos backfill
**Why:** Highest visible impact. The app goes from "navy initials everywhere" to "real faces" — huge demo lift.

**Steps:**
1. Verify `school_rosters.espn_player_id` column exists; if not, `ALTER TABLE school_rosters ADD COLUMN espn_player_id text;` and run a scrape pass to populate it (Oregon first as proof)
2. Add `photo_url text` columns to `school_rosters` and `prospect_shortlist`
3. Python script (~40 lines) that:
   - Pulls all rows with `espn_player_id` set
   - Constructs ESPN headshot URL: `https://a.espncdn.com/i/headshots/mens-college-basketball/players/full/{espn_player_id}.png`
   - Optionally HEAD-checks the URL exists
   - Batch-updates `photo_url` via Supabase REST
4. Frontend: extend `photoAv(name, size, photoUrl)` calls in `_recCard`, `renderRoster`, `renderCRM`, etc. to pass `p.photo_url`

**Risk:** Medium. Depends on `espn_player_id` being available. Could be a 2-session arc if scrape needed first.

### Option C: `prospect_contacts` schema generalization
**Why:** Replaces `dummyContactsFor()` JS generator with real schema. Unblocks the contact profile modal's "Edit contact" button.

**Backend:**
```sql
CREATE TABLE prospect_contacts (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  prospect_id uuid REFERENCES prospect_shortlist(id) ON DELETE CASCADE,
  contact_name text NOT NULL,
  role text NOT NULL,  -- 'FAMILY' | 'HS_COACH' | 'AAU_COACH' | 'AGENT' | 'TRAINER' | 'SIBLING' | 'OTHER'
  relationship text,    -- e.g. 'Mother', 'Father', 'Stepfather' for FAMILY role
  phone text,
  email text,
  social_handle text,
  photo_url text,
  notes text,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now(),
  deleted_at timestamptz
);
-- RLS + migration from prospect_family
INSERT INTO prospect_contacts (prospect_id, contact_name, role, relationship, phone, email)
  SELECT prospect_id, contact_name, 'FAMILY', relationship, phone, email FROM prospect_family;
```

**Frontend:**
1. `loadData()`: replace `prospect_family` query with `prospect_contacts`
2. Replace `familyMap` with `contactsMap` keyed by prospect_id, value = full array
3. `renderCRM` nested rows: render real contacts with role tag from `contact.role`, drop `dummyContactsFor()` calls
4. `openContactProfile`: full edit form wired to PATCH `/rest/v1/prospect_contacts`

**Risk:** Medium. Schema migration is irreversible-ish (keep `prospect_family` as backup). Estimated 8-10 str_replaces.

### Option D: + New Visit flow with `prospect_visits` table
**Why:** Currently visits are seeded only. This unblocks the "log a real recruiting visit from your phone" use case.

**Backend:**
```sql
CREATE TABLE prospect_visits (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  prospect_id uuid REFERENCES prospect_shortlist(id),
  school_id uuid REFERENCES schools(id),
  type text NOT NULL,  -- 'official' | 'unofficial' | 'home'
  date_iso timestamptz NOT NULL,
  date_display text,
  location text,
  staff_assigned text[],  -- array of staff initials
  itinerary jsonb,        -- [{time, duration_min, title, location}, ...]
  materials jsonb,        -- [{name, size, updated, type}, ...]
  status text DEFAULT 'scheduled',
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now()
);
-- RLS + migration from VISITS_SEED (manual port)
```

**Frontend:**
1. `newVisit()` — open modal with form (prospect picker, type, date/time, location, staff multi-select)
2. `addItineraryStop()` / `removeItineraryStop()` helpers
3. `saveVisit()` — POST to `/rest/v1/prospect_visits`, refresh list
4. Update `loadData()` to pull from `prospect_visits` instead of `VISITS_SEED.slice()`

**Risk:** Higher. Schema is more complex (jsonb fields). Probably needs full session.

### My recommendation for tonight: **Option A**
- Smallest scope means highest probability of actually shipping
- Both tables are tiny + identical shape
- Frontend changes are surgical (4 spots: `loadData`, `toggleAddCrm`, `toggleWatch`, optionally a "show only added" CRM filter)
- Validates the persistence pattern that all future Sets-to-DB conversions will reuse
- After this ships, Options B/C/D can each be their own future session

---

## Session starter for next chat

Open a fresh chat in claude.ai. Paste this as your first message:

````
Loading project context for Phase 5b — Apex Intel persistence work.

Please fetch:
https://github.com/Apexver1/apex-intel/raw/main/CLAUDE.md
https://github.com/Apexver1/Apex-App/raw/main/handoffs/HANDOFF-2026-04-15-night-phase-5a-shipped.md

If GitHub fetch fails, I'll paste both files directly.

Tonight I want to ship Option A from the Phase 5b candidate list:
real Add to CRM + Watchlist persistence. Two tiny Supabase tables,
four surgical frontend changes. Walk me through:

1. The two CREATE TABLE statements + RLS policies (one statement at
   a time — Supabase SQL editor only runs the first)
2. The frontend changes to loadData, toggleAddCrm, toggleWatch
3. Any new state initialization needed on page load

Working agreements that are critical:
1. Map the index.html function structure ONCE at the start, work from
   memory after that — don't re-grep for line numbers.
2. Use larger contiguous str_replace blocks (full function replacements).
3. Run `node --check` after each major batch of edits.
4. Validate at midpoint AND end before present_files.
5. Deliver as a downloadable file via present_files. I'll drag-replace
   into ~/code/Apex-App/index.html, then commit + push from Terminal.
6. Numbered step-by-step lists for everything. No "do X then Y" prose.
7. Every URL as a clickable markdown link.
````

Then in the next message, drag in your production `~/code/Apex-App/index.html` (now Phase 5a shipped) and let the new session execute.

---

## Final notes

- The file presented at the end of this session (`/mnt/user-data/outputs/index.html`) IS the Phase 5a build. Drag-replace and ship.
- After shipping Phase 5a tonight, save this handoff to `~/code/Apex-App/handoffs/HANDOFF-2026-04-15-night-phase-5a-shipped.md` and commit it alongside `index.html` in the same push.
- The CLAUDE.md in the apex-intel repo doesn't need updating from this session — no DB schema or data-source changes.
- If you want to backfill the CLAUDE.md frontend section with Phase 5a function map + new state vars, that's a 2-minute fast-follow in any future session.
