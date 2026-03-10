---
name: Call Chain Analysis
description: Traces method call chains across multiple files and classes in VB.NET projects
---

# Call Chain Analysis

## 描述

追蹤 VB.NET 專案中跨檔案、跨類別的方法呼叫鏈。當一個方法呼叫了其他類別或模組中的方法時，持續追蹤直到找到最終的資料來源或操作。

## 使用者

- Form Analyzer

## 執行步驟

### Step 1：識別方法呼叫

在目標方法中找到所有外部方法呼叫：

- 搜尋 `{ClassName}.{MethodName}(` 格式的呼叫
- 搜尋模組方法呼叫 `{ModuleName}.{MethodName}(`
- 搜尋實例方法呼叫 `{variable}.{MethodName}(`
- 識別 Shared（靜態）方法呼叫

### Step 2：定位方法定義

對每個被呼叫的方法，找到其定義位置：

1. 使用 Grep 搜尋 `Sub {MethodName}` 或 `Function {MethodName}`
2. 確認所屬類別與檔案
3. 讀取方法的完整程式碼

### Step 3：遞迴追蹤

對找到的方法重複 Step 1-2，直到滿足以下任一終止條件：

- 到達資料庫操作（SQL 語句、Stored Procedure 呼叫）
- 到達檔案 I/O 操作
- 到達外部 API / Web Service 呼叫
- 到達 .NET Framework 內建方法（無需繼續追蹤）
- 追蹤深度超過 10 層（標記為「追蹤截斷」並記錄原因）

### Step 4：記錄呼叫鏈

以樹狀結構記錄完整的呼叫鏈，每個節點包含：

- 方法全名（ClassName.MethodName）
- 檔案位置（file:line）
- 參數列表
- 回傳值類型
- 該方法的核心邏輯摘要（一句話）

## 特殊處理

### 事件驅動呼叫

WinForms 常見的間接呼叫模式：

- `RaiseEvent` → 搜尋對應的 `Handles` 或 `AddHandler`
- `Delegate.Invoke` → 搜尋 delegate 的賦值位置
- `BackgroundWorker.RunWorkerAsync` → 追蹤 `DoWork` 和 `RunWorkerCompleted` 事件

### 介面呼叫

如果呼叫的是介面方法：

- 搜尋 `Implements {InterfaceName}` 找到實作類別
- 確認執行時期使用的具體實作

### 多載方法

搜尋方法定義時注意多載（Overloads）：

- 根據參數數量和型別匹配正確的多載版本

## 輸出格式

```markdown
## 呼叫鏈：{起始方法名}

### 呼叫樹
{ClassName1}.{Method1}（{file1}:{line1}）
├── {ClassName2}.{Method2}（{file2}:{line2}）
│   ├── {ClassName3}.{Method3}（{file3}:{line3}）
│   │   └── [SQL] SELECT * FROM Table WHERE ...
│   └── {ClassName4}.{Method4}（{file4}:{line4}）
│       └── [API] POST https://...
└── {ClassName5}.{Method5}（{file5}:{line5}）
    └── [FILE] Write to log.txt

### 各節點摘要
| # | 方法 | 位置 | 參數 | 回傳 | 核心邏輯 |
|---|---|---|---|---|---|
```

## 範例

**輸入**：追蹤 `frmOrder.btnSave_Click` 中呼叫的 `OrderService.SaveOrder()`

**輸出**：

```markdown
## 呼叫鏈：OrderService.SaveOrder

### 呼叫樹
OrderService.SaveOrder(orderData)（Services/OrderService.vb:30）
├── OrderValidator.Validate(orderData)（Validators/OrderValidator.vb:15）
│   ├── OrderValidator.CheckStock(items)（Validators/OrderValidator.vb:45）
│   │   └── [SQL] SELECT Stock FROM Products WHERE ProductID = @id
│   └── OrderValidator.CheckCredit(customerId)（Validators/OrderValidator.vb:60）
│       └── [SQL] SELECT CreditLimit, Balance FROM Customers WHERE ID = @id
├── OrderDAL.InsertOrder(order)（DAL/OrderDAL.vb:50）
│   └── [SQL] INSERT INTO Orders (OrderNo, CustomerID, ...) VALUES (...)
├── OrderDAL.InsertOrderItems(items)（DAL/OrderDAL.vb:70）
│   └── [SQL] INSERT INTO OrderItems (OrderID, ProductID, ...) VALUES (...)
└── StockService.UpdateStock(items)（Services/StockService.vb:25）
    └── [SQL] UPDATE Products SET Stock = Stock - @qty WHERE ProductID = @id

### 各節點摘要
| # | 方法 | 位置 | 核心邏輯 |
|---|---|---|---|
| 1 | OrderService.SaveOrder | Services/OrderService.vb:30 | 協調訂單儲存：驗證 → 寫入 → 更新庫存 |
| 2 | OrderValidator.Validate | Validators/OrderValidator.vb:15 | 驗證訂單合法性（庫存、信用額度） |
| 3 | OrderDAL.InsertOrder | DAL/OrderDAL.vb:50 | 寫入訂單表頭至 Orders 表 |
| 4 | StockService.UpdateStock | Services/StockService.vb:25 | 扣減產品庫存數量 |
```
