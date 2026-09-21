# SPEC v1.3 — Mid-Autumn Prompt Builder (FROZEN)

> **Status**: ✅ **FROZEN 2026-09-22**。Scope locked: **A + B + C + E**(4 sprints)。Skip D (offline-first) + F (field-test retrospective patch)。Cadence 7-day field data per sprint。
> **Owner**: kencheng
> **Repo**: <https://github.com/ihateusingai-beep/mid-autumn-prompt-builder>
> **Live**: <https://ihateusingai-beep.github.io/mid-autumn-prompt-builder/>
> **Previous**: [`SPEC-v1.2.md`](./SPEC-v1.2.md) (✅ COMPLETED 2026-09-22)

---

## 0. TL;DR (30s 讀完)

v1.3 scope = 4 sprints frozen from 6 candidates。每 sprint ≥7 日 field data window。Total estimate ~15-21 hr, LoC ~480。

| Sprint | Features | 估時 | 風險 | Source |
|---|---|---|---|---|
| **v1.3.1** | A Image-gen UX iteration (cache / seed / rate-limit / error states) | 2-3 hr | 🟢 | Retrospective from v1.2.3 |
| **v1.3.2** | B Prompt Export (PDF / TXT / rich clipboard) | 4-6 hr | 🟢 | Deferred from v1.2.2 |
| **v1.3.3** | C Class Roster (multi-student + auto-migrate) | 7-10 hr | 🟡 (schema migration) | Deferred from v1.2.2 |
| **v1.3.4** | E Teacher PIN upgrade + session timeout | 1-2 hr | 🟢 | C privacy consideration |
| ~~v1.3.4~~ | ~~D Service Worker offline-first~~ | — | — | ❌ DEFERRED to v1.4 |
| ~~v1.3.6~~ | ~~F Field-test retrospective patch~~ | — | — | ❌ Independent commit if needed |

**Trigger**: v1.2 closed (`86c9141`+`8b0b393`) → ≥7-day field test → open Sprint v1.3.1 plan doc。

**Decisions locked 2026-09-22** (per user questionnaire `ask_d7c2b8d9ddc6e49f14db21a1`):
1. Scope = A + B + C + E
2. Cadence = keep 7 日 field data per sprint
3. C Roster migration = auto-migrate + backup `midautumn_history_v12`

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

**Migration strategy** (RESOLVED 2026-09-22 per user questionnaire):
- ✅ **Auto-migrate + backup** (NOT prompt per-entry)
- On first load after deploy:
  1. 讀 `midautumn_history`,check if `studentName` field exists on every entry
  2. Backup: 寫整個舊 array 到 `midautumn_history_v12` key(永久保留,可手動 restore)
  3. Migrate: 對冇 `studentName` 嘅 entry, 設 `studentName = '(未命名)'`(之後 user 可手動改名 / 刪除)
  4. Write back to `midautumn_history`
  5. Toast notification: 「已 migrate X 個舊 prompt 到新 schema, 舊 data 備份咗喺 v12」
- Backup key 永久保留, manual restore via DevTools if needed

**Bug Family Audit**:
- F1 default-state desync: ✅ Dropdown default = first student in roster
- F4 silent data loss: ✅ Auto-backup to `midautumn_history_v12` BEFORE write-back
- F3 enable-condition: ✅ Dropdown enable = roster.length > 0
- F2 reset-on-render: ⚠️ Roster 改完 dropdown 唔 crash,要小心 re-render 順序
- **CRITICAL**: Schema migration — old `midautumn_history` entries without `studentName` must migrate or risk F4 silent data loss → auto-backup 防呆

---

### D. Service Worker offline-first (v1.3.4) — ❌ DEFERRED to v1.4

**Status**: Not in v1.3 scope per user 2026-09-22 decision。Reason: 22 張 JPG cache ~6MB 對 offline 用有意義,但 v1.3 集中喺 user-facing features + Roster schema migration,offline work 屬 v1.4 DX improvement。

**Carries to v1.4**:
- Service worker registration (root `service-worker.js`)
- Cache strategy: stale-while-revalidate (HTML/CSS/JS) + cache-first 30-day (img/)
- Online/offline indicator + new-version toast

**Why deferred**:
- D Roster schema migration 同 sprint ship 風險高,offline work 會加 conflict surface
- Roster 已 ship 後, v1.4 再加 offline-first 更穩
- 22 張 JPG 第一次 load 慢 but 學生班房通常有 wifi

### E. Teacher PIN upgrade + session timeout (v1.3.4) ✅ IN SCOPE

**What**: 因為 C Roster 涉及學生個私隱,加 PIN session timeout + 強度選項。

**How**:
- 老師 mode 進去 → 30 分鐘 idle timeout → 自動退出(per SPEC-v1.2 §5 Q3 答咗「30 min」)
- 老師 mode panel 加「PIN 強度」選項 (4-digit default / 6-digit / alphanumeric)
- 加 「🔒 自動鎖定」 toggle,default ON

**Surface**:
- 老師 mode panel: 2 個 toggle (PIN 強度 / 自動鎖定)
- Idle warning toast: 「5 分鐘後自動鎖定」(30s 前 grace)

**Acceptance**:
- 入老師 mode → 30 分鐘無動作 → 自動退出返 home
- Idle 25 分鐘 → warn toast 顯示
- 強度升級到 alphanumeric → 舊 PIN 強制 reset,新 PIN 設定 wizard

**Bug Family Audit**:
- F4 silent data loss: ⚠️ Auto-lock 唔 loss unsaved edits,先 toast warn + 30s grace
- F1 default-state desync: ✅ Timeout default = 30min,鎖定 default = ON
- F3 enable-condition: ✅ Auto-lock enable = 老師 mode active AND toggle ON

### F. Field-test retrospective — ❌ NOT in v1.3 scope (independent commit if needed)

**Source**: 7-day field test with SEN teacher + student。

**Status**: NOT a v1.3 sprint per user decision。如果 field test 揭發 critical bug(silent data loss / a11y regression / schema corruption),獨立 hotfix commit 入 v1.2.x maintenance,唔阻 v1.3 sprint cadence。

**Watch categories** (same as §2 F original):
- Cognitive load issues
- a11y issues (screen reader / keyboard nav)
- Print issues (worksheet layout when actually printed)
- Image-gen issues (rate limit / quality)
- localStorage edge cases (multi-device, quota, schema)

---

## 3. Sprint Breakdown — FROZEN (4 sprints, A + B + C + E)

### Sprint v1.3.1 — Image-gen UX iteration (2-3 hr, 🟢)

**Trigger**: v1.2 closeout (`86c9141`+`8b0b393`) + 7-day field test ≥50% complete (3.5 days)。

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

**Trigger**: v1.3.2 ship + 7-day field data。

**Migration strategy** (RESOLVED): auto-migrate + backup `midautumn_history_v12`。

**Commits**:
1. `chore(D): schema migration utility — backup midautumn_history to midautumn_history_v12 + auto-assign '(未命名)'`
2. `feat(D): roster CRUD in teacher mode + localStorage schema`
3. `feat(D): student-name dropdown populated from roster + free-text fallback`
4. `feat(D): history filter by student + per-student export`
5. `feat(D): roster 10+ warning toast`
6. `docs: README v1.3.3 changelog + migration guide`

**Test gate**:
- ✅ Roster add 11 個 → toast warn
- ✅ Dropdown empty 時 fallback free-text input
- ✅ Schema migration: 5 個舊 entry (no studentName) → all preserved + 顯示 「(未命名)」+ backup key 存在
- ✅ LocalStorage quota < 5MB (20 students × 50 prompts)
- ✅ axe-core: 0 violations

**Risk**: 🟡 中 (schema migration can silent-fail if backup write fails)

**Rollback**: Backup key (`midautumn_history_v12`) 永久保留 → 可以 restore via DevTools。

---

### Sprint v1.3.4 — Teacher PIN upgrade (1-2 hr, 🟢)

**Trigger**: v1.3.3 ship + 7-day field data (或同 v1.3.3 一齊 ship,如果時間軸 OK)。

**Rationale**: C Roster 涉及學生私隱,PIN 升級一齊 ship 老師 user 唔使 await 過 session timeout warning。

**Commits**:
1. `feat(security): PIN strength options (4-digit default / 6-digit / alphanumeric)`
2. `feat(security): session timeout 30min + grace warning toast (30s before)`
3. `feat(security): auto-lock toggle default ON`
4. `docs: README v1.3.4 changelog`

**Test gate**:
- ✅ Idle 30min → 自動退出返 home
- ✅ Idle 25min → warn toast 顯示
- ✅ 強度升級到 alphanumeric → 舊 PIN reset + 新 PIN wizard

---

### ❌ Deferred (NOT in v1.3)

- **D Service Worker offline-first** → v1.4 (見 §2 D)
- **F Field-test retrospective patch** → independent hotfix if needed (見 §2 F)

---

## 4. Bug Family Risk Register (per memory rule 13)

| Family | Risk v1.3 | Mitigation |
|---|---|---|
| F1 default-state desync | 🟡 A, C, E | localStorage 記住 user 偏好;Schema migration default = old schema |
| F2 reset-on-render | 🟡 C | Dropdown re-render 唔 crash — explicit re-render check |
| F3 enable-condition off-by-concept | 🟢 B, E | Export enable = entries.length > 0;Auto-lock enable = 老師 mode active AND toggle ON |
| F4 silent default data loss | 🔴 C | Schema migration auto-backup to `midautumn_history_v12` BEFORE write-back (per Q2 RESOLVED) |
| F5 dead code via name collision | 🟢 A, C | Cache key vs prompt variable (review);roster array vs history array naming (review) |

**D Service Worker**: NOT in v1.3 scope → see v1.4 (F4 + F5 risk reduce when added)。

---

## 5. Open Questions for User — FROZEN

**Resolved 2026-09-22** (per user questionnaire `ask_d7c2b8d9ddc6e49f14db21a1`):
1. ✅ **v1.3 scope**: A + B + C + E (skip D + F)
2. ✅ **C Roster migration**: Auto-migrate + backup `midautumn_history_v12`
3. ⏸ **C Roster cap**: 10 students hard cap? — **DEFERRED to v1.3.3 sprint-time** (not blocking freeze, user can confirm during v1.3.3 spec)
4. ⏸ **D Service Worker scope**: N/A — deferred to v1.4 (唔擋 v1.3 freeze)
5. ✅ **Teacher PIN session timeout**: 30 min default (per SPEC-v1.2 §5 Q3 answer)
6. ✅ **Sprint cadence**: keep 7-day field data per sprint
7. ✅ **v1.3.6 (field-test retrospective)**: NOT in v1.3 scope, independent hotfix if needed

**Open during sprint execution** (NOT blocking v1.3 freeze):
- v1.3.1 (Image UX): 揀邊 1-2 個 image UX features 優先 (cache / seed / rate-limit / error states)
- v1.3.2 (Export): TXT filename format + rich clipboard HTML structure
- v1.3.3 (Roster): roster cap (10 hard vs warn-only), 學生名 validation rules

**Previous open questions (superseded by freeze 2026-09-22):**
1. ~~v1.3 揀邊 3-4 sprints ship?~~ → §0 TL;DR locked: A + B + C + E
2. ~~D Roster migration strategy~~ → auto-migrate + backup `midautumn_history_v12`
3. ~~D Roster 上限~~ → deferred to v1.3.3 sprint-time
4. ~~Service Worker scope~~ → N/A (deferred to v1.4)
5. ~~Teacher PIN session timeout~~ → 30 min default
6. ~~Sprint cadence~~ → keep 7-day field data
7. ~~v1.3.6 retrospective patch~~ → NOT in v1.3 scope (independent hotfix)

---

## 6. Reference Table — Feature × Property (memory rule 13 §X.5)

| | A Image UX | B Export | C Roster | E PIN |
|---|---|---|---|---|
| Touches localStorage | +1 key (cache) | 0 (read existing) | +1 key + schema change | 0 (existing PIN key) |
| New UI element | 1 section + toggles | 3 buttons | 1 section + 1 dropdown | 2 toggles |
| New file | 0 | 0 | 0 | 0 |
| New CSS class | 0 | 0 | 0 | 0 |
| Bug families touched | F1, F4, F5 | F1, F3, F4 | F1, F2, F3, F4 | F1, F3, F4 |
| Dependency on others | 0 | History (existing) | B (recommended order) | C (privacy context) |
| Rollback ease | Easy (toggle) | Easy (revert) | Hard (schema migration) | Easy (toggle) |
| Test device | Desktop + iPad | Desktop + iPad | Desktop + iPad | Desktop + iPad |
| Estimated LoC | +120 | +120 | +180 | +60 |
| **Status 2026-09-22** | ✅ FROZEN v1.3.1 | ✅ FROZEN v1.3.2 | ✅ FROZEN v1.3.3 | (skipped) | ✅ FROZEN v1.3.4 |

**Deferred to v1.4+**:
- **D Service Worker offline-first** — F1, F4, F5 (LoC +100, 1 new file `service-worker.js`)
- **F Field-test retrospective patch** — ad-hoc hotfix if needed (LoC TBD)

**Total LoC estimate (4 sprints v1.3)**: ~480 (從 1352 → ~1830). Single file integrity preserved (冇新 file, SW 留 v1.4).

---

## 7. Acceptance Criteria — FROZEN

Ready to start sprint 1 (v1.3.1) when:
- [x] ✅ v1.2 closed (`86c9141` + `8b0b393` push verified)
- [ ] ⏳ ≥7 day field test complete (after v1.2 closeout)
- [x] ✅ User confirms v1.3 scope (this doc) — **FROZEN 2026-09-22 per questionnaire**
- [x] ✅ User answers §5 open questions — **Q1/Q2/Q5/Q6/Q7 RESOLVED**
- [x] ✅ Bug family audit done (F1/F4/F5 for A, F1/F2/F3/F4 for C)

**Trigger for sprint 2 (v1.3.2 B Export)**: v1.3.1 ship + 7-day field data。
**Trigger for sprint 3 (v1.3.3 C Roster)**: v1.3.2 ship + 7-day field data。
**Trigger for sprint 4 (v1.3.4 E PIN)**: v1.3.3 ship + 7-day field data (or co-ship with v1.3.3 if field data allows)。

---

## 8. Change Log

- **2026-09-22** v1.3 DRAFT opened。6 sprint candidates from v1.2 retrospective + field-test signals。Scope freeze after 7-day field test.
- **2026-09-22** v1.3 **FROZEN** — scope locked to A + B + C + E (skip D + F). C Roster migration = auto-migrate + backup `midautumn_history_v12`. Cadence = keep 7-day field data per sprint. Per user questionnaire `ask_d7c2b8d9ddc6e49f14db21a1`.

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

**v1.3 FROZEN 2026-09-22**。Open Sprint v1.3.1 plan doc after:
- ≥7 day field test from v1.2 closeout (2026-09-22)
- Or user pre-approves sprint v1.3.1 plan without 7-day buffer (per §5 Q7)

Sprint v1.3.1 plan doc will need:
- Pick 1-2 image UX features (cache / seed / rate-limit / error states) per §5 open
- LoC + commit plan per memory rule 4 worker discipline
- Bug family re-audit (per memory rule 13 senior pass)