---
name: Analysis Depth Standard
description: Defines minimum analysis depth requirements for legacy VB.NET system analysis
---

# Analysis Depth Standard

## 適用範圍

- 適用於：`architecture-scout`、`form-analyzer`、`analysis-reviewer`

## 規則內容

### 欄位分析深度

每個資料欄位的分析必須達到以下深度：

1. **資料綁定層**：識別欄位的綁定方式（DataBinding / 手動賦值 / DataSource）
2. **資料來源層**：追蹤到具體的資料來源（SQL 查詢、Stored Procedure、API 回傳值、使用者輸入）
3. **運算層**：記錄資料從來源到顯示之間的所有運算與轉換，以虛擬碼或公式表示
4. **參數層**：如果資料來源有參數（如 SQL 的 WHERE 條件），追蹤參數值的來源

分析結果中，每一層都必須附上程式碼引用（檔案路徑:行號）。

### 按鈕分析深度

每個按鈕的分析必須包含：

1. **完整動作流程**：從 Click 事件到最終操作的每一步
2. **所有方法呼叫**：追蹤到最終的資料存取操作（SQL / 檔案 / API）
3. **條件分支**：記錄所有 If-Else 和 Select Case 的分支邏輯
4. **錯誤處理**：記錄 Try-Catch 的範圍和處理方式
5. **交易控制**：記錄 Transaction 的邊界（Begin / Commit / Rollback）

### 呼叫鏈追蹤深度

- 最大追蹤深度：10 層
- 超過 10 層時標記為「追蹤截斷」，並記錄截斷原因
- .NET Framework 內建方法（System.* 命名空間）不需追蹤內部實作
- 第三方元件的方法記錄其呼叫方式和參數，不追蹤內部實作

### 不可省略的項目

以下項目在分析中不得省略：

- Form_Load 事件中的初始化邏輯
- Timer 事件的處理邏輯
- 控件間的連動關係（例如 ComboBox.SelectedIndexChanged 觸發其他控件更新）
- 隱藏控件（Visible = False）的邏輯
- 在 Designer 中設定的預設值

## 違反判定

- 欄位分析缺少資料來源層（只記錄了綁定方式但沒追蹤到具體 SQL 或 API）→ 違反
- 按鈕分析缺少動作流程（只列出事件名稱但沒記錄具體步驟）→ 違反
- 分析結論沒有附上程式碼引用（檔案:行號）→ 違反
- 呼叫鏈超過 10 層但未標記截斷 → 違反
- 省略了 Form_Load 中的初始化邏輯 → 違反

## 例外

- 純 UI 排版控件（如 Panel 僅作為容器、Separator 分隔線）不需要追蹤資料邏輯
