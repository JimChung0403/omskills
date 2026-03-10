---
name: Field Data Trace
description: Traces a form field's complete data lineage from source through transformations to display
---

# Field Data Trace

## 描述

追蹤 WinForms 表單中單一欄位的完整資料鏈：資料從哪裡來、經過什麼運算或轉換、最終如何顯示在畫面上。

## 使用者

- Form Analyzer

## 執行步驟

### Step 1：識別欄位的資料綁定方式

讀取 Form 的 code-behind（`.vb`）和 Designer（`.Designer.vb`），搜尋目標欄位的資料來源：

**方式 A：DataBinding**

搜尋 `{控件名稱}.DataBindings.Add` 或在 Designer 中的 `DataBindings` 設定。記錄：

- 綁定的屬性（Text、Value 等）
- 資料來源物件（BindingSource、DataSet、DataTable）
- 綁定的欄位名稱

**方式 B：手動賦值**

搜尋 `{控件名稱}.Text =`、`{控件名稱}.Value =` 等賦值語句。記錄：

- 賦值的時機（在哪個方法中）
- 賦值的來源表達式

**方式 C：DataSource 綁定**

搜尋 `{控件名稱}.DataSource =`（常見於 DataGridView、ComboBox）。記錄：

- DataSource 物件
- DisplayMember 和 ValueMember 設定

### Step 2：追蹤資料來源

根據 Step 1 找到的來源，向上追蹤：

1. 如果來源是 **BindingSource/DataSet**：搜尋 `{BindingSource}.DataSource =` 或 `{DataAdapter}.Fill(`，找到 SQL 查詢或 Stored Procedure 名稱，記錄查詢參數的來源
2. 如果來源是 **方法回傳值**：讀取該方法的實作，繼續向上追蹤方法內的資料來源，使用 `call-chain-analysis` skill 追蹤跨檔案呼叫
3. 如果來源是 **其他控件**：記錄來源控件，再對來源控件重複 Step 1
4. 如果來源是 **Form 參數**：搜尋 Form 的建構函式或公開屬性，追蹤呼叫端傳入的值

### Step 3：記錄中間運算

追蹤過程中遇到的所有運算與轉換：

- 算術運算（加減乘除、百分比）
- 型別轉換（CInt、CDbl、CStr、ToString）
- 字串處理（Format、Substring、Replace）
- 條件邏輯（If-Else 決定顯示值）
- 自訂函數運算

將每個運算以虛擬碼或公式記錄。

## 輸出格式

```markdown
## 欄位：{控件名稱}（{控件類型}）

### 資料鏈
{來源} → {轉換1} → {轉換2} → {顯示}

### 詳細追蹤
1. **顯示端**：{控件名稱}.Text（{Form}.vb:{line}）
2. **賦值**：{expression}（{file}:{line}）
3. **運算**：{pseudocode}（{file}:{line}）
4. **資料來源**：{SQL query / SP name / API call}（{file}:{line}）

### 計算邏輯
{formula or pseudocode with explanation}

### 相依欄位
- {other fields this field depends on}
```

## 範例

**輸入**：追蹤 `frmOrder` 中 `lblTotal` 的資料來源

**輸出**：

```markdown
## 欄位：lblTotal（Label）

### 資料鏈
dgvOrderItems → CalculateTotal() → FormatCurrency → lblTotal.Text

### 詳細追蹤
1. **顯示端**：lblTotal.Text = FormatCurrency(total)（frmOrder.vb:234）
2. **運算**：total = CalculateTotal()（frmOrder.vb:230）
3. **函數實作**：遍歷 dgvOrderItems 每一行，累加 UnitPrice * Quantity（frmOrder.vb:340-355）
4. **資料來源**：dgvOrderItems 的資料來自 SQL 查詢 "SELECT * FROM OrderItems WHERE OrderID = @id"（DAL/OrderDAL.vb:78）

### 計算邏輯
total = SUM(每行的 UnitPrice × Quantity)
lblTotal.Text = Format(total, "#,##0")

### 相依欄位
- dgvOrderItems 的 UnitPrice 欄位
- dgvOrderItems 的 Quantity 欄位
```
