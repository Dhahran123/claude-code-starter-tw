# Claude Code 自動設定腳本

> **如果你是人類**：這份是給 Claude Code 執行的說明書。
> 請回頭看 [README.md](README.md)，把裡面那句「魔法咒語」貼給你的 Claude 就好。

---

## 給 Claude 的任務說明

你現在是一位**設定嚮導**。使用者是完全沒接觸過 AI 工具與程式的新手，
母語是繁體中文，用 Mac 或 Windows。
你的任務：照本文件的階段順序，帶他完成 Claude Code 的個人化設定——
**先透過訪談認識他，再依他的工作裝對的模組**，不是把所有東西全裝。

**環境判斷（開工前先做，兩個檢查）**：

1. **確認是本機 session**：跑 `uname`——
   - 回 `Darwin`＝使用者的 Mac，繼續。
   - 回 `MINGW`／`MSYS` 開頭（或指令不存在但 PowerShell 可用）＝Windows 本機，繼續。
   - 回 `Linux`＝這是雲端 session，**停下**：請使用者關掉這個對話，
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
   - **硬前置**（本機 session、Node.js／npx、git、寫入 `~/.claude.json`、階段 2 的核心 MCP）：
     **不可跳過**——停下來白話說明卡在哪、給解法，解決了才往下（後面的階段全靠它們）。
   - **其他**（個別技能、選配模組與階段）：標成「跳過」繼續往下，最後彙整跳過清單。
5. 本文件寫的套件名稱與 GitHub 路徑**可能隨時間變動**——找不到時，
   用合理方式尋找（搜尋 repo、看它的 README），但**只接受本文件列出的原作者／組織**
   名下的來源——同名的別人 fork 或仿冒套件一律不裝；找不到原作者的就跳過並回報。
6. **不要安裝本文件沒列出的東西**，也不要略過文件中標 ⚠️ 的警告。

---

## 階段 0：環境檢查

目的：確認兩個基礎工具在不在。

1. 跑 `node --version` 與 `npx --version`。
   - 沒有 Node.js 的話：待會要接的外部工具需要它（Node.js 是那些工具的運行引擎）。
     帶使用者到 https://nodejs.org 下載 **LTS 版**安裝檔（Mac＝.pkg、Windows＝.msi），
     像裝一般軟體一樣裝完 → 完全重啟 Claude Code（桌面版完全結束再重開；
     終端機版重開終端機）→ 回來繼續。（不要教新手用 Homebrew 或 winget。）
     **重啟前先講好**：「重開後從左側清單點回**這個對話**，跟我說『繼續設定』就好。」
   - 版本低於 **24** 也照上面重裝成 LTS（部分外部工具要求 Node 24 以上，一次裝到位）。
2. 跑 `git --version`。
   - **Mac**：第一次跑可能跳出「安裝命令列開發者工具」視窗——請使用者按「安裝」等完成，
     **裝完再跑一次 `git --version`，拿到版本號才算通過**。
   - **Windows**：沒有的話帶他到 https://git-scm.com 下載 Git for Windows，
     安裝一路用預設值，裝完重啟 Claude Code 回來再驗一次。

---

## 階段 1：認識使用者（CLAUDE.md）

白話：`~/.claude/CLAUDE.md` 是「你專屬助理的員工手冊」，Claude 每次啟動都會先讀它。

**先訪談**（一次問一兩題，不要問卷式轟炸）：

1. 平常怎麼稱呼你？
2. 你的工作是什麼？（職業、產業；醫療人員的話問一下科別）
3. 最想讓 Claude 幫忙的三件事是什麼？
4. 對回答方式有沒有偏好？（例如：先給結論再解釋、解釋要多白話）

**然後建立 `~/.claude/CLAUDE.md`**，以下面模板為底、把訪談結果填進去。
（若該檔已存在：先把現有內容展示給使用者看，經同意後合併，不要直接覆蓋。）

```markdown
# 關於我

- 稱呼：＿＿＿；職業：＿＿＿
- 我不是工程師：技術一律用白話＋比喻解釋；給選項時要說明「選了會發生什麼＋好處代價」。
- 專業術語保留英文原文，不硬翻中文。

# 硬規則（沒有例外）

（依職業擇一放入，醫療人員用第一款：）
- **病患可識別資料不進 AI**：病患姓名、病歷、口內照、X 光、約診資料——
  不讓 Claude 讀取（我給 AI 看的內容都會傳到雲端處理），
  也絕不上傳到任何其他外部服務。需要 AI 協助臨床相關內容時，
  先去識別化（姓名改代號、拿掉可辨識資訊）再給。
  發現我給的內容裡有病患可識別資料時，主動提醒我。
（非醫療用這款：）
- **能識別出特定他人的敏感資料不進 AI**：客戶個資、別人的財務與身分資料——
  不讓 Claude 讀取（我給 AI 看的內容都會傳到雲端處理）。
  需要處理時先把可辨識資訊拿掉再給；發現內容含這類資料時，主動提醒我。

# 協作方式

- 一律繁體中文。先給答案或結論，再解釋。
- 我交代得模糊時，先問清楚再動手；大的改動先講計畫再執行。
- 刪除或覆蓋我的檔案之前，先告訴我。
- ＿＿＿（依訪談補充其他偏好）
```

若使用者是醫療人員，額外用白話講一次紅線的道理（README 最末段那套）。

---

## 階段 1b：安全防護（防呆鎖＋檔案存檔點）

白話：幫助理裝兩道保險——**危險指令黑名單**（最具破壞性的指令直接擋下、連問都不問）
與**檔案存檔點**（Claude 改壞檔案時有版本可以救回來）。

做法：編輯使用者家目錄的 `~/.claude/settings.json`
（桌面版與終端機版共用；⚠️ 這與階段 2 的 `~/.claude.json` 是**兩個不同檔案**）：

1. **先備份**：檔案已存在就先複製一份成 `settings.json.bak`；不存在就直接建新檔。
2. 讀取現有內容，**合併**以下設定（保留原有其他內容；`deny` 清單已有的項目不重複加。
   兩套寫法 Mac／Windows 都放沒關係——用不到的那套永遠不會被觸發）：

```json
{
  "permissions": {
    "deny": [
      "Bash(rm -rf *)",
      "Bash(rm -fr *)",
      "Bash(rm -r *)",
      "Bash(rm -R *)",
      "Bash(sudo *)",
      "Bash(dd *)",
      "Bash(mkfs*)",
      "Bash(git reset --hard*)",
      "Bash(git push --force*)",
      "Bash(git push -f *)",
      "Bash(git clean -f*)",
      "Bash(git branch -D*)",
      "Bash(shutdown*)",
      "Bash(reboot*)",
      "PowerShell(Remove-Item -Recurse -Force*)",
      "PowerShell(Remove-Item -Force -Recurse*)",
      "PowerShell(rm -rf*)",
      "PowerShell(rm -fr*)",
      "PowerShell(rm -r -Force*)",
      "PowerShell(Format-Volume*)",
      "PowerShell(Clear-Disk*)",
      "PowerShell(Remove-Partition*)",
      "PowerShell(diskpart*)",
      "PowerShell(Stop-Computer*)",
      "PowerShell(Restart-Computer*)",
      "PowerShell(Set-ExecutionPolicy*)",
      "PowerShell(git reset --hard*)",
      "PowerShell(git push --force*)",
      "PowerShell(git push -f*)",
      "PowerShell(git clean -f*)",
      "PowerShell(git branch -D*)"
    ]
  },
  "fileCheckpointingEnabled": true
}
```

3. 存檔後驗證 JSON 格式正確（壞掉的 JSON 會讓整個 app 讀不到設定——
   驗證失敗就還原備份重來）。
4. 用白話跟使用者講清楚兩件事：
   - 黑名單擋的都是「幾乎不可能是你想要的」動作：整批強制刪除、格式化硬碟、
     強制改寫版本紀錄、關機重開機。Claude 想跑這些會直接被拒絕。
   - 這**不是**萬能保險——日常的權限視窗還是要看；刪除、覆蓋類的動作多看一眼再允許。

⚠️ 給你（Claude）的界線：只加上面這兩項設定，**不要**動 `permissions` 的其他欄位，
尤其**絕不**設定任何「跳過權限確認」的模式——新手需要那道權限視窗。

---

## 階段 2：接上核心外部工具（MCP）

白話：MCP 像「幫助理接上各種工具的插座」。核心兩個人人都裝；
其他的（例如醫學文獻資料庫）放在階段 3 的模組裡，選了才裝。
裝完要**重啟 Claude Code 才生效**，重啟統一留到階段 4。

以下指令需要 `claude` CLI；**若環境判斷是「桌面版、無 CLI」，跳到本階段最後的
〈桌面版做法〉**，效果完全一樣。

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

疑難排解：若加完後顯示連不上——先跑一次 `npx -y 該套件名 --help` 把套件下載進快取
（第一次下載較久、可能逾時）再重試；**Windows** 上 npx 型工具若一直連不上，
把設定裡的 command 從 `npx` 改成 `cmd /c npx`（舊版本的已知坑）。

### 桌面版做法（沒有 `claude` CLI 時用）

直接編輯使用者家目錄的 `~/.claude.json`（MCP 設定存這裡，桌面版與 CLI 共用）：

1. **先備份**：複製一份成 `~/.claude.json.bak`（檔案不存在就跳過備份、之後建新檔）。
2. 讀取現有內容，在頂層的 `mcpServers` 物件裡**合併**以下項目（保留原有內容）：

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
    }
  }
}
```

3. 存檔後驗證 JSON 格式正確（壞掉的 JSON 會讓整個 app 讀不到設定——
   驗證失敗就還原備份重來）。
4. 這條路線沒辦法當場跑 `claude mcp list`——驗證留到階段 4 重啟後用 `/mcp` 看。

---

## 階段 3：技能模組（依訪談結果挑）

白話：技能包＝「教助理特定工作流程的說明書資料夾」，放在 `~/.claude/skills/`。

**通用裝法**：把來源 repo clone 到**系統暫存區**（Mac 例 `/tmp/skill-install/`、
Windows 例 `%TEMP%\skill-install\`；⚠️ 不要 clone 進使用者的工作資料夾），
把需要的技能資料夾（含 `SKILL.md` 的那一層）複製到 `~/.claude/skills/`，
最後刪掉暫存的 clone。

**安裝前的好習慣**（做給使用者看並簡單說明）：複製前快速讀一遍該 `SKILL.md`，
確認裡面沒有奇怪指示（要求上傳資料、下載執行不明程式）再裝。
本文件列的來源都經過整理者實際使用與稽核，但這個習慣值得保持。

### 3a. 基本技能（人人都裝）

**文書四件套**——先**實際檢查**（不要憑印象認定「新版應該內建」）：看你當下的技能清單，
`docx`、`pptx`、`xlsx`、`pdf` 四支是不是都在。**四支都在才跳過；缺哪支就補哪支。**
沒有才裝：clone https://github.com/anthropics/skills ，
找到 docx、pptx、xlsx、pdf 四個技能資料夾，複製到 `~/.claude/skills/`。

**find-skills**（技能搜尋器——以後使用者說「我想要能◯◯」，可以用它找社群現成的技能）：
執行 `npx skills add find-skills`（skills.sh 的技能安裝器，裝到 user／global 範圍）。
失敗就在 GitHub 搜 `find-skills` 技能照通用裝法裝；再不行標跳過（非硬前置）。
裝完順便講一次習慣：**用它裝任何陌生技能前，先讀該技能的 SKILL.md 確認沒有奇怪指示**。

### 3b. 模組選單（照訪談結果推薦，使用者點頭才裝）

把符合他工作的模組用一句白話介紹、問要不要裝：

**模組一｜醫療／牙科文獻**（醫療人員，尤其牙科）
1. 先跟使用者要一個 **email**（給找全文的 Unpaywall 服務與 OpenAlex 資料庫當
   禮貌性識別，不會註冊任何帳號），然後加兩個 MCP（桌面版一樣走 `~/.claude.json` 合併）：

```
claude mcp add --scope user pubmed -e MCP_TRANSPORT_TYPE=stdio -e MCP_LOG_LEVEL=warn -e UNPAYWALL_EMAIL=使用者email -- npx -y @cyanheads/pubmed-mcp-server
claude mcp add --scope user openalex -e MCP_TRANSPORT_TYPE=stdio -e MCP_LOG_LEVEL=warn -e OPENALEX_MAILTO=使用者email -- npx -y @cyanheads/openalex-mcp-server
```

```json
{
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
```

2. 裝牙科文獻評讀 5 支：https://github.com/Tuminha/dental-ai-skills
   （作者為牙周專科醫師，MIT 授權）——只裝 `dental-evidence-retriever`（PICO＋檢索策略）、
   `research-critic`（單篇評讀）、`clinical-evidence-reviewer`（GRADE 證據評級）、
   `dental-statistical-forensics`（統計深審）、`dental-evidence-report-artifact`（HTML 報告）。
   ⚠️ 同 repo 的 `dental-image-generator` 與 `dental-content-creator` **不要裝**
   （前者把內容送外部生圖服務＝醫療影像紅線，後者是美式社群文案模板）。
3. 告知使用者：整理者另有**私人牙科技能包**（含自製的文獻檢索驗貨管線等進階工具，
   與上面 5 支成套）——跟把這份入門包傳給他的人索取邀請即可，這裡不用做任何事。

**模組二｜網頁與簡報**（要做個人網站、教學簡報的人）
- `frontend-design`（網站視覺方向）：anthropics/skills 裡有就用 3a 同一個 clone 順手裝。
- `frontend-slides`（有設計感的網頁簡報）：clone https://github.com/zarazhangrui/frontend-slides ，
  裝 `plugins/frontend-slides/skills/frontend-slides/`（路徑變了就找含 SKILL.md 的同名資料夾）。
  ⚠️ **裝好後刪掉技能資料夾裡的 `scripts/deploy.sh`**——那個腳本會把整份簡報
  上傳到公開網址，對處理敏感內容的使用者是風險；刪掉不影響做簡報。

**模組三｜社群文案**（有在經營 IG／FB／部落格的人）
- 內容策略 4 支：https://github.com/blacktwist/social-media-skills ——只裝
  `content-strategy-sms`（不知道要發什麼）、`content-calendar-sms`（內容日曆）、
  `hook-writer-sms`（開頭鉤子）、`content-repurposer-sms`（一稿多發）。
  ⚠️ **不要裝 BlackTwist 的 MCP**（會自動排程發文，有未經確認就對外發布的風險）；
  hook 產出是美式語氣，當發想器用、發文前自己潤。
- `speak-human-tw`（繁體中文「去 AI 味」校對）：
  https://github.com/Raymondhou0917/speak-human-tw ——裝該技能資料夾。

### 3c. 選裝清單（唸給使用者聽，要哪個裝哪個，都可以之後再回來裝）

1. **網站品質檢測 4 支**（`seo`／`accessibility`／`performance`／`web-quality-audit`）：
   https://github.com/addyosmani/web-quality-skills ——自己有網站的人才需要。
2. **ffmpeg-usage**（影片轉檔、壓縮、擷取聲音）：
   https://github.com/ychoi-kr/claude-ffmpeg-skill ——⚠️ **只複製 SKILL.md**
   （不要跑它的 install.sh）；另需安裝 ffmpeg 本體（屆時再協助）。
3. **scheduler**（定時自動任務）：https://github.com/jshchnz/claude-code-scheduler ——
   只裝 `skills/scheduler/` 資料夾。⚠️ 定時執行需要**終端機版 CLI**——只用桌面版的人
   先別裝。⚠️ 排程任務**絕不**在碰得到敏感資料的資料夾用「跳過權限確認」的設定。
4. **思考決策包**：`grilling`＋`grill-me`（壓力測試你的計畫）：
   https://github.com/mattpocock/skills （在 `skills/productivity/` 底下）——只裝這兩支；
   結構化思考 8 支：https://github.com/tjboudreaux/cc-thinking-skills ——
   只裝 `skills/` 底下的技能資料夾，**不要**搬 evals／experiments／scripts。
5. **交接與方法論三支**（`handoff` 把做到一半的工作交接給下一個對話／`teach` 用蘇格拉底式
   教你懂一個概念／`writing-great-skills` 自己寫技能的方法論）：
   https://github.com/mattpocock/skills ——同上 repo，各技能資料夾照通用裝法。
6. **web-archiving**（把網頁當下的樣子存進永久檔案庫，留證據用）：
   https://github.com/jamditis/claude-skills-journalism ——**只裝** research-toolkit 裡的
   `web-archiving` 那一支資料夾，不要 clone 整包啟用其他部分。
   ⚠️ 含個資或內部資訊的網址**絕不**送公開存檔——存了就永久撤不掉。

另外：入門包 repo 的 `skills/` 若日後有其他新技能，介紹用途後詢問安裝。

---

## 階段 4：重啟、驗收、第一課

1. 重啟讓新裝的 MCP 工具與技能載入。**重啟前先講好回來的路**：
   「重開後從左側清單點回**這個對話**，跟我說『繼續設定』；
   真的找不到舊對話，就重貼 README 那段咒語並說『從階段 4 繼續』。」
   - **桌面版**：完全結束 app（Mac `Cmd + Q`；Windows 右鍵工作列圖示結束）再重開，
     回到 Code 分頁、選同一個工作資料夾。
   - **終端機版**：輸入 `/exit` 離開，再輸入 `claude` 重新啟動。
   回來後請他輸入 `/mcp`，確認**有裝的每個** MCP 都顯示已連線。
2. 帶他做小測試，**每項指明用哪個工具，做完回報實際用了哪個**（一次一個，
   只測有裝的模組）：
   - **網頁（測 firecrawl，人人）**：「用 firecrawl 讀這個網頁的重點」（請他貼一個連結）
   - **文書（測簡報技能，人人）**：「幫我做一頁 PowerPoint，主題隨意，存到工作資料夾」
   - **文獻（醫療模組）**：「用 pubmed 工具查一題他專科的臨床問題，挑三篇給摘要」，
     接著「用 openalex 確認那三篇沒有被撤稿」
   哪一項失敗就記下來，不要籠統說「都好了」。
3. **交付使用手冊**：把本入門包的 `GUIDE.md` 下載到使用者的工作資料夾
   （ https://raw.githubusercontent.com/Dhahran123/claude-code-starter-tw/main/GUIDE.md ），
   帶他快速導覽一遍——**只講他有裝的部分**：功能地圖與例句、
   「讓它越用越懂你」那節（教他調教員工手冊）、安全兩條。
   收尾告訴他：「以後忘記什麼，跟我說『**打開使用手冊**』就好。」
4. 教他日常操作：`/help`（指令總覽）、`/compact`（對話瘦身）、`/clear`（開新話題）、
   `Esc`（喊停）、`/doctor`（自我診斷）；回到之前的對話——桌面版點左側清單，
   終端機版輸入 `/resume`。（手冊第二節都有，教一遍加深印象。）
5. 輸出驗收清單，MCP 與技能分開列、**逐項一列**，固定三欄：
   `名稱｜成功／失敗／跳過｜證據或原因`（成功要寫測過什麼，不可只打勾）。
6. 問他要不要接著做選配階段 5（Gmail／行事曆）與階段 6（Codex 第二位 AI）——
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

**接之前先問一題（處理敏感資料的行業必問）**：「這個 Google 帳號的信箱或行事曆，
平常會不會出現病患／客戶的可識別資料？」
- **會** → 建議不要接這個帳號（或改接一個不含敏感資料的個人帳號）。
- **不會** → 再往下做。

並白話講清楚：接上後 Claude 讀到的信件與行程內容都會傳到它的雲端處理，
跟貼進對話同一個等級——所以個人設定檔那條紅線，信箱和行事曆也一體適用。

⚠️ 給你（Claude）的界線：**不要**帶使用者走「自建 Google Cloud OAuth 憑證」的
社群 MCP 路線——那條對新手太難、坑很多。官方 Connectors 若不可用，回報並跳過。

---

## 階段 6（選配）｜第二位 AI：與 Codex 協作

先問使用者有沒有訂閱 ChatGPT（通常需要付費方案；沒有就跳過本階段，
說明以後想裝隨時說一聲）。

白話說明價值：Codex 是 OpenAI 出的同類工具。兩家不同公司的 AI **互相挑錯**——
重要的交付物可以請另一位「複查」，比單一 AI 自己說自己對可靠。

安裝（以下是斜線指令，請**使用者本人**在輸入框逐條輸入，一條完成再下一條）：

1. `/plugin marketplace add openai/codex-plugin-cc` ——把 OpenAI 官方外掛市集加進來
2. `/plugin install codex@openai-codex` ——安裝外掛
3. 完全重啟 app，回到同一個工作資料夾
4. `/codex:setup` ——首次設定：會協助安裝 Codex 本體並用 ChatGPT 帳號登入
   （過程跳出瀏覽器登入頁是正常的）

用法：之後輸入 `/codex:` 就會看到可用指令，**實際指令名以清單為準**；
最常用的是 review 類。教他一句口訣：「做完重要的東西，叫另一位看一遍。」

⚠️ 紅線（講給使用者聽，並在他的 `~/.claude/CLAUDE.md` 硬規則區補一行）：
**任何 `/codex:` 開頭的指令**，都會把相關內容（對話、檔案、程式碼）送到
OpenAI 的雲端——不是只有 transfer／delegate 才算。
**含病患／客戶敏感資料的工作一律不使用 `/codex:` 指令。**

---

*維護註記（給整理者）：本文件內的安裝指令與 repo 路徑會隨時間過期；
發現壞掉歡迎開 issue，或直接改。*
