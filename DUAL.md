# 雙棲指南——Claude Code × Codex

誰要讀哪一節：
- **只用 Claude Code** → 整份可跳過。
- **只用 Codex** → 至少讀第一節（讓 Codex 能執行 SETUP.md）。
- **已有一個、想加另一個** → 第二節。
- **想讓兩個 AI 共用記憶、互相審稿** → 第三、四、五節。

---

## 一、Codex 對照表（給執行 SETUP.md 的 Codex 讀）

SETUP.md 是以 Claude Code 為主線寫的。你是 Codex 的話，把下面左欄的東西換成右欄，其餘照做：

| SETUP.md 裡的 Claude 做法 | Codex 的對應做法 |
|---|---|
| 環境判斷 `command -v claude` | Mac 用 `command -v codex`；Windows PowerShell 用 `Get-Command codex`。本路線一定裝了 CLI，沒有「桌面版無 CLI」這個分支 |
| 「桌面版做法」（編 `~/.claude.json`） | 不需要，一律走指令版 |
| 員工手冊 `~/.claude/CLAUDE.md` | **`~/.codex/AGENTS.md`**（內容格式一樣，照 SETUP 階段 1 的模板寫；若已有 `AGENTS.override.md` 它會優先生效，先看有沒有） |
| **階段 1b 安全防護**（Claude 的 `settings.json` 黑名單＋檔案存檔點） | **不能照搬那份 JSON**——Codex 不讀它。Codex 的權限靠 `~/.codex/config.toml` 的 `approval_policy` 與 `sandbox_mode`：**保持預設的「要問就問」**，不要設成免確認。檔案存檔點 Codex 沒有對應功能：改動重要檔案前請它先備份一份（`.bak`），並讓使用者知道「這邊沒有一鍵還原」 |
| `claude mcp add --scope user 名稱 -- npx -y 套件` | `codex mcp add 名稱 -- npx -y 套件` |
| `claude mcp add … -e KEY=VAL …` | `codex mcp add 名稱 --env KEY=VAL -- npx -y 套件`（多個變數就重複 `--env`） |
| HTTP 型 MCP（Firecrawl）`--transport http` | 先試 `codex mcp add firecrawl --url https://mcp.firecrawl.dev/v2/mcp`；版本不支援就寫進 `~/.codex/config.toml`（已有同名區塊就更新它，不要重複加）：<br>`[mcp_servers.firecrawl]`<br>`url = "https://mcp.firecrawl.dev/v2/mcp"` |
| `claude mcp list`／`/mcp` | `codex mcp list`；對話裡輸入 `/mcp` |
| Windows 上 npx 型 MCP 連不上要用 `cmd /c npx` | 一樣，但**程式與參數要分開**：指令版 `codex mcp add playwright -- cmd /c npx -y @playwright/mcp@latest`；設定檔版 `command = "cmd"`、`args = ["/c", "npx", "-y", "@playwright/mcp@latest"]`。只在直接用 `npx` 連不上時才這樣改 |
| 技能資料夾 `~/.claude/skills/` | **`~/.agents/skills/`**（官方位置）。裝完技能清單沒出現：先看 `codex --version` 與技能載入有無錯誤，確定是舊版才改用 `~/.codex/skills/`——**不要兩邊都放**，同名技能會重複出現 |
| 呼叫技能 `/技能名` | `$技能名`（Codex 也會依 description 自動觸發） |
| 技能內容本身含 Claude 專屬指令（例：`scheduler` 要靠 `claude` CLI 排程、找 `~/.claude/` 路徑的技能） | **相容性檢查**：裝前讀該 SKILL.md，含 Claude 專屬指令的技能在 Codex 不裝，或改寫並驗收後才啟用 |
| 重啟 Claude Code | 結束 codex 再重新啟動；**回到原本的設定對話用 `codex resume`**（單純打 `codex` 是開新對話） |
| `/doctor` 自我診斷 | 終端機跑 `codex doctor` |
| `/clear`、`/resume`、`/help`、`/compact` | Codex 也有這些指令，照用 |
| 階段 5 Gmail／行事曆 Connectors | Claude 那組選單步驟不適用 Codex。Codex 有自己的 Plugins 介面可接 Gmail 等服務——**本包暫不設定**，想接再另外處理 |
| 階段 6「第二位 AI：Codex」 | 你就是 Codex，跳過；反向做法看本文件第四節 |
| find-skills 的 `npx skills add` | 一樣能用（skills.sh 支援 Codex）；裝好後確認它落在上面的技能資料夾 |

---

## 二、已經有一個，加裝另一個

### 已有 Claude Code → 加 Codex

1. 帳號：用 ChatGPT 帳號登入。**免費／Go 方案也能用 Codex，但額度很少；本包建議 Plus 以上**。
   也可用另外計費的 API key，但新手不建議。
2. 裝 CLI（擇一）：
   - **Mac**：終端機貼 `curl -fsSL https://chatgpt.com/codex/install.sh | sh`（或 `brew install --cask codex`）
   - **Windows**：先確認有 Node.js（`node --version`，沒有就到 nodejs.org 裝 LTS），
     再在 PowerShell 貼 `npm install -g @openai/codex`
3. **關掉終端機重開**，輸入 `codex --version` 有版本號才算裝好。
   Windows 若出現「執行原則」錯誤，改打 `codex.cmd`。
4. 在工作資料夾啟動：終端機 `cd` 到資料夾（Mac 可把資料夾拖進終端機視窗取得路徑），
   輸入 `codex` → 選 **Sign in with ChatGPT** → 瀏覽器登入。
5. 想要圖形介面：在終端機輸入 `codex app`（會開 ChatGPT 桌面 app，Windows 會提示路徑），
   開啟後選同一個工作資料夾。
6. 接著做第三節「共同記憶」。

### 已有 Codex → 加 Claude Code

1. 帳號：Claude **Pro 或 Max**（Claude Code 不支援免費帳號）。
2. 照 README 第 1、2 步裝桌面版並登入（或終端機版：README 末段一行指令）。
3. 做 SETUP 的**階段 1b 安全防護**（那是 Claude 這邊才有的保險，Codex 那邊沒做過）。
4. 接著做第三節「共同記憶」——做完之後**不必**重跑整份 SETUP：
   員工手冊與技能會直接共用，只有 MCP 要在 Claude 這邊再加一次（照 SETUP 階段 2）。

---

## 三、共同記憶：一份員工手冊、一套技能，兩個 AI 共用

原理：兩邊各自讀自己的設定檔，但我們讓那些路徑**指向同一份檔案**——改一次、兩邊都變。
（對話紀錄不共用，也不需要；跨工具接手看第五節。）

### 3a. 員工手冊一份兩用

**先看哪一邊已經有內容**——有內容的那份是「正本」，另一邊做連結指向它：

| 狀況 | 做法 |
|---|---|
| 只有 `~/.claude/CLAUDE.md` | 正本＝它；建 `~/.codex/` 資料夾後，把 `~/.codex/AGENTS.md` 連到它 |
| 只有 `~/.codex/AGENTS.md` | 正本＝它；建 `~/.claude/` 後，把 `~/.claude/CLAUDE.md` 連到它 |
| 兩邊都有 | 先把兩份內容合成一份（去重、保留兩邊都要的規則）存進其中一份當正本，
另一份**改名備份**（例：`AGENTS.md.bak`）後再建連結 |

連結指令（以「Claude 為正本、Codex 連過去」為例；反過來就把兩個路徑對調）：

**Mac**（符號連結）：
```bash
mkdir -p ~/.codex
ln -s ~/.claude/CLAUDE.md ~/.codex/AGENTS.md
```

**Windows**（硬連結，家目錄內不需要管理員；PowerShell）：
```powershell
New-Item -ItemType Directory -Force "$HOME\.codex" | Out-Null
New-Item -ItemType HardLink -Path "$HOME\.codex\AGENTS.md" -Target "$HOME\.claude\CLAUDE.md"
```
前提：來源檔存在、目的地**不存在**（存在就先改名備份）；硬連結兩邊要在同一顆磁碟。

⚠️ **硬連結會被「另存新檔再取代」式的存檔默默拆掉**，變成兩份各自獨立的檔——
不論是人用編輯器改、還是 AI 用工具改，都可能發生。所以規矩是：
**每次改完員工手冊，都跑一次 `fsutil hardlink list "$HOME\.claude\CLAUDE.md"`**，
要列出兩條路徑才算還連著；只剩一條就重做連結。

兩件小事：Codex 讀指引檔有預設 32 KiB 的**載入上限**（可在 config.toml 用 `project_doc_max_bytes` 調高）——
員工手冊寫太長會被截斷；`~/.codex/AGENTS.override.md` 若存在會**優先**生效，做連結前確認沒有它。

驗收：**各開一個全新對話**，在 Claude 說「你的員工手冊第一行是什麼」、在 Codex 說「你的 AGENTS.md 第一行是什麼」
——答案一樣才算兩邊都自動載入了（叫它們打開檔案只能證明讀得到，不能證明啟動時有載）。

### 3b. 技能共用

**先看哪一邊已經有技能**：

| 狀況 | 做法 |
|---|---|
| 只有 `~/.claude/skills/` 有東西 | 建 `~/.agents/` 後，把 `~/.agents/skills` 整夾連到 `~/.claude/skills` |
| 只有 `~/.agents/skills/` 有東西 | 反向：把 `~/.claude/skills` 整夾連到 `~/.agents/skills` |
| 兩邊都有 | **不要整夾覆蓋**——逐個技能建連結（每支技能一個連結），同名的先比對內容再決定留哪份 |

整夾連結指令（以 Claude 為正本為例）：

**Mac**：
```bash
mkdir -p ~/.agents
ln -s ~/.claude/skills ~/.agents/skills
```
（若 `~/.agents/skills` 已存在，`ln -s` 會把連結建在**裡面**而不是取代它——這就是為什麼「兩邊都有」要逐個處理。）

**Windows**（junction，不需管理員）：
```powershell
New-Item -ItemType Directory -Force "$HOME\.agents" | Out-Null
New-Item -ItemType Junction -Path "$HOME\.agents\skills" -Target "$HOME\.claude\skills"
```

驗收：重啟 Codex 開新對話，輸入 `$` 看技能清單有沒有另一邊裝的技能。
沒有：先看 `codex --version` 與有無載入錯誤，是舊版才把連結另建到 `~/.codex/skills`（擇一，不要兩邊都放）。
另外，含 Claude 專屬指令的技能在 Codex 這邊不會正常運作（第一節的相容性檢查）。

### 3c. 不共用的東西（各裝一次）

- **MCP**：兩邊設定檔格式不同，照第一節對照表在另一邊再加一次。
- **對話紀錄**：各自的，不同步。
- **權限與安全設定**：各自管（Claude 的階段 1b、Codex 的 config.toml）。

---

## 四、互審機制：做完重要的東西，叫另一家看一遍

兩家公司的 AI 盲點不同——同一份東西讓另一個獨立看，比自己說自己對可靠。規則：

1. **什麼要審**：重要交付物（給客戶的網站、對外發布的文章、要用很久的腳本或設定）。日常小改不用。
2. **怎麼給**：**開一個全新對話**給審查方（沿用舊對話會帶著既有結論），只給「目標＋受審檔案＋限制」，
   **不要給它你的結論**。
3. **怎麼收**：主責方（做的那一方）依證據決定採納哪些。**初審一次、修正後複審一次**，
   還有分歧就列出來讓你決定；沒問題就結束，不要讓兩個 AI 無限辯論。
4. **紅線**：送去審的內容——包括你貼的文字、**以及審查方依路徑讀進去的檔案**——都會上另一家的雲端。
   含病患／客戶可識別資料的東西不送。

### Claude 做完 → 叫 Codex 審

- 有裝 SETUP 階段 6 的 Codex 外掛：輸入 `/codex:` 看 review 類指令。
- 沒裝外掛：開一個新的 Codex 對話，貼下面的審查模板。

### Codex 做完 → 叫 Claude 審

- 有終端機版 Claude 的人，在 Codex 對話裡請它跑（先確認 Claude 已登入、工作目錄正確）：
  ```bash
  claude -p "（貼下面的審查模板）"
  ```
  注意：`-p` 只是「不開互動視窗」，**不等於唯讀**——所以模板第一句就要求它不動檔案；
  需要核准的動作在這個模式下不會跳視窗，別給它改東西的任務。
- 只有桌面版的人：開一個新的 Claude 對話，貼模板即可。

### 審查模板（複製貼上，換掉◯◯）

```
請以獨立審查者身分審查以下內容：只讀、只回報，不要修改任何檔案。
目標：◯◯（這份東西要達成什麼）
受審檔案：◯◯（路徑或貼上內容）
限制：◯◯（例如：不能改視覺定案、要相容 Windows）
輸出格式：
1. 先列出你實際讀到的檔案；讀不到的明說「未完成審查」，不要猜。
2. 問題按嚴重度分 🔴 會出錯／🟡 有疑慮／🔵 建議，每條附位置、問題、建議修法；沒發現的級別明說。
```

---

## 五、跨工具接手（一個專案兩邊輪流做）

在專案資料夾放一份 `CURRENT.md`——「現在做到哪」的交接單：目標、目前誰在改、已定案的決定、
下一步、還沒解決的問題。換工具接手時先讀它、核對實際檔案再動手；改完更新它。
同一份檔案同時只讓一邊改，避免互相覆蓋。

---

*維護註記：Codex 的安裝指令、skills 路徑與 MCP 語法為 2026-09-11 依官方文件查證，並經 Codex 本尊複審修正；日後變動以官方為準。*
