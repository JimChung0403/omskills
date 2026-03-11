# Phase 4：Endpoint 完整邏輯追蹤

針對 Phase 1 找到的每一個接口端點，追蹤完整的商業邏輯、計算邏輯、和資料流向。

前置：
- 讀取 `analysis-memory/01-own-protocols.md`
- 讀取 `analysis-memory/02-upstream-downstream.md`
- 讀取 `analysis-memory/03-data-model.md`
- 檢查 `analysis-memory/endpoints/` 已完成的檔案，避免重複
- 檢查 `analysis-memory/shared/` 已完成的共用服務，追蹤時引用而非重複展開

---

## Pre-Step：共用服務識別（首次執行 Phase 4 時執行一次）

掃描哪些 service class 被廣泛注入（3 次以上），這些是共用服務：

```
Grep: @Autowired                glob: **/*.java   → 統計每個被注入型別的出現次數
Grep: @Inject                   glob: **/*.java
Grep: @EJB                      glob: **/*.java
Grep: @Resource                  glob: **/*.java
```

**判定規則**：同一 class 被注入超過 3 次 → 視為共用服務。

對每個共用服務獨立分析，寫入 `analysis-memory/shared/{ServiceName}.md`：
- class 名稱、檔案位置
- 所有 public 方法摘要
- 關鍵商業邏輯（計算、驗證、轉換）
- 注入的下游依賴

後續 endpoint 追蹤中遇到共用服務呼叫時，引用 `→ 見 shared/{ServiceName}.md` 而非重複展開。

---

## Step 1：定位入口方法

讀取對應的 Controller / Servlet / EJB class，記錄：

- **協定**：REST / SOAP / Servlet / EJB Remote / EJB Local / JMS / ...
- **路徑**：URL path / Queue name / JNDI name
- **HTTP Method**（如適用）
- **方法簽名**
- **參數**：名稱、型別、來源
- **回傳型別**
- **檔案位置**

**EJB 入口特殊處理**：
- 入口方法定義在 Remote/Local **interface**，實作在 **Bean class**
- 同一 Bean 多協定曝露（@Stateless + @WebService + @Path）→ 記錄所有曝露方式，呼叫鏈只追蹤一次
- 有 `@Interceptors` → 在 Step 7 追蹤

---

## Step 2：追蹤呼叫鏈

從入口方法逐層追蹤，最大深度 10 層。

**追蹤規則**：
- 跟隨 `this.xxx()` 和注入物件的 `xxx.method()` 呼叫
- 讀取被呼叫方法完整內容
- 遞迴追蹤直到：資料存取層 / 外部呼叫 / 共用服務（引用 shared/） / 10 層深度 / 框架內部

**注入物件解析**：

| 注入方式 | 解析方法 |
|---|---|
| `@Autowired` / `@Inject` / `@Resource` | 從欄位型別找實作 class；interface → `Grep: implements\s+{Name}` |
| `@EJB` | 從欄位型別找 Bean 實作；檢查 `@EJB(lookup=...)` 或 `@EJB(beanName=...)` |
| JNDI Lookup | `Grep: InitialContext\|lookup\(\|ServiceLocator`；對照 Phase 1 EJB 清單 |
| Service Locator | 找到 ServiceLocator class 讀取 lookup 邏輯 |
| XML Wiring | 檢查 `applicationContext.xml` 或 `ejb-jar.xml` |

多實作 → 記錄所有實作，標記「⚠ 多實作，需確認」。

**事件 / 非同步連線追蹤**：

呼叫鏈中遇到以下模式時，追蹤事件的接收端：

```
Grep: publishEvent|ApplicationEventPublisher  glob: **/*.java
Grep: @EventListener|@TransactionalEventListener  glob: **/*.java
Grep: @Observes                               glob: **/*.java
Grep: @Async                                  glob: **/*.java
Grep: Event\.fire|BeanManager                 glob: **/*.java
```

| 模式 | 追蹤方法 |
|---|---|
| `publisher.publishEvent(XxxEvent)` | 找所有 `@EventListener` 參數型別為 `XxxEvent` 的方法 |
| CDI `event.fire(xxx)` | 找所有 `@Observes` 參數型別匹配的方法 |
| `@Async` 方法呼叫 | 標記為非同步分支，追蹤該方法但標註「非同步執行」 |

記錄為呼叫鏈的「非同步分支」，在 Mermaid 圖中用虛線表示。

**記錄格式**：

```
Layer N: {ClassName}.{methodName}()
  檔案：{path}:{line}
  呼叫了：
    → {NextClass}.{nextMethod}()  (path:line)
    → [共用服務] → 見 shared/{ServiceName}.md
  非同步分支：
    → [Event] XxxEvent → {ListenerClass}.{method}()  (path:line)
    → [Async] {Class}.{asyncMethod}()  (path:line)
  外部呼叫：
    → [REST] POST https://xxx/api  (path:line)
    → [SQL] SELECT * FROM TABLE WHERE id = ?  (path:line)
```

---

## Step 3：資料存取分析

參照 `analysis-memory/03-data-model.md` 中的 Entity/Table 清單，交叉比對。

**JPA / Hibernate**：
- Repository/DAO 方法 → `@Query` 中的 JPQL/SQL
- 方法名稱衍生查詢（`findByUserId`）
- Entity → Table（`@Table(name=...)`）

**MyBatis / iBatis**：
- Mapper interface → 對應 XML mapper
- 讀取 SQL（含動態 SQL `<if>`, `<choose>`, `<foreach>`）

**JDBC / JdbcTemplate**：
- `prepareStatement`、`jdbcTemplate.query` → 提取 SQL 字串（注意字串拼接）

**Stored Procedure**：
- `CallableStatement` 或 `@Procedure` → SP 名稱和參數

**對每個 SQL 記錄**：SQL 類型、完整 SQL、Table 名稱、參數來源、檔案位置。
**交叉比對**：記錄存取的 Table 是否在 Phase 3 ER 圖中，標注 Entity 關聯。

**⚠ Stored Procedure 限制**：Phase 4 只能取得 SP 名稱與輸入/輸出參數。SP 內部的商業邏輯（條件分支、計算公式、資料操作）存在於資料庫中，非 Java 原始碼的一部分。若偵測到 SP 呼叫，標記為「⚠ SP 內部邏輯需另行從資料庫取得」，並在改寫時決定是保留 SP 還是改寫為 Java 邏輯。

---

## Step 4：外部系統呼叫分析

參照 Phase 2 下游系統清單。

**REST Client**：目標 URL、HTTP Method、Request Body 內容與來源、Response 處理方式。

**SOAP Client**：Operation 名稱、請求參數、回應處理。

**Message Queue**：Queue/Topic、消息內容（序列化/JSON/XML）、欄位來源。

---

## Step 5：商業邏輯與計算邏輯

**條件分支**（虛擬碼 + 原始碼位置）：

```
IF {condition}
  → 執行 A
ELSE IF {condition}
  → 執行 B
```

**計算邏輯**（公式 + 變數來源）：

```
totalAmount = sum(items.price * items.quantity)
tax = totalAmount * taxRate    // taxRate from config.properties
```

**資料轉換**：DTO ↔ Entity 欄位 mapping、格式轉換。

**驗證邏輯**：@Valid / 手動驗證、商業規則（庫存/權限/狀態檢查）、失敗處理。

**設定驅動邏輯溯源**：

追蹤程式碼中從設定檔讀取的值：

```
Grep: @Value\(                        glob: {追蹤涉及的檔案}
Grep: @ConfigurationProperties        glob: {追蹤涉及的檔案}
Grep: Environment\.getProperty|getenv glob: {追蹤涉及的檔案}
Grep: getProperty|getConfig           glob: {追蹤涉及的檔案}
```

對每個設定值：找到 key → 在 properties/yml 中找到值 → 記錄。

**規則引擎偵測**：

```
Glob: **/*.drl
Glob: **/*.bpmn
Grep: RuleEngine|KieSession|Drools    glob: **/*.java
Grep: ProcessEngine|RuntimeService    glob: **/*.java
Grep: DecisionTable|RuleFlow          glob: **/*.java
```

若偵測到規則引擎：記錄規則檔案位置、觸發條件、影響的商業邏輯。

---

## Step 6：錯誤處理

```
Grep: try\s*\{|catch\s*\(     glob: {追蹤涉及的檔案}
Grep: @ExceptionHandler       glob: {Controller package}/**/*.java
Grep: @ControllerAdvice        glob: **/*.java
```

記錄：try-catch 範圍、catch 的 Exception 類型、處理方式、全域 handler 規則。

### 自訂 Exception 層級結構（首次分析 endpoint 時建立，後續共用）

```
Grep: extends\s+\w*Exception          glob: **/*.java
Grep: extends\s+\w*RuntimeException   glob: **/*.java
Grep: @ResponseStatus                 glob: **/*.java
```

建立 Exception 繼承樹：
- 列出所有自訂 Exception class 與繼承關係
- 記錄對應的 HTTP status code（若有 `@ResponseStatus`）
- 記錄 `@ControllerAdvice` / `@ExceptionHandler` 中每個 Exception 的處理方式

寫入 `analysis-memory/shared/exception-hierarchy.md`，後續 endpoint 引用此檔案。改寫 Spring Boot 時用於設計 `@ControllerAdvice`。

---

## Step 7：Interceptor / AOP

### EJB Interceptor

```
Grep: @Interceptors            glob: {Bean class}
Grep: @AroundInvoke            glob: {Interceptor classes}
```

記錄：執行順序、每個 @AroundInvoke 的動作、是否修改參數/回傳值。

### Spring AOP

```
Grep: @Aspect                  glob: **/*.java
Grep: @Around|@Before|@After|@AfterReturning|@AfterThrowing  glob: **/*.java
Grep: @Pointcut                glob: **/*.java
Grep: <aop:config|<aop:aspect  glob: **/*.xml
```

對每個 @Aspect class：
1. 讀取完整檔案
2. 分析 `@Pointcut` expression，判斷影響哪些 class/method
3. 比對當前追蹤的 endpoint 方法是否被 AOP 攔截
4. 記錄 @Around/@Before/@After 的動作

| Aspect Class | Pointcut | 類型 | 影響的方法 | 動作 | 檔案位置 |
|---|---|---|---|---|---|

---

## Step 8：安全控制

```
Grep: @RolesAllowed|@PermitAll|@DenyAll|@RunAs  glob: {追蹤涉及的檔案}
Grep: isCallerInRole|getCallerPrincipal  glob: {追蹤涉及的檔案}
Grep: @Secured|@PreAuthorize|@PostAuthorize  glob: {追蹤涉及的檔案}
```

記錄：方法角色限制、手動角色檢查、`@RunAs` 身份切換。

---

## Step 9：交易控制

**Spring**：
- `@Transactional` 位置、propagation、rollbackFor

**EJB CMT**：
- `@TransactionAttribute` 值：REQUIRED / REQUIRES_NEW / MANDATORY / NOT_SUPPORTED / SUPPORTS / NEVER
- Class level vs Method level
- EJB 預設 = CMT + REQUIRED（無 annotation 也由容器管理）

**EJB BMT**：
- `UserTransaction` 的 begin/commit/rollback 邊界

**跨 EJB 交易傳播**：
- 記錄呼叫鏈中每層的交易屬性，標明交易邊界

```
[TX-1] OrderServiceBean.createOrder() (REQUIRED)
  → [TX-1] ItemService.addItems() (REQUIRED, 加入 TX-1)
  → [TX-2] NotificationService.notify() (REQUIRES_NEW, 開新交易)
```

---

## 輸出格式

每個 endpoint 寫入 `analysis-memory/endpoints/{METHOD}-{path-slug}.md`。

檔名規則：REST → `GET-api-users.md`、SOAP → `SOAP-createOrder.md`、Servlet → `SERVLET-processPayment.md`、JMS → `JMS-queue-order-create.md`、EJB → `EJB-OrderService-createOrder.md`

```markdown
# Endpoint 分析：{METHOD} {path}

## 基本資訊

| 項目 | 值 |
|---|---|
| 協定 | REST / SOAP / EJB / JMS |
| 路徑 | {path or JNDI name} |
| 入口方法 | {Class.method(params)} |
| 檔案位置 | {path:line} |
| 回傳型別 | {type} |

## 呼叫鏈

{Layer-by-layer trace with file:line references}
{共用服務引用：→ 見 shared/{ServiceName}.md}

## 呼叫鏈圖（Mermaid flowchart TD）

{非同步分支用虛線：A -.->|Event/Async| B}

## 資料存取

| # | 操作 | SQL / 方法 | Table | Entity 關聯 | 參數來源 | 檔案位置 |
|---|---|---|---|---|---|---|

## 外部系統呼叫

| # | 類型 | 目標 | Request | Response | 檔案位置 |
|---|---|---|---|---|---|

## 商業邏輯

### 計算邏輯
### 條件分支
### 驗證規則
### 資料轉換
### 設定驅動邏輯

| Config Key | 值 | 來源 | 用途 |
|---|---|---|---|

## 資料流向圖（Mermaid flowchart LR）

## 錯誤處理

| Exception 類型 | 觸發條件 | 處理方式 | 位置 |
|---|---|---|---|

## Interceptor / AOP

| # | 類型 | Class | 動作 | 影響 | 檔案位置 |
|---|---|---|---|---|---|

## 安全控制

| 方法 | 限制 | 來源 |
|---|---|---|

## 交易控制

| 層級 | 管理方式 | 設定 | 位置 |
|---|---|---|---|

交易傳播鏈：{if cross-EJB}

## 備註

- ⚠ {待確認項目}
```

---

## 分析節奏

- 首次進入 Phase 4 時先執行 Pre-Step（共用服務識別）
- 每次 session 開始時：讀取 `analysis-memory/endpoint-checklist.md`，從下一個「⬜ 未開始」或「🔄 進行中」的端點繼續
- 使用者指定端點時：將該端點狀態改為 P0，優先分析
- 每次 session 分析 3-5 個 endpoint
- 按 Phase 1 的「建議分析順序」執行
- 每個完成後：
  1. 寫入 `analysis-memory/endpoints/{METHOD}-{path-slug}.md`
  2. 更新 `analysis-memory/endpoint-checklist.md`：狀態改為「✅ 完成」，填入分析檔案路徑
  3. 更新 `analysis-memory/progress.md`
