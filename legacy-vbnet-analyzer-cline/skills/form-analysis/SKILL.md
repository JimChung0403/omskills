---
name: Form Analysis
description: 對單一 WinForms 表單執行完整商業邏輯分析，追蹤欄位資料來源、按鈕動作鏈、跨檔案呼叫鏈與事件流
role: WinForms 商業邏輯分析師
---

# 單一表單深度分析

## 前置輸入

1. `memory-bank/progress.md` — 確認 Phase 1-2 已完成
2. `memory-bank/01-project-structure.md` — 專案結構
3. `memory-bank/02-protocols.md` — 通訊協定（了解資料存取方式）
4. `memory-bank/03-form-inventory.md` — 找到目標 Form 的檔案位置和控件清單

## 目標 Form

使用者指定的 Form 名稱。例如使用者說「分析 frmOrder」，則目標為 `frmOrder`。

## 執行步驟

### Step 1：讀取 Form 基本資訊

從 `03-form-inventory.md` 取得目標 Form 的檔案位置、控件清單、關聯 Form。

### Step 2：讀取 Form 原始碼

讀取目標 Form 的兩個檔案：
- `{FormName}.vb` — code-behind
- `{FormName}.Designer.vb` — 控件宣告

---

### Part A：欄位資料追蹤

對每個資料輸入/顯示欄位（TextBox、ComboBox、Label、DataGridView 等）執行追蹤。

**A1：識別資料綁定方式**

- 方式 A（DataBinding）：搜尋 `{控件名稱}.DataBindings.Add`。記錄綁定屬性、資料來源物件、欄位名稱。
- 方式 B（手動賦值）：搜尋 `{控件名稱}.Text =`、`{控件名稱}.Value =`。記錄賦值時機和來源表達式。
- 方式 C（DataSource）：搜尋 `{控件名稱}.DataSource =`。記錄 DataSource 物件、DisplayMember、ValueMember。

**A2：向上追蹤資料來源**

1. **BindingSource/DataSet** → 搜尋 `.DataSource =` 或 `.Fill(` → 找到 SQL 或 SP
2. **方法回傳值** → 讀取方法實作 → 繼續追蹤（跨檔案時執行 Part D）
3. **其他控件** → 記錄來源控件並對其重複 A1
4. **Form 參數** → 搜尋建構函式或公開屬性 → 追蹤呼叫端傳入的值

**A3：記錄中間運算**

以虛擬碼或公式記錄所有運算與轉換：算術運算、型別轉換（CInt、CDbl、CStr）、字串處理（Format、Substring）、條件邏輯（If-Else）、自訂函數。

---

### Part B：按鈕邏輯追蹤

對每個按鈕和觸發事件的控件執行追蹤。

**B1：找到事件處理方法**

- 搜尋 `Handles {按鈕名稱}.Click`
- 搜尋 `Private Sub {按鈕名稱}_Click`
- 讀取事件處理方法完整程式碼

**B2：分析動作流程**

逐行分析，用以下標記分類：

- `[驗證]` — 輸入值檢查、商業規則驗證
- `[收集]` — 從控件讀取值
- `[運算]` — 計算公式、條件分支、迴圈
- `[存取]` — 資料庫操作、檔案操作、外部服務呼叫
- `[更新]` — UI 更新、Form 切換、訊息顯示

**B3：追蹤跨方法呼叫**

呼叫其他類別/模組的方法時，執行 Part D 呼叫鏈追蹤。

**B4：識別錯誤處理**

- Try-Catch 區塊的範圍和例外類型
- Transaction 的 Begin / Commit / Rollback

---

### Part C：事件流分析

**C1：Form 生命週期**

搜尋 `_Load`、`Handles Me.Load`、`_Shown`、`_FormClosing` 事件，分析初始化邏輯。

**C2：控件連動**

搜尋：`SelectedIndexChanged`、`SelectedValueChanged`、`TextChanged`、`ValueChanged`、`CellValueChanged`、`CellClick`、`CheckedChanged`。記錄觸發控件和被影響控件。

**C3：Timer / 背景任務**

搜尋 Timer 和 BackgroundWorker 的事件處理。

---

### Part D：呼叫鏈追蹤

當 Part A 或 Part B 遇到跨檔案方法呼叫時執行。

**D1：識別方法呼叫**

- `{ClassName}.{MethodName}(`、`{ModuleName}.{MethodName}(`、`{variable}.{MethodName}(`

**D2：定位方法定義**

搜尋 `Sub {MethodName}` 或 `Function {MethodName}`，確認所屬類別與檔案，讀取完整程式碼。

**D3：遞迴追蹤**

重複 D1-D2，直到到達 SQL / 檔案 I/O / API / Framework 方法，或深度超過 10 層（標記「追蹤截斷」）。

**D4：特殊呼叫模式**

- `RaiseEvent` → 搜尋 `Handles` 或 `AddHandler`
- `BackgroundWorker.RunWorkerAsync` → 追蹤 `DoWork` 和 `RunWorkerCompleted`
- 介面呼叫 → 搜尋 `Implements {InterfaceName}` 找到實作類別
- 多載方法 → 根據參數數量和型別匹配

---

### Part E：Mermaid 圖表產出

**E1：欄位資料流圖**（選最重要的 3-5 個欄位，`flowchart LR`）

**E2：按鈕動作流程圖**（每個按鈕一張，`flowchart TD`）

---

## 輸出

寫入 `memory-bank/forms/{FormName}.md`：

```markdown
# {FormName} 分析結果

## 1. 表單概述
- 檔案位置：{path}
- 用途：{description}
- 關聯表單：{list}

## 2. 欄位清單與資料來源
| 欄位名稱 | 控件類型 | 資料來源摘要 | 運算邏輯摘要 | 程式碼位置 |
|---|---|---|---|---|

## 3. 欄位詳細分析
### {控件名稱}（{控件類型}）
**資料鏈**：{來源} → {轉換} → {顯示}
**詳細追蹤**：
1. **顯示端**：{控件}.Text（{file}:{line}）
2. **賦值**：{expression}（{file}:{line}）
3. **運算**：{pseudocode}（{file}:{line}）
4. **資料來源**：{SQL / SP / API}（{file}:{line}）
**計算邏輯**：{formula}
**相依欄位**：{list}

## 4. 按鈕與動作
### {按鈕名稱}（{按鈕文字}）
**事件方法**：{method}（{file}:{line}）
**動作流程**：
1. **[驗證]** {description}（{file}:{line}）
2. **[收集]** {description}（{file}:{line}）
3. **[運算]** {description}（{file}:{line}）
4. **[存取]** {description}（{file}:{line}）
5. **[更新]** {description}（{file}:{line}）
**商業邏輯**：{pseudocode}
**資料去向**：{destination}（{INSERT/UPDATE/DELETE}）
**呼叫鏈**：
{Class1}.{Method1}（{file1}:{line1}）
├── {Class2}.{Method2}（{file2}:{line2}）
│   └── [SQL] {query}
└── {Class3}.{Method3}（{file3}:{line3}）
**錯誤處理**：{types} → {handling}
**交易控制**：{scope}

## 5. 事件流
### Form_Load
{initialization logic with code references}
### 控件連動
| 觸發控件 | 觸發事件 | 影響控件 | 影響動作 | 程式碼位置 |
|---|---|---|---|---|
### Timer / 背景任務
{timer logic, or "無"}

## 6. 相依性
- 共用模組/類別：{list with paths}
- 外部服務：{list}
- 存取的資料表：{list}

## Mermaid 圖表
### 欄位資料流
```mermaid
flowchart LR
    ...
```
### 按鈕動作流程
#### {按鈕名稱}
```mermaid
flowchart TD
    ...
```
```

## 更新進度

更新 `memory-bank/progress.md`：
- 在 Phase 3 表格中將目標 Form 狀態改為 `✅ 完成`
- 如果有待確認事項，加入「待確認事項」區段
- 如果發現共用模式，加入「分析備註」區段
