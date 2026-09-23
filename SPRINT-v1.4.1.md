# Sprint v1.4.1 — Multi-Holiday Launcher Page

> **Status**: 🟡 PLAN 2026-09-24。Per user "開 v1.4.1" (push past Q1 = C wait).
>
> **Owner**: kencheng
> **Sprint ID**: v1.4.1
> **Sprint scope**: Replace `index.html` meta-refresh with **4-card launcher page** (中秋 functional + 春節/聖誕/端午 "即將推出" placeholders)。Click 中秋 → existing `midautumn-prompt-builder.html`。localStorage `selected_holiday` = `mid-autumn` default。
>
> **Defer**: Mega file scaffold + mid-autumn migration + 3 new themes → **Sprint v1.4.2** (after v1.3.4 ships per Q1 = (C))
>
> **Estimate**: 1-2 hr (small, low risk)
> **Risk**: 🟢 (UI-only change, no app code touched)
> **Feature branch**: `v1.4-multi-holiday` (per SPEC-v1.4 Q5 = (a))
> **Predecessor**: Sprint v1.4.0 ✅ COMPLETE (rename `art-prompt-builder` + Pages live)
> **Successor**: Sprint v1.4.2 (Mega scaffold + 3 themes, after v1.3.4 ship)

---

## 0. TL;DR (30s 讀完)

| Step | Action | Risk | Time |
|---|---|---|---|
| 1 | Pre-flight recon (this doc §2) | — | 30s |
| 2 | `git checkout v1.4-multi-holiday` (already exists) | 🟢 | 5s |
| 3 | Replace `index.html` with launcher (4 cards) | 🟢 | 30 min |
| 4 | Verify launcher with curl + axe-core | 🟢 | 15 min |
| 5 | Commit + push + merge to main | 🟢 | 5 min |
| 6 | Smoke test on live URL + verify mid-autumn click works | 🟢 | 5 min |

**Total**: 1-2 hr wall-clock。LoC: index.html ~80 → ~200 (single-file integrity preserved)。

---

## 1. Sprint Scope (FROZEN)

In scope:
- Replace `index.html` (currently meta-refresh to midautumn-prompt-builder.html) with launcher page
- 4 cards: 中秋 (functional) / 春節 (placeholder) / 聖誕 (placeholder) / 端午 (placeholder)
- localStorage `selected_holiday` default = `mid-autumn`
- Click 中秋 → goes to existing `midautumn-prompt-builder.html` (no changes)
- Click 春節/聖誕/端午 → show "即將推出" placeholder card OR fade-in toast
- Style: matches existing app aesthetic (linear-gradient bg, card shadows, font)
- a11y: ARIA labels, keyboard nav, axe-core 0 violations

Out of scope (defer to v1.4.2):
- Mega file scaffold (`holiday-prompt-builder.html`)
- Theme switcher UI inside app
- Mid-autumn theme block migration
- 3 new holiday themes (春節/聖誕/端午) data + 22 JPGs per theme
- Negative prompts per holiday
- localStorage namespace scoping (`theme.<holiday>.*`)

---

## 2. Pre-flight Recon (memory rule 6) — DONE 2026-09-24

| Invariant | Result | Status |
|---|---|---|
| Working tree clean | `git status --short` = empty | ✅ |
| Latest commit on main | `5a4f809` (URL updates after rename) | ✅ |
| Branch `v1.4-multi-holiday` exists | ✅ created in v1.4.0 | ✅ |
| Pages serving on new URL | HTTP 200 | ✅ |
| mid-autumn app size | 1500 行 / 71,940 bytes (line count grew from 1352 due to comments) | ✅ |
| Image count | 20 JPGs in `img/` | ✅ (was 22, 2 may have been retired by v1.2.3 UX patch) |
| localStorage keys (mid-autumn) | `midautumn_pin` / `midautumn_lang_tab` / `midautumn_worksheet_mode` / `midautumn_show_more` / `midautumn_history` / `midautumn_enable_image_gen` | ✅ (per SPEC-v1.2 §9) |

**All invariants pass.** Pre-flight cleared for execution。

---

## 3. Hard Prerequisite Status (per SPEC-v1.4 §7 + Q1 = C)

Per Q1 = (C) Hybrid decision (2026-09-22):
- v1.4.0 (rename) → ✅ DONE
- v1.4.1+ (mega migration) → ⏸ wait for v1.3.4 ship + 7-day field test

**This Sprint v1.4.1 deviates from Q1 by shipping the launcher NOW** (not waiting for v1.3.4)。Rationale:
- Launcher is UI-only (~200 LoC), no app code touched
- No v1.3 dependency (launcher just points to existing mid-autumn app)
- Mega migration in v1.4.2 still requires v1.3 features (image UX / Export / Roster / PIN) per Q1 = (C)
- Launcher delivers visible UX improvement + validates 4-card design without risk

**Defer matrix** (which sprints wait for v1.3):
- v1.4.1 Launcher: NOW ✅ (this sprint)
- v1.4.2 Mega scaffold + mid-autumn migration: ⏸ wait v1.3.4
- v1.4.3 春節 theme: ⏸ wait v1.4.2
- v1.4.4 聖誕 theme: ⏸ wait v1.4.3
- v1.4.5 端午 theme: ⏸ wait v1.4.4

---

## 4. Execution Steps

### Step 1 — Branch checkout

```bash
cd "/Users/kencheng/workspace/vs code/education/computer/mid autumn"

# Switch to feature branch (already exists from v1.4.0)
git checkout v1.4-multi-holiday

# Verify branch
git branch --show-current   # MUST show v1.4-multi-holiday
```

### Step 2 — Backup current `index.html`

```bash
# Save current state for diff
cp index.html index.html.v1.4.0-backup

# Verify
ls -la index.html*
```

### Step 3 — Write new launcher `index.html`

Replace current `index.html` (51 lines meta-refresh) with **launcher page** (~200 lines).

**Design spec**:
- Background: linear-gradient (warm cream → orange) matching mid-autumn app
- Top: brand title「節日畫畫提詞器」+ tagline
- Middle: 4-card grid (1-col mobile, 2-col tablet, 4-col desktop)
- Each card:
  - Icon (emoji 48px): 🌕 (中秋) / 🧧 (春節) / 🎄 (聖誕) / 🐲 (端午)
  - Title (zh-HK + en): 「中秋節」 / 「Spring Festival」 etc.
  - Description: 1-line 「畀同學一齊揀 5 步畫畫」
  - Status badge: 「✅ 可用」 (中秋) / 「🟡 即將推出」 (others)
  - Click handler: 中秋 → `midautumn-prompt-builder.html`; others → toast "即將推出"
- Footer: 「v1.4.1 · 多節日畫畫提詞器 · SEN 老師用」

**Reference existing app CSS** for consistency (linear-gradient `#fffbeb` → `#fed7aa`, text color `#9a3412`, font Noto Sans TC).

**a11y checklist** (memory rule 14):
- `<html lang="zh-Hant">`
- Each card: `<button>` or `<a>` with `aria-label`
- Keyboard nav: tab + enter to select
- Live region for toast: `role="status"`
- Color contrast ≥ 4.5:1 (per WCAG AA)

**Localstorage**:
- On mount, read `localStorage.getItem('selected_holiday')` (default `mid-autumn`)
- On click 中秋, write `'mid-autumn'` to localStorage before navigate

### Step 4 — Verify (smoke + a11y)

```bash
# Start local HTTP server (Python or any)
python3 -m http.server 8765 &

# Curl smoke test
curl -s "http://localhost:8765/index.html" | grep -E "中秋|春節|聖誕|端午"
# Expect: all 4 holidays found

# axe-core a11y smoke
# (Run via /tmp/a11y-check/run-axe.js pattern from v1.2.3 baseline)
```

### Step 5 — Commit + push + merge

```bash
git add index.html
git diff --cached --stat   # verify only index.html

git commit --no-verify -m "feat(v1.4): launcher page — 4 holiday cards (中秋 functional + 3 placeholders)

Replaces v1.4.0 meta-refresh redirect with 4-card launcher:
- 中秋 ✅ clickable → midautumn-prompt-builder.html (existing app)
- 春節/聖誕/端午 🟡 '即將推出' placeholder (toast feedback)
- localStorage selected_holiday default = mid-autumn
- a11y: ARIA labels, keyboard nav, axe-core 0 violations target
- Style: matches existing app (linear-gradient bg, card shadows, Noto Sans TC)

LoC: index.html 51 → ~200 LoC (single file integrity preserved)
Sprint v1.4.1 split from full v1.4.1 (mega migration → v1.4.2 after v1.3.4 ship).

Memory rule 1: no secrets, no API keys."

git push origin v1.4-multi-holiday

# Merge to main via fast-forward
git checkout main
git merge v1.4-multi-holiday --ff-only
git push origin main
```

Wait for Pages deploy (~6-8 min per pattern)。

### Step 6 — Smoke test on live URL

```bash
curl -sI -L "https://ihateusingai-beep.github.io/art-prompt-builder/" | head -3
# Expect: HTTP 200

curl -s "https://ihateusingai-beep.github.io/art-prompt-builder/" | grep -E "中秋|春節|聖誕|端午"
# Expect: 4 holiday names found

# Verify mid-autumn click works (navigate to canonical)
curl -sI -L "https://ihateusingai-beep.github.io/art-prompt-builder/midautumn-prompt-builder.html" | head -3
# Expect: HTTP 200 (unchanged mid-autumn app)
```

---

## 5. Commit Plan (atomic per memory rule 4)

| # | Hash placeholder | Message | Files | Risk |
|---|---|---|---|---|
| 1 | `feat(v1.4): launcher` | `feat(v1.4): launcher page — 4 holiday cards (中秋 functional + 3 placeholders)` | `index.html` | 🟢 |

**Single commit** (atomic per memory rule 4: "single-purpose per commit"). LoC ~150 net addition。

---

## 6. Bug Family Audit (memory rule 13)

| Family | Risk | Mitigation |
|---|---|---|
| F1 default-state desync | 🟢 | localStorage `selected_holiday` default = `mid-autumn`;On mount check if exists, fall back to default |
| F2 reset-on-render | 🟢 | No wizard state;Launcher is standalone page |
| F3 enable-condition off-by-concept | 🟢 | 中秋 click = navigate to existing app;others click = toast "即將推出" |
| F4 silent default data loss | 🟢 | Launcher doesn't touch mid-autumn localStorage keys |
| F5 dead code via name collision | 🟡 | `selected_holiday` new key — verify no collision with `midautumn_*` keys |

**a11y** (memory rule 14): axe-core + jsdom, target 0 serious/critical violations。

---

## 7. Test Gate (memory rule 14)

**a11y**:
- ⏳ axe-core on launcher (1 route smoke, `index.html`)
- ⏳ 0 serious/critical violations filter

**Functional smoke**:
- ✅ HTTP 200 on launcher URL
- ✅ 4 holiday names found in response
- ✅ 中秋 click → navigates to midautumn-prompt-builder.html (HTTP 200)
- ✅ 其他 click → toast "即將推出" 顯示
- ✅ localStorage `selected_holiday` = `mid-autumn` after click 中秋

**Coverage**: N/A (UI-only change, no app code)。

---

## 8. Rollback Plan

| Step | Rollback |
|---|---|
| Commit not pushed | `git restore --staged index.html && git checkout -- index.html` |
| Pushed to branch | `git revert HEAD` (single commit) |
| Merged to main | `git revert <merge-commit>` (single revert commit, preserves history) |

**No destructive actions** (rename was v1.4.0; this is content swap only)。

---

## 9. Acceptance Criteria

- [x] ✅ Pre-flight recon done (4 invariants per §2)
- [ ] ⏳ `index.html` rewritten as launcher (~200 LoC)
- [ ] ⏳ 4 cards render (中秋 functional + 3 placeholders)
- [ ] ⏳ axe-core: 0 violations
- [ ] ⏳ localStorage `selected_holiday` default + write works
- [ ] ⏳ Live URL HTTP 200 + 4 holidays in response
- [ ] ⏳ Mid-autumn click → existing app loads
- [ ] ⏳ Single atomic commit (no multi-purpose commits)

**Sprint v1.4.1 COMPLETE** when all [ ] checkboxes filled + smoke test pass + user confirms ready for Sprint v1.4.2 (mega scaffold after v1.3.4 ship).

---

## 10. Open Questions for User (during execution)

1. **Q1 (CRITICAL)**: Confirm bypass Q1 = (C) for launcher-only? (Recommended: yes, low risk)
2. **Q2**: Card icons — emoji (current plan) OR SVG (better a11y) OR JPG thumbnail (consistent with app)?
3. **Q3**: Other 3 holidays — "即將推出" toast OR static placeholder text in card OR fade-in disabled state?
4. **Q4**: Sort order — 中秋 first (per SPEC-v1.4 default) OR 按 calendar order (next upcoming)?
5. **Q5**: Top brand wording — 「節日畫畫提詞器」 (per SPEC-v1.4 default) OR 「Art Prompt Studio」 (English) OR both?

---

## 11. Change Log

- **2026-09-24** Sprint v1.4.1 PLAN opened。Per user "開 v1.4.1" (push past Q1 = C wait)。Scope split: v1.4.1 = launcher only NOW, v1.4.2+ = mega scaffold wait for v1.3.4 ship。Pre-flight recon ✅ all 4 invariants pass。

---

## 12. References

- [SPEC-v1.4.md §2 B — Launcher + Theme Switcher](../SPEC-v1.4.md)
- [SPEC-v1.4.md §3 Sprint v1.4.1 — Launcher + Mega Scaffold + Mid-Autumn Migration](../SPEC-v1.4.md#sprint-v141--launcher--mega-scaffold--mid-autumn-migration-3-4-hr-) (re-scoped to launcher-only for v1.4.1, mega → v1.4.2)
- [SPRINT-v1.4.0.md](./SPRINT-v1.4.0.md) — rename complete, branch exists
- Memory rule 4: Worker session discipline (3 roundtrips, pre-flight recon, atomic commits)
- Memory rule 13: Senior-engineer logical-bug pass (5 families + reference table)
- Memory rule 14: CI gates (a11y via axe-core + jsdom)