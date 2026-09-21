# SPEC v1.3 — Mid-Autumn Prompt Builder (DRAFT)

> **Status**: 🟡 DRAFT 2026-09-22。Scope 喺 7-day field test 之後 freeze。Carries over v1.2 deferred items + 新 retrospective candidates。
> **Owner**: kencheng
> **Repo**: <https://github.com/ihateusingai-beep/mid-autumn-prompt-builder>
> **Live**: <https://ihateusingai-beep.github.io/mid-autumn-prompt-builder/>
> **Previous**: [`SPEC-v1.2.md`](./SPEC-v1.2.md) (✅ COMPLETED 2026-09-22)

---

## 0. TL;DR (30s 讀完)

v1.3 carries 2 個 deferred v1.2 features + 3 個 retrospective candidates from v1.2.3 + 1 個 known-limitation fix。共 6 個 sprints 可行,**會 freeze 落 3-4 個** based on field test data。

| Sprint | Features | 估時 | 風險 | Source |
|---|---|---|---|---|
| **v1.3.1** | Image-gen UX iteration (rate-limit / cache / seed) | 2-3 hr | 🟢 | Retrospective from v1.2.3 |
| **v1.3.2** | B Prompt Export (PDF / TXT / rich) | 4-6 hr | 🟢 | Deferred from v1.2.2 |
| **v1.3.3** | D Class Roster (multi-student) | 7-10 hr | 🟡 (schema migration) | Deferred from v1.2.2 |
| **v1.3.4** | Service Worker offline-first | 3-5 hr | 🟢 | Known Limitation #1 |
| **v1.3.5** | Teacher PIN upgrade + session timeout | 1-2 hr | 🟢 | D privacy consideration |
| **v1.3.6** | (TBD by field test) | — | — | — |

**Trigger**: v1.2 closed + 7-day field test (SEN teacher + student) + user confirms scope。

---

## 1. v1.2 Baseline (carryover)

- 11 commits on `main`(2026-09-21 → 2026-09-22)
- 1352 行 / ~71KB single-file HTML
- Live URL: <https://ihateusingai-beep.github.io/mid-autumn-prompt-builder/>
- 0 axe-core a11y violations
- 0 secrets in localStorage / git
- 5 features shipped (A / C / E + UX patch)
- 2 features deferred (B / D)
- See [`SPEC-v1.2.md`](./SPEC-v1.2.md) §9 Closeout for full inventory

---

## 2. v1.3 Feature Spec (6 candidates → 3-4 ship)

### A. Image-gen UX Iteration (v1.3.1)

**What**: 根據 field test 結果 iterate image generation UX。

**Candidates** (rank by field-test signal):
1. **Image caching** — localStorage cache by prompt hash,避免 re-fetch 同一 prompt
2. **Seed control** — let teacher pin seed for reproducibility (e.g. 學生想重做)
3. **Rate-limit handling** — Pollinations soft limits ~100 req/min,加 429 backoff + quota warn toast
4. **Better error states** — timeout (60s) vs network fail vs rate-limit 三個 distinct messages
5. **Image gallery** — localStorage store last 10 generated images (base64 → quota warning)
6. **Prompt refinement hint** — if image quality poor, suggest adding more style / color detail

**How**: Incremental edits to `generateImage()` + 老師 mode panel 加新 toggles。

**Surface**:
- Result panel: 1 「⚙️」掣 expand options (seed, retry count)
- 老師 mode panel: 1 new section「Image Caching」with clear-cache button

**Acceptance** (TBD per field test — minimum):
- Re-generate 同一 prompt → 即時 return cached image (no network)
- Generate 60 圖 / hr → quota warn toast 提示
- Timeout 60s + network fail + 429 rate-limit 三個 distinct UX
- 老師 cache clear 按鈕 work

**Bug Family Audit (memory rule 13)**:
- F1 default-state desync: ✅ Cache first-lookup 唔 miss
- F4 silent data loss: ✅ Retry button + clear error message
- F5 dead code: ⚠️ Cache key naming vs `prompt` variable scope — review carefully

---

### B. Prompt Export (v1.3.2 — quick win)

**What**: 老師可一鍵 export 全部 prompt(歷史 + 當前)。3 種 format。

**How**:
- 加「📥 全部 Export」掣(老師 mode panel 內)
- **PDF**: 用 print CSS(已有)→ 叫 user「揀列印 → 存 PDF」(零 backend,最 portable)
- **Plain Text**: `Blob` + `URL.createObjectURL` + `<a download>`. Filename: `midautumn-prompts-YYYY-MM-DD.txt`
- **Rich Clipboard**: `navigator.clipboard.write([new ClipboardItem({'text/html': ...})])` 直接 paste 入 Google Doc

**Surface**:
- 老師 mode panel 加 3 個 export 按鈕(PDF / TXT / GDoc)
- 每個 prompt entry 旁加「複製」獨立 button

**Acceptance**:
- Export 0 entry → button disable + toast warn
- TXT export 包含 timestamp + student name + zh-HK/zh-CN/en 三段 prompt
- Rich clipboard paste 入 Google Doc → 有 basic formatting (heading + bullet)
- PDF via print → 4-page handout (per UX patch worksheet layout)

**Bug Family Audit**:
- F4 silent data loss: ✅ Empty history 唔 export,先 show count
- F1 default-state desync: ✅ Default = 當前 + 歷史 all,user 可 toggle
- F3 enable-condition: ✅ Export enable = entries.length > 0

---

### C. Class Roster (v1.3.3 — biggest risk)

**What**: 老師可管理學生 roster,每個 student 嘅 prompt 自動歸入該 student 名下。

**How**:
- `localStorage` schema 改:
  - 舊 `midautumn_history` (array of prompt) → 加 `studentName` 欄位(已有 optional field)
  - 新 `midautumn_roster` (array of `{name, createdAt}`)
- 老師 mode panel 加「👥 學生名單」section:add / remove student
- 學生名 input 變 dropdown,只列 roster 入面嘅 name
- Roster 10 人上限,超過 warn

**Surface**:
- 老師 mode: roster CRUD + 「清除全部」+ 「Export Roster」
- Step 5 學生名 input 變 `<select>` populated from roster
- History view 加 filter by student + 「匯出呢個學生」掣

**Acceptance**:
- 加 5 個 student → 揀 prompt → 歷史 filter 返出正確 entry
- Roster 超 10 → toast warn,但仍然 add
- Empty roster → drop-down 自動 hide,fallback 舊 free-text input
- Export 1 個學生 → 該學生所有 prompt 1-click TXT

**Bug Family Audit**:
- F1 default-state desync: ✅ Dropdown default = first student in roster
- F4 silent data loss: ✅ Roster 寫 localStorage 前 toast confirm + auto-backup to `midautumn_history_v12`
- F3 enable-condition: ✅ Dropdown enable = roster.length > 0
- F2 reset-on-render: ⚠️ Roster 改完 dropdown 唔 crash,要小心 re-render 順序
- **CRITICAL**: Schema migration — old `midautumn_history` entries without `studentName` must migrate or risk F4 silent data loss

**Pre-flight blockers**:
- Q1 (per SPEC-v1.2 §5): Auto-migrate old history? (recommend yes + backup)
- Q3 (per SPEC-v1.2 §5): Add teacher PIN session timeout?

---

### D. Service Worker offline-first (v1.3.4)

**What**: 解決 Known Limitation #1 — 第一次 load 要裝晒 22 張 JPG(~6 MB),離線 reload 會空白。

**How**:
- 加 `service-worker.js`(root)
- Cache strategy:
  - **HTML / CSS / JS**: stale-while-revalidate
  - **JPG (img/)**: cache-first + 30-day expiry
  - **Image generation response (Pollinations)**: network-only (don't cache external AI)
- Register on first load, update on subsequent

**Surface**:
- Status indicator (top-right): 「📶 Online」/「✈️ Offline」
- First-load install progress (optional)

**Acceptance**:
- 第一次 load → reload → 完全 offline 仍 work (HTML + 22 JPG 從 cache)
- 新版 HTML push → user 收到 update toast 「有新版本,撳重載」
- Service worker fail (e.g. file://) → gracefully degrade (no error)

**Bug Family Audit**:
- F1 default-state desync: ✅ Online/offline status match real network
- F4 silent data loss: ⚠️ Cache invalidation on HTML update must not silent-fail
- F5 dead code: ⚠️ Service worker scope vs GitHub Pages base path(`/mid-autumn-prompt-builder/`) — review carefully

---

### E. Teacher PIN upgrade + session timeout (v1.3.5)

**What**: 因為 D Roster 涉及學生個私隱,加 PIN session timeout。

**How**:
- 老師 mode 進去 → 30 分鐘 idle timeout → 自動退出
- 老師 mode panel 加「PIN 強度」選項 (4-digit / 6-digit / alphanumeric)
- 加 「🔒 自動鎖定」 toggle,default ON

**Surface**:
- 老師 mode panel: 3 個 toggle (強度 / timeout / 自動鎖定)
- Idle warning toast: 「5 分鐘後自動鎖定」(30s 前)

**Acceptance**:
- 入老師 mode → 30 分鐘無動作 → 自動退出返 home
- Idle 25 分鐘 → warn toast 顯示
- 強度升級到 alphanumeric → 舊 PIN 強制 reset

**Bug Family Audit**:
- F4 silent data loss: ⚠️ Auto-lock 唔 loss unsaved edits,先 toast warn + 30s grace
- F1 default-state desync: ✅ Timeout default = 30min,鎖定 default = ON

---

### F. Field-test retrospective (v1.3.6 — TBD)

**Source**: 7-day field test with SEN teacher + student。

**Categories** to watch for:
- **Cognitive load issues** (student confusion during 5-step flow)
- **a11y issues** (screen reader / keyboard nav gap)
- **Print issues** (worksheet layout when actually printed)
- **Image-gen issues** (rate limit / quality / safety prompts hit)
- **localStorage edge cases** (multi-device, quota hit, schema migration)

**Output**: Use findings to ship **1 ad-hoc patch** between sprints or extend a sprint scope。

---

## 3. Sprint Breakdown (DRAFT)

### Sprint v1.3.1 — Image-gen UX iteration (2-3 hr, 🟢)

**Trigger**: v1.2 closeout + 7-day field test ≥50% complete (3.5 days)。

**Commits** (TBD per chosen features):
1. `feat(image): localStorage cache by prompt hash + clear button`
2. `feat(image): seed control + reproducibility UI`
3. `feat(image): rate-limit backoff + quota warn toast`
4. `fix(image): distinct error states (timeout vs network vs 429)`
5. `docs: README v1.3.1 changelog`

**Test gate** (memory rule 14):
- ✅ Re-fetch 同一 prompt → instant return from cache
- ✅ 60 req/hr → quota warn toast 1 次
- ✅ axe-core: 0 violations

**Rollback**: Teacher toggle「disable cache」即時 disable, no code revert。

---

### Sprint v1.3.2 — Prompt Export (4-6 hr, 🟢)

**Trigger**: v1.3.1 ship + 7-day field data。

**Commits** (carry-over from v1.2.2 plan):
1. `feat(B): plain-text export with Blob download`
2. `feat(B): PDF print-to-PDF instructions`
3. `feat(B): rich clipboard (text/html) for Google Doc`
4. `feat(B): per-entry 複製 button`
5. `docs: README v1.3.2 changelog`

**Test gate**:
- ✅ Export 0 entry → button disable
- ✅ TXT export 包含 timestamp + student name + 3-lang prompts
- ✅ Rich clipboard paste 入 Google Doc → formatting 正常
- ✅ axe-core: 0 violations

**Rollback**: revert commits。

---

### Sprint v1.3.3 — Class Roster (7-10 hr, 🟡) — biggest risk

**Trigger**: v1.3.2 ship + 7-day field data + user answers Q1 (auto-migrate) + Q3 (PIN upgrade)。

**Commits**:
1. `chore(D): schema migration utility — backup midautumn_history to midautumn_history_v12`
2. `feat(D): roster CRUD in teacher mode + localStorage schema`
3. `feat(D): student-name dropdown populated from roster + free-text fallback`
4. `feat(D): history filter by student + per-student export`
5. `feat(D): roster 10+ warning toast`
6. `docs: README v1.3.3 changelog + migration guide`

**Test gate**:
- ✅ Roster add 11 個 → toast warn
- ✅ Dropdown empty 時 fallback free-text input
- ✅ Schema migration: 5 個舊 entry (no studentName) → all preserved + 顯示 「(未命名)」
- ✅ LocalStorage quota < 5MB (20 students × 50 prompts)
- ✅ axe-core: 0 violations

**Risk**: 🟡 中 (schema migration can silent-fail if backup write fails)

**Rollback**: Backup key (`midautumn_history_v12`) 永久保留 → 可以 restore。

---

### Sprint v1.3.4 — Service Worker offline-first (3-5 hr, 🟢)

**Trigger**: v1.3.3 ship + 7-day field data。

**Commits**:
1. `feat(offline): service-worker.js — stale-while-revalidate for HTML/CSS/JS`
2. `feat(offline): cache-first for img/ with 30-day expiry`
3. `feat(offline): online/offline status indicator`
4. `fix(offline): update toast on new HTML version`
5. `docs: README v1.3.4 changelog`

**Test gate**:
- ✅ Offline reload → HTML + 22 JPG from cache
- ✅ New version push → user 收到 update toast
- ✅ Service worker scope match GitHub Pages base path

**Risk**: 🟢 (cache-first 已 well-tested pattern)

---

### Sprint v1.3.5 — Teacher PIN upgrade (1-2 hr, 🟢)

**Trigger**: v1.3.3 同時 ship (PIN upgrade 同 Roster 一齊做合理),否則 v1.3.4 後做。

**Commits**:
1. `feat(security): PIN strength options (4-digit / 6-digit / alphanumeric)`
2. `feat(security): session timeout 30min + grace warning`
3. `feat(security): auto-lock toggle default ON`
4. `docs: README v1.3.5 changelog`

**Test gate**:
- ✅ Idle 30min → 自動退出
- ✅ Idle 25min → warn toast
- ✅ 強度升級 → 舊 PIN reset

---

## 4. Bug Family Risk Register (per memory rule 13)

| Family | Risk v1.3 | Mitigation |
|---|---|---|
| F1 default-state desync | 🟡 A, C, D, E | localStorage 記住 user 偏好;Schema migration default = old schema |
| F2 reset-on-render | 🟡 C | Dropdown re-render 唔 crash — explicit re-render check |
| F3 enable-condition off-by-concept | 🟢 B, D | Export enable = entries.length > 0;Service worker enable = HTTPS only |
| F4 silent default data loss | 🔴 C, D | Schema migration backup + Service worker cache invalidation toast |
| F5 dead code via name collision | 🟡 D | Service worker scope path vs GitHub Pages base path;Cache key vs prompt variable |

---

## 5. Open Questions for User (要答先開 sprint 1+)

1. **v1.3 揀邊 3-4 sprints ship?** (見 §0 TL;DR)
2. **D Roster migration strategy**: Auto-migrate all old history? (建議 yes + backup) **OR** prompt user per-entry?
3. **D Roster 上限**: 10 students hard cap? **OR** warn-only (let user add 50 if they want)?
4. **Service Worker scope**: Apply to whole `midautumn-prompt-builder/` path? **OR** separate paths per file?
5. **Teacher PIN session timeout**: 30 min? **OR** configurable per session?
6. **Sprint cadence keep 7 days?** Or compress after v1.3.1?
7. **v1.3.6 (field-test retrospective)** ship as ad-hoc patch? Or fold into v1.3.1?

---

## 6. Reference Table — Feature × Property (memory rule 13 §X.5)

| | A Image UX | B Export | C Roster | D Offline | E PIN |
|---|---|---|---|---|---|
| Touches localStorage | +1 key (cache) | 0 (read existing) | +1 key + schema change | +1 key (SW version) | 0 (existing PIN key) |
| New UI element | 1 section + toggles | 3 buttons | 1 section + 1 dropdown | 1 status indicator | 3 toggles |
| New file | 0 | 0 | 0 | `service-worker.js` | 0 |
| New CSS class | 0 | 0 | 0 | 1 (.offline-banner) | 0 |
| Bug families touched | F1, F4, F5 | F1, F3, F4 | F1, F2, F3, F4 | F1, F4, F5 | F1, F4 |
| Dependency on others | 0 | History (existing) | B (recommended order) | 0 | C (privacy context) |
| Rollback ease | Easy (toggle) | Easy (revert) | Hard (schema migration) | Medium (cache clear) | Easy (toggle) |
| Test device | Desktop + iPad | Desktop + iPad | Desktop + iPad | Desktop offline mode | Desktop + iPad |
| Estimated LoC | +120 | +120 | +180 | +100 | +60 |
| **Status 2026-09-22** | 🟡 DRAFT | 🟡 DRAFT | 🟡 DRAFT | 🟡 DRAFT | 🟡 DRAFT |

**Total LoC estimate (3 sprints avg)**: ~420 (從 1352 → ~1770). Single file integrity preserved if SW kept separate.

---

## 7. Acceptance Criteria (Sprint 1 Ready Check)

Ready to start sprint 1 (v1.3.1) when:
- [x] ✅ v1.2 closed (`86c9141`)
- [ ] ⏳ ≥7 day field test complete
- [ ] ⏳ User confirms v1.3 scope (this doc)
- [ ] ⏳ User answers §5 open questions (Q1, Q7 priority)
- [x] ✅ Bug family audit done (F1/F4/F5 for A)

**Trigger for sprint 2**: v1.3.1 ship + 7-day field data。
**Trigger for sprint 3 (D Roster)**: v1.3.2 ship + user answers Q1/Q2/Q3 + 7-day field data。

---

## 8. Change Log

- **2026-09-22** v1.3 DRAFT opened。6 sprint candidates from v1.2 retrospective + field-test signals。Scope freeze after 7-day field test.

---

## 9. Pre-flight recon before freeze (memory rule 6)

When user picks v1.3 scope, run this before locking sprint plan:
1. `git log --oneline -20` — confirm v1.2 closeout commit (`86c9141`) on main
2. `gh run list --limit 5` — confirm last 5 Pages deploys success
3. `curl -sI -L <live URL>` — confirm live URL serves the app
4. Drift table — actual v1.2 commits vs SPEC §6 Reference Table

Token cost: ~30s. Stop + surface + await user if any fail.

---

## 10. Next Action

User 揀 v1.3 嘅 sprint priority (見 §5 Q1) + 答 §5 Q7 (cadence) → freeze scope → open Sprint v1.3.1 plan doc。