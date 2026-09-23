# 中秋節畫畫提詞器 ｜ Mid-Autumn Prompt Builder

> 給同學一起揀 — 中度智障程度嘅學生用大圖卡揀主角、活動、畫風、顏色，自動砌出**中文 + 英文** AI 圖像提詞，老師複製去 Midjourney / DALL·E / 即夢就用得。

一個 zero-backend 嘅 single-file HTML app — 淨係一個 `midautumn-prompt-builder.html`，配 22 張預載 JPG。所有 state 喺 localStorage，可離線用。

---

## 🌕 Live Demo

**Status: ✅ Live since 2026-09-22**

**👉 <https://ihateusingai-beep.github.io/art-prompt-builder/>**

> ⚠️ **2026-09-23 改名**: Repo 由 `mid-autumn-prompt-builder` 改名做 `art-prompt-builder`(v1.4 準備 multi-holiday)。新 URL live, 舊 URL 返 404(GitHub Pages 唔支援 auto-redirect)。請 update bookmark / share link。

每次 push `main` 自動 deploy(GitHub Actions `actions/deploy-pages@v4`)。`/` URL 經 `index.html` meta-refresh 跳去 `midautumn-prompt-builder.html`(保留 git history, 0ms redirect hop)。

---

## 🎯 點用(老師版,30 秒)

1. 開 Live Demo 嘅 URL,喺 iPad / Chromebook / 投影機邊開邊畀學生睇
2. **5 步流程**(每步都係大圖卡,撳一下揀,再撳一下確認 ✅):
   - **第 1 步** 👤 邊個係主角?(7 個選項:小朋友 / 一家人 / 兔仔 / 嫦娥 / 小貓 / 我自己 / 爺爺奶奶…)
   - **第 2 步** 🎯 佢哋做緊咩?(可揀 1–2 件:望月 / 拎燈籠 / 食月餅 / 放天燈…)
   - **第 3 步** 🎨 畫面係咩感覺?(6 種:可愛卡通 / 圖畫書 / 剪紙畫 / 夜空 / 簡筆畫 / 溫柔夜景)
   - **第 4 步** 🌙 夜空同顏色(可揀 1–2 種:金黃 / 紅色 / 紫色 / 桂花黃 / 銀白 / 橙紅)
   - **第 5 步** ✅ 確認你嘅選擇,填學生名(可選),撳「生成中秋圖提詞」
3. 底部結果面板有 **中文提詞 / 英文 Prompt** 兩個 tab,撳「複製提詞」就 paste 入 AI 圖像工具
4. 老師可撳右上角 ⚙️ 入「**老師模式**」(預設 PIN `7444`),即場加 / 改選項、睇歷史、改 PIN

**無障礙設計(點解學生用得到)：**
- 大字模式(右上角「**大字**」掣)— 適合視障學生
- 語音反饋(右上角 🔊)— 撳一下讀出選項名(廣東話 `zh-HK`)
- ARIA labels + 鍵盤導航(← → 切步驟,Enter 確認)
- 隨機掣 🎲 — 唔識揀嘅學生一撳就有 starter idea
- 重來掣 — 唔好結果即清

---

## ✨ Features(已 ship,12 commits)

| 區域 | 功能 |
|---|---|
| **流程** | 5 步 wizard + 確認步 + 進度點 + 「返去 / 下一頁」 |
| **選擇** | 撳 → 揀 → 再撳確認(two-tap 防誤觸)+ 多選上限 2 |
| **結果** | 中文 / 英文 prompt 兩 tab + 複製 + 儲存到 localStorage 歷史 + 列印(print CSS ready) |
| **老師模式** | PIN gate(預設 `7444`)+ 即場編輯 who/act/style 選項 + 歷史 10 條 + 改 PIN |
| **體驗** | 大字模式 / 聲音 toggle / 隨機 / 重來 / 語音 feedback / colored toasts / pick flash 動畫 |
| **無障礙** | ARIA + keyboard nav + role="status" for live toasts |
| **工程** | AudioContext singleton / history by id / XSS escape / TDZ bug fixed |

---

## 🛠 Tech Stack

| 層 | 用咩 |
|---|---|
| Markup | Single-file HTML,零 build step |
| CSS | Tailwind v3 (CDN) + 自訂 print CSS |
| Icons | Font Awesome 6.5 (CDN) |
| Font | Noto Sans TC (Google Fonts CDN) |
| State | Vanilla JS + `localStorage`(PIN / 歷史) |
| Voice | `window.speechSynthesis` + fallback chain (zh-HK → zh-CN → en) |
| Deploy | GitHub Actions → GitHub Pages(`actions/deploy-pages@v4`) |

**無 build,無 npm,無 backend** — 改完 HTML 推上 `main` 即 deploy。

---

## 🧑‍💻 本地跑 / 開發

任何 HTTP server 都得:

```bash
# Option 1: Python
python3 -m http.server 8000
# → http://localhost:8000/midautumn-prompt-builder.html

# Option 2: Node
npx serve .

# Option 3: 直接 double-click HTML(注意:語音 API 要 https / localhost 先 work)
open midautumn-prompt-builder.html
```

**改完之後:**
1. 改 `midautumn-prompt-builder.html`
2. `git add . && git commit -m "..."`
3. `git push origin main`(GitHub Actions 自動 deploy)

**改預設 PIN:** 入老師模式 → 「更改 PIN」,會寫入 `localStorage` 嘅 `midautumn_pin` key,跟住先 load 嘅機就用新 PIN。要 reset 返 default,瀏覽器 DevTools → Application → Local Storage → 刪 `midautumn_pin`。

---

## 🧒 設計取捨(畀同行老師睇)

1. **vocabulary 簡化到 2-3 字**:「小朋友」「兔仔」「拎燈籠」,唔用「嫦娥奔月」「闔家團圓」呢類對中度智障學生太抽象嘅詞
2. **5 步上限**:再加步驟 cognitive load 過重;每步獨立 confirm-step,訓練「揀 → 確認」routine
3. **Negative prompt 內嵌**:中英文 prompt 都內置「no horror / no dark / no messy text」,減低 AI 出恐怖圖嘅機會
4. **圖卡先行,文字輔助**:每張卡有實體 JPG(預載 22 張),唔係 icon — 學生睇圖揀,認知門檻低過讀字
5. **多選限 2**:act 同 color 限 1–2,超出 2 太多 prompt 會變混亂(實測過 3+ 出圖率跌)

---

## 🐛 Known Limitations(老師要知道)

- **無 offline-first cache strategy**:第一次 load 要裝晒 22 張 JPG(~6 MB),之後 service worker 冇做,離線 reload 會空白
- **Mobile keyboard 輸入學生名** 喺細芒(<375px)會擋住「生成」掣,目前要 scroll 一下
- **多於 5 個學生同時用** localStorage quota 易撞,但呢個 app 設計係 1 部機 1 個老師用,history 共用,唔算 bug

---

## 🆕 v1.2.1 Changelog (2026-09-21)

### A. 多語言 Prompt — 3 tabs

- 結果面板由 2 tab(繁中 / 英)變 3 tab:**繁體中文 / 简体中文 / English**
- 每個 data item 都有 `zhCn` field(簡體中文版本)— 28 個選項全部覆蓋
- Negative prompt 都分中港兩地版本(繁 vs 簡)
- Tab 選擇記住喺 `localStorage`(`midautumn_lang_tab`),下次 reload 自動還原
- 適合中港跨境班房 / 普通話學生

### C. Worksheet 列印模式

- 老師模式 panel 加「🖨️ Worksheet 列印」section
- Toggle 啟用後:**每張卡獨立一頁**,學生可貼紙 / 圈出嚟揀
- 列印時自動隱藏 nav / 結果面板 / 語音掣
- Toggle 狀態記住喺 `localStorage`(`midautumn_worksheet_mode`)

### Bonus: 語音 fallback chain

- 之前限制「無 zh-HK voice 嘅 Android 機會讀錯」已解決
- Fallback 順序: `zh-HK` → `zh-CN` → `en`(first available wins)
- 即係內地 Android / 國際 Chrome 都有機會讀到廣東話

### a11y fix

- `#soundToggle` button 加 `aria-label="音效開關"`(axe-core: 0 violations)

**Commits in this sprint**:
- `59e75b1` feat(A): zh-CN prompt data + 3-tab result panel
- `75ef399` feat(A): voice fallback chain + localStorage
- `1d4beeb` feat(C): worksheet-mode CSS + worksheet-card class
- `1aa6443` feat(C): teacher-mode toggle + printWorksheet
- `de926a5` fix(a11y): aria-label on soundToggle
- 詳細 spec 見 [`SPEC-v1.2.md`](./SPEC-v1.2.md)

---

## 🆕 v1.2.2 UX Patch (2026-09-21)

Field test 之前嘅 UX 修正 — 4 個 commit 喺 v1.2.1 同 v1.2.3 之間,sprint 2 (B + D) 因為呢啲修正更優先 skip 咗:

### UX fix 1 — 結果面板 hide until ready
- 一開始 4 個 category 未揀齊,結果面板完全隱藏(`opacity transition`)
- SEN accessibility:避免學生見到空白 prompt 困惑

### UX fix 2 — 3-col card grid on tablet+
- Tablet(≥500px viewport)自動轉 3-col grid,6 張卡一次過見到
- 用 raw CSS `@media (min-width: 500px)`,**唔靠 Tailwind responsive**(FilePanel Browser @ DPR 2x 唔 trigger `sm:`/`md:`/`lg:`)
- Mobile 維持 1-col scroll

### UX fix 3 — Worksheet = 4 pages, 3-col grid
- 原本 worksheet 每張卡 1 頁 → 25 pages 太癲
- 改成 **每 category 1 頁**,共 4 頁,3-col grid 細卡
- 列印時 4 頁 handout 比 25 頁可行

### UX fix 4 — Trim defaults + 「顯示更多」toggle
- 預設選項減: who 7→6 / act 8→6 / style 6→4 / color 6→4
- 每 category 有「**顯示更多**」掣 per-user toggle (`localStorage` `midautumn_show_more`)
- Advanced 選項對中度智障學生 cognitive load 太重, defaults 留俾主流使用

**Commits in this patch**:
- `8c29bca` fix(UX): hide result panel until all 4 categories picked
- `0bc3729` fix(UX): 3-col card grid on tablet+ (raw CSS @media)
- `bfe38a8` fix(UX): worksheet mode = 4 pages (1 per category), 3-col grid
- `8a2c2c3` feat(UX): trim default options to 6/6/4/4 + 'show more' toggle

---

## 🆕 v1.2.3 Changelog — Image Generation Hook (2026-09-22)

🔒 **Security baseline**: Pollinations.ai 唔需要 API key → 零秘密風險。

### E. Image Generation (optional, OFF by default)

- 老師 mode panel 加 **「🎨 Image Generation」** section
- Toggle「啟用 Image Generation」(OFF by default, 學生 user 唔見到掣)
- Result panel 加 **「🎨 生成圖」** 掣 → fetch `https://image.pollinations.ai/prompt/...` 直接 GET, CORS open, ~50-200KB JPEG
- 圖片 render 喺 result panel inline,可撳「下載」save
- **`AbortController`**: `resetAll()` 中止 in-flight request(防止 stale image 寫入 history)
- **Loading spinner** + retry button(失敗時)
- **`localStorage` `midautumn_enable_image_gen`** 記住 toggle 狀態

### 🛡️ Security

- ✅ Zero API key → 唔需要保護
- ✅ **OFF by default** 對 SEN 學生(家長 / 老師 opt-in)
- ✅ Threat model 寫入 [`SECURITY.md`](./SECURITY.md)

### 🐛 Fix: GitHub Pages root URL 404
- 新加 `index.html`(51 行),meta-refresh 0 秒跳去 `midautumn-prompt-builder.html`
- `/` URL 而家 serve 個 app,唔再 404

**Commits in this sprint**:
- `0b8a1ab` chore(E): security audit + SPEC v1.2.3 frozen
- `427ecd8` feat(E): image generation hook via Pollinations.ai (free, no key)
- `959a0a7` fix(pages): add index.html redirect so / serves the app

---

## ✅ v1.2 Status: COMPLETED

**3 個 sprint + 1 個 UX patch,total 11 commits:**

| Sprint | Status | Outcome |
|---|---|---|
| v1.2.1 (A + C) | ✅ done | zh-CN tab, worksheet print, a11y fix |
| v1.2.2 (B + D) | ⏸ skipped | Deferred to v1.3 (UX fixes prioritized) |
| UX Patch | ✅ done | panel-hide, 3-col, worksheet 4-page, trim defaults |
| v1.2.3 (E) | ✅ done | image-gen via Pollinations, OFF by default |

詳細 spec / 風險 register / reference table 見 [`SPEC-v1.2.md`](./SPEC-v1.2.md)。

---

## 🛣 Roadmap — v1.3 candidates

詳細 spec 見 [`SPEC-v1.3.md`](./SPEC-v1.3.md)(draft)。每 sprint ≥7 日 field data,user 確認先開下一個。

**Deferred from v1.2**:
- **B** Prompt Export (PDF/TXT/rich clipboard)— 7-10 hr, 🟡 中
- **D** Class Roster (multi-student)— 7-10 hr, 🟡 中 (schema migration)

**New from v1.2.3 retrospective**:
- Image-gen UX iteration (rate limits / retry count / CDN caching after field test)
- Pollinations quality variance mitigation (negative prompt tuning / seed control)

**Long-term**:
- Service worker offline-first(離線 reload 空白問題)
- PWA installable (Add to Home Screen)
- 多語言 i18n framework(完整 i18n,唔只 prompt)
- Native mobile app(Tauri / Capacitor)

---

## 📜 License

Code: MIT。圖片(22 張 JPG)係 AI 生成,自用 OK,轉售 / 再 train 請自行評估。

---

## 🙏 Credits

Built for: 香港 SEN 老師(中度智障學生班房)。Thanks to all teachers who trialed it during dev.
Stack: GitHub Pages / GitHub Actions / Tailwind / Font Awesome / Noto Sans TC.

🐰 中秋快樂,祝同學畫出佢哋嘅月亮。