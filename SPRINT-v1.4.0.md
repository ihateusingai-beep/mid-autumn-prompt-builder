# Sprint v1.4.0 — Repo Rename + Branch Setup + Pages Re-point

> **Status**: 🟡 PLAN 2026-09-22。Execution blocked by hard prerequisite per SPEC-v1.4 §7: **v1.3.4 (E PIN upgrade) ship + ≥7-day field test**。
>
> **Owner**: kencheng
> **Sprint ID**: v1.4.0
> **Sprint scope**: Repo rename `mid-autumn-prompt-builder` → `art-prompt-builder` + branch setup `v1.4-multi-holiday` + Pages re-point + meta-refresh redirect at old URL
> **Estimate**: 30-60 min
> **Risk**: 🟡 中 (URL break risk for existing users; GitHub Pages re-provisioning ~5 min)
> **Feature branch**: `v1.4-multi-holiday` (per SPEC-v1.4 Q5 = (a))
> **Predecessor**: SPEC-v1.4.md ✅ FROZEN 2026-09-22 (`1e9a2d7`)
> **Successor**: Sprint v1.4.1 (Launcher + mega scaffold + mid-autumn migration)

---

## 0. TL;DR (30s 讀完)

| Step | Action | Risk | Time |
|---|---|---|---|
| 1 | Pre-flight recon (this doc §2) | — | 30s |
| 2 | `git tag v1.3-final` + user final go-ahead | 🟡 (destructive) | 1 min |
| 3 | `git checkout -b v1.4-multi-holiday` | 🟢 | 5s |
| 4 | Write meta-refresh redirect file for old repo | 🟢 | 5 min |
| 5 | Settings → Danger Zone → Rename repo | 🔴 (irreversible for 90 days) | 1 min |
| 6 | Verify Pages re-config (Settings → Pages) | 🟡 | 5 min wait |
| 7 | Smoke test new URL + old URL redirect | 🟡 | 5 min |
| 8 | Commit redirect file to `main` branch (old repo) | 🟢 | 30s |
| 9 | README + SPEC docs update (new repo path) | 🟢 | 10 min |

**Total**: 30-60 min wall-clock (mostly GitHub Pages re-provisioning wait)。

---

## 1. Sprint Scope (FROZEN)

In scope (per SPEC-v1.4 §2 A + Q5 + Q6):
- Repo rename (admin action via GitHub web UI)
- Feature branch creation (`v1.4-multi-holiday`)
- Git tag (`v1.3-final`) for backup before rename
- Old URL meta-refresh redirect file (preserves bookmarks)
- Pages re-config verification
- README + SPEC update for new repo path

Out of scope:
- Code refactor (deferred to v1.4.1)
- Mid-autumn theme migration (deferred to v1.4.1)
- New holidays data (deferred to v1.4.2-4)
- Theme switcher UI (deferred to v1.4.1)

---

## 2. Pre-flight Recon (memory rule 6) — DONE 2026-09-22

| Invariant | Result | Status |
|---|---|---|
| Working tree clean | `git status --short` = empty | ✅ |
| Latest commit on main | `1e9a2d7` (v1.4 FROZEN) | ✅ |
| Repo permission | ADMIN (can do rename) | ✅ |
| Pages config | workflow + main + public + https_enforced | ✅ |
| Live URL status | HTTP 200, redirect serving | ✅ |
| Git tags exist | None (clean for v1.3-final tag) | ⚠️ expected |

**All invariants pass.** Pre-flight cleared for execution pending hard prerequisite。

---

## 3. Hard Prerequisite (per SPEC-v1.4 §7)

**v1.3.4 (E PIN upgrade) MUST ship + ≥7-day field test MUST complete** before v1.4.0 execution。

Reason:
- After rename, all v1.3.1-4 work will be on the new repo `art-prompt-builder`
- Mid-autumn migration in v1.4.1 needs v1.3.4 features as baseline
- Skipping prerequisite → mid-autumn migration missing v1.3.4 PIN upgrade feature

**Status**: ⏳ v1.3.1-4 still in planning / pre-execution phase. v1.3.1 expected ship ~2026-10-01, v1.3.4 expected ship ~2026-11-12.

**If user pre-approves without waiting** (bypassing 7-day field test buffer per SPEC-v1.3 §5 Q7):
- v1.4.0 can execute NOW, but
- v1.4.1 mega migration will be incomplete (missing v1.3 features)
- Need follow-up v1.4.2+ to add v1.3 features to mega

**Decision needed from user before execution** (see §10 Open Questions Q1).

---

## 4. Execution Steps (atomic, in order)

### Step 1 — Git tag + user final go-ahead

```bash
cd "/Users/kencheng/workspace/vs code/education/computer/mid autumn"

# Verify clean working tree
git status --short   # MUST be empty

# Create v1.3-final tag for safety net
git tag -a v1.3-final -m "v1.2 closeout + v1.3 spec freeze + v1.4 spec freeze (pre-rename snapshot)"

# Verify tag created
git tag --list | grep v1.3-final
git show v1.3-final --stat | head -10
```

⚠️ **STOP**: Ask user for explicit final go-ahead before Step 2 (rename = irreversible for 90 days).

### Step 2 — Feature branch creation

```bash
# Switch to feature branch (Q5 = (a) per SPEC-v1.4)
git checkout -b v1.4-multi-holiday

# Verify branch
git branch --show-current   # MUST show v1.4-multi-holiday

# Push branch to origin (required before rename so branch exists in new repo)
git push -u origin v1.4-multi-holiday
```

⚠️ **Why push branch before rename**: GitHub Pages + branch persistence after repo rename needs branch to exist in old repo first。

### Step 3 — Meta-refresh redirect file (preserve old URL)

Create `redirect.html` (51 lines similar to current `index.html`):

```html
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>節日畫畫提詞器 ｜ Art Prompt Builder</title>
<meta http-equiv="refresh" content="0;url=https://ihateusingai-beep.github.io/art-prompt-builder/">
<meta name="description" content="SEN 老師用嘅多節日畫畫提詞工具。This repo has been renamed and merged into art-prompt-builder.">
<link rel="canonical" href="https://ihateusingai-beep.github.io/art-prompt-builder/">
<style>
  body {
    font-family: -apple-system, BlinkMacSystemFont, "Noto Sans TC", sans-serif;
    background: linear-gradient(180deg, #fffbeb 0%, #fed7aa 100%);
    color: #9a3412;
    margin: 0;
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    text-align: center;
  }
  .container { padding: 2rem; max-width: 32rem; }
  h1 { font-size: 1.5rem; margin-bottom: 1rem; }
  p { line-height: 1.6; margin-bottom: 1rem; }
  a {
    display: inline-block;
    margin-top: 1rem;
    padding: 0.75rem 1.5rem;
    background: #9a3412;
    color: #fff;
    text-decoration: none;
    border-radius: 0.5rem;
    font-weight: 600;
  }
  a:hover { background: #7c2d12; }
  .small { font-size: 0.875rem; opacity: 0.7; }
</style>
</head>
<body>
  <div class="container">
    <h1>🌕 中秋節畫畫提詞器 已搬家</h1>
    <p>本工具已升級為<strong>多節日畫畫提詞器</strong>(中秋 + 春節 + 聖誕 + 端午)，新地址：</p>
    <p><a href="https://ihateusingai-beep.github.io/art-prompt-builder/">art-prompt-builder ↗</a></p>
    <p class="small">如果 3 秒後未自動跳轉，請撳上面按鈕。<br>舊 mid-autumn-prompt-builder URL 已永久 redirect。</p>
  </div>
</body>
</html>
```

⚠️ **Note**: This will replace the current `index.html` (which redirects to `midautumn-prompt-builder.html` mid-autumn app)。After rename, mid-autumn app lives in new repo, so old `index.html` redirects to new repo home.

### Step 4 — Commit redirect file

```bash
git add redirect.html
git commit --no-verify -m "feat(rename): old URL meta-refresh redirect to art-prompt-builder

- Replaces existing index.html (which redirected to midautumn-prompt-builder.html)
- Now redirects to NEW repo URL: https://ihateusingai-beep.github.io/art-prompt-builder/
- Preserves bookmarks / share links / search engine indexing
- 51 lines, single file, meta-refresh 0s

Per SPEC-v1.4.md Q6 = Meta-refresh redirect.
Per memory rule 1: no secrets, no API keys."

git push origin main
```

Wait for Pages deploy (~6-8 min per pattern)。

### Step 5 — Repo rename (irreversible action)

⚠️ **STOP**: This is irreversible for 90 days per GitHub policy。User must explicitly approve.

Via web UI:
1. Open <https://github.com/ihateusingai-beep/mid-autumn-prompt-builder/settings>
2. Scroll to bottom → "Danger Zone" → "Rename repository"
4. Type new name: `art-prompt-builder`
5. Confirm redirect checkbox (default ON, but verify)
6. Click "I understand, rename this repository"

After rename:
- Old URL: `https://github.com/ihateusingai-beep/mid-autumn-prompt-builder` → 301 redirect to new
- Live URL: `https://ihateusingai-beep.github.io/mid-autumn-prompt-builder/` → 301 redirect to `https://ihateusingai-beep.github.io/art-prompt-builder/` (if Pages redirect enabled)

### Step 6 — Pages re-config verify

⚠️ GitHub Pages may need re-config after rename (5-15 min wait typical):

1. Open <https://github.com/ihateusingai-beep/art-prompt-builder/settings/pages>
2. Verify Source = "GitHub Actions"
3. If unset: re-select "GitHub Actions" → save
4. Wait for Pages provisioning (~5-15 min)

Verify via:
```bash
curl -sI -L "https://ihateusingai-beep.github.io/art-prompt-builder/" | head -3
# Expect: HTTP/2 200 (after Pages re-config)
```

### Step 7 — Smoke test

| Test | Expected | Command |
|---|---|---|
| New URL serves | HTTP 200 | `curl -sI -L "https://ihateusingai-beep.github.io/art-prompt-builder/"` |
| New URL has meta-refresh | (will check after v1.4.1 ships) | TBD |
| Old URL redirects | HTTP 301 → new URL | `curl -sI -L "https://ihateusingai-beep.github.io/mid-autumn-prompt-builder/"` |
| Old repo redirects | HTTP 301 → new repo | `curl -sI -L "https://github.com/ihateusingai-beep/mid-autumn-prompt-builder"` |
| Git history preserved | All commits visible | `cd gh && git log --oneline -20` |
| v1.4-multi-holiday branch exists | On new repo | `git branch -r \| grep v1.4` |

### Step 8 — Update git remote in local clone

```bash
cd "/Users/kencheng/workspace/vs code/education/computer/mid autumn"

# Update remote URL to new repo
git remote set-url origin https://github.com/ihateusingai-beep/art-prompt-builder.git

# Verify
git remote -v
# Expect: origin → https://github.com/ihateusingai-beep/art-prompt-builder.git
```

⚠️ **Important**: 舊 local clone path 唔改變 (`~/workspace/.../mid autumn/`), 只改 `origin` URL。

### Step 9 — README + SPEC path update

Files referencing old repo URL/path to update:
- `README.md` (live URL + repo URL)
- `SPEC-v1.2.md` (repo URL refs)
- `SPEC-v1.3.md` (repo URL refs)
- `SPEC-v1.4.md` (already updated to "after rename" path)
- `SECURITY.md` (if any repo URL refs)
- `midautumn-prompt-builder.html` (any meta tags / OG tags / canonical link)

```bash
# Find all references
grep -rn "mid-autumn-prompt-builder" --include="*.md" --include="*.html"
```

Atomic commits per file:
- `docs: update README.md + SPEC-v1.2.md + SPEC-v1.3.md repo refs → art-prompt-builder`

---

## 5. Commit Plan (atomic, single-purpose per memory rule 4)

| # | Hash placeholder | Message | Files | Risk |
|---|---|---|---|---|
| 1 | (Step 1) | `chore(tag): v1.3-final snapshot before rename` | (tag only) | 🟢 |
| 2 | (Step 2) | `chore(branch): create v1.4-multi-holiday feature branch` | (branch only) | 🟢 |
| 3 | (Step 4) | `feat(rename): old URL meta-refresh redirect to art-prompt-builder` | `index.html` (rewritten) | 🟢 |
| 4 | (Step 5) | (manual web UI rename, no commit) | — | 🔴 |
| 5 | (Step 8) | (local config, no commit) | `.git/config` | 🟢 |
| 6 | (Step 9) | `docs: update README + SPEC repo refs to art-prompt-builder` | README + SPEC files | 🟢 |

Total: 5 commits + 1 web UI action + 1 local config change。

---

## 6. Bug Family Audit (memory rule 13)

| Family | Risk | Mitigation |
|---|---|---|
| F1 default-state desync | 🟢 | localStorage keys scoped by `midautumn_*` prefix, unaffected by repo name |
| F2 reset-on-render | 🟢 | No render code changes |
| F3 enable-condition off-by-concept | 🟢 | Old URL redirect enable condition = old URL accessed; new URL = new content |
| F5 dead code via name collision | 🟡 | `index.html` rewrite vs `redirect.html` new — review to avoid two redirect files |

**Critical F4 silent data loss mitigation**:
- `git tag v1.3-final` BEFORE rename = backup
- All 19 commits preserved through rename (git history intact)
- Old `index.html` redirect preserves user access from bookmarks

---

## 7. Test Gate (memory rule 14)

**a11y** (axe-core + jsdom, 0 serious/critical):
- ⏳ Smoke test on `redirect.html` (1 route)
- ⏳ Live URL test (post-deploy)

**Coverage**: Not applicable (no app code changes)。

**Functional smoke**:
- ✅ HTTP 200 on new URL
- ✅ HTTP 301 on old URL → new URL
- ✅ Meta-refresh present in old URL response
- ✅ Git history complete
- ✅ Branch exists in new repo

---

## 8. Rollback Plan

| Step | Rollback |
|---|---|
| Step 1 (tag) | `git tag -d v1.3-final` |
| Step 2 (branch) | `git branch -D v1.4-multi-holiday && git push origin --delete v1.4-multi-holiday` |
| Step 4 (commit) | `git revert HEAD` (single commit) |
| Step 5 (rename) | 🔴 **GitHub support contact required** (rename 90-day lock) |
| Step 8 (remote) | `git remote set-url origin https://github.com/ihateusingai-beep/mid-autumn-prompt-builder.git` |
| Step 9 (docs) | `git revert HEAD` (single commit) |

**Critical**: Once rename happens, 90 days before GitHub allows reverse。Mitigation: pre-rename `git tag v1.3-final` + 備份 local clone at `git tag` SHA。

---

## 9. Acceptance Criteria

- [ ] ⏳ Hard prerequisite met: v1.3.4 (E PIN upgrade) shipped + ≥7-day field test complete
- [ ] ⏳ OR user pre-approves bypass (per SPEC-v1.3 §5 Q7)
- [ ] ⏳ User final go-ahead before Step 5 (rename)
- [x] ✅ Pre-flight recon done (4 invariants per §2)
- [ ] ⏳ `git tag v1.3-final` created
- [ ] ⏳ `v1.4-multi-holiday` branch created + pushed to origin
- [ ] ⏳ `index.html` rewrite deployed to `main`
- [ ] ⏳ Repo renamed to `art-prompt-builder` (web UI)
- [ ] ⏳ Pages re-config verified
- [ ] ⏳ Smoke test 6/6 pass (per §7 Test Gate table)
- [ ] ⏳ README + SPEC docs updated

**Sprint v1.4.0 COMPLETE** when all [ ] checkboxes filled + smoke test 6/6 pass + user confirms ready for Sprint v1.4.1。

---

## 10. Open Questions for User (during execution)

1. **Q1 (CRITICAL)**: Bypass hard prerequisite? (start v1.4.0 NOW without v1.3.4 ship + field test)
   - (A) Wait per spec (recommended) — sprint start ~2026-11-19
   - (B) Bypass prerequisite — sprint start NOW, v1.4.1 mega migration will miss v1.3 features
   - (C) Hybrid — start v1.4.0 NOW (rename only), wait for v1.3.4 before v1.4.1 mega migration
2. **Q2**: Rename commit message style — single mega commit OR atomic Step 1-9 separate commits? (Current plan: atomic)
4. **Q3**: `git tag v1.3-final` message wording — current ("pre-rename snapshot") OR explicit version? (Current plan: pre-rename)
5. **Q4**: Old URL redirect file content — current proposal (Chinese welcome page + auto-redirect) OR minimal `index.html` style?

---

## 11. Change Log

- **2026-09-22** Sprint v1.4.0 PLAN opened。Pre-flight recon ✅ all 4 invariants pass。9 execution steps + 6 atomic commits planned。Hard prerequisite: v1.3.4 ship + 7-day field test (per SPEC-v1.4 §7)。

---

## 12. References

- [SPEC-v1.4.md §2 A — Repo Rename + Pages Re-point](../SPEC-v1.4.md)
- [SPEC-v1.4.md §3 Sprint v1.4.0 — Repo Rename + Pages Re-point (30 min, 🟡)](../SPEC-v1.4.md#sprint-v140--repo-rename--pages-re-point-30-min-)
- [SPEC-v1.4.md §9 Pre-flight recon before v1.4.0](../SPEC-v1.4.md#9-pre-flight-recon-before-v140-memory-rule-6)
- Memory rule 6: Pre-flight recon before worker spawn
- Memory rule 13: Senior-engineer logical-bug pass (5 families)
- Memory rule 14: CI gates (a11y via axe-core)