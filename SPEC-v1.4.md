# SPEC v1.4 — Art Prompt Builder (multi-holiday) [DRAFT]

> **Status**: 🟡 DRAFT 2026-09-22。Decisions locked per user questionnaire `ask_3fc8b6a7a0576ed388186eb0`:
>
> 1. Repo rename: `mid-autumn-prompt-builder` → **`art-prompt-builder`** (per user Other field)
> 2. Architecture: **1 mega file + theme switcher**
> 3. First 4 holidays: **中秋 + 春節 + 聖誕 + 端午** (HK default)
>
> **Owner**: kencheng
> **Repo (current)**: <https://github.com/ihateusingai-beep/mid-autumn-prompt-builder>
> **Repo (after rename)**: <https://github.com/ihateusingai-beep/art-prompt-builder>
> **Live (current)**: <https://ihateusingai-beep.github.io/mid-autumn-prompt-builder/>
> **Live (after rename)**: <https://ihateusingai-beep.github.io/art-prompt-builder/>
> **Previous**: [`SPEC-v1.3.md`](./SPEC-v1.3.md) (✅ FROZEN 2026-09-22 — 4 sprints in flight)
> **Cadence**: 7-day field data per sprint (same as v1.3)

---

## 0. TL;DR (30s 讀完)

從單節日 mid-autumn app 升級做 multi-holiday platform。架構 = 1 mega HTML + runtime theme switcher。Launcher page 顯示 4 個節日 icon, click 入 mega app 自動切 theme。

| Sprint | Feature | 估時 | 風險 | Depends on |
|---|---|---|---|---|
| **v1.4.0** | Repo rename + Pages URL 重新指向 | 30 min | 🟡 (URL redirect) | — |
| **v1.4.1** | Launcher (`index.html`) + mega file scaffold + theme switcher + mid-autumn migrate | 3-4 hr | 🟡 | v1.4.0 |
| **v1.4.2** | 春節 theme (data + 22 JPG + prompts × 3 lang) | 2-3 hr | 🟢 | v1.4.1 |
| **v1.4.3** | 聖誕 theme | 2-3 hr | 🟢 | v1.4.1 |
| **v1.4.4** | 端午 theme | 2-3 hr | 🟢 | v1.4.1 |

**Total**: ~10-13 hr。LoC: 1830 (current mid-autumn) + 300×3 (new themes) + 100 (launcher + theme switcher) = ~2830 LoC single file.

**Trigger**: v1.3.4 (E PIN upgrade) ship + ≥7-day field test + user confirms v1.4 freeze。

---

## 1. v1.3 Carry-over (mid-autumn baseline)

What we keep / what we migrate:

| Aspect | v1.3 state | v1.4 action |
|---|---|---|
| `midautumn-prompt-builder.html` (1352 行 / ~71KB) | Live | Migrate to `holiday-prompt-builder.html` mid-autumn theme block |
| `index.html` (51 行 meta-refresh) | Redirect | Replace with launcher page (4 holiday icons) |
| `img/` (22 JPGs) | Flat directory | Move to `img/mid-autumn/` subdirectory |
| `midautumn_*` localStorage keys | Various | Keep as-is (mid-autumn namespace scoped) |
| `SECURITY.md` | Pollinations threat model | Keep (per-theme API key same model — none) |
| `SPEC-v1.2.md`, `SPEC-v1.3.md` | Frozen | Archive (kept for history) |

**Migration safety** (memory rule 13 F4 silent data loss): Mid-autumn user localStorage keys unchanged. Theme switcher reads `selected_holiday` (new key, default `mid-autumn`), no data loss.

---

## 2. v1.4 Feature Spec

### A. Repo Rename + Pages Re-point (v1.4.0)

**What**: GitHub repo rename from `mid-autumn-prompt-builder` → `art-prompt-builder`。Pages URL 重新指派。

**How**:
1. Settings → General → "Rename repository" → `art-prompt-builder`
2. Settings → Pages → re-confirm source = `GitHub Actions`, branch = `main`, path = `/`
3. Add `index.html` redirect at OLD URL `mid-autumn-prompt-builder/index.html` (meta-refresh to `https://ihateusingai-beep.github.io/art-prompt-builder/`)
4. Verify both URLs work

**Risks**:
- 🔴 URL break: Old `mid-autumn-prompt-builder` URL → 404 if redirect not set. Mitigation: meta-refresh redirect at old index.html
- 🔴 Pages re-config: GitHub Pages sometimes needs ~5 min after repo rename. Mitigation: verify with curl
- 🟢 Code history preserved (rename keeps git history)

**Acceptance**:
- New URL `https://ihateusingai-beep.github.io/art-prompt-builder/` → 200 (eventually, after Pages re-config)
- Old URL → meta-refresh to new URL
- Git history intact (`git log` shows all 17 commits)

**Bug Family Audit**:
- F4 silent data loss: ✅ Git history preserved through rename
- F5 dead code: ⚠️ Old `index.html` redirect file vs new launcher — review to ensure no collision

---

### B. Launcher + Theme Switcher (v1.4.1)

**What**: `index.html` 變 launcher page, 顯示 4 個節日 icon grid + click 入 mega app with theme pre-selected。Mega `holiday-prompt-builder.html` 加 theme switcher UI (top-right dropdown / 4-icon tab)。

**How**:

**`index.html` (launcher)**:
- 4 holiday cards: 中秋 / 春節 / 聖誕 / 端午 (按時間排序, 最近節日 first)
- Each card: icon (24-48px emoji or SVG) + name (zh-HK + en) + 1-line description
- Click → navigate to `holiday-prompt-builder.html?holiday=<id>` or set localStorage then navigate
- 預設排列順序: 中秋 first (現有最大 user base)

**`holiday-prompt-builder.html` (mega)**:
- Theme switcher UI (top-right corner): 4 icon buttons
- Click → re-render UI with selected theme data
- localStorage `selected_holiday` 記住 choice
- Mid-autumn theme data: migrate from `midautumn-prompt-builder.html`
- Other themes: 3 theme blocks defined in §2 D / E / F

**Surface**:
- Launcher: 4-card grid (responsive — 1-col mobile, 2-col tablet, 4-col desktop)
- Mega app: theme switcher icon row (top-right, next to 「老師模式」 entry)

**Acceptance**:
- Click 中秋 → mid-autumn 主题啟動, vocab + 22 JPG + prompts correct
- Click 春節 → spring festival 主题啟動 (placeholder during v1.4.1, full data after v1.4.2)
- Theme switcher in mega → instant switch (no page reload)
- Refresh page → theme choice persisted via localStorage

**Bug Family Audit (memory rule 13)**:
- F1 default-state desync: ✅ `selected_holiday` localStorage default = `mid-autumn`
- F4 silent data loss: ⚠️ Theme switcher 唔 loss in-progress prompt state (warning before switch if mid-flow)
- F5 dead code via name collision: ⚠️ `selected_holiday` key vs existing `midautumn_*` keys — name scoped, no collision

---

### C. Mid-autumn Theme Migration (v1.4.1, alongside B)

**What**: 將 v1.2 `midautumn-prompt-builder.html` 嘅所有 inline data (vocab + images + prompts × 3 lang) 搬到新 mega file 嘅 mid-autumn theme block。

**How**:
- Copy data block: who[] / act[] / style[] / color[] (with zh-HK/zh-CN/en labels)
- Copy prompts template generator (zh-HK/zh-CN/en negative prompts)
- Copy image paths → update from `img/who-1.jpg` to `img/mid-autumn/who-1.jpg`
- Move 22 JPGs: `img/*.jpg` → `img/mid-autumn/*.jpg` (git mv preserves history)
- Copy teacher mode + 老師 mode + worksheet mode + image-gen toggle (all v1.2.3 features preserved)
- Add v1.3 features: image cache + history per student + per-student export (these get deferred — wait for v1.3 to ship first, then migrate to mega)

**⚠️ Critical timing**:
- v1.4.1 should migrate AFTER v1.3.1-4 ships (image UX + Export + Roster + PIN upgrade)
- Otherwise we lose v1.3 features in migration
- OR: v1.4.1 scaffolds without v1.3 features, then v1.4.2+ re-adds them (more work)

**Decision needed**: Migrate v1.3 features into v1.4.1 mega (delays v1.4 launch) OR ship v1.4 with v1.2.3 baseline + add v1.3 features later?

**Acceptance**:
- Mid-autumn theme in mega = feature parity with v1.2.3 baseline (image-gen, worksheet, 老師 mode)
- 22 JPGs moved to `img/mid-autumn/`
- All existing localStorage keys preserved (`midautumn_*`)

---

### D. Spring Festival Theme (春節) (v1.4.2)

**What**: 春節 theme data + 22 JPGs + prompts × 3 lang。

**Theme content** (research needed):
- **who**: 媽媽 / 爸爸 / 小朋友 / 爺爺奶奶 / 舞獅 / 財神 / 全家福
- **act**: 拎紅包 / 貼揮春 / 放煙花 / 拜年 / 食年糕 / 舞獅 / 逗利是
- **style**: 中國水墨 / 紅金剪紙 / 國畫工筆 / 卡通可愛 / 傳統國畫 / 現代插畫
- **color**: 紅色 / 金色 / 桃紅 / 橙色 / 紫色 / 銀色

**Image generation**:
- 22 JPGs needed: who (7) + act (7) + style (6) + color (variants)
- Use Pollinations.ai (same as v1.2.3) for free generation
- Prompt engineering: 「Chinese New Year red envelope lantern, watercolor style, warm red gold palette」

**Prompts** (zh-HK/zh-CN/en template):
- Same structure as mid-autumn, replace festival/chinese terms
- Negative prompt: 「no horror, no dark, no messy text, no Christian symbols」 (avoid mixing holidays)

**Acceptance**:
- 22 JPGs generated + pre-loaded
- Vocab × 3 lang: each item has `zh / zhCn / en` fields
- Theme switcher: 春節 click → correct data + images render

**LoC estimate**: ~300 LoC (vocab + images + prompts)

**Bug Family Audit**:
- F1 default-state desync: ✅ Theme state isolated from mid-autumn
- F5 dead code: ⚠️ Same `who/act/style/color` keys across themes — name scoped as `theme.<holiday>.who`, OK

---

### E. Christmas Theme (聖誕) (v1.4.3)

**Theme content**:
- **who**: 聖誕老人 / 麋鹿 / 小朋友 / 一家人 / 雪人 / 精靈精靈 / 天使
- **act**: 收禮物 / 佈置聖誕樹 / 堆雪人 / 唱聖誕歌 / 拆禮物 / 烤薑餅 / 等聖誕老人
- **style**: 北歐童話 / 卡通可愛 / 復古水彩 / 雪景寫實 / 剪紙風 / 溫暖夜景
- **color**: 紅色 / 綠色 / 金色 / 銀色 / 白色 / 冰藍

**Negative prompt**: 「no horror, no dark, no religion symbols, no cross imagery」 (avoid mixing holidays)

**Same 22 JPGs + LoC ~300 estimate**.

---

### F. Dragon Boat Theme (端午) (v1.4.4)

**Theme content**:
- **who**: 龍舟隊員 / 小朋友 / 爸爸 / 阿嬤 / 屈原 / 艾草人偶 / 全家
- **act**: 扒龍舟 / 食糉子 / 掛艾草 / 戴五彩繩 / 划船 / 飲雄黃酒 / 包糉子
- **style**: 中國水墨 / 卡通可愛 / 國畫工筆 / 傳統插畫 / 端午海報 / 古典風
- **color**: 綠色 / 紅色 / 黃色 / 紫色 / 青色 / 金色

**Negative prompt**: 「no horror, no dark, no western holiday elements」

**Same 22 JPGs + LoC ~300 estimate**.

---

## 3. Sprint Breakdown (DRAFT)

### Sprint v1.4.0 — Repo Rename + Pages Re-point (30 min, 🟡)

**Trigger**: v1.3.4 (E PIN upgrade) ship + 7-day field test ≥50%。

**Commits**:
1. `chore(rename): update repo references in README + SPEC docs (rename SPEC-v1.4.md content refs)`
2. `feat(rename): Settings → rename repo mid-autumn-prompt-builder → art-prompt-builder (manual step via web)`
3. `fix(rename): add legacy URL redirect at old mid-autumn-prompt-builder/index.html (meta-refresh to new URL)`
4. `docs: README v1.4.0 changelog`

**Test gate**:
- ✅ New URL serves the app (after Pages re-config ~5 min)
- ✅ Old URL meta-refresh to new URL
- ✅ Git history preserved (all 17 commits still in `git log`)

**Risk**: 🟡 中 — URL break for existing users without redirect。

**Rollback**: Repo rename GitHub support can reverse within 90 days。Old URL data still in git history。

---

### Sprint v1.4.1 — Launcher + Mega Scaffold + Mid-Autumn Migration (3-4 hr, 🟡)

**Trigger**: v1.4.0 ship + Pages verify。

**Commits**:
1. `feat(v1.4): scaffold index.html launcher with 4 holiday cards (mid-autumn functional, others placeholder)`
3. `feat(v1.4): scaffold holiday-prompt-builder.html mega file with theme switcher`
4. `refactor(v1.4): migrate mid-autumn theme data from midautumn-prompt-builder.html into mega file`
5. `chore(v1.4): move img/ to img/mid-autumn/ (git mv preserves history)`
6. `fix(v1.4): update mid-autumn image paths in mega file`
7. `docs: README v1.4.1 changelog (multi-holiday architecture)`

**Test gate**:
- ✅ Launcher renders 4 cards (mid-autumn clickable, others placeholder)
- ✅ Mega file mid-autumn theme = feature parity with v1.2.3
- ✅ Theme switcher UI present (mid-autumn selected by default)
- ✅ axe-core: 0 violations
- ✅ Localstorage keys preserved (`midautumn_*` unchanged)

**Risk**: 🟡 中 — large refactor risk (memory rule 5 >500 LoC → 1.5x budget, mini-audit required)

**Rollback**: revert commits, restore old `midautumn-prompt-builder.html` as canonical

---

### Sprint v1.4.2 — Spring Festival (2-3 hr, 🟢)

**Trigger**: v1.4.1 ship + 7-day field data。

**Commits**:
1. `feat(theme): spring-festival vocab data block (who/act/style/color × 3 lang)`
2. `feat(theme): generate 22 spring-festival JPGs via Pollinations.ai`
3. `feat(theme): spring-festival prompt templates (zh-HK/zh-CN/en)`
4. `feat(theme): theme switcher integration test (mid ↔ spring)`
5. `docs: README v1.4.2 changelog`

**Test gate**:
- ✅ Click 春節 → correct images + vocab + prompts
- ✅ Prompt generation in 3 langs (zh-HK/zh-CN/en)
- ✅ Negative prompt 內嵌
- ✅ axe-core: 0 violations

**Risk**: 🟢 — pure additive, no breaking change

---

### Sprint v1.4.3 — Christmas (2-3 hr, 🟢)

Same pattern as v1.4.2, different theme data。

---

### Sprint v1.4.4 — Dragon Boat (2-3 hr, 🟢)

Same pattern, different theme data。

---

## 4. Bug Family Risk Register (per memory rule 13)

| Family | Risk v1.4 | Mitigation |
|---|---|---|
| F1 default-state desync | 🟡 B, C, D, E, F | `selected_holiday` localStorage default = `mid-autumn`;Theme switcher instant update |
| F2 reset-on-render | 🟢 B | Theme switcher 唔 reset wizard progress if mid-flow (warn before) |
| F3 enable-condition off-by-concept | 🟢 B | Launcher card clickable = theme data loaded; placeholder = not clickable |
| F4 silent default data loss | 🔴 A, C | Repo rename git history preserved;Mid-autumn migration 唔 loss localStorage keys |
| F5 dead code via name collision | 🟡 B, C, D, E, F | Theme keys scoped `theme.<holiday>.*`;Image paths scoped `img/<holiday>/` |

---

## 5. Open Questions for User

1. **v1.3 features migration timing**: v1.4.1 migrate WITH v1.3 features (image UX / Export / Roster / PIN upgrade after they ship) OR ship v1.4.1 with v1.2.3 baseline + add v1.3 features later?
2. **Brand wording** (中文): "節日畫畫提詞器" (current) OR "節日藝術創作工具" OR "Art Prompt Studio"?
3. **Launcher default sort**: 按時間 (next upcoming holiday first) OR 按 user base (中秋 first)?
4. **Theme switcher position**: Top-right corner icon row OR 「⚙️ 設定」 dropdown menu OR 「主題」 tab?
5. **v1.4 dev branch**: work on `main` directly OR feature branch `v1.4-multi-holiday` and merge after?
6. **Old URL behavior**: meta-refresh redirect (current plan) OR 410 Gone OR keep old `midautumn-prompt-builder.html` as separate legacy file?

---

## 6. Reference Table — Feature × Property (memory rule 13 §X.5)

| | A Rename | B Launcher | C Migration | D Spring | E Christmas | F Dragon |
|---|---|---|---|---|---|---|
| Touches localStorage | 0 | +1 key (`selected_holiday`) | 0 (preserve) | 0 (theme scoped) | 0 | 0 |
| Adds `renderTheme()` function | — | ✅ | ✅ | — | — | — |
| Generates 22 JPGs | — | — | 0 (moves existing) | ✅ (Pollinations) | ✅ | ✅ |
| New UI element | — | 1 launcher page | 0 (rebrand) | 1 theme switch entry | 1 | 1 |
| New file | — | `index.html` (rewrite) | `holiday-prompt-builder.html` (new) | 0 | 0 | 0 |
| New directory | — | — | `img/mid-autumn/` | `img/spring-festival/` | `img/christmas/` | `img/dragon-boat/` |
| Bug families touched | F4 | F1, F2, F5 | F4, F5 | F1, F5 | F1, F5 | F1, F5 |
| Dependency on others | — | C | v1.3.4 (PIN) | B, C | B, C | B, C |
| Rollback ease | Hard (URL break risk) | Easy (revert) | Medium (large refactor) | Easy (additive) | Easy | Easy |
| Test device | Desktop + iPad | Desktop + iPad | Desktop + iPad | Desktop + iPad | Desktop + iPad | Desktop + iPad |
| Estimated LoC | ~50 (redirect file + docs) | ~150 (launcher + theme switcher) | ~200 (migration scaffold) | ~300 (theme block) | ~300 | ~300 |
| **Status 2026-09-22** | 🟡 DRAFT | 🟡 DRAFT | 🟡 DRAFT | 🟡 DRAFT | 🟡 DRAFT | 🟡 DRAFT |

**Total LoC estimate (5 sprints v1.4)**: ~1300 (從 1352 → ~2650). Single mega file integrity preserved with theme blocks。

---

## 7. Acceptance Criteria — DRAFT

Ready to start sprint v1.4.0 when:
- [ ] ⏳ v1.3.4 (E PIN upgrade) shipped to main
- [ ] ⏳ ≥7 day field test complete after v1.3.4
- [ ] ⏳ User confirms v1.4 freeze (this doc, all §5 open Q resolved)
- [ ] ⏳ Pre-flight recon: confirm GitHub repo rename permissions + Pages access
- [x] ✅ Bug family audit done (F1/F4/F5 for all themes)

**Trigger for v1.4.1**: v1.4.0 ship + URL verify
**Trigger for v1.4.2-4**: v1.4.1 ship + 7-day field data per sprint

---

## 8. Change Log

- **2026-09-22** v1.4 DRAFT opened。3 decisions locked: repo rename `art-prompt-builder`, 1 mega file + theme switcher, 4 holidays (中秋/春節/聖誕/端午)。6 sprint candidates v1.4.0-4。Per user questionnaire `ask_3fc8b6a7a0576ed388186eb0`。
- **2026-09-22** SPEC-v1.3.md linkage updated (Next: SPEC-v1.4.md DRAFT).

---

## 9. Pre-flight recon before v1.4.0 (memory rule 6)

Before repo rename:
1. `gh repo view ihateusingai-beep/mid-autumn-prompt-builder --json name,viewerPermission,defaultBranchRef` — verify rename permission
2. `curl -sI -L <old URL>` — confirm old URL working pre-rename (baseline)
3. Check GitHub Pages settings access (Settings → Pages → source = GitHub Actions)
4. Backup plan: git tag `v1.3-final` on current main BEFORE rename

Token cost: ~30s. STOP + surface + await user if any fail.

---

## 10. Next Action

User 答 §5 open questions (priority: Q1 migration timing + Q5 branch strategy + Q6 URL redirect) → freeze scope → open Sprint v1.4.0 plan doc after v1.3.4 ships + 7-day field test.

**Hard prerequisite**: v1.3 (4 sprints A+B+C+E) MUST ship before v1.4.0 starts. Per user decision 2026-09-22 "(i) 繼續 ship v1.3 as-is"。

**Timeline estimate** (v1.3 → v1.4):
- v1.3.1 Image UX: ship ~2026-10-01 (after 7-day field test from 2026-09-22)
- v1.3.2 Export: ship ~2026-10-15
- v1.3.3 Roster: ship ~2026-10-29
- v1.3.4 PIN: ship ~2026-11-12
- v1.4.0 start: ~2026-11-19 (after 7-day field test)
- v1.4.4 ship: ~2026-12-17 (4 sprints × 7-day cadence)

Total: 3 months from now to v1.4.4 ship。