# Legacy VB.NET Analyzer

## 部署模式

單一 Agent 模式。不使用多 Agent 架構。透過單一分析師 Agent 執行所有分析任務，以 skill 系統組織分析步驟，以 Memory Bank 實現跨對話狀態保存。

模型：gpt-oss

## Memory Bank 機制

`memory-bank/` 資料夾是跨對話的記憶體。結構如下：

```
memory-bank/
├── progress.md                  ← 進度追蹤（核心，每次必讀）
├── 01-project-structure.md      ← Phase 1a 產出
├── 02-protocols.md              ← Phase 1b 產出
├── 03-form-inventory.md         ← Phase 2 產出
└── forms/                       ← Phase 3 產出（每個 Form 一個檔案）
    ├── frmMain.md
    ├── frmOrder.md
    └── ...
```

**規則**：
- 每次對話開始，必須先讀取 `progress.md`
- 每個 skill 執行完成後，必須更新 `progress.md`
- 所有分析結果必須寫入 `memory-bank/`，不得只存在對話上下文中

## 通用行為規範

- 使用繁體中文撰寫所有分析報告
- 技術術語保留英文原文，首次出現時附上中文說明
- 所有分析結論必須附上原始碼位置引用（檔案路徑:行號）
- 不得猜測邏輯，所有分析必須基於實際程式碼
- 遇到無法確認的部分，明確標記為「⚠ 待確認」而非跳過

## 分析深度標準

### 欄位分析必須達到的深度

1. 資料綁定層：識別綁定方式（DataBinding / 手動賦值 / DataSource）
2. 資料來源層：追蹤到具體來源（SQL、Stored Procedure、API、使用者輸入）
3. 運算層：記錄所有運算與轉換，以虛擬碼或公式表示
4. 參數層：追蹤 SQL 參數、方法參數的來源

每一層必須附上程式碼引用（檔案路徑:行號）。

### 按鈕分析必須包含

1. 完整動作流程：從 Click 事件到最終操作的每一步
2. 所有方法呼叫：追蹤到最終的資料存取操作（SQL / 檔案 / API）
3. 條件分支：記錄所有 If-Else 和 Select Case
4. 錯誤處理：記錄 Try-Catch 範圍和處理方式
5. 交易控制：記錄 Transaction 邊界

### 呼叫鏈追蹤

- 最大追蹤深度：10 層
- 超過 10 層標記為「追蹤截斷」
- .NET Framework 內建方法不需追蹤內部實作
- 第三方元件記錄呼叫方式和參數，不追蹤內部

### 不可省略的項目

- Form_Load 事件中的初始化邏輯
- Timer 事件的處理邏輯
- 控件間的連動關係
- 隱藏控件（Visible = False）的邏輯
- Designer 中設定的預設值

## Mermaid 圖表

在以下情境產出 Mermaid 圖表（用 ```mermaid 區塊包裹）：

- 系統架構：`graph TD`（protocol-detection 產出）
- Form 呼叫關係：`graph TD`（form-inventory 產出）
- 欄位資料流：`flowchart LR`（form-analysis 產出）
- 按鈕動作流程：`flowchart TD`（form-analysis 產出）

## 報告格式標準

### 表單分析報告 6 個必要章節

1. 表單概述（檔案位置、用途、關聯表單）
2. 欄位清單與資料來源（表格格式）
3. 欄位詳細分析（資料鏈、計算邏輯、程式碼引用）
4. 按鈕與動作（動作流程、商業邏輯、資料去向、錯誤處理）
5. 事件流（Form_Load、控件連動、Timer）
6. 相依性（共用模組、外部服務、資料表）

### 系統架構報告 6 個必要章節

1. 專案基本資訊
2. 專案結構
3. 通訊協定與資料存取
4. Form 清單與關係
5. 共用元件
6. 第三方相依性
