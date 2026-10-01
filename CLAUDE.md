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

## 部署

- 已發布為 Claude Artifact（私人連結）；尚未推 GitHub／Pages（依偏好，實驗性新工具上線前先問）。
- 本機預覽 port 8823；注意 `localhost:8823` 在 Playwright 瀏覽器有其他專案殘留的 service worker，測試請改用 `127.0.0.1:8823`。
