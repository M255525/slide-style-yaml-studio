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

- **頂部跑馬燈**：獨立 IIFE，逐字沿用 `ecommerce-health-dashboard` 的實作（深色細長版型 `#111827`／`#fbbf24`、共用工作區公告 Apps Script 端點 `MARQUEE_CHECK_URL`），localStorage key `slideStyleYamlStudioMarquee`。**多了 `DEFAULT_ITEMS` 預設公告**：Claude Artifact 檢視器的 CSP 會擋掉對 script.google.com 的 fetch，沒有快取時就顯示預設三則；本機或一般網站上會抓到共用公告取代。
- **使用警語＋創作者資訊**：footer 兩欄（警語｜創作者卡片），信箱用可選取文字＋複製按鈕而非 mailto（Artifact 內 mailto 不可靠）。
- **操作手冊 `manual.html`**：與主頁同一組色彩 token、支援深色模式；含快速開始、圖鑑、產生器、九種頁面類型對照表、YAML 結構、NotebookLM 套用步驟、常見問題、使用警語、創作者資料與授權。Artifact 發布時用 `files` 一併上傳，主頁以相對連結 `manual.html` 開啟。

## 部署

- 已發布為 Claude Artifact（私人連結）；尚未推 GitHub／Pages（依偏好，實驗性新工具上線前先問）。
- 本機預覽 port 8823；注意 `localhost:8823` 在 Playwright 瀏覽器有其他專案殘留的 service worker，測試請改用 `127.0.0.1:8823`。
