---
name: dental-lit
description: 牙科文獻五步管線：PICO 定形→雙軌搜（語意＋PubMed MeSH）→三層驗貨（撤稿／來源／定位，逐筆解析 PMID/DOI）→滾雪球→證據表。觸發：「找◯◯的文獻」（快查）、「深挖◯◯」（完整跑）、「查證出處」、「抓全文／圖表」，及文獻查詢單。需 pubmed／openalex MCP。
---

# dental-lit — 牙科文獻檢索與驗貨管線

> 分享版 2026-08-28：源自多輪臨床實跑後的封裝（雙題快查、10 單批次、出處鏈查證、OA 圖表抓取）。
> 前置需求：pubmed 與 openalex MCP（入門包 SETUP.md 階段 2 已含安裝）；Firecrawl 可選。

## 鐵則（每個任務型態都適用）

1. **絕不憑記憶給引用。** 每一筆文獻都要當場經 PubMed／OpenAlex 實際解析後才能出現在交付物裡（實測：LLM 憑記憶生牙科文獻最高 99% 捏造；接網搜仍 3–13% 連結捏造）。
2. **每筆附 PMID＋DOI（有就附）＋查證日期。** 無 DOI 者標「（無 DOI，PubMed 驗證存在）」。
3. **Publication type 要看、要標。** Editorial／Letter／Comment／Case report 照實標（曾抓到候選清單把 2 頁 editorial 當研究引）。
4. **研究定位要核對。** implant-supported vs tooth-supported、上顎 vs 下顎、veneered vs monolithic——摘要層就要對齊題目，錯位引用比沒引用更傷。
5. **組 case-specific 論述前，先回頭核對整串累積的案件事實**（牙位、年齡、病人條件）——別從上一張單慣性推斷（實際錯過一次：把前牙案例慣性寫成後牙）。
6. **誠實聲明是交付物的一部分**：無 SR 就明講、外推（如上顎資料推下顎）要標、教科書出處不可代驗就註明由使用者自填、證據缺口主動寫出來。
7. 輸出預設繁體中文、術語保留英文原文；醫療專業判斷永遠是使用者的。

## 任務型態（先判斷是哪一種）

| 口令／情境 | 型態 | 規模 |
|:--|:--|:--|
| 「找◯◯的文獻」 | 快查 | Step 0–2 輕量，5–10 篇 |
| 「深挖◯◯」 | 完整五步 | 含滾雪球與飽和停止 |
| 「◯◯的出處是哪篇／可引出處」 | 出處查證 | 見§出處查證 |
| 「抓◯◯的全文／圖表／圖說」 | OA 抓取 | 見§OA 抓取 |
| 收到文獻查詢單（多題一批） | 批次快查 | 見§查詢單協定 |
| 領域級盤點（「把◯◯的證據全面盤一次」） | 升級 | 拆成多個「深挖」子題分次跑，並事先跟使用者對齊範圍 |

## 五步管線

### Step 0｜定形
- 白話問題→PICO（Population／Intervention／Comparison／Outcome）＋研究設計偏好＋年份範圍（預設近 10 年＋landmark 另標）＋篇數上限（預設 5–8）。
- 已驗證牙科濾器可直接套：牙周炎 RCT＝Lyrio 2021（PMC8019511，sens 93.2%）；牙科 SR＝Fontanive 2025（PMC12482493，高特異版 sens 96.7%/spec 99.1%）。
- 深評讀需求交棒姊妹 skill（入門包 3b 那 5 支）：檢索策略模板→`dental-evidence-retriever`；GRADE 逐 outcome→`clinical-evidence-reviewer`；單篇評讀→`research-critic`；數值審→`dental-statistical-forensics`；HTML 報告→`dental-evidence-report-artifact`。本 skill 是它們前端的**檢索與驗貨引擎**。

### Step 1｜雙軌搜（並行呼叫）
- **語意軌**（依 Firecrawl 帳號狀態擇一）：
  - 有 Firecrawl 帳號（API key／OAuth）→ `firecrawl_research_search_papers`（同題換 2–3 種問法撈得更全；引用圖 `related_papers` 覆蓋不全，滾雪球別靠它）。
  - 免註冊版沒有 research 系列工具 → 改用 `firecrawl_search` 加 `categories: ["research"]`，或 pubmed MCP 的 `pubmed_europepmc_search` 補一軌。
- **MeSH 軌**：pubmed MCP——`pubmed_lookup_mesh` 確認詞彙→`pubmed_search_articles`（MeSH＋tiab 同義詞 OR 塊、`systematic[sb]`／`randomized controlled trial[pt]` 過濾、`summaryCount` 拿簡表省 context）。
- 對方給候選 PMID 時：**先 `pubmed_fetch_articles` 逐筆覆核**（存在？type？定位？）再補搜——這一步曾抓到 editorial 誤植與 implant/tooth-supported 錯位。
- 兩軌合併去重，挑完再 `pubmed_fetch_articles` 抓完整摘要。

### Step 2｜三層驗貨
- **撤稿關**：OpenAlex 批次——`openalex_search_entities`，`entity_type: works`、`filters: {"doi": "A|B|C|…"}`（pipe 串接一發驗完）、`select: ["doi","is_retracted"]`。備援：Crossref REST `works/{DOI}` 看 `updated-by`（Retraction Watch 已併入）。
- **來源關**：DOAJ 收錄查 OpenAlex（`is_in_doaj`）；MEDLINE 收錄 OpenAlex **沒有**旗標——查 NLM Catalog／PubMed 的期刊資訊，查不到就標「收錄狀態未驗證」。predatory 疑慮用收錄狀態判斷，不靠黑名單。
- **定位關**：鐵則 3、4——type 與研究定位逐筆對齊題目。
- 立場抽驗（選配，深挖時做）：`pubmed_find_related` `cited_by` 抽讀引用文獻摘要，看後續文獻支持或反駁；寧可少報、報了要準。

### Step 3｜滾雪球（深挖才做）
- 後向：`pubmed_find_related` `references`（找 landmark）；前向：`cited_by`＋OpenAlex `openalex_get_citation_graph`（找最新發展與翻案）。
- 停止條件（事先宣告、回報哪條觸發）：已知關鍵論文全部找到＋連續一輪無新相關文獻＋估計剩餘 <5%（Undermind 飽和曲線 f=1−e^(−n/τ)，評估 150 篇≈85%）。

### Step 4｜收網
交付物格式（Markdown 表）：

```
| 文獻 | 設計/等級 | 跟問題相關的重點（含數字） | 連結 |
```
＋固定尾段：**「哪裡還在吵」**（爭議與證據缺口）＋驗貨聲明（N 篇解析、零撤稿、查證日期）＋誠實聲明（鐵則 6）。
- 供上台報告或答辯用時加 **defense 邏輯鏈**（把文獻串成可講的論證順序，並把「證據缺口主動講」寫進去）。
- 需要引用格式時用 `pubmed_format_citations`。

## 出處查證

某分類／概念「慣稱出處」的驗證流程：
1. PubMed 作者搜尋＋`firecrawl_search` 查原典完整書目（德文等非英語原典常不在 PubMed——用**英文文獻參考清單交叉驗證存在性**，兩個獨立來源以上）。
2. 找**原作者本人的英文可引版本**（期刊 OA 最佳）：`pubmed_fetch_fulltext` 開全文確認內容真的在裡面（含 ref 追蹤——作者自己引哪一版，那就是現行正式載體）。
3. 交付：原典書目＋英文可引出處＋教科書載體三層，加「簡報最站得住的寫法」＋易混淆系統的防雷（例：Terheyden quarter ≠ Cologne CCARD）。
4. 頁碼在引用者間不一致時照實說，建議抄英文論文參考清單寫法。

## OA 全文／圖表抓取

- 全文：`pubmed_fetch_fulltext`（PMC→Europe PMC→Unpaywall 鏈）。先 `overflowMode: "outline"` 掃結構，再 `sections` 指定段落抓全文；`includeReferences: true` 追參考文獻。
- **圖說**：JATS 全文不含 figure captions——用 `firecrawl_scrape` 抓 PMC 頁面、`includeTags: ["figcaption","figure"]` 只抽圖說與圖檔連結（含 CDN 原圖 URL）。
- 交付時附授權判斷（MDPI/BMC 等 CC BY 可改作、註明出處；非 OA 只能連結不能重製）。
- 誠實聲明：使用者想要的內容若不在該文（如逐級術式表在教科書不在論文），明講並警告別掛錯出處。

## 查詢單協定（多題一批）

- 收單格式（提供方應給）：臨床問題／用途（簡報頁）／證據等級需求／年份範圍／篇數上限／deadline。整批傳優於逐單傳（可平行搜、跨題去重）。
- 回件＝上述 Step 4 Markdown 表；超上限要說明理由並指名可砍哪篇。
- 交付對象要上台報告或答辯時，把「考官／聽眾會怎麼問」的 defense 視角寫進交付物。

## 環境備忘

- 需要 pubmed／openalex MCP（入門包 SETUP.md 階段 2 已裝）；工具未載入時先用 ToolSearch 找。
- Firecrawl 免註冊版只有 search／scrape／parse；`firecrawl_research_*` 系列要有帳號（API key）才會出現。scrape 額度用罄會回 402，圖說改走內建 WebFetch 備援。
- OpenAlex 免費申請 key 可 10× 日額度（不設也能跑）；NCBI API key 未設＝3 req/s（免費申請可到 10）。
