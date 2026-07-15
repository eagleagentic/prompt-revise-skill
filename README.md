# prompt-revise

> 依據 GPT-5.6 prompting guidance，將冗長、含糊或互相矛盾的 prompt 改寫成精簡、可執行且可驗證的版本。

`prompt-revise` 是一個用於審閱與改寫 prompt 的 agent skill。它保留原始目標與硬性約束，同時整理重複規則、權限邊界、工具路由、證據要求及完成條件，讓 prompt 更容易交給 Codex 或其他支援 skills 的 agent 使用。

這個 skill 的重點不是只列出問題，而是預設直接交付可使用的 revised prompt，並簡短說明關鍵變更；若有不應寫入 prompt 的模型或執行設定，則另外列為 runtime 建議。

## 快速開始

1. 複製 repository，並將 skill 安裝到 `~/.agents/skills/prompt-revise`：

   ```sh
   git clone https://github.com/eagleagentic/prompt-revise-skill.git
   test ! -e ~/.agents/skills/prompt-revise
   mkdir -p ~/.agents/skills
   cp -R prompt-revise-skill/prompt-revise ~/.agents/skills/prompt-revise
   ```

2. 開啟支援 skills 的 agent，使用 `$prompt-revise` 進行第一次改寫：

   ```text
   $prompt-revise 請改寫以下 prompt，使目標、輸出格式與成功條件更清楚：
   「幫我寫一篇好的產品介紹」
   ```

若 skill 未在目前工作階段出現，請開始新的對話或 agent session，讓宿主重新載入 skill。需要檢查既有安裝或更新來源時，請參閱[安裝指南](#安裝指南)；更多範例見[使用方式](#使用方式)。

## 這是甚麼

`prompt-revise` 適合處理需要精簡、校正、遷移或補齊執行契約的 prompt，包括：

- system prompt
- developer instructions
- agent instructions
- tool descriptions
- 由多個角色層級組成的 prompt stacks

它會辨識 prompt 的預期成果、受眾、runtime 角色、可用工具、輸出格式、批准邊界與完成標準；對多層 prompt，則保留各層角色並檢查跨層矛盾。只要意圖足以判斷，skill 便會直接產出完整 revised prompt，而不是停在批評或評分。只有不同解讀會實質改變結果時，才需要追問。

## 運作流程

```text
原始 prompt
     |
     v
辨識目標與約束
     |
     v
移除重複與矛盾
     |
     v
補充成功及停止條件
     |
     v
Revised prompt
```

## 設計依據

本 skill 依據 OpenAI 官方的 [Prompting guidance for GPT-5.6](https://developers.openai.com/api/docs/guides/prompt-guidance-gpt-5p6) 設計。與本專案直接相關的重點包括：

- 先定義成果、重要約束、可用證據與完成門檻，再讓模型選擇有效的執行路徑。
- 優先移除重複指令、無法改變行為的範例與過度複雜的工具描述。
- 把可觀察的成功條件、停止規則、fallback 與驗證要求寫清楚，而非堆疊過度細碎的流程指示。
- prompt 與 runtime configuration 分開處理，並以代表性任務驗證修改後的效果。

README 只摘要本 skill 實際採用的原則；完整背景、模型細節與最新建議請以官方文件為準。

## 核心原則

| 原則 | 實際做法 |
| --- | --- |
| Outcome-first prompting | 先描述成功成果與完成門檻，再補充真正影響結果的限制。 |
| 保留使用者目標與硬性約束 | 保留可見成果、必要事實、格式、安全、隱私及商業限制，不自行增加政策或能力。 |
| 移除重複、矛盾及無效 scaffolding | 合併同義規則，刪除不改變行為的範例、流程旁白與無關工具。 |
| 明確定義成功條件和停止規則 | 只在影響正確性時加入成功、停止、重試、fallback、放棄或澄清規則。 |
| 區分 personality 與 collaboration style | personality 管理語氣；collaboration style 管理假設、主動性、提問、取捨與驗證。 |
| 集中定義自主權和批准邊界 | 一次說清楚唯讀分析、本地修改、外部寫入、破壞性操作、成本與擴大範圍的授權。 |
| 精簡工具描述並清楚設定 tool routing | 每個相關工具只保留能力、觸發條件、重要輸出與失敗處理；獨立讀取可平行，有相依性的工作依序執行。 |
| 對證據、引用與 fallback 設定規則 | 指定哪些主張需要證據、何謂充分證據、引用位置，以及證據不足或衝突時如何縮小結論。 |
| 在完成前執行相關驗證 | 使用針對性的測試、lint、型別檢查、受影響的 build 或最小 smoke test；無法執行時說明原因。 |
| 將 runtime configuration 與 prompt 內容分開 | reasoning effort、`text.verbosity` 等執行設定放在 runtime notes，不重複塞入 prompt。 |

## 安裝指南

### 首次安裝

以下指令使用 repository 的實際目錄 `prompt-revise/`，並在目標已存在時停止，避免無提示覆寫既有內容：

```sh
git clone https://github.com/eagleagentic/prompt-revise-skill.git
cd prompt-revise-skill
test ! -e ~/.agents/skills/prompt-revise
mkdir -p ~/.agents/skills
cp -R prompt-revise ~/.agents/skills/prompt-revise
```

安裝結果應包含：

```text
~/.agents/skills/prompt-revise/
├── SKILL.md
└── agents/
    └── openai.yaml
```

skill 何時被載入取決於使用中的 agent 宿主。若安裝後無法立即使用 `$prompt-revise`，請開啟新的對話或 agent session，再進行呼叫。

### 更新既有安裝

先更新本地 repository，再比較來源與已安裝版本：

```sh
cd prompt-revise-skill
git pull --ff-only
diff -ru prompt-revise ~/.agents/skills/prompt-revise
```

`diff` 結束碼為 `0` 代表內容相同，`1` 代表有差異。請先檢視差異；若安裝目錄含有個人修改，請自行合併，不要直接覆寫。確認要以 repository 版本取代既有安裝後，才執行：

```sh
rm -rf ~/.agents/skills/prompt-revise
cp -R prompt-revise ~/.agents/skills/prompt-revise
```

上述更新會刪除目標目錄，因此只適用於已確認不需保留其中個人內容的情況。

## 使用方式

在支援 skills 的 agent 對話中，以 `$prompt-revise` 明確呼叫 skill，並附上要改寫的 prompt。若有指定模型、角色層級、不可變更的限制或期望輸出，請一併寫入請求。

### 範例一：內容生成 prompt

```text
$prompt-revise 請把以下 prompt 改寫成可直接交給一般 LLM 的版本，保留繁體中文輸出：
「幫我寫一篇好的產品介紹，讓大家想買。」
```

### 範例二：含工具、權限與驗證規則的 agent prompt

```text
$prompt-revise 請為 GPT-5.6 改寫以下 coding agent prompt，保留權限邊界並補齊完成條件：

「檢查 repository 並修正登入失敗。可以讀取與修改本地檔案，也可以執行測試；不得部署、推送、讀取 secrets 或刪除使用者資料。先找出根因，遵循現有架構，只做最小修改。完成前執行相關測試；若測試無法執行，說明原因與剩餘風險。」
```

## 輸出內容

skill 一般先交付完整 prompt，再提供精簡說明：

| 區段 | 內容 | 是否屬於 prompt |
| --- | --- | --- |
| `Revised prompt` | 完整、可直接使用的改寫結果。 | 是 |
| `What changed` | 3 至 7 項影響可靠性、清晰度或 token 效率的主要變更。 | 否，為變更摘要 |
| `Runtime notes` | reasoning effort、`text.verbosity` 或實作層設定等不應寫入 prompt 的建議。 | 否；僅適用時提供 |
| `Assumptions` | 改寫時採用的重大假設。 | 否；僅適用時提供 |

這項區分可避免把模型或 API 的 runtime configuration 誤寫成任務內容。若沒有適用的 runtime 建議或重大假設，對應區段會省略。

## Repository 結構

```text
prompt-revise-skill/
├── README.md
└── prompt-revise/
    ├── SKILL.md
    └── agents/
        └── openai.yaml
```

- [`prompt-revise/SKILL.md`](prompt-revise/SKILL.md)：skill 的適用範圍、改寫規則、輸出契約與完成檢查。
- [`prompt-revise/agents/openai.yaml`](prompt-revise/agents/openai.yaml)：顯示名稱、簡短描述與預設呼叫 prompt 等介面 metadata。
