# CLAUDE.md

`slide-style-yaml-studio` 是「簡報風格工坊」：NotebookLM 簡報風格圖鑑＋YAML 設計框架產生器（單檔前端，無後端、無建置步驟）。

## 由來

參考一份外部 Artifact「簡報風格圖鑑」（12 種風格卡片＋每張附 NotebookLM 用 YAML，結構為 `global_design` ＋ `slides`）建置，2026-10-01 建立。差異：
- 12 種風格**取材自本工作區既有工具的視覺主題**（提示詞控制台、ai-prompt-generator、亞馬遜工具組、animal-ecommerce-adventure、costar-game、six-thinking-hats-generator、where-what-how-strategy-studio、政府補助產生器、social-post-grader、crispe-game、annual-marketing-calendar、ppt-course-video），範例題目取自 `ai-course-hub` 真實課程。
- 多了「YAML 產生器」分頁：選風格 → 從 `ai-course-hub` 33 門課帶入大綱（自動拆成封面／大綱／各單元／回顧）→ 逐頁改類型／標題／重點 → 即時產生完整 YAML 與 NotebookLM「自訂說明」。

## 架構重點

- 全部在 `index.html`。`STYLES` 陣列每筆含 `colors`（[key, value, 註解]）、`typo`、`rules`、`m`（7 種版面語彙 motifs：cover/list/card/flow/emph/data/scene）、`sample`、`pv(t,st)` 縮圖函式。
- `visualFor(style, slide, n)` 用「頁面類型 × 風格 motifs」組出每頁 `visual_description`；`buildYaml()` 組 YAML，字串一律雙引號並跳脫 `\` 與 `"`（已用 PyYAML 驗證可解析）。
- `const COURSES` 是從 `../ai-course-hub/data/courses.json` 抽出的精簡版（id/t/cat/o）**內嵌快照**；課程更新後要手動重抽：
  ```bash
  python -c "import json,io;d=json.load(open('../ai-course-hub/data/courses.json',encoding='utf-8'));print(json.dumps([{'id':c['id'],'t':c['title'],'cat':c['category'],'o':c['outline']} for c in d],ensure_ascii=False,separators=(',',':')))"
  ```
  再取代 `index.html` 中 `const COURSES = [...]` 那一行。
- 縮圖用 container query 單位（cqw）：**`.pv` 本身的 padding／border 不能用 cqw**（對自己無效，會退回 viewport 單位），要放到內層 `.abs` 元素。
- 草稿存 localStorage（`ssy.draft`、`ssy.tab`），全部 try/catch。
- 刻意不提供下載 .yaml 按鈕（Artifact 檢視器會擋下載），只提供複製。

## 週邊功能（2026-10-01 補上）

- **頂部跑馬燈**：資料來源為 Google 試算表「跑馬燈」（`1sSBXW2dAc-4u0j21Q72MzNEBIhDccShhr1iJcAdG0UE`）的「工作表1」A 欄（A1「內容」略過），跟工作區共用公告是同一張表。讀取順序：localStorage 快取（`slideStyleYamlStudioMarquee`）→ `DEFAULT_ITEMS`（2026-10-01 的工作表1快照）→ 依環境二選一：
  - **一般網頁（GitHub Pages／本機）**：沒有 `window.claude`，打共用公告 Apps Script 端點 `MARQUEE_CHECK_URL`。
  - **Claude Artifact 檢視器**：CSP 擋外部連線，改用宣告的 `mcp` capability（`Google Drive` 連接器的 `read_file_content`，`watchTool` 每 20 分鐘刷新），用 `parseSheetText()` 從回傳的 markdown 表格取出工作表1 A 欄並還原 `\~` 跳脫。檢視者沒連 Drive 或不同意時，保留快取／快照，不顯示錯誤。宣告 mcp 後此 Artifact 不能公開分享。
  - 公告文字支援 `[文字](網址)` 與單獨網址自動轉連結。
- **PWA 加入主畫面**：`manifest.json`＋`service-worker.js`（network-first＋同源快取，比照 ecommerce-health-dashboard）＋`icons/`（PIL 繪製：深綠底＋兩張疊放的投影片卡＋縮排的 YAML 線條＋琥珀圓點，192/512/maskable-512/apple-touch-icon；產生腳本未進 repo）。頁首 `#installBtn`，iOS／Mac Safari 退回文字提示（`#toast`）。2026-10-01 在正式網域用 Playwright 驗證：`beforeinstallprompt` 有觸發、`Page.getInstallabilityErrors` 為空、service worker 已接管。Artifact 檢視器內（偵測到 `window.claude`）不能安裝，按鈕隱藏。
- **訪客計數器**：footer `visitor-badge.laobi.icu`，`page_id=m255525.slidestyleyamlstudio`；圖片載入失敗（Artifact CSP 擋外部圖片）時整列隱藏。
- **使用警語＋創作者資訊**：footer 兩欄（警語｜創作者卡片），信箱用可選取文字＋複製按鈕而非 mailto（Artifact 內 mailto 不可靠）。
- **操作手冊 `manual.html`**：與主頁同一組色彩 token、支援深色模式；含快速開始、圖鑑、產生器、九種頁面類型對照表、YAML 結構、NotebookLM 套用步驟、常見問題、使用警語、創作者資料與授權。Artifact 發布時用 `files` 一併上傳，主頁以相對連結 `manual.html` 開啟。

## 部署

- **Claude Artifact**（私人）：<https://claude.ai/artifact/K3mSFBE3qDJhGiAZMs5Auf>，用 `files` 一併上傳 `manual.html`，`capabilities` 宣告 mcp Google Drive。重新發布時省略 `capabilities` 會沿用既有宣告。
- **GitHub Pages**（公開）：repo <https://github.com/M255525/slide-style-yaml-studio>，Actions workflow `.github/workflows/deploy-pages.yml`（比照 ecommerce-health-dashboard，push 到 `master` 即部署）→ <https://m255525.github.io/slide-style-yaml-studio/>。
- `index.html` 開頭保留 `<!doctype html>`（GitHub Pages 需要；Artifact 發布時多餘的 doctype 會被忽略）。
- 本機預覽 port 8823；注意 `localhost:8823` 在 Playwright 瀏覽器有其他專案殘留的 service worker，測試請改用 `127.0.0.1:8823`。
