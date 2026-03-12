---
name: Report Generation
description: 根據所有分析結果產出系統架構摘要、各 Form 分析報告與商業邏輯索引
role: 技術文件撰寫師
---

# 報告產出

## 前置輸入

1. `memory-bank/progress.md` — 確認所有 Form 已分析完成
2. `memory-bank/01-project-structure.md` — 專案結構
3. `memory-bank/02-protocols.md` — 通訊協定
4. `memory-bank/03-form-inventory.md` — 表單盤點
5. `memory-bank/forms/*.md` — 所有已完成的 Form 分析結果

## 執行步驟

### Step 1：產出系統架構摘要

整合 `01-project-structure.md` 和 `02-protocols.md`，寫入 `analysis-reports/00-system-architecture.md`。

必須包含 6 個章節：
1. 專案基本資訊
2. 專案結構（目錄樹、檔案統計）
3. 通訊協定與資料存取
4. Form 清單與關係（含 Mermaid 關係圖）
5. 共用元件
6. 第三方相依性

包含 Mermaid 系統架構圖（`graph TD`）。

### Step 2：產出各 Form 分析報告

對每個已分析的 Form，從 `memory-bank/forms/{formName}.md` 讀取，整理為正式報告。

寫入 `analysis-reports/{序號}-form-{formName}.md`（序號兩位數，從 01 開始）。

每份報告必須包含 6 個章節：
1. 表單概述
2. 欄位清單與資料來源
3. 欄位詳細分析
4. 按鈕與動作
5. 事件流
6. 相依性

每份報告必須包含 Mermaid 圖表。

### Step 3：產出商業邏輯索引

彙整所有 Form 的分析結果，寫入 `analysis-reports/99-business-logic-index.md`：

```markdown
# 商業邏輯索引

## 資料表使用索引
| 資料表名稱 | 操作類型 | 使用的 Form | 使用的方法 | 程式碼位置 |
|---|---|---|---|---|

## 共用方法/模組索引
| 方法名稱 | 所在檔案 | 呼叫的 Form | 功能摘要 |
|---|---|---|---|

## 外部服務呼叫索引
| 服務 | 協定 | 呼叫的 Form | 用途 |
|---|---|---|---|

## 計算邏輯索引
| 計算名稱 | 公式/邏輯 | 使用的 Form | 程式碼位置 |
|---|---|---|---|

## 待確認事項彙整
{從 progress.md 整理}
```

### Step 4：自我檢查

- [ ] 每份 Form 報告包含 6 個章節
- [ ] 所有結論附有程式碼引用
- [ ] Mermaid 語法正確
- [ ] 商業邏輯索引涵蓋所有 Form
- [ ] 使用繁體中文

不符合的項目立即修正。

## 輸出位置

```
analysis-reports/
├── 00-system-architecture.md
├── 01-form-{name}.md
├── 02-form-{name}.md
├── ...
└── 99-business-logic-index.md
```

## 更新進度

更新 `memory-bank/progress.md`：將「Phase 4」狀態改為 `✅ 完成`，填入完成日期。

## 注意

如果 Form 數量很多，可分批產出。先告知使用者總共需要產出多少份報告，建議分幾次對話完成。
