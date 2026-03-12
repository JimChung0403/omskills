# Phase 5：跨端點業務流程組裝

將 Phase 4 各端點的獨立分析串接成完整的業務流程，還原真實的端對端執行路徑。

前置：
- 讀取 `analysis-memory/03-data-model.md`
- 讀取 `analysis-memory/endpoints/` 所有已完成的端點分析
- 讀取 `analysis-memory/shared/` 所有共用服務分析

---

## Step 1：MQ 生產者 ↔ 消費者配對

從所有 endpoint 分析中，提取：
- **生產端**（Phase 4「外部系統呼叫」中的 MQ 發送）：Queue/Topic 名稱、消息內容
- **消費端**（Phase 1 中的 JMS Consumer / MDB）：Queue/Topic 名稱、Handler

**配對規則**：Queue/Topic 名稱完全匹配 → 建立連線。

```
| 生產者 Endpoint | Queue/Topic | 消費者 Endpoint | 消息內容 |
|---|---|---|---|
```

---

## Step 2：Event 發布者 ↔ 監聽者配對

從 Phase 4 中標記的事件連線，提取：
- `ApplicationEventPublisher.publishEvent(XxxEvent)` → 發布者
- `@EventListener` / `@TransactionalEventListener` 處理 `XxxEvent` → 監聽者
- CDI `Event.fire()` → `@Observes` 配對

**配對規則**：Event class 型別匹配。

```
| 發布者 | Event Class | 監聯者 | 同步/非同步 |
|---|---|---|---|
```

---

## Step 3：同系統內部呼叫串接

從 endpoint 分析的「外部系統呼叫」中，找出目標 URL 指向本系統自身的呼叫：

- REST 呼叫的 URL path 匹配 Phase 1 的 REST endpoint
- SOAP 呼叫的 operation 匹配 Phase 1 的 SOAP endpoint
- EJB 呼叫的 JNDI name 匹配 Phase 1 的 EJB bean

```
| 呼叫者 Endpoint | 呼叫方式 | 被呼叫 Endpoint |
|---|---|---|
```

---

## Step 4：資料流串接（DB 寫入 → 讀取）

從 Phase 4 的資料存取分析中：
- Endpoint A 對 Table X 執行 INSERT/UPDATE
- Endpoint B 對 Table X 執行 SELECT

交叉比對 `analysis-memory/03-data-model.md` 中的 Table 清單，找出透過資料庫間接串接的端點。

**篩選**：只記錄業務上有意義的串接（同一 Table 的寫→讀），忽略共用 config table。

```
| 寫入 Endpoint | Table | 操作 | 讀取 Endpoint | 操作 |
|---|---|---|---|---|
```

---

## Step 5：Timer/排程觸發串接

Phase 1 中的排程任務（@Scheduled、@Schedule、Quartz）可能：
- 讀取某個 endpoint 寫入的資料，執行後續處理
- 呼叫內部或外部 API
- 發送 MQ 消息觸發下游

將排程任務納入業務流程鏈中。

---

## Step 6：組裝業務流程

根據 Step 1-5 的配對結果，組裝完整的業務流程：

**組裝規則**：
1. 從一個「觸發入口」開始（REST/SOAP/Servlet/外部 MQ/Timer）
2. 跟隨所有連線（MQ、Event、內部呼叫、DB 間接串接）
3. 直到沒有後續連線（終端操作：回應 caller、寫入 DB、發送通知）
4. 一條完整路徑 = 一個業務流程

**命名規則**：以業務動作命名（如 `order-creation`、`payment-processing`、`user-registration`）。

---

## Step 7：識別孤立端點

列出未被任何業務流程覆蓋的端點：
- 沒有上游觸發、沒有下游連線
- 可能是：工具 API、管理接口、遺留未使用的端點

標記為「⚠ 孤立端點，待確認用途」。

---

## 輸出格式

每個業務流程寫入 `analysis-memory/workflows/{workflow-name}.md`：

```markdown
# 業務流程：{流程名稱}

## 流程摘要

{一段話描述此業務流程的目的與觸發條件}

## 涉及端點

| # | Endpoint | 角色 | 協定 |
|---|---|---|---|
| 1 | POST /api/orders | 觸發入口 | REST |
| 2 | MDB: queue/order-process | 非同步處理 | JMS |
| 3 | Timer: checkOrderTimeout | 定時檢查 | Scheduler |

## 流程圖（Mermaid sequenceDiagram）

sequenceDiagram
    participant Client
    participant OrderAPI as POST /api/orders
    participant MQ as JMS Queue
    participant MDB as OrderProcessor MDB
    participant DB as Database
    participant ExtAPI as Payment API

    Client->>OrderAPI: 建立訂單
    OrderAPI->>DB: INSERT INTO orders
    OrderAPI->>MQ: send(OrderCreatedMsg)
    MQ->>MDB: onMessage()
    MDB->>ExtAPI: POST /payment
    ExtAPI-->>MDB: 200 OK
    MDB->>DB: UPDATE orders SET status='PAID'

## 資料流向（Mermaid flowchart LR）

flowchart LR
    A[Client Request] --> B[OrderAPI]
    B --> C[(orders table)]
    B --> D[JMS Queue]
    D --> E[MDB Processor]
    E --> F[Payment API]
    E --> C

## 共用服務引用

| 共用服務 | 使用位置 | 用途 |
|---|---|---|

## 交易邊界

{描述此流程中的交易邊界，哪些操作在同一交易內，哪些跨交易}

## 備註

- ⚠ {待確認項目}
```

同時更新 `analysis-memory/05-workflow-summary.md`，包含：

```markdown
# Phase 5：業務流程總覽

## 流程清單

| # | 流程名稱 | 觸發方式 | 涉及端點數 | 涉及 Table |
|---|---|---|---|---|

## 全域流程圖（Mermaid graph TD）

graph TD
    subgraph 訂單流程
        A[POST /orders] -->|JMS| B[OrderProcessor MDB]
        B -->|Timer| C[checkOrderTimeout]
    end
    subgraph 付款流程
        D[POST /payments] -->|Event| E[PaymentListener]
    end
    A -->|DB| D

## 孤立端點

| # | Endpoint | 協定 | 可能原因 |
|---|---|---|---|
```
