---
name: Button Logic Trace
description: Traces a button's complete action chain from click event through business logic to data destination
---

# Button Logic Trace

## 描述

追蹤 WinForms 表單中單一按鈕被點擊後的完整動作鏈：事件處理 → 資料驗證 → 商業邏輯 → 資料存取 → 結果回饋。

## 使用者

- Form Analyzer

## 執行步驟

### Step 1：找到事件處理方法

搜尋按鈕的 Click 事件處理方法：

- 在 Designer 中搜尋 `AddHandler {按鈕名稱}.Click` 或 `Handles {按鈕名稱}.Click`
- 在 code-behind 中搜尋 `Private Sub {按鈕名稱}_Click`
- 讀取事件處理方法的完整程式碼

### Step 2：分析動作流程

逐行分析事件處理方法，識別以下動作類型：

**資料驗證**

- 輸入值檢查（空值、格式、範圍）
- 商業規則驗證（例如庫存是否足夠）
- 記錄驗證失敗時的處理（MessageBox、ErrorProvider）

**資料收集**

- 從控件讀取使用者輸入的值
- 從其他資料來源取得輔助資料
- 記錄每個資料項的來源控件和轉換方式

**商業邏輯運算**

- 計算公式（金額、數量、比率等）
- 條件分支（不同情境的處理邏輯）
- 迴圈處理（批次運算）
- 記錄完整的運算邏輯（以虛擬碼表示）

**資料存取**

- 資料庫寫入（INSERT、UPDATE、DELETE）
- 檔案操作（讀寫檔案）
- 外部服務呼叫（API、Web Service）
- 記錄 SQL 語句或方法呼叫

**UI 更新**

- 更新畫面控件的顯示
- 切換 Form（開啟/關閉）
- 顯示訊息（MessageBox、StatusBar）

### Step 3：追蹤跨方法呼叫

如果事件處理方法呼叫了其他方法：

- 讀取被呼叫方法的完整程式碼
- 遞迴分析該方法的動作流程
- 使用 `call-chain-analysis` skill 追蹤跨檔案呼叫
- 標記遞迴追蹤的深度

### Step 4：識別錯誤處理

- 記錄 Try-Catch 區塊的範圍和捕獲的例外類型
- 記錄錯誤發生時的處理方式（記錄日誌、顯示訊息、回滾交易）
- 識別 Transaction 的使用（BeginTransaction、Commit、Rollback）

## 輸出格式

```markdown
## 按鈕：{按鈕名稱}（{按鈕文字}）

### 事件處理方法
- 方法名稱：{method name}
- 位置：{file}:{line}

### 動作流程
1. **[驗證]** {description}（{file}:{line}）
2. **[收集]** {description}（{file}:{line}）
3. **[運算]** {description}（{file}:{line}）
4. **[存取]** {description}（{file}:{line}）
5. **[更新]** {description}（{file}:{line}）

### 商業邏輯
{pseudocode or formula}

### 資料去向
- 目的地：{database table / file / API endpoint}
- 操作：{INSERT / UPDATE / DELETE / POST}
- SQL/指令：{actual SQL or method call}

### 錯誤處理
- 例外類型：{exception types caught}
- 處理方式：{error handling description}
- 交易控制：{transaction scope if any}

### 影響範圍
- 修改的資料表：{table names}
- 觸發的 UI 更新：{controls updated}
- 開啟的 Form：{forms opened}
```

## 範例

**輸入**：追蹤 `frmOrder` 中 `btnSave` 的動作鏈

**輸出**：

```markdown
## 按鈕：btnSave（儲存）

### 事件處理方法
- 方法名稱：btnSave_Click
- 位置：frmOrder.vb:150

### 動作流程
1. **[驗證]** 檢查 txtOrderNo 不為空（frmOrder.vb:152）
2. **[驗證]** 檢查 cboCustomer 已選擇（frmOrder.vb:155）
3. **[驗證]** 檢查 dgvOrderItems 至少一筆資料（frmOrder.vb:158）
4. **[收集]** 從控件收集訂單表頭資料（frmOrder.vb:162-170）
5. **[運算]** 計算訂單總金額 = SUM(數量 × 單價)（frmOrder.vb:172）
6. **[運算]** 計算稅額 = 總金額 × 0.05（frmOrder.vb:173）
7. **[存取]** 呼叫 OrderDAL.SaveOrder(orderData)（frmOrder.vb:176）
   - 內部：BEGIN TRAN → INSERT Orders → INSERT OrderItems → COMMIT（DAL/OrderDAL.vb:45-80）
8. **[更新]** 顯示 "儲存成功" MessageBox（frmOrder.vb:180）
9. **[更新]** 清空表單控件（frmOrder.vb:182）

### 商業邏輯
totalAmount = SUM(每行: quantity × unitPrice)
taxAmount = totalAmount × 0.05
grandTotal = totalAmount + taxAmount

### 資料去向
- 目的地：SQL Server — Orders 表、OrderItems 表
- 操作：INSERT（新增）/ UPDATE（修改，依據 OrderID 判斷）
- SQL：見 DAL/OrderDAL.vb:50-75

### 錯誤處理
- 例外類型：SqlException, Exception
- 處理方式：Rollback Transaction + MessageBox 顯示錯誤訊息
- 交易控制：SqlTransaction（整筆訂單原子操作）
```
