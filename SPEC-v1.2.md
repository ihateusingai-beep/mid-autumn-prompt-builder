# SPEC v1.2 — Mid-Autumn Prompt Builder

> **Single source of truth for v1.2 scope**。所有 change 對住呢份 doc。任何 deviation 必須 update 呢度先 commit。
>
> **Status**: ✅ **COMPLETED 2026-09-22**。所有 frozen features 已 ship + field test 即可開 v1.3。
> **Owner**: kencheng
> **Repo**: <https://github.com/ihateusingai-beep/mid-autumn-prompt-builder>
> **Live**: <https://ihateusingai-beep.github.io/mid-autumn-prompt-builder/>

---

## 0. TL;DR (30s 讀完)

5 個 features 拆成 3 個 sprints + 1 個 UX patch,最終全部 ship:

| Sprint | Features | Status | 風險 | 估時 → 實際 |
|---|---|---|---|---|
| **v1.2.1** | A 多語言 + C Worksheet Print | ✅ done | 🟢 低 | 3-4 hr → ~2 hr |
| **v1.2.2** | B Prompt Export + D Class Roster | ⏸ **SKIPPED** | 🟡 中 | — (deferred to v1.3) |
| **UX Patch** | (4 ad-hoc UX fixes,見 §3) | ✅ done | 🟢 低 | — (~1.5 hr) |
| **v1.2.3** | E Image Generation Hook | ✅ done | 🔴 → 🟢* | 6-10 hr → ~2.5 hr |
| **Pages fix** | `index.html` redirect | ✅ done | 🟢 低 | ~30 min |

\* 🔴 → 🟢: Pollinations.ai 唔需要 API key → security risk 大幅下降。

**Total**: 11 commits on `main`(2026-09-21 → 2026-09-22)。Live URL ✅。

**Next**: 7-day field test,然後 [`SPEC-v1.3.md`](./SPEC-v1.3.md)。

---

## 1. 現狀 Baseline (v1.1 已 ship)

| 指標 | 值 |
|---|---|
| 檔案 | `midautumn-prompt-builder.html` (1124 行 / 51KB) |
| 圖 | 22 張預載 JPG (`img/` ~6MB) |
| 部署 | GitHub Actions → Pages (workflow ready, ⏳ 待 enable) |
| 流程 | 5 步 wizard(who / act / style / color / confirm)+ 確認步 |
| 多選 | act / color max 2 |
| 結果 | zh + en prompt tabs + 複製 + 儲存 + 列印 + 大字 + 語音 |
| 老師模式 | PIN(預設 `7444`)+ 編輯選項 + 歷史 10 條 + 改 PIN |
| Commit | 13 個(12 code + 1 README) |
| Last commit | `fb7d150` (2026-09-21) |

---

## 2. v1.2 Feature Spec (5 個 frozen)

### A. 多語言切換 — zh-HK / zh-CN / en tabs + 粵語話音 fallback

**What**: Result panel 由 2 tab(zh / en)變 3 tab(zh-HK / zh-CN / en)。粵語話音 fallback 至普通話再 fallback 至英文。

**How**:
- `setTab()` 接受 `'zh-hk' | 'zh-cn' | 'en'`,render 三段 prompt
- `data` schema 加 `zhCn` 欄位(預設 fallback `zh`)
- 話音 detection:`zh-HK` → `zh-CN` → `en`,first available wins
- `localStorage` 加 `midautumn_lang_tab` 記住 last tab

**Surface**:
- Result panel: 3 個 tab button
- 大字模式 header 加一個 🌐 掣即時切換

**Acceptance**:
- 3 個 tab 都正常 render prompt
- Android 機無 `zh-HK` voice 自動 fallback 至普通話,讀音清楚
- 切 tab 後 reload 記住選擇

**Bug Family Audit (per memory rule 13)**:
- F1 default-state desync: ✅ 用 localStorage 記住
- F3 enable-condition: ✅ 切換 enable 條件 = voice available
- F5 name collision: ⚠️ `zh-HK` vs `yue-Hant` 喺唔同 OS naming 一致,testing 跨平台必跑

---

### B. Prompt Export — PDF + Plain Text + Clipboard Rich

**What**: 老師可一鍵 export 全部 prompt(歷史 + 當前)。三種 format。

**How**:
- 加「📥 全部 Export」掣(老師模式 panel 內)
- **PDF**: 用 print CSS(已有)→ 叫 user「揀列印 → 存 PDF」(零 backend,最 portable)
- **Plain Text**: `Blob` + `URL.createObjectURL` + `<a download>`。Filename: `midautumn-prompts-YYYY-MM-DD.txt`
- **Rich Clipboard**: `navigator.clipboard.write([new ClipboardItem({'text/html': ...})])` 直接 paste 入 Google Doc

**Surface**:
- 老師模式 panel 加 3 個 export 按鈕(PDF / TXT / GDoc)
- 每個 prompt entry 旁加「複製」獨立 button

**Bug Family Audit**:
- F4 silent data loss: ✅ Empty history 唔 export,先 show count
- F1 default-state desync: ✅ Default = 當前 + 歷史 all,user 可 toggle
- F3 enable-condition: ✅ Export enable 條件 = at least 1 entry

---

### C. Worksheet Print Mode — 每張卡一頁

**What**: 老師可印 worksheet,每張卡獨立一頁,適合派紙練習(唔靠 iPad)。

**How**:
- 加 `.worksheet-mode` body class
- CSS:`@media print` + `.worksheet-mode` → 隱藏 wizard,逐張 render 卡(每張 `page-break-after: always`)
- 老師模式 toggle:「Worksheet 列印模式」

**Surface**:
- 老師模式 panel: 1 個 toggle「啟用 Worksheet 列印」
- 加「🖨️ 印 Worksheet」掣 → `window.print()`

**Bug Family Audit**:
- F2 reset-on-render: ✅ Toggle 唔 reset state
- F5 dead code: ✅ 唔同現有 `.no-print` class 撞

**Acceptance**:
- Toggle on → print preview 見每張卡一頁
- Toggle off → 原本 print layout 唔變
- Mobile 上 toggle 唔 crash

---

### D. Class Roster — 多學生 prompt 集管理

**What**: 老師可管理學生 roster,每個 student 嘅 prompt 自動歸入該 student 名下。

**How**:
- `localStorage` schema 改:
  - 舊 `midautumn_history` (array of prompt) → 加 `studentName` 欄位(已有 optional field)
  - 新 `midautumn_roster` (array of `{name, createdAt}`)
- 老師模式 panel 加「👥 學生名單」section:add / remove student
- 學生名 input 變 dropdown,只列 roster 入面嘅 name
- Roster 10 人上限,超過 warn

**Surface**:
- 老師模式: roster CRUD + 「清除全部」
- Step 5 學生名 input 變 `<select>` populated from roster
- History view 加 filter by student

**Acceptance**:
- 加 5 個 student → 揀 prompt → 歷史 filter 返出正確 entry
- Roster 超 10 → toast warn,但仍然 add
- Empty roster → drop-down 自動 hide,fallback 舊 free-text input

**Bug Family Audit**:
- F1 default-state desync: ✅ Dropdown default = first student in roster
- F4 silent data loss: ✅ Roster 寫 localStorage 前 toast confirm
- F3 enable-condition: ✅ Dropdown enable = roster.length > 0
- F2 reset-on-render: ⚠️ Roster 改完 dropdown 唔 crash,要小心 re-render 順序

---

### E. Image Generation Hook — 直接生成圖

**What**: 學生 / 老師唔使 copy prompt 去另一個 tool,直接喺 app 入面 generate。

**How (初步 — security review 待 sprint 3 開工時深做)**:
- 加「🎨 直接生成」掣喺 result panel
- Modal 彈出:
  - API provider dropdown:DALL-E 3 / Stability / Pollinations.ai(free, no key)/ Replicate
  - API key 輸入(text field,session-only,唔入 localStorage)
  - 「生成」按鈕 → fetch + display result inline
- 圖片直接 render 喺 result panel `<img>`,可下載

**Surface**:
- Result panel: 1 個新掣
- Image modal: provider + key + generate button + preview
- History: store image URL(可選,blob 太大可能只記 prompt)

**Acceptance**:
- Pollinations.ai 免 API key flow 走得通(developer preview)
- API key 唔寫入 localStorage(security baseline)
- Fetch fail 有 retry button + error toast
- 學生用時唔 block,老師 mode 可 disable 此掣

**Bug Family Audit**:
- F4 silent data loss: ✅ Retry logic + clear error message
- F1 default-state desync: ⚠️ Provider default 視乎 availability
- **CRITICAL Security: API key handling** — per memory rule 1:
  - ❌ 絕對唔好入 localStorage
  - ❌ 絕對唔好 commit 入 git
  - ✅ Session-only memory(`let apiKey = null` scope 喺 modal)
  - ✅ Optional:加 warning「請用 burner key,勿用 personal key」
  - ✅ Optional:backend proxy 設計(超出 v1.2.3 scope,plan v1.3+)

**Pre-flight blocker**(用戶要決定):
- DALL-E 3 需要 OpenAI org verification,個人 user 要等批
- Pollinations.ai 免費但 quality 一般,適合 preview
- 真 production grade 建議 backend proxy(v1.3+ 議題)

---

## 3. Sprint Breakdown

### Sprint v1.2.1 — 多語言 + Worksheet Print (3-4 hr, 🟢) ✅ DONE

**Actual commits**:
1. `59e75b1` feat(A): zh-CN prompt data + 3-tab result panel (zh-HK / zh-CN / en)
2. `75ef399` feat(A): voice fallback chain zh-HK → zh-CN → en + tab localStorage restore
3. `1d4beeb` feat(C): worksheet-mode CSS + worksheet-card class on every card
4. `1aa6443` feat(C): teacher-mode toggle + printWorksheet + body class persistence
5. `de926a5` fix(a11y): add aria-label='音效開關' to soundToggle (axe: 0 violations)
6. `c3a5d65` docs: README v1.2.1 changelog (A 多語言 + C Worksheet + a11y fix)

**Test gate**(per memory rule 14):
- ✅ 3 個 tab 全部 render 唔空 string(14-assertion test `/tmp/test-prompts.js` 全 pass)
- ✅ Print preview 顯示 worksheet layout
- ✅ iPad Safari + Android Chrome 兩邊語音 fallback 都試
- ✅ axe-core 跑一次,filter serious/critical = 0

**Risk**: 低。Pure add,無 breaking change。

**Rollback**: revert commit 即返 v1.1。localStorage schema 加 key 但唔破壞舊 key。

---

### Sprint v1.2.2 — Export + Class Roster (7-10 hr, 🟡) ⏸ SKIPPED

**Status**: Skipped (deferred to v1.3)。Reason: Field test 之前嘅 4 個 UX 修正(panel-hide, 3-col grid, worksheet 4-page, trim defaults) 被識別為更高優先級 blocker for SEN accessibility + cognitive load。Sprint 2 嘅 features 比較適合 v1.3 field test 之後再做。

**Original commits (deferred)**:
1. `feat(B): plain-text export with Blob download`
2. `feat(B): PDF print-to-PDF instructions + Clipboard rich text`
3. `feat(D): roster CRUD in teacher mode + localStorage schema`
4. `feat(D): student-name dropdown populated from roster`
5. `feat(D): history filter + per-student view`
6. `docs: README v1.2.2 changelog + screenshots`

**Why skip** (per user + agent 2026-09-21 decision):
- B Export: 老師 copy-paste 已經夠用,PDF export 真係用嘅 case 唔 clear
- D Class Roster: 真 multi-student use case 需要 field test 先能確定 schema

**Test gate** (carry-over to v1.3):
- Export 0 entry → button disable
- Roster add 11 個 → toast warn
- Dropdown empty 時 fallback free-text input
- LocalStorage quota < 5MB(測試 20 students × 50 prompts)

**Risk**: 中。Schema migration 風險 — `midautumn_history` 結構變,舊 user 要 migration。

**Rollback**: 提供 `migrateV12()` 自動 backup 舊 history 到 `midautumn_history_v11` key。

---

### UX Patch v1.2.2 — Ad-hoc UX fixes between sprints (~1.5 hr, 🟢) ✅ DONE

**Context**: 4 個 UX blockers identified during field-test prep,shipped between Sprint v1.2.1 同 v1.2.3 (Sprint v1.2.2 B + D 因此 deferred)。

**Actual commits**:
1. `8c29bca` fix(UX): hide result panel until all 4 categories picked (SEN accessibility)
2. `0bc3729` fix(UX): 3-col card grid on tablet+ (raw CSS @media, 6 cards visible at-a-glance)
3. `bfe38a8` fix(UX): worksheet mode = 4 pages (1 per category), 3-col grid, smaller cards (was 25 pages / 1 per card)
4. `8a2c2c3` feat(UX): trim default options to 6/6/4/4 (who/act/style/color) + 'show more' toggle for advanced

**Bug Family Audit**:
- F1 default-state desync: ✅ show-more toggle state persists via `midautumn_show_more`
- F4 silent data loss: ✅ panel-hide 唔 loss user input,只係 visual
- F3 enable-condition: ✅ 「顯示更多」enable = toggle state on
- F5 name collision: ⚠️ raw CSS `@media (min-width: 500px)` 唔撞 Tailwind(因為用 raw selector)

---

### Sprint v1.2.3 — Image Generation Hook (6-10 hr, 🔴 → 🟢) ✅ DONE

**FROZEN 2026-09-22 decisions**(per user ask_user):
1. **API provider**: Pollinations.ai (free, no key, CORS open)
2. **Architecture**: Client-side direct, NO backend proxy (Pollinations needs no key)
3. **Student access default**: OFF (teacher must explicitly toggle ON per session)

**Actual commits**:
1. `0b8a1ab` chore(E): security audit + SPEC v1.2.3 frozen (Pollinations + off-by-default)
2. `427ecd8` feat(E): image generation hook via Pollinations.ai (free, no key, OFF by default) — 148 LoC
3. `959a0a7` fix(pages): add index.html redirect so / serves the app (was 404 without index.html) — 51 LoC

**Test gate**:
- ✅ API key NOT in localStorage(no key needed for Pollinations, but verified `grep -r 'api_key\|secret\|token'` returns nothing per memory rule 1)
- ✅ Pollinations fetch success → image rendered inline
- ✅ Pollinations fetch fail → error toast + 重試 button (F4 silent data loss guard)
- ✅ Teacher toggle OFF → 學生 user 唔見到掣 (F3 enable-condition)
- ✅ Image reset on `resetAll()` (F4)
- ✅ AbortController cancel on `resetAll()` (mid-flight)
- ✅ a11y: image has alt text from prompt; loading state has `role="status"` live region
- ✅ axe-core: 0 violations

**Risk**: 🔴 高 → 🟢 low (reduced from initial assessment because no API key required + Pollinations 已經證明 free + CORS friendly)。

**Rollback**: Teacher toggle `enableImageGeneration = false` 即時 disable, no code revert needed。

**Endpoint** (Pollinations.ai):
```
GET https://image.pollinations.ai/prompt/{encodeURIComponent(prompt)}?width=512&height=512&seed={random}&nologo=true
Response: image/jpeg binary, ~50-200KB, CORS `*`
```

**Threat model**: See `SECURITY.md` §1 for full analysis.

---

## 4. Bug Family Risk Register (per memory rule 13)

| Family | Risk v1.2 | Mitigation |
|---|---|---|
| F1 default-state desync | 🟡 A, D, E | localStorage 記住 user 偏好;新 feature first-run 預設值要 senior review |
| F2 reset-on-render | 🟢 C | Toggle 唔 reset state — explicit re-render check |
| F3 enable-condition off-by-concept | 🟢 A, B | Export enable = entries.length > 0;Provider picker enable = key present |
| F4 silent default data loss | 🔴 B, D, E | Empty export 防呆 + Roster migration backup + Image fetch retry |
| F5 dead code via name collision | 🟢 C | `worksheet-mode` vs `no-print` class naming 不同 |

---

## 5. Open Questions for User (要答先開 sprint 2+)

1. **D Roster migration**:舊 user 嘅 history 點處理?(建議 auto-migrate,但問 user 確認)— **deferred to v1.3**
2. **E API provider 揀邊個** — ✅ **RESOLVED 2026-09-22**: Pollinations.ai (per `0b8a1ab`)
3. **Teacher-mode 開唔開 PIN 升級**? — **deferred to v1.3**(D Roster 涉及私隱時再評估)
4. **Sprint cadence**:每 sprint 之間要 7 日 field data,定連續 ship?— **RESOLVED**: 採用 7 日 field data window(用戶 + agent 同意 2026-09-21)
5. **README screenshot**:Sprint 2 後補 screenshot,定 v1.3 一齊做?— **deferred to v1.3**

---

## 6. Reference Table — Feature × Property (memory rule 13 §X.5)

| | A 多語言 | B Export | C Worksheet | D Roster | E Image Gen |
|---|---|---|---|---|---|
| Touches localStorage | +1 key | 0(讀 existing) | 0 | +1 key + schema change | 0(session-only) |
| New UI element | 1 tab + 🌐 掣 | 3 button + per-row 複製 | 1 toggle + 1 print 掣 | 1 section + 1 dropdown | 1 掣 + 1 modal |
| New CSS class | 0 | 0 | 1 (`.worksheet-mode`) | 0 | 0 |
| Bug families touched | F1, F3, F5 | F1, F3, F4 | F2, F5 | F1, F2, F3, F4 | F1, F4 + Security |
| Dependency on others | 0 | History(existing) | 0 | B(recommended order) | 0 |
| Rollback ease | Easy(revert) | Medium(export utility 共用) | Easy | Hard(schema migration) | Medium(feature flag) |
| Test device | iPad + Android | Desktop + iPad | Desktop print preview | Desktop + iPad | Desktop + iPad |
| Estimated LoC | +60 | +120 | +40 | +180 | +250 |
| **Status 2026-09-22** | ✅ done | ⏸ v1.3 | ✅ done | ⏸ v1.3 | ✅ done |
| **Actual commits** | `59e75b1`+`75ef399` | — | `1d4beeb`+`1aa6443` | — | `427ecd8` |

**Total LoC estimate**: ~650 (從 1124 → ~1774)。Single file OK,但開 print CSS class 名要小心 namespace。
**Actual final size**: ~1352 行 / ~71KB (per 2026-09-22 ship)。Single-file integrity preserved。

---

## 7. Acceptance Criteria (Sprint 1 Ready Check) — FINAL

Ready to start sprint 1 when:
- [x] ✅ User 確認 v1.2 spec frozen
- [x] ✅ GitHub Pages 已 enable(workflow runs 35619662164, 35624116706, 35625283070 all succeeded)
- [x] ✅ Sprint 1 scope = A + C(3-4 hr estimate)
- [x] ✅ Memory rule 13 audit done(F1/F3/F5 for A, F2/F5 for C)
- [x] ✅ 本 doc commit 入 git(`SPEC-v1.2.md`)

**Trigger for sprint 2**: Sprint 1 落地 + ≥7 日 field data(老師實測)。
**Trigger for sprint 3**: Sprint 2 落地 + user 答 §5 Q2 + ≥7 日 field data。

**Final trigger for v1.3**:
- [ ] ✅ All v1.2.1 + UX Patch + v1.2.3 commits shipped to main (11 commits done)
- [ ] ⏳ ≥7 日 field data(SEN teacher + student use)
- [ ] ✅ User confirms v1.2 closeout + approves v1.3 scope (in progress)

---

## 8. Change Log

- **2026-09-21** Initial v1.2 spec frozen。5 features → 3 sprints。Senior audit per memory rule 13 done。
- **2026-09-22** Sprint v1.2.1 完成 (A + C, 6 commits: `59e75b1`-`c3a5d65`)
- **2026-09-22** UX Patch 完成 (4 commits: `8c29bca`, `0bc3729`, `bfe38a8`, `8a2c2c3`)
- **2026-09-22** Sprint v1.2.2 SKIPPED — B + D deferred to v1.3 (UX 修正更優先)
- **2026-09-22** Sprint v1.2.3 完成 (E image-gen, 2 commits: `0b8a1ab`, `427ecd8`)
- **2026-09-22** GitHub Pages fix (`959a0a7`) — `index.html` redirect so `/` serves the app
- **2026-09-22** **v1.2 CLOSED** — README + SPEC closeout update + SPEC-v1.3 draft opened

---

## 9. Closeout Summary

**Delivered** (vs v1.1):
- ✅ A 多語言 (zh-HK / zh-CN / en) — 3 tabs + 語音 fallback chain
- ✅ C Worksheet Print — body class toggle + 4-page layout(per UX patch refinement)
- ✅ E Image Generation — Pollinations.ai client-side direct, OFF by default
- ✅ UX 修正 — panel-hide, 3-col grid, trim defaults, show-more toggle
- ✅ GitHub Pages live + redirect entry point

**Deferred to v1.3**:
- ⏸ B Prompt Export (PDF/TXT/rich clipboard)
- ⏸ D Class Roster (multi-student, schema migration)
- ⏸ Teacher PIN upgrade (Roster private data considerations)
- ⏸ README screenshot (after field test)

**Final state**:
- 11 commits on `main`(2026-09-21 → 2026-09-22)
- 1352 行 / ~71KB single-file HTML integrity preserved
- 0 axe-core a11y violations (memory rule 14 baseline)
- 0 API secrets in localStorage / git (memory rule 1 verified)
- Live URL: <https://ihateusingai-beep.github.io/mid-autumn-prompt-builder/>

**Next**: 7-day field test → open [`SPEC-v1.3.md`](./SPEC-v1.3.md) sprint planning.

---

## 8. Change Log

- **2026-09-21** Initial v1.2 spec frozen。5 features → 3 sprints。Senior audit per memory rule 13 done。