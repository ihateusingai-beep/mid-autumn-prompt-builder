# 中秋節畫畫提詞器 ｜ Mid-Autumn Prompt Builder

> 給同學一起揀 — 中度智障程度嘅學生用大圖卡揀主角、活動、畫風、顏色，自動砌出**中文 + 英文** AI 圖像提詞，老師複製去 Midjourney / DALL·E / 即夢就用得。

一個 zero-backend 嘅 single-file HTML app — 淨係一個 `midautumn-prompt-builder.html`，配 22 張預載 JPG。所有 state 喺 localStorage，可離線用。

---

## 🌕 Live Demo

**Status: ⏳ GitHub Pages 等待啟用**

Intended URL: <https://ihateusingai-beep.github.io/mid-autumn-prompt-builder/>

目前回傳 `HTTP 404`。原因係 workflow `.github/workflows/deploy.yml` 雖然就位，但 GitHub repo settings → Pages 尚未指派 environment / source。**啟用步驟(老師 / 維護者)：**

1. 去 <https://github.com/ihateusingai-beep/mid-autumn-prompt-builder/settings/pages>
2. **Source** 揀 `GitHub Actions`(唔係 `Deploy from a branch`)
3. 儲存後 1-2 分鐘 deploy 完成,URL 自動生效

啟用後,README 頂呢段會更新做正式 link。

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
| Voice | `window.speechSynthesis`(廣東話 `zh-HK`) |
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
- **語音廣東話要 iOS / macOS / Chrome 內置 voice**:部分 Android 機(尤其大陸品牌)冇 `zh-HK` voice,fallback 會讀普通話或者英文
- **Mobile keyboard 輸入學生名** 喺細芒(<375px)會擋住「生成」掣,目前要 scroll 一下
- **多於 5 個學生同時用** localStorage quota 易撞,但呢個 app 設計係 1 部機 1 個老師用,history 共用,唔算 bug
- **GitHub Pages URL 未啟用**(見頂部 ⏳)— 啟用後即可用

---

## 🛣 Roadmap(v1.2 — 等待 teacher input 揀方向)

候選 scope,未 frozen:

- **A. 多語言**:除繁中 + 英,加簡中 / 廣東話話音 prompt
- **B. 教師版 prompt export**:一鍵 export 全部 prompt 成 PDF / Google Doc,方便改作業
- **C. 列印 layout 改善**:現時 print CSS 基本,想加每張卡一頁嘅 worksheet mode
- **D. 自動 class roster**:支援多學生,自動存每個 student 嘅 prompt 集
- **E. 圖像生成 backend hook**:直接 generate Midjourney / DALL·E API,唔使人手 copy

---

## 📜 License

Code: MIT。圖片(22 張 JPG)係 AI 生成,自用 OK,轉售 / 再 train 請自行評估。

---

## 🙏 Credits

Built for: 香港 SEN 老師(中度智障學生班房)。Thanks to all teachers who trialed it during dev.
Stack: GitHub Pages / GitHub Actions / Tailwind / Font Awesome / Noto Sans TC.

🐰 中秋快樂,祝同學畫出佢哋嘅月亮。