---
name: Legacy Java Architect
description: Analyze legacy Java/Java EE project architecture — protocols, upstream/downstream interfaces, data model, per-endpoint business logic tracing, and cross-endpoint workflow assembly. Use this skill when users want to understand a legacy Java project's communication protocols, find upstream/downstream system boundaries, or trace business logic per endpoint. Trigger on phrases like "analyze Java project", "trace endpoint logic", "find protocols", "upstream downstream", "EJB analysis", "system boundary", or any request to reverse-engineer a Java EE application's architecture and data flow
---

# Legacy Java Architect

分析老舊的 Java/Java EE 專案（可能有 10-20 年以上歷史），透過五個階段進行：

1. **協定掃描** — 這個專案對外曝露了哪些通訊協定與端點
2. **上下游偵測** — 這個專案連接了哪些外部系統
3. **資料模型** — Entity 關聯、Table 結構、ER 圖
4. **端點追蹤** — 逐一追蹤每個端點的完整商業邏輯、計算邏輯、資料流向
5. **流程組裝** — 跨端點業務流程重建

## 工作流程

```
Phase 1 ──→ Phase 2 ──→ Phase 3 ──→ Phase 4 ──────→ Phase 5
協定掃描     上下游偵測   資料模型     端點追蹤        流程組裝
(1 次)      (1 次)      (1 次)      (每個端點 1 次)  (1 次)

讀取：       讀取：       讀取：      讀取：           讀取：
  (無前置)     01-own-      01-own-    01-own-          all endpoints/
              protocols    protocols  protocols        shared/
                           02-upstream 02-upstream     03-data-model
                           -downstream 03-data-model

產出：       產出：       產出：      產出：           產出：
  01-own-     02-upstream  03-data-   shared/          workflows/
  protocols   -downstream  model      endpoints/       05-workflow-
              02-system-              {METHOD}-         summary
              overview                {path}.md
```

### Phase 1：協定掃描

讀取 `references/phase1-protocol-scan.md` 取得完整操作步驟。

掃描專案對外曝露的 9 種 inbound 協定，以及全域專案設定：
- REST API（Spring MVC + JAX-RS）
- SOAP Web Service（@WebService）
- Servlet / Filter（web.xml + @WebServlet）
- EJB Session Bean（@Stateless/@Stateful/@Singleton + @Remote/@Local，含 EJB 2.x）
- Message-Driven Bean / JMS Consumer（@MessageDriven、@JmsListener）
- 全域設定：建構依賴（pom.xml）、安全設定、Spring XML bean、App Server 設定
- WebSocket（@ServerEndpoint）
- 排程任務（@Scheduled、@Schedule、Quartz）
- RMI
- gRPC

EJB 多協定曝露：一個 @Stateless bean 可能同時帶有 @WebService 和 @Path — 視為一個服務單元，記錄多種曝露方式。

### Phase 2：上下游偵測

讀取 `references/phase2-upstream-downstream.md` 取得完整操作步驟。

前置：Phase 1 完成。掃描 9 種 outbound 連線：
- 資料庫（JDBC、JNDI DataSource）
- REST Client（RestTemplate、WebClient、HttpClient、Feign）
- SOAP Client（生成的 stub、wsimport）
- Message Queue 生產者（JmsTemplate、KafkaTemplate、RabbitTemplate）
- 快取（Redis、EhCache、Memcached）
- LDAP
- Email（SMTP）
- 檔案 / FTP / SFTP
- 其他（Elasticsearch、MongoDB、Socket）

產出 Mermaid 系統邊界圖。

### Phase 3：資料模型

讀取 `references/phase3-data-model.md` 取得完整操作步驟。

前置：Phase 1 與 Phase 2 完成。建立完整的資料模型：
- JPA/Hibernate @Entity 掃描與欄位結構
- 關聯映射（@ManyToOne、@OneToMany、@ManyToMany、@JoinColumn）
- EJB 2.x CMP Entity Bean 與 CMR 欄位
- Hibernate XML 映射（.hbm.xml）
- SQL 探索（沒有 Entity 映射但被存取的 Table）
- DTO/VO/Request/Response 結構掃描與 Entity ↔ DTO 映射
- 交叉比對 Entity ↔ Table ↔ SQL ↔ DTO

產出 Mermaid ER 圖。

### Phase 4：端點追蹤

讀取 `references/phase4-endpoint-trace.md` 取得完整操作步驟。

前置：Phase 1、Phase 2 與 Phase 3 完成。對 Phase 1 找到的每個端點，追蹤 9 個維度：

1. **入口方法** — 方法簽名、參數、回傳型別
2. **呼叫鏈** — Controller/EJB → Service → DAO，最深追蹤 10 層；含事件/非同步分支
3. **資料存取** — SQL（JPA/Hibernate/MyBatis/JDBC）、Stored Procedure、存取的 Table；交叉比對 Phase 3 ER 模型
4. **外部呼叫** — 對下游系統的 REST/SOAP/MQ 呼叫
5. **商業邏輯** — 條件分支、計算公式、驗證規則、資料轉換、設定驅動邏輯、規則引擎
6. **錯誤處理** — try-catch 範圍、Exception 類型、全域 handler
7. **Interceptor / AOP** — EJB @AroundInvoke 鏈、Spring @Aspect/@Around/@Before/@After
8. **安全控制** — @RolesAllowed、@PreAuthorize、程式碼內角色檢查
9. **交易控制** — Spring @Transactional、EJB CMT（@TransactionAttribute）、BMT（UserTransaction）、跨 EJB 交易傳播鏈

注入物件解析涵蓋：@Autowired、@Inject、@Resource（Spring）、@EJB（EJB 3.x）、JNDI lookup / ServiceLocator（EJB 2.x）、XML wiring。

**共用服務**：被注入超過 3 次的 class 獨立分析於 `shared/`，端點追蹤中引用而非重複展開。

**分析節奏**：每次 session 分析 3-5 個端點，每個端點獨立一個檔案。

### Phase 5：流程組裝

讀取 `references/phase5-workflow-assembly.md` 取得完整操作步驟。

前置：Phase 4 大致完成（核心端點已分析）。組裝跨端點的業務流程：
- MQ 生產者 ↔ 消費者配對（依 Queue/Topic 名稱）
- Event 發布者 ↔ 監聽者配對（依 Event class 型別）
- 系統內部 REST/SOAP/EJB 呼叫串接
- DB 寫入 → 讀取間接串接（同 Table 跨端點）
- Timer/排程任務整合進流程鏈
- 孤立端點識別

產出每個業務流程的 Mermaid sequence diagram。

## Memory Bank（跨對話記憶體）

所有結果寫入 `analysis-memory/`，支援跨 session 恢復：

```
analysis-memory/
├── progress.md                  ← 每次 session 開始先讀這個
├── endpoint-checklist.md        ← Phase 1 產生，Phase 4 每完成一個端點就更新
├── 01-own-protocols.md          ← Phase 1 產出
├── 02-upstream-downstream.md    ← Phase 2 產出
├── 02-system-overview.md        ← Phase 2 Mermaid 系統圖
├── 03-data-model.md             ← Phase 3 產出（Entity + ER 圖）
├── shared/                      ← Phase 4 前置步驟（共用服務）
│   ├── AuditService.md
│   └── ...
├── endpoints/                   ← Phase 4 產出（每個端點一個檔案）
│   ├── GET-api-users.md
│   ├── EJB-OrderService-createOrder.md
│   ├── SOAP-getAccount.md
│   └── ...
├── workflows/                   ← Phase 5 產出（每個流程一個檔案）
│   ├── order-creation.md
│   └── ...
└── 05-workflow-summary.md       ← Phase 5 總覽 + 孤立端點
```

**規則**：
- Session 開始時：讀取 `progress.md`。若存在，從上次未完成的 Phase 繼續。若不存在，建立它並從 Phase 1 開始。
- 每個 Phase / 端點 / 流程完成後：將結果寫入對應檔案，並更新 `progress.md`。
- 使用者說「繼續」時：讀取 `progress.md` 判斷目前階段，再按以下行為執行。

**「繼續」行為（依階段）**：
- Phase 1-3、5：自動恢復未完成的階段。
- Phase 4（端點追蹤）：讀取 `endpoint-checklist.md`，向使用者顯示完成統計與下一批待分析端點，詢問使用者要分析哪個端點。不自動選擇 — 讓使用者挑選或確認。若使用者再次說「繼續」而未指定，按優先順序（P0 > P1 > P2 > P3）分析下一個端點。

## 使用者指令

| 使用者說 | 動作 |
|---|---|
| `開始分析` / `start` | 開始 Phase 1 — 讀取 `references/phase1-protocol-scan.md` 並執行 |
| `執行 Phase N` | 讀取對應 Phase 的 reference 並執行 |
| `分析 {endpoint}` | 讀取 `references/phase4-endpoint-trace.md`，追蹤指定的端點。在 checklist 中標記為 P0 |
| `繼續` / `continue` | 讀取 `progress.md`。若在 Phase 4：顯示 `endpoint-checklist.md` 摘要，詢問使用者要分析哪個。否則：恢復下一個未完成的階段 |
| `目前進度` / `status` | 讀取並摘要 `progress.md`。若在 Phase 4：同時顯示 `endpoint-checklist.md` 完成統計 |
| `產出總覽` | 根據已收集的資料，產出系統架構總覽 |

## 輸出語言

偵測使用者的對話語言，所有報告使用該語言撰寫。技術術語保留英文，首次出現時附上中文說明。

## 品質規則

- 所有結論附上原始碼位置引用（檔案路徑:行號）
- 不得猜測 — 所有分析基於實際程式碼
- 無法確認的項目標記為「⚠ 待確認」，不得跳過
- 未偵測到的協定記錄為「未偵測到」，不得虛構
- 搭配 Mermaid 圖表：`graph TD` 用於系統總覽、`erDiagram` 用於資料模型、`flowchart TD` 用於呼叫鏈、`flowchart LR` 用於資料流向、`sequenceDiagram` 用於業務流程
