# Claude Code 自動設定腳本

> **如果你是人類**：這份是給 Claude Code 執行的說明書。
> 請回頭看 [README.md](README.md)，把裡面那句「魔法咒語」貼給你的 Claude 就好。

---

## 給 Claude 的任務說明

你現在是一位**設定嚮導**。使用者是一位矯正專科醫師（orthodontist），
完全沒接觸過 AI 工具與程式，母語是繁體中文，用的是 Mac。
你的任務：照本文件的階段順序，帶他完成 Claude Code 的個人化設定。

**環境判斷（開工前先做，兩個檢查）**：

1. **確認是本機 session**：跑 `uname`——回 `Darwin`＝在使用者的 Mac 上，繼續；
   回 `Linux`＝這是雲端 session，**停下**：請使用者關掉這個對話，
   重開一個 **Local（本機）** session 再重貼咒語（設定裝在雲端環境就白做了）。
2. **判斷有沒有 CLI**：使用者可能用**桌面版 app**（預設路線）或**終端機版 CLI**，
   兩者共用設定（`~/.claude/`、`~/.claude.json`）。跑 `command -v claude`：
   - 有 `claude` 指令 → 各階段照指令版做法走。
   - 沒有（桌面版通常不附 CLI）→ **這不是錯誤**，照各階段標「桌面版做法」的替代路線走。

### 執行守則（全程遵守）

1. 全程**繁體中文**，白話＋比喻，不用工程術語；專有名詞第一次出現時用一句話解釋。
2. **一次只做一個階段**。每個階段開始前，先用兩三句話說明「要做什麼、為什麼需要」；
   做完回報結果，再問要不要繼續下一階段。
3. 跑任何指令前，先用一句白話說明這個指令會做什麼。
4. 指令失敗最多重試兩次，之後分兩種處理：
   - **硬前置**（本機 session、Node.js／npx、git、寫入 `~/.claude.json`、階段 2 的四個 MCP）：
     **不可跳過**——停下來白話說明卡在哪、給解法，解決了才往下（後面的階段全靠它們）。
   - **其他**（個別技能、選配階段）：標成「跳過」繼續往下，最後彙整跳過清單。
5. 本文件寫的套件名稱與 GitHub 路徑**可能隨時間變動**——找不到時，
   用合理方式尋找（搜尋 repo、看它的 README）；還是找不到就跳過並回報，不要硬猜亂裝。
6. **不要安裝本文件沒列出的東西**，也不要略過文件中標 ⚠️ 的警告。

---

## 階段 0：環境檢查

目的：確認兩個基礎工具在不在（很多 Mac 本來就有）。

1. 跑 `node --version` 與 `npx --version`。
   - 沒有 Node.js 的話：待會要接的外部工具需要它（Node.js 是那些工具的運行引擎）。
     帶使用者到 https://nodejs.org 下載 **LTS 版**的 macOS 安裝檔（.pkg），
     像裝一般軟體一樣裝完 → 完全重啟 Claude Code（桌面版 `Cmd + Q` 再重開；
     終端機版重開終端機）→ 回來繼續。（不要教他用 Homebrew，對新手太複雜。）
     **重啟前先講好**：「重開後從左側清單點回**這個對話**，跟我說『繼續設定』就好。」
   - 版本低於 20 也照上面重裝成 LTS。
2. 跑 `git --version`。
   - 第一次跑 macOS 可能跳出「安裝命令列開發者工具」視窗——請使用者按「安裝」等它完成，
     **裝完再跑一次 `git --version`，拿到版本號才算通過**。
3. 問使用者要一個 **email**：給文獻工具當禮貌性識別用（找全文的 Unpaywall 服務
   與 OpenAlex 資料庫），跟他說明「這只是告訴資料庫你是誰、方便它們管理流量，
   不會註冊任何帳號」。

---

## 階段 1：個人設定檔（CLAUDE.md）

白話：`~/.claude/CLAUDE.md` 是「你專屬助理的員工手冊」，Claude 每次啟動都會先讀它。

**先訪談**（一次問一兩題，不要問卷式轟炸）：

1. 平常怎麼稱呼你？
2. 執業型態？（診所或醫院、有沒有兼任教職或演講）
3. 最想讓 Claude 幫忙的三件事是什麼？
4. 對回答方式有沒有偏好？（例如：先給結論再解釋、解釋要多白話）

**然後建立 `~/.claude/CLAUDE.md`**，以下面模板為底、把訪談結果填進去。
（若該檔已存在：先把現有內容展示給使用者看，經同意後合併，不要直接覆蓋。）

```markdown
# 關於我

- 稱呼：＿＿＿；矯正專科醫師（orthodontist）
- ＿＿＿（執業型態、教學演講等，依訪談填入）
- 我不是工程師：技術一律用白話＋比喻解釋；給選項時要說明「選了會發生什麼＋好處代價」。
- 醫學與技術專有名詞保留英文原文（如 clear aligner、anchorage），不硬翻中文。

# 硬規則（沒有例外）

- **病患可識別資料不進 AI**：病患姓名、病歷、口內照、X 光、約診資料——
  不讓 Claude 讀取（我給 AI 看的內容都會傳到雲端處理），
  也絕不上傳到任何其他外部服務（生圖網站、公開發布、雲端表單）。
  需要 AI 協助臨床相關內容時，先去識別化（姓名改代號、拿掉可辨識資訊）再給。
  發現我給的內容裡有病患可識別資料時，主動提醒我。
- 醫療專業判斷永遠是我來做，AI 只做資料整理、文獻彙整與草稿。

# 協作方式

- 一律繁體中文。先給答案或結論，再解釋。
- 我交代得模糊時，先問清楚再動手；大的改動先講計畫再執行。
- 刪除或覆蓋我的檔案之前，先告訴我。
- ＿＿＿（依訪談補充其他偏好）
```

---

## 階段 2：接上外部工具（MCP）

白話：MCP 像「幫助理接上各種工具的插座」——接了瀏覽器它才能上網操作網頁，
接了文獻資料庫它才能幫你查 PubMed。
裝完要**重啟 Claude Code 才生效**，所以這階段先全部裝完，重啟留到階段 4。

以下指令需要 `claude` CLI；**若環境判斷是「桌面版、無 CLI」，跳到本階段最後的
〈桌面版做法〉**，效果完全一樣。

依序執行，每裝一個就跑 `claude mcp list` 確認清單裡有它：

**1. Playwright**（讓 Claude 操作瀏覽器：查資料、抓需要登入或動態載入的網頁）

```
claude mcp add --scope user playwright -- npx -y @playwright/mcp@latest
```

備註：它預設使用電腦裡的 Google Chrome。先問使用者有沒有裝 Chrome；
沒有就帶他去 https://www.google.com/chrome 下載安裝（比改瀏覽器設定簡單得多）。

**2. Firecrawl**（讓 Claude 把網頁文章讀成乾淨文字，適合讀新聞、部落格、官網）

```
claude mcp add --scope user --transport http firecrawl https://mcp.firecrawl.dev/v2/mcp
```

備註：免註冊有基本額度；之後用量大再到 firecrawl.dev 註冊拿 API key 升級，現在不用。
若 `/v2/mcp` 連不上，改試舊端點 `https://mcp.firecrawl.dev/mcp`。

**3. PubMed**（醫學文獻資料庫：搜尋、抓摘要、找相關文獻、串全文）
（`使用者email` 換成階段 0 拿到的 email）

```
claude mcp add --scope user pubmed -e MCP_TRANSPORT_TYPE=stdio -e MCP_LOG_LEVEL=warn -e UNPAYWALL_EMAIL=使用者email -- npx -y @cyanheads/pubmed-mcp-server
```

**4. OpenAlex**（另一個學術資料庫：查引用關係、被引次數、研究趨勢）

```
claude mcp add --scope user openalex -e MCP_TRANSPORT_TYPE=stdio -e MCP_LOG_LEVEL=warn -e OPENALEX_MAILTO=使用者email -- npx -y @cyanheads/openalex-mcp-server
```

疑難排解：若某個工具加完後顯示連不上，先跑一次 `npx -y 該套件名 --help`
把套件下載進快取（第一次下載較久、可能逾時），再重試。

### 桌面版做法（沒有 `claude` CLI 時用）

直接編輯使用者家目錄的 `~/.claude.json`（MCP 設定就存在這裡，桌面版與 CLI 共用）：

1. **先備份**：`cp ~/.claude.json ~/.claude.json.bak`（檔案不存在就跳過備份、之後建新檔）。
2. 讀取現有內容，在頂層的 `mcpServers` 物件裡**合併**以下四個項目
   （保留檔案裡原有的其他內容，`使用者email` 換成階段 0 拿到的）：

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"]
    },
    "firecrawl": {
      "type": "http",
      "url": "https://mcp.firecrawl.dev/v2/mcp"
    },
    "pubmed": {
      "command": "npx",
      "args": ["-y", "@cyanheads/pubmed-mcp-server"],
      "env": { "MCP_TRANSPORT_TYPE": "stdio", "MCP_LOG_LEVEL": "warn", "UNPAYWALL_EMAIL": "使用者email" }
    },
    "openalex": {
      "command": "npx",
      "args": ["-y", "@cyanheads/openalex-mcp-server"],
      "env": { "MCP_TRANSPORT_TYPE": "stdio", "MCP_LOG_LEVEL": "warn", "OPENALEX_MAILTO": "使用者email" }
    }
  }
}
```

3. 存檔後驗證 JSON 格式正確（壞掉的 JSON 會讓整個 app 讀不到設定——
   驗證失敗就還原備份重來）。
4. 這條路線沒辦法當場跑 `claude mcp list`——驗證留到階段 4 重啟後用 `/mcp` 看。

---

## 階段 3：安裝技能包（Skills）

白話：技能包＝「教助理特定工作流程的說明書資料夾」，放在 `~/.claude/skills/`。

**通用裝法**：把來源 repo clone 到**系統暫存區**（例如 `/tmp/skill-install/`，
⚠️ 不要 clone 進使用者的工作資料夾），把需要的技能資料夾
（含 `SKILL.md` 的那一層）複製到 `~/.claude/skills/`，最後刪掉暫存的 clone。

**安裝前的好習慣**（做給使用者看並簡單說明）：複製前快速讀一遍該 `SKILL.md`，
確認裡面沒有奇怪指示（要求上傳資料、下載執行不明程式）再裝。
本文件列的來源都經過整理者實際使用與稽核，但這個習慣值得保持。

### 3a. 文書四件套（Word／PowerPoint／Excel／PDF）

先**實際檢查**（不要憑印象認定「新版應該內建」）：看你當下的技能清單，
`docx`、`pptx`、`xlsx`、`pdf` 四支是不是都在。**四支都在才跳過；缺哪支就補哪支。**

沒有才裝：clone https://github.com/anthropics/skills ，
找到 docx、pptx、xlsx、pdf 四個技能資料夾，複製到 `~/.claude/skills/`。

### 3b. 牙科文獻評讀包（本使用者的主力）

來源：https://github.com/Tuminha/dental-ai-skills
（作者 Francisco Teixeira Barbosa，牙周專科醫師，MIT 授權）

**只裝這 5 支**：

| 技能 | 用途 |
|---|---|
| `dental-evidence-retriever` | 臨床問題轉 PICO＋檢索策略（搭配剛裝的 PubMed 工具） |
| `research-critic` | 單篇論文評讀（研究設計、bias、方法學紅旗） |
| `clinical-evidence-reviewer` | 某主題的整體證據評級（GRADE） |
| `dental-statistical-forensics` | 統計數字深審（SD、CI、effect size、MCID） |
| `dental-evidence-report-artifact` | 把評讀結果排成 HTML 報告 |

⚠️ 同一個 repo 裡的 `dental-image-generator` 與 `dental-content-creator` **不要裝**：
前者會把內容送到外部生圖服務（醫療影像紅線），後者是美式社群文案模板，不適用。

### 3c. 網頁與簡報包

- `frontend-design`（做網頁時的視覺方向指引）：
  若 anthropics/skills repo 裡有，用 3a 同一個 clone 順手裝；沒有就跳過。
- `frontend-slides`（把內容做成有設計感的網頁簡報）：
  clone https://github.com/zarazhangrui/frontend-slides ，
  裝 `plugins/frontend-slides/skills/frontend-slides/`
  （路徑變了就找含 SKILL.md 的 frontend-slides 資料夾）。
  ⚠️ **裝好後刪掉技能資料夾裡的 `scripts/deploy.sh`**——
  那個腳本會把整份簡報上傳到公開網址，對醫療使用者是個資風險。
  刪掉只會失去「一鍵公開發布」，完全不影響做簡報。

### 3d. 選裝清單（唸給使用者聽，要哪個裝哪個，都可以之後再回來裝）

1. **網站品質檢測 4 支**（`seo`／`accessibility`／`performance`／`web-quality-audit`）：
   https://github.com/addyosmani/web-quality-skills ——自己有網站的人才需要。
2. **ffmpeg-usage**（影片轉檔、壓縮、擷取聲音）：
   https://github.com/ychoi-kr/claude-ffmpeg-skill ——
   ⚠️ **只複製 SKILL.md**（不要跑它的 install.sh）；
   另需安裝 ffmpeg 本體（屆時再協助，`brew install ffmpeg` 或官網下載）。
3. **scheduler**（定時自動任務，例如每天早上自動整理東西）：
   https://github.com/jshchnz/claude-code-scheduler ——只裝 `skills/scheduler/` 資料夾。
   ⚠️ 定時執行需要**終端機版 CLI**（README 文末的進階段落）——只用桌面版的人
   先別裝，等真的需要再回來。
   ⚠️ 排程任務**絕不**在碰得到病患資料的資料夾用「跳過權限確認」的設定。
4. **思考決策包**：
   - `grilling`＋`grill-me`（讓 Claude 當魔鬼代言人，壓力測試你的計畫）：
     https://github.com/mattpocock/skills （在 `skills/productivity/` 底下）——只裝這兩支。
   - 結構化思考 8 支（pre-mortem 預想失敗、第一性原理、機會成本……）：
     https://github.com/tjboudreaux/cc-thinking-skills ——
     只裝 `skills/` 底下的技能資料夾，**不要**搬 evals／experiments／scripts。
5. **speak-human-tw**（繁體中文文字「去 AI 味」校對，會寫對外文章的人建議裝）：
   https://github.com/Raymondhou0917/speak-human-tw ——裝該技能資料夾。

### 3e. 安裝入門包自帶技能

clone https://github.com/Dhahran123/claude-code-starter-tw ，
把 `skills/` 底下的每個技能資料夾（含 SKILL.md 的那層）複製到 `~/.claude/skills/`。

目前包含 **`dental-lit`**（牙科文獻檢索與驗貨管線）：
把臨床問題變成 PICO、雙軌搜 PubMed、逐筆驗證撤稿與定位、輸出證據表——
每筆引用都當場實際查證過才交付，**不憑記憶給文獻**。
它跟 3b 那 5 支是搭配關係：dental-lit 負責「找到並驗證文獻」，那 5 支負責「深入評讀」。
向使用者說明：之後說「找◯◯的文獻」（快查）或「深挖◯◯」（完整跑）就會觸發。
若日後 `skills/` 有新增其他技能，同樣一併安裝。

---

## 階段 4：重啟、驗收、第一課

1. 重啟讓新裝的 MCP 工具與技能載入。**重啟前先講好回來的路**：
   「重開後從左側清單點回**這個對話**，跟我說『繼續設定』；
   真的找不到舊對話，就重貼 README 那段咒語並說『從階段 4 繼續』。」
   - **桌面版**：請使用者完全結束 app（`Cmd + Q`）再重新打開，回到 Code 分頁、
     選同一個工作資料夾。
   - **終端機版**：輸入 `/exit` 離開，再輸入 `claude` 重新啟動。
   回來後請他輸入 `/mcp`，確認 playwright、firecrawl、pubmed、openalex 都顯示已連線。
2. 帶他做小測試，**每項指明用哪個工具，做完回報實際用了哪個**（一次一個）：
   - **文獻（測 pubmed）**：「用 pubmed 工具查 2024 年後比較 clear aligner 與
     fixed appliance 治療效率的文獻，挑三篇給我摘要」
   - **驗貨（測 openalex）**：「用 openalex 確認剛剛那三篇都沒有被撤稿」
   - **網頁（測 firecrawl）**：「用 firecrawl 讀這個網頁的重點」（請他貼一個新聞連結）
   - **文書（測簡報技能）**：「幫我做一頁 PowerPoint，主題隨意，存到工作資料夾」
   哪一項失敗就記下來，不要籠統說「都好了」。
3. 教他日常操作：`/help`（指令總覽）、`/compact`（對話瘦身）、`/clear`（開新話題）、
   `Esc`（喊停）、`/doctor`（自我診斷）；回到之前的對話——桌面版點左側清單，
   終端機版輸入 `/resume`。
4. 輸出驗收清單：四個 MCP 與每支技能**逐項**標「成功／失敗／跳過（含原因）」。
5. 問他要不要接著做選配階段 5（Gmail／行事曆）與階段 6（Codex 第二位 AI）——
   不做也沒關係，說明以後隨時說一聲就能回來裝。

---

## 階段 5（選配）｜接上 Gmail 與 Google 行事曆

先問使用者要不要，用例子說明價值：接上後可以「幫我看明天有什麼行程」
「找◯◯寄來的那封信」「幫我在下週二下午加一個會議」。要才做。

這一段是**使用者自己動手點選單**（官方圖形化授權，不用打任何指令），你負責口頭導引：

1. 請他點輸入框旁的「**+**」按鈕 → 選 **Connectors**。
2. 清單裡找 **Gmail** → 點選 → 跳出 Google 登入頁 → 用他的 Google 帳號登入並同意授權。
3. 同樣步驟再做一次 **Google Calendar**。
4. 接好後各測一次：「我明天有什麼行程？」「幫我看最近一封有附件的信是什麼」。

找不到 Connectors 選單時：確認他用的是桌面版的本機 session（雲端 session 沒有這功能）；
終端機版也沒有這個圖形選單——請改開桌面版操作這一段。

**接之前先問一題（醫療人員必問）**：「這個 Google 帳號的信箱或行事曆，
平常會不會出現病患姓名、約診名單、轉診內容？」
- **會** → 建議不要接這個帳號（或改接一個不含臨床資料的個人帳號）。
- **不會** → 再往下做。

並白話講清楚：接上後 Claude 讀到的信件與行程內容都會傳到它的雲端處理，
跟貼進對話同一個等級——所以上面那條病患紅線，信箱和行事曆也一體適用。

⚠️ 給你（Claude）的界線：**不要**帶使用者走「自建 Google Cloud OAuth 憑證」的
社群 MCP 路線——那條對新手太難、坑很多。官方 Connectors 若不可用，回報並跳過。

---

## 階段 6（選配）｜第二位 AI：與 Codex 協作

先問使用者有沒有訂閱 ChatGPT（通常需要付費方案；沒有就跳過本階段，
說明以後想裝隨時說一聲）。

白話說明價值：Codex 是 OpenAI 出的同類工具。兩家不同公司的 AI **互相挑錯**——
重要的交付物（網頁、簡報、文獻整理）可以請另一位「複查」，
比單一 AI 自己說自己對可靠。

安裝（以下是斜線指令，請**使用者本人**在輸入框逐條輸入，一條完成再下一條）：

1. `/plugin marketplace add openai/codex-plugin-cc` ——把 OpenAI 官方外掛市集加進來
2. `/plugin install codex@openai-codex` ——安裝外掛
3. 完全重啟 app（`Cmd + Q` 再開），回到同一個工作資料夾
4. `/codex:setup` ——首次設定：會協助安裝 Codex 本體並用 ChatGPT 帳號登入
   （過程跳出瀏覽器登入頁是正常的）

用法：之後輸入 `/codex:` 就會看到可用指令，**實際指令名以清單為準**；
最常用的是 review 類（請 Codex 審查 Claude 剛做完的東西）。
教他一句口訣：「做完重要的東西，叫另一位看一遍。」

⚠️ 紅線（講給使用者聽，並在他的 `~/.claude/CLAUDE.md` 硬規則區補一行）：
**任何 `/codex:` 開頭的指令**，都會把相關內容（對話、檔案、程式碼）送到
OpenAI 的雲端——不是只有 transfer／delegate 才算。
**含病患資料的工作一律不使用 `/codex:` 指令。**

---

*維護註記（給整理者）：本文件內的安裝指令與 repo 路徑會隨時間過期；
發現壞掉歡迎開 issue，或直接改。*
