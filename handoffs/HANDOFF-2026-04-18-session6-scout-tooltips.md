# Handoff — April 18, 2026 (Session 6: Scout v2 hero + metric tooltips)

## Session result

Two clean commits shipped. Session 6 split into 6a (Scout v2 hero redesign) and 6b/2a (metric tooltips via tap-for-info popovers). Third attempted commit (2c — LLM default engine + 1a popover-copy polish) broke the app; surgically reverted, preserving 2a. Drawer work (Commit 2b — logo swap, role label, subhead) deferred to Session 7 because grep recon kept returning 0 hits on the dynamic-text render target — needs DevTools to unblock.

| Commit | Hash | Delta |
|---|---|---|
| Scout v2 hero | `d30a3ca` | 1 file, +69 / -21 lines |
| Metric tooltips | `7ee3241` | 1 file, +4 / -4 lines (4288 net-new bytes on disk) |

**Live on production at [apexver1.github.io/Apex-App/](https://apexver1.github.io/Apex-App/).**

Session ~3 hours total, most of it on diagnostic recon for anchor patterns. Actual patch application time was tight; the overhead was iterating to find the right byte-form for five separate anchor sites.

---

## What shipped — Commit 1 (Scout v2 hero, d30a3ca)

### Replaced
- `AI Scout` heading + gold italic `Pro` badge
- `v1.1.1 · 4,122 D1 players · BT advanced` meta row (developer-facing noise)
- `Rules · free / Apex AI · $0.01/q` engine-toggle segment control
- `Mic` text button (replaced by circular icon inside input)
- Flat `Quick queries` pill strip

### With
- **Hero title block** — small uppercase "APEX SCOUT" label, large serif "Build your roster." headline, dynamic subhead "Ask anything. Get real prospects that fit [School]." The subhead interpolates the current school name via a regex that strips common mascot suffixes (Ducks/Blue Devils/Tigers/etc.) so "Oregon Ducks" becomes just "Oregon."
- **Pill input with inline circular mic** — `border-radius: 999px` on the input, circular navy mic button absolutely positioned inside the input's right side, SVG microphone icon
- **Full-width Search button** below the input
- **4 smart preset cards** — dynamic per school:
  - `Defensive upgrades` — subtitle pulls `kenpomFor(school).adjD`
  - `Value picks` — subtitle static "Under $500K"
  - `Rebuild targets` — subtitle pulls `torvikFor(school).rank`
  - `Tempo match` — subtitle pulls `torvikFor(school).tempo`
- **4 static preset cards** — evergreen basketball archetypes:
  - `Elite shooters · 35%+ from 3`
  - `Rim protectors · 8+ rpg, 2+ bpg`
  - `Stretch 4s · 6'8+, 35% 3PT`
  - `Two-way wings · Score + defend`
- **2×2 per section grid** — `grid-template-columns: repeat(2, minmax(0, 1fr))`. Initially shipped as 4-col grid with a media-query fallback to 2-col on mobile; wraps on labels like "Defensive upgrades" and "Rebuild targets" forced the switch to 2-col everywhere. Cleaner, more readable, no media query needed.
- **Below-the-fold "How to use Scout" instructions card** — renders only when `!aiResults`. Cream background, small uppercase heading, brief description of what Scout searches (4,122 D1 players / 1,192 portal / 657 HS) + "Try:" suggestion text.

### Engine toggle fate
The `aiMode === "rules"` / `aiMode === "llm"` branch logic is still fully intact in `renderScout` results-rendering code. Only the toggle UI was removed. `aiMode` at runtime is `undefined` (never declared with `var`), which falls through to the rules-mode branch internally. **This is now the known source of the "rim protectors returns Juke Harris" quality issue** — BACKLOG item.

### Mid-session iteration
Three patches to ship Commit 1 cleanly:

1. **First patch:** Hero block with 3×2 grid (3 smart + 3 static = 6 presets). Shipped clean.
2. **Micro-patch to 8 presets in 4×2 grid:** added Tempo match + Two-way wings. P6 (responsive CSS media query) failed because the anchor `.ai-chips{display:flex;flex-wrap:wrap;gap:6px}` was off — real file had `;margin-top:10px` appended. Fixed via byte-extraction.
3. **Second pivot to 2×2 grid:** user saw the 4-col grid with label wrap on "Defensive upgrades" / "Rebuild targets" / "Two-way wings" and called it. Switched to 2-col everywhere, dropped the responsive media query as redundant.

## What shipped — Commit 2a (metric tooltips, 7ee3241)

### CSS infrastructure
Added to the `<style>` block right after the `.ai-chips` rule:

- `.apx-info` — 14px circular icon with `i` character inside, dark gray bg (turns brand-blue on hover), inline-flex centered, used inline with text
- `.apx-chip-tap` — adds `cursor: pointer` to make chip tappability obvious
- `.apx-pop` — fixed-position dark popover with 260px max-width, 10px border-radius, 0 8px 24px shadow
- `.apx-pop-ttl` / `.apx-pop-acr` / `.apx-pop-body` / `.apx-pop-val` — popover content sections (title / monospace acronym / body text / values-row with top border)
- `.apx-pop-close` — × button in top-right of popover

### JS helpers
Inserted right before `window.startCompare`:

```javascript
window.apxMetricInfo = {
  "team_strength": { ttl, acr, body },
  "team_offense":  { ttl, acr, body },
  "team_defense":  { ttl, acr, body },
  "team_pace":     { ttl, acr, body },
  "team_winprob":  { ttl, acr, body },
  "player_offense":{ ttl, acr, body }
};
window.apxShowInfo = function(ev, key, extraVal) {
  // 1. stopPropagation, remove any existing popovers
  // 2. Create .apx-pop div, populate from apxMetricInfo[key] + extraVal
  // 3. Position near event target (fallback above/below if overflow)
  // 4. Attach deferred click-outside listener that removes popover
};
```

Only 2 of the 6 definitions are wired to live UI so far: `team_strength` (via the chip helper) and `player_offense` (via the stat-box). The other four are ready to use when more surfaces get plain-English treatment.

### Usage sites
1. **`window.kenpomChipHTML(p, isPr)` rewritten.** One function rewrite propagates to 5+ surfaces:
   - Home Next Up KenPom chip
   - Recruiting prospect cards (both renderers — line 2500 + line 3480)
   - Player modal subtitle chip
   - Scout result cards in LLM mode
   - Scout result cards in rules mode

   The chip visible output is unchanged (`#101 · +7.08 AdjEM`) but the wrapping `<span>` now has `class="kp-chip apx-chip-tap"`, `onclick="window.apxShowInfo(event, 'team_strength', 'Wake Forest: +10.58 AdjEM · #80')"`, and a trailing `<span class="apx-info">i</span>`.

2. **Home Roster Pulse 4th tile label rewrite.** Changed the `.pulse-l` (label) content from `+7.08 AdjEM` to `Team strength <span class="apx-info" onclick="...">i</span>`. Value column still shows `#101`. Attaches the popover via the same `apxShowInfo` helper.

3. **Player modal BART TORVIK ADVANCED stat-box.** The `AdjOE` stat-box label rewrote to `Offense <span class="apx-info" onclick="window.apxShowInfo(event, 'player_offense', 'AdjOE: 118.5')">i</span>`. Uses the `player_offense` key in `apxMetricInfo` which has a player-specific tooltip body (vs. the team-level `team_offense`).

### Popover visual
Dark charcoal background, white/near-white text. Structure:
```
[×]
Team strength
AdjEM                           (monospace, subtle)
KenPom's measure of overall team strength.
Offensive efficiency minus defensive efficiency,
pace-adjusted. Higher is better.

────────────────────────────────
+7.08 AdjEM · #101              (values row, subtle)
```

Looks exactly like the kind of pro-product tooltip a coach would expect. Sammy Gelfand's "this should look expensive" bar cleared.

---

## Attempted and reverted — Commit 2c + 1a

Two small follow-on changes attempted together after Commit 2a shipped, then rolled back.

### 2c — LLM default engine
Attempted: change `aiMode` init from `undefined` (default to rules) to `"llm"` (default to the Scout 1.2 tool-use path).

Implementation: prepend `var aiMode="llm";` immediately before the existing `setAiMode=function(m){aiMode=m;render();};` declaration. This was my mistake — `setAiMode=function...` is NOT prefixed with `var` or `window.` in the source. It's hanging off a larger statement or expression chain. Inserting a free-standing `var aiMode="llm";` in front of it broke statement context; the JS parser threw, and the whole app's JS died on page load. Auto-login failed (auto-login is JS-driven), and the user landed on the sign-in form with password-entry nonfunctional.

### 1a — "Team ·" prefix on chip popover values
Attempted: change the chip popover's values-row copy from `Wake Forest: +10.58 AdjEM · #80` to `Team · Wake Forest · +10.58 AdjEM · #80`. Motivation: the current `Wake Forest:` framing doesn't make clear the rating is about the TEAM, not the player, on the player's modal. Adding `Team ·` prefix closes the loop.

This change by itself is fine. It got rolled back along with 2c because both were in the same fix patch, and when 2c broke the app we reverted the entire patch surgically — leaving Commit 2a's tooltip infrastructure intact but removing the 1a prefix fix along with the bad 2c declaration.

### Revert mechanics
The fix patch was only `+43` bytes total (small) and the revert patch was `-38` bytes (small). Asserts confirmed: no residual `Team \u00b7` prefix, no residual `var aiMode="llm"` declaration, but `window.apxMetricInfo`, `window.apxShowInfo`, `.apx-info{` CSS, `apx-chip-tap` class, `Team strength` Home label, `Offense` stat-box label all still present. Commit 2a v3 tooltip state fully preserved.

Both 1a and 2c filed to BACKLOG as separate projects for standalone re-attempt.

---

## Mid-session lessons — folded into CLAUDE.md working agreements

### Byte-extraction for anchors
**The ONE big lesson from this session.** For any patch anchor containing nested quotes, escape sequences, or unicode literals, extracting the anchor from live file bytes via Python `.find()` + save to `/tmp/` + read back is the only reliable method. Typing the anchor as a Python string literal burned us four times this session (chip return, pulse label, stat-box label, valline) and once in Session 5 (fTier colon/equals).

Pattern:
```python
import pathlib
txt = pathlib.Path('index.html').read_text()
start = txt.find('window.kenpomChipHTML=function')
end = txt.find('window.startCompare', start)
body = txt[start:end]
pathlib.Path('/tmp/chip_exact.txt').write_text(body)
print(repr(body[-200:]))  # see exact bytes of return statement
```

Then in the patch script:
```python
OLD_CHIP = pathlib.Path("/tmp/chip_exact.txt").read_text()
# extract the substring we want to replace
ret_start = OLD_CHIP.find('return "<span class=')
ret_end = OLD_CHIP.find(';', ret_start) + 1
OLD_ANCHOR = OLD_CHIP[ret_start:ret_end]
assert txt.count(OLD_ANCHOR) == 1
txt = txt.replace(OLD_ANCHOR, NEW_STRING, 1)
```

When debugging a count=0 mismatch, run `repr()` on the real bytes vs the attempted anchor, compare char-by-char in Python. Takes 30 seconds.

### DevTools for dynamic-render targets
Four rounds of `grep` recon in Session 6 couldn't find the drawer render function. User-visible strings ("Dana Altman", "Coach Stewart", "Head coach", "Logged in as") don't grep because they're built via JS string concatenation at runtime. For Session 7 drawer work: start with Safari DevTools → Inspect Element → examine the drawer's DOM → trace the HTML back to the render function. Don't repeat grep-loop.

### Statement-context before injecting new declarations
When inserting a `var`, `let`, `const`, or function declaration next to existing code, verify the outer statement context first. `setAiMode=function(m){...};` with no `var`/`window.` prefix means it's hanging off a larger expression — you can't just prepend a standalone `var` next to it without breaking parsing. If you need to inject, find an unambiguous safe location (right after a `;` that ends a known statement) or use `window.aiMode = "llm";` style (assignment, no declaration).

---

## Files changed

| File | Repo | Status |
|---|---|---|
| `index.html` | Apex-App | MODIFIED — commits `d30a3ca` (Scout v2, +4743 bytes) + `7ee3241` (tooltips, +4288 bytes). Net Session 6: +9031 bytes (236834 → ~246674). |
| `index.html.s6-backup-20260418-*` | Apex-App | LOCAL ONLY, gitignored (multiple safety backups from the session) |
| `/tmp/s6a_patch.py`, `/tmp/s6a_micro_patch_v2.py`, `/tmp/s6a_2x2_patch.py`, `/tmp/s6b_2a_patch_v3.py`, `/tmp/s6b_2a_fix_patch.py`, `/tmp/s6b_revert.py` | local | Patch scripts, not committed (one-shot artifacts) |
| `/tmp/chip_exact.txt`, `/tmp/pulse_exact.txt`, `/tmp/sb_exact.txt`, `/tmp/chip_valline.txt`, `/tmp/aimode_init.txt` | local | Byte-extraction artifacts (essential for the final patches to work) |
| `DATA-STATUS.md` | Apex-App | MODIFIED (this session close) |
| `BACKLOG.md` | Apex-App | MODIFIED (this session close — 4 new named projects logged) |
| `CLAUDE.md` | Apex-App | MODIFIED (this session close — byte-extraction + DevTools working agreements added) |
| `handoffs/HANDOFF-2026-04-18-session6-scout-tooltips.md` | Apex-App | NEW (this file) |

---

## Still open

### Session 7 pickup — Commit 2b (drawer + role label + Howard fallback)
- **Drawer logo swap:** replace the `DA` initials circle with 72px centered school logo via `window.schoolLogo(name, size)` (confirmed at line 1086 of index.html)
- **Coach name subhead:** change from `Head coach · Oregon` to `Oregon Ducks` (matching the top-left app header pattern)
- **Role label:** change `Coach Stewart · HC` (orange) to `Coach Stewart · Assistant Coach` (Title Case, muted gray). Bug filed that HC was the hardcoded placeholder; real RBAC still deferred.
- **Howard `head_coach_name` NULL fallback (B3):** `Good morning, Coach.` reads awkward. Fix with a more defensive string — either omit the comma when name is empty, OR fallback to "Welcome back, Coach Stewart" using the logged-in user's name.

**Session start approach:** Safari DevTools → Inspect Element on the drawer `DA` initials circle → screenshot the element HTML + parent structure → trace back to the render function → build byte-extraction anchors from there.

Scope: ~60–90 min once the recon is solid.

### Deferred to future sessions (per 14-day plan)
- **Session 7-ish:** Apex Picks preset strip in Smart Filter (maybe redundant now that Scout v2 absorbed the preset concept — re-evaluate)
- **Session 8:** NIL Budget per-school defaults
- **Session 9:** Patrons data for Power 5 + NIL Vault stub
- **Session 10:** Beta-2 dry run + polish pass

### New BACKLOG items from Session 6
- **Scout engine auto-routing (LLM default)** — the 2c Commit attempted. Needs correct JS context for the `aiMode` declaration, or a different approach (e.g., patch `aiRun()` to default-to-LLM without touching `aiMode` init).
- **"Team ·" prefix on KenPom chip popovers** — the 1a polish, needs standalone retry without the 2c baggage.
- **Player-level stat chips on prospect cards** — larger Sammy-pitch-aligned project to show player's own AdjOE/AdjDE (from `prospect_stats`) alongside or instead of the team's rating. Directly addresses "stop paying for the ranking, pay for the player."

### S6-minor items (see BACKLOG)
- S6a-minor-1: Remove dead `setAiMode` function (no UI calls it anymore)
- S6a-minor-2: Fix the `Math.random` fake-ranking logic in Defensive upgrades subtitle — should show real rank
- S6a-minor-3: Dim/disable Search button when input matches last-run query
- S6b-minor-1: Add arrow/caret to tooltip popover pointing at the ⓘ trigger
- S6b-minor-2: Refactor inline `onclick` strings into event delegation via `data-metric-key`
- S6b-minor-3: Four of the six `apxMetricInfo` definitions are ready but not yet wired to any UI — use as more metrics surface

---

## Reference constants (unchanged)

- Oregon: UUID `d4dd73c7-2ec6-49bf-b63b-5c4a333a2bf7`, KenPom #101 / +7.08, Torvik #91 / 0.7068
- Duke: UUID `279abc4c-77dd-4a3a-8f46-5daa6f9d8da1`, KenPom #3 / +37.37, Torvik #3 / 0.9768
- Supabase project: `midyxvjfoggchbugxzkd`
- Publishable key (in `index.html`, safe): `sb_publishable_MizrCkmoqSNlQAF3ty_FUA_WkATU4Ai`
- **Session 6 commits:** `d30a3ca` (Scout v2 hero) + `7ee3241` (metric tooltips)

---

## Session starter for next chat (Session 7 — Commit 2b drawer + Howard)

Paste this as your first message in Session 7:

```
Loading project context — Apex Intel, Session 7 (Commit 2b — drawer logo swap + role label + Howard blank-greeting fallback).

Please read from Project Knowledge:
- CLAUDE.md
- handoffs/HANDOFF-2026-04-18-session6-scout-tooltips.md  ← latest
- DATA-STATUS.md
- BACKLOG.md
- PLAN-2026-04-16-to-2026-04-30.md

Session 7 target (one sentence):
Ship Commit 2b — 72px centered school logo in the drawer (replacing DA initials), subhead rewrite to 'Oregon Ducks', role label rewrite to 'Coach Stewart · Assistant Coach' (Title Case, muted gray), and Howard NULL head-coach fallback copy.

Session 6 Commit 2b was deferred because four rounds of grep recon couldn't find the drawer render function — the user-visible strings are dynamically built from variables at runtime. Session 7 starts with Safari DevTools inspection to unblock.

Expected scope: 60–90 min once recon nails the anchor. Approach:

1. Safari DevTools → Inspect Element on the DA initials circle in the drawer
2. Screenshot the element HTML + 2-3 parent levels
3. Trace to the render function (likely CSS class or React-like data attr gives us the route)
4. Byte-extract the render function using the Session 6 pattern (python .find + /tmp/ save + read back)
5. Single patch: drawer logo swap + subhead rewrite + role label rewrite + Howard fallback in one atomic commit

Also committed in this session: final Sessions 6-wrap docs updated and present in Project Knowledge.

I'm on Mac, Terminal ready. Let's plan before code.
```

---

## ⚠️ Reminder — upload updated docs to Project Knowledge

Four docs changed/created this session and need to be re-uploaded to Project Knowledge before Session 7:

- `DATA-STATUS.md` (updated — Session 6 shipped entries; tooltip status across every surface)
- `BACKLOG.md` (updated — 4 new named projects: Scout engine auto-routing, Player-level stat chips, "Team ·" popover prefix; plus S6 minor items)
- `CLAUDE.md` (updated — byte-extraction working agreement, DevTools recon working agreement, Phase 6 marked complete)
- `handoffs/HANDOFF-2026-04-18-session6-scout-tooltips.md` (this file, new)

Without the re-upload, Session 7's `project_knowledge_search` will return stale Session 5 data and we'll boot with wrong state. **Please re-upload all four immediately after this session closes.**
