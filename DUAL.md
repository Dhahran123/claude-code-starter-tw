# 雙棲指南——Claude Code × Codex

這份給三種人：
- **用 Codex 跑這套設定的人**（第一節：對照表，讓 Codex 能執行同一份 SETUP.md）
- **已經在用其中一個、想加裝另一個的人**（第二節）
- **想讓兩個 AI 共用記憶、互相審稿的人**（第三、四節）

只用一個工具的人，這份可以整份跳過。

---

## 一、Codex 對照表（給執行 SETUP.md 的 Codex 讀）

SETUP.md 是以 Claude Code 為主線寫的。你是 Codex 的話，把下面左欄的東西換成右欄，其餘照做：

| SETUP.md 裡的 Claude 做法 | Codex 的對應做法 |
|---|---|
| 環境判斷 `command -v claude` | `command -v codex`；Codex 一律有 CLI，沒有「桌面版無 CLI」這個分支 |
| 「桌面版做法」（編 `~/.claude.json`） | 不需要，一律走指令版 |
| 員工手冊 `~/.claude/CLAUDE.md` | **`~/.codex/AGENTS.md`**（內容格式一樣，照 SETUP 階段 1 的模板寫） |
| `claude mcp add --scope user 名稱 -- npx -y 套件` | `codex mcp add 名稱 -- npx -y 套件` |
| `claude mcp add … -e KEY=VAL …` | `codex mcp add 名稱 --env KEY=VAL -- npx -y 套件` |
| HTTP 型 MCP（Firecrawl）`--transport http` | 寫進 `~/.codex/config.toml`：<br>`[mcp_servers.firecrawl]`<br>`url = "https://mcp.firecrawl.dev/v2/mcp"` |
| `claude mcp list`／`/mcp` | `codex mcp list`；對話裡輸入 `/mcp` |
| 技能資料夾 `~/.claude/skills/` | **`~/.agents/skills/`**（官方位置；舊版 Codex 用 `~/.codex/skills/`，裝完技能清單沒出現就兩邊都放連結） |
| 呼叫技能 `/技能名` | `$技能名`（Codex 也會依 description 自動觸發） |
| 重啟 Claude Code | 結束 codex 再重新啟動（`codex`） |
| 回到之前的對話 | `codex resume` |
| 階段 5 Gmail／行事曆 Connectors | **Codex 沒有這個圖形連接器**——跳過；想要的話用 Claude 那邊接 |
| 階段 6「第二位 AI：Codex」 | 你就是 Codex，跳過；反向做法看本文件第四節 |
| find-skills 的 `npx skills add` | 一樣能用（skills.sh 支援 Codex）；裝好後確認它落在上面的技能資料夾 |

Windows 補充：Codex 用 PowerShell 時，`npx` 型 MCP 若連不上，command 改 `cmd /c npx`（跟 Claude 那邊同一個坑）。

---

## 二、已經有一個，加裝另一個

### 已有 Claude Code → 加 Codex

1. 需要 **ChatGPT 付費方案**（Plus／Pro／Business／Edu／Enterprise 之一）。
2. 裝 CLI（擇一）：
   - **Mac**：`curl -fsSL https://chatgpt.com/codex/install.sh | sh`（或 `brew install --cask codex`）
   - **Windows／有 Node 的人**：`npm install -g @openai/codex`
3. 在工作資料夾輸入 `codex` → 選 **Sign in with ChatGPT** → 瀏覽器登入。
4. 想要圖形介面：輸入 `codex app`（或到官方頁下載 Codex App）。
5. 接著做第三節「共同記憶」。

### 已有 Codex → 加 Claude Code

1. 需要 **Claude 付費方案**（Pro 或 Max）。
2. 照 README 第 1、2 步裝桌面版並登入（或終端機版：README 末段一行指令）。
3. 接著做第三節「共同記憶」——做完之後**不必**重跑整份 SETUP：
   員工手冊與技能會直接共用，只有 MCP 要在 Claude 這邊再加一次（照 SETUP 階段 2）。

---

## 三、共同記憶：一份員工手冊、一套技能，兩個 AI 共用

原理：兩邊各自讀自己的設定檔，但我們讓那些路徑**指向同一份檔案**——改一次、兩邊都變。
（對話紀錄不共用，也不需要共用；跨工具接手看第五節。）

### 3a. 員工手冊一份兩用

先看 `~/.codex/AGENTS.md` 在不在：
- **不在** → 直接建連結（下面指令）。
- **在** → 先把它的內容併進 `~/.claude/CLAUDE.md`（兩份合一、去重），把舊的 AGENTS.md 改名備份，再建連結。

**Mac**（符號連結）：
```bash
mkdir -p ~/.codex
ln -s ~/.claude/CLAUDE.md ~/.codex/AGENTS.md
```

**Windows**（硬連結，不需要管理員權限；PowerShell）：
```powershell
New-Item -ItemType Directory -Force "$HOME\.codex" | Out-Null
New-Item -ItemType HardLink -Path "$HOME\.codex\AGENTS.md" -Target "$HOME\.claude\CLAUDE.md"
```
⚠️ Windows 硬連結的坑：有些編輯器存檔是「另存新檔再取代」，會把連結**默默拆掉**變成兩份獨立檔案。
之後改員工手冊**請 AI 改**（用工具直接寫入，不會拆），或改完跑
`fsutil hardlink list "$HOME\.claude\CLAUDE.md"` 確認還列出兩條路徑。

驗收：在 Claude 說「打開你的員工手冊」、在 Codex 說「讀你的 AGENTS.md」——內容一模一樣就成功。

### 3b. 技能共用

**Mac**：
```bash
mkdir -p ~/.agents
ln -s ~/.claude/skills ~/.agents/skills
```
（`~/.agents/skills` 已經有東西的話，改成逐個技能建連結，不要整夾覆蓋。）

**Windows**（junction，不需管理員）：
```powershell
New-Item -ItemType Directory -Force "$HOME\.agents" | Out-Null
New-Item -ItemType Junction -Path "$HOME\.agents\skills" -Target "$HOME\.claude\skills"
```

驗收：重啟 Codex，輸入 `$` 看技能清單有沒有 Claude 那邊裝的技能；沒有就再對 `~/.codex/skills` 做一次同樣的連結（舊版路徑）。

### 3c. 不共用的東西（各裝一次）

- **MCP**：兩邊設定檔格式不同，照第一節對照表在 Codex 再加一次。
- **對話紀錄**：各自的，不同步。
- **權限設定**：各自管。

---

## 四、互審機制：做完重要的東西，叫另一家看一遍

兩家公司的 AI 盲點不同——同一份東西讓另一個獨立看，比自己說自己對可靠。規則很簡單：

1. **什麼要審**：重要交付物（給客戶的網站、對外發布的文章、要用很久的腳本或設定）。
   日常小改不用。
2. **怎麼給**：審查方只拿「目標＋受審檔案＋限制」，**不要給它你的結論**（會被帶著走）。
3. **怎麼收**：主責方（做的那一方）依證據決定採納哪些；**最多兩輪**，還有分歧就列出來讓你決定，
   不要讓兩個 AI 無限辯論。
4. **紅線**：審查請求裡貼的內容會送到另一家的雲端——含病患／客戶可識別資料的東西不送。

### Claude 做完 → 叫 Codex 審

- 有裝 SETUP 階段 6 的 Codex 外掛：輸入 `/codex:` 看 review 類指令。
- 沒裝外掛：開 Codex 對話，貼下面的審查模板。

### Codex 做完 → 叫 Claude 審

- 有終端機版 Claude 的人，在 Codex 對話裡直接請它跑：
  ```bash
  claude -p "（貼下面的審查模板）"
  ```
- 只有桌面版的人：開 Claude 對話，貼模板即可。

### 審查模板（複製貼上，換掉◯◯）

```
請以獨立審查者身分審查以下內容，不要幫我改，只回報問題。
目標：◯◯（這份東西要達成什麼）
受審檔案：◯◯（路徑或貼上內容）
限制：◯◯（例如：不能改視覺定案、要相容 Windows）
輸出格式：按嚴重度分 🔴 會出錯／🟡 有疑慮／🔵 建議，每條附位置、問題、建議修法；沒發現的級別明說。
```

---

## 五、跨工具接手（一個專案兩邊輪流做）

在專案資料夾放一份 `CURRENT.md`——「現在做到哪」的交接單：目標、目前誰在改、已定案的決定、
下一步、還沒解決的問題。換工具接手時先讀它、核對實際檔案再動手；改完更新它。
同一份檔案同時只讓一邊改，避免互相覆蓋。

---

*維護註記：Codex 的安裝指令、skills 路徑與 MCP 語法為 2026-09-11 依官方文件查證；日後變動以官方為準。*
