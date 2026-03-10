---
name: Form Inventory
description: Discovers and catalogs all WinForms forms with their controls and inter-form relationships
---

# Form Inventory

## 描述

盤點 VB.NET WinForms 專案中的所有 Form，列出每個 Form 的控件清單，以及 Form 之間的呼叫關係。

## 使用者

- Architecture Scout

## 執行步驟

### Step 1：收集所有 Form

從 Project Scan 的結果中取得所有 Form 檔案清單。如果沒有現成清單，使用 Grep 搜尋：

- `Inherits System.Windows.Forms.Form`
- `Inherits Form`

### Step 2：分析每個 Form 的控件

讀取每個 Form 的 `.Designer.vb` 檔案，提取所有控件宣告。搜尋 `Friend WithEvents` 或 `Private WithEvents` 宣告，記錄控件名稱與類型。

將控件分類為：

| 分類 | 控件類型 |
|---|---|
| 資料輸入 | TextBox, ComboBox, DateTimePicker, NumericUpDown, CheckBox, RadioButton, MaskedTextBox |
| 資料顯示 | Label, DataGridView, ListView, TreeView, RichTextBox |
| 動作觸發 | Button, ToolStripButton, ToolStripMenuItem, MenuStrip |
| 容器 | Panel, GroupBox, TabControl, SplitContainer |
| 其他 | Timer, BackgroundWorker, BindingSource, ErrorProvider |

### Step 3：分析 Form 間呼叫關係

在所有 `.vb` 檔案中搜尋 Form 的實例化和顯示：

- `New {FormName}` — 建立 Form 實例
- `.Show()`, `.ShowDialog()` — 顯示 Form
- 記錄呼叫方（哪個 Form/Module）和被呼叫方

### Step 4：識別共用 UserControl

列出所有 UserControl，並搜尋它們在哪些 Form 的 Designer 中被使用。

## 輸出格式

```markdown
# Form 盤點結果

## 統計摘要
- Form 總數：{count}
- UserControl 總數：{count}
- 控件總數：{count}

## Form 清單

### {FormName}
- 檔案：{file path}
- 推測用途：{inferred purpose from name and controls}
- 控件統計：
  - 資料輸入：{n} 個（{list}）
  - 資料顯示：{n} 個（{list}）
  - 動作觸發：{n} 個（{list}）
- 呼叫的 Form：{list of forms this form opens}
- 被呼叫自：{list of forms that open this form}

## Form 呼叫關係圖
{text-based diagram showing form relationships}

## 共用 UserControl
| UserControl | 使用的 Form |
|---|---|
```

## 範例

**輸入**：盤點專案中所有 Form

**輸出**：

```markdown
# Form 盤點結果

## 統計摘要
- Form 總數：45
- UserControl 總數：12
- 控件總數：387

## Form 清單

### frmMain
- 檔案：Forms/frmMain.vb
- 推測用途：主選單
- 控件統計：
  - 資料輸入：0 個
  - 資料顯示：1 個（lblStatus）
  - 動作觸發：8 個（btnOrder, btnInventory, btnReport, ...）
- 呼叫的 Form：frmOrder, frmInventory, frmReport, frmSettings
- 被呼叫自：（啟動 Form）

### frmOrder
- 檔案：Forms/Order/frmOrder.vb
- 推測用途：訂單管理
- 控件統計：
  - 資料輸入：12 個（txtOrderNo, cboCustomer, dtpOrderDate, ...）
  - 資料顯示：2 個（dgvOrderItems, lblTotal）
  - 動作觸發：5 個（btnSave, btnDelete, btnPrint, btnSearch, btnClose）
- 呼叫的 Form：frmCustomerLookup, frmOrderDetail
- 被呼叫自：frmMain

## 共用 UserControl
| UserControl | 使用的 Form |
|---|---|
| ucAddress | frmCustomer, frmSupplier, frmOrder |
| ucPager | frmOrder, frmInventory, frmReport |
```
