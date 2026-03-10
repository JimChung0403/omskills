# Legacy VB.NET Analyzer 使用指南

## 適用環境

- **工具**：Cline（VS Code 擴充套件）
- **LLM**：OpenAI Completion API（或相容 API）
- **目標**：分析 VB.NET WinForms 舊系統，產出結構化的邏輯追蹤文件

---

## 一、部署（一次性設定）

將以下檔案複製到你的 VB.NET 專案根目錄：

```
你的 VB.NET 專案/
├── .clinerules                    ← Cline 自動載入（agent 定義 + 全域規範）
├── skills/                        ← 分析技能
│   ├── project-scan/SKILL.md
│   ├── protocol-detection/SKILL.md
│   ├── form-inventory/SKILL.md
│   ├── form-analysis/SKILL.md
│   └── report-generation/SKILL.md
├── memory-bank/                   ← Memory Bank（跨對話記憶體）
│   └── progress.md
└── （你的 .vb、.sln 等原始碼...）
```

用 VS Code 開啟專案，Cline 會自動載入 `.clinerules`。

---

## 二、使用方式

你只需要用自然語言告訴 Cline 要做什麼，它會自動讀取對應的 skill 檔案並執行。

### 常用指令

| 你說的話 | Cline 的動作 |
|---|---|
| `開始分析` | 從 Phase 1a 開始，執行 project-scan skill |
| `執行 project-scan` | 讀取 `skills/project-scan/SKILL.md` 並執行 |
| `執行 protocol-detection` | 讀取 `skills/protocol-detection/SKILL.md` 並執行 |
| `執行 form-inventory` | 讀取 `skills/form-inventory/SKILL.md` 並執行 |
| `分析 frmOrder` | 讀取 `skills/form-analysis/SKILL.md`，以 frmOrder 為目標執行 |
| `執行 report-generation` | 讀取 `skills/report-generation/SKILL.md` 並執行 |
| `繼續` | 讀取 `progress.md`，判斷下一步，自動執行 |
| `目前進度` | 讀取並摘要 `progress.md` |

---

## 三、分析流程

```
Phase 1a ──→ Phase 1b ──→ Phase 2 ──→ Phase 3 ──→ Phase 4
project-     protocol-     form-        form-         report-
scan         detection     inventory    analysis      generation
(1次)       (1次)        (1次)       (每Form 1次)  (1次)
```

### Phase 1a：專案結構掃描

```
你：執行 project-scan
```

結果寫入 `memory-bank/01-project-structure.md`。

### Phase 1b：通訊協定偵測

```
你：執行 protocol-detection
```

結果寫入 `memory-bank/02-protocols.md`（含 Mermaid 系統架構圖）。

### Phase 2：表單盤點

```
你：執行 form-inventory
```

結果寫入 `memory-bank/03-form-inventory.md`（含 Mermaid 關係圖 + 建議分析順序）。

### Phase 3：逐表單分析（每個 Form 一次）

```
你：分析 frmOrder
```

完整深度分析該 Form 的所有欄位、按鈕、呼叫鏈、事件流。
結果寫入 `memory-bank/forms/frmOrder.md`（含 Mermaid 資料流圖 + 動作流程圖）。

**依照 Phase 2 的「建議分析順序」逐一分析。**

### Phase 4：報告產出

```
你：執行 report-generation
```

產出 `analysis-reports/` 下的所有最終報告。

---

## 四、何時需要開新對話？

### Token 消耗估計

本分析工具使用 Claude Haiku 4.5（快速、低成本），128K context window（~131K 個 token）。

#### 各階段 Token 消耗

| 階段 | Token 低估 | 典型估計 | 高估 |
|---|---|---|---|
| Phase 1a（專案掃描） | 2,000 | 3,500 | 6,000 |
| Phase 1b（協定偵測） | 2,500 | 4,000 | 6,500 |
| Phase 2（表單盤點） | 2,400（5 Form） | 5,000（20 Form） | 10,250（50+ Form） |
| **Phase 1a+1b+2** | — | ~**12,500 合計** | — |
| Phase 3 / Form（中等複雜度） | 6,000 | 10,000 | 18,000 |

#### 單次對話容量

- **可用 Token**：~104,858 tokens（留 20% 緩衝）
- **Phase 1a+1b+2 後剩餘**：~92,000 tokens
- **Phase 3 容納 Form 數**（每 Form ~10k token）：
  - **保守策略**（留 30% 緩衝）：5 Form / 對話 ✅ 推薦
  - **中等策略**（留 20% 緩衝）：6-7 Form / 對話
  - **激進策略**（留 10% 緩衝）：8+ Form / 對話（危險，易截斷）

#### 推薦對話計劃

**小型專案（< 10 Form）**
```
對話 1: Phase 1a + 1b + 2
對話 2: Phase 3（所有 Form）+ Phase 4
= 2 次對話
```

**中型專案（10-30 Form）**
```
對話 1: Phase 1a + 1b + 2
對話 2: Phase 3（Form 1-5）
對話 3: Phase 3（Form 6-10）
對話 4: Phase 3（Form 11-15）
對話 5: Phase 3（Form 16-20）...
最後:   Phase 4（全部 Form）
```

**大型專案（30+ Form）**
- 每 4-6 個 Form 開一次新對話（Phase 3）
- Phase 4 單獨對話

### 何時開新對話

| 狀況 | 建議 |
|---|---|
| Phase 1a+1b+2 完成，準備 Phase 3 | **一定要開新對話** |
| Phase 3 已分析 5 個 Form | 開新對話（保險起見） |
| Cline 回應開始變慢或遺漏內容 | **立即開新對話** |
| 看到「context limit」或「output truncated」警告 | **立即開新對話** |

開新對話**不會丟失進度**。新對話中只需說 `繼續`，Cline 會讀取 `memory-bank/progress.md` 自動接續。

---

## 五、中斷與恢復

隨時可中斷。進度保存在 `memory-bank/progress.md`。

恢復方式：開啟 Cline，說 `目前進度` 或 `繼續`。

重新分析某個 Form：說 `重新分析 frmOrder`。

---

## 六、Mermaid 圖表轉圖片

```bash
# 安裝
npm install -g @mermaid-js/mermaid-cli

# 單張轉換
mmdc -i diagram.mmd -o diagram.png

# 批次轉換（請 Cline 幫你）
你：讀取 analysis-reports/ 下所有 .md，提取 mermaid 區塊，
    存為 analysis-reports/diagrams/{name}-{index}.mmd，
    然後對每個 .mmd 執行 mmdc -i {input} -o {output}.png
```

---

## 七、架構說明

```
.clinerules                              ← Cline 自動載入（Agent 定義）
│
├── skills/                              ← Skills（5 個）
│   ├── project-scan/SKILL.md            ← Phase 1a：專案結構掃描
│   ├── protocol-detection/SKILL.md      ← Phase 1b：通訊協定偵測
│   ├── form-inventory/SKILL.md          ← Phase 2：表單盤點
│   ├── form-analysis/SKILL.md           ← Phase 3：表單深度分析（欄位+按鈕+呼叫鏈+事件流）
│   └── report-generation/SKILL.md       ← Phase 4：報告產出
│
└── memory-bank/                         ← Memory Bank（跨對話記憶體）
    ├── progress.md                      ← 進度追蹤（核心）
    ├── 01-project-structure.md          ← Phase 1a 產出
    ├── 02-protocols.md                  ← Phase 1b 產出
    ├── 03-form-inventory.md             ← Phase 2 產出
    └── forms/                           ← Phase 3 產出
        └── {formName}.md
```

---

## 八、常見問題

### Q：Form 太多，要分析很久怎麼辦？

依照 Phase 2 的「建議分析順序」，先分析最核心的 10-20 個。

### Q：可以只分析特定功能嗎？

```
你：我想了解「訂單儲存」的完整邏輯，請找出相關的 Form 和方法，
    追蹤完整呼叫鏈，結果寫入 memory-bank/feature-order-save.md
```

### Q：分析結果不準確怎麼辦？

```
你：frmOrder 的 btnSave 分析遺漏了 Try-Catch，請重新分析這個按鈕，
    更新 memory-bank/forms/frmOrder.md
```
