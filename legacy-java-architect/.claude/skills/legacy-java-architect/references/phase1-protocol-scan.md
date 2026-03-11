# Phase 1：自身通訊協定掃描

掃描這個 Java 專案「自己提供」給外部呼叫者的所有通訊協定與接口端點。

---

## Step 1：REST API 端點

搜尋 Spring MVC 風格：

```
Grep: @RestController          glob: **/*.java
Grep: @Controller              glob: **/*.java
Grep: @RequestMapping          glob: **/*.java
Grep: @GetMapping              glob: **/*.java
Grep: @PostMapping             glob: **/*.java
Grep: @PutMapping              glob: **/*.java
Grep: @DeleteMapping           glob: **/*.java
Grep: @PatchMapping            glob: **/*.java
```

搜尋 JAX-RS 風格：

```
Grep: @Path                    glob: **/*.java
Grep: @GET|@POST|@PUT|@DELETE  glob: **/*.java
Grep: @Produces|@Consumes      glob: **/*.java
```

**對每個 Controller/Resource class**：
1. 讀取完整檔案
2. 記錄 class 層級的 `@RequestMapping` / `@Path`（base path）
3. 列出每個 method 的：HTTP Method、完整 URL path、參數（型別與來源）、回傳型別、檔案路徑:行號

---

## Step 2：SOAP Web Service

```
Grep: @WebService              glob: **/*.java
Grep: @WebMethod               glob: **/*.java
Glob: **/*.wsdl
Grep: <jaxws:|cxf:|<service    glob: **/*.xml
```

**對每個 @WebService class**：
1. 讀取完整檔案
2. 記錄 serviceName、portName、targetNamespace
3. 列出每個 @WebMethod 的方法名、參數、回傳型別
4. 若有 WSDL 檔案，記錄位置

---

## Step 3：Servlet / Filter

讀取 `**/web.xml`：
1. 列出所有 `<servlet>` 與 `<servlet-mapping>`（URL pattern）
2. 列出所有 `<filter>` 與 `<filter-mapping>`
3. 列出所有 `<listener>`
4. 記錄 Servlet API 版本（`<web-app version="...">`）

搜尋 annotation 式：

```
Grep: @WebServlet              glob: **/*.java
Grep: @WebFilter               glob: **/*.java
```

**對每個 Servlet**：讀取 `doGet`、`doPost`、`service` 方法，記錄 URL pattern。

---

## Step 4：EJB（Session Bean / MDB）

一個 EJB class 可能同時透過多種協定曝露（EJB Remote + SOAP + REST）。將每個 EJB class 視為一個「服務單元」。

### 4a：EJB 3.x（Annotation 式）

```
Grep: @Stateless               glob: **/*.java
Grep: @Stateful                glob: **/*.java
Grep: @Singleton               glob: **/*.java
Grep: @MessageDriven           glob: **/*.java
Grep: @Remote                  glob: **/*.java
Grep: @Local                   glob: **/*.java
Grep: @LocalBean               glob: **/*.java
Grep: @Startup                 glob: **/*.java
Grep: @EJB                     glob: **/*.java
Grep: @Interceptors            glob: **/*.java
Grep: @AroundInvoke            glob: **/*.java
Grep: @TransactionAttribute    glob: **/*.java
Grep: @RolesAllowed|@PermitAll|@DenyAll  glob: **/*.java
Grep: @Timeout|@Schedule       glob: **/*.java
```

### 4b：EJB 2.x（20 年的專案可能使用）

```
Grep: implements\s+SessionBean      glob: **/*.java
Grep: extends\s+EJBHome|EJBObject   glob: **/*.java
Grep: implements\s+MessageDrivenBean  glob: **/*.java
Grep: ejbCreate|ejbRemove|ejbActivate|ejbPassivate  glob: **/*.java
```

### 4c：EJB 部署描述檔

```
Glob: **/ejb-jar.xml
Glob: **/jboss.xml
Glob: **/jboss-ejb3.xml
Glob: **/weblogic-ejb-jar.xml
Glob: **/ibm-ejb-jar-bnd.xml
Glob: **/sun-ejb-jar.xml
Glob: **/glassfish-ejb-jar.xml
```

讀取每個部署描述檔，提取：`<session>`/`<entity>`/`<message-driven>` 定義、JNDI 綁定、Transaction 設定（CMT/BMT）、Security role、Interceptor。

### 4d：對每個 EJB class 記錄

1. **Bean 基本資訊**：type（Stateless/Stateful/Singleton/MDB）、class name、位置
2. **多協定曝露**：是否同時有 @Remote + @WebService + @Path，列出每種曝露的接口方法
3. **介面方法清單**：Remote/Local interface 上的所有方法（方法名、參數型別、回傳型別）
4. **JNDI 名稱**：`mappedName`、`name`、或部署描述檔中的 binding
5. **Interceptor**：`@Interceptors` 列表、`@AroundInvoke` 方法、執行順序
6. **交易屬性**：CMT `@TransactionAttribute` 值 / BMT `@TransactionManagement(BEAN)`，class vs method level
7. **安全設定**：`@RolesAllowed`、`@PermitAll`、`@DenyAll`、`@RunAs`
8. **Timer/排程**：`@Timeout`、`@Schedule`
9. **注入的其他 EJB**：`@EJB` 注入的型別（Phase 4 追蹤線索）

---

## Step 5：JMS / Message Consumer

```
Grep: @JmsListener             glob: **/*.java
Grep: MessageListener          glob: **/*.java
Grep: onMessage                glob: **/*.java
Grep: @RabbitListener          glob: **/*.java
Grep: @KafkaListener           glob: **/*.java
```

**對每個 Consumer**：記錄 Queue/Topic 名稱、消息型別、檔案位置。

---

## Step 6：WebSocket

```
Grep: @ServerEndpoint          glob: **/*.java
Grep: @OnMessage|@OnOpen|@OnClose  glob: **/*.java
Grep: WebSocketHandler|WebSocketConfigurer  glob: **/*.java
```

---

## Step 7：排程任務

```
Grep: @Scheduled               glob: **/*.java
Grep: @Schedules               glob: **/*.java
Grep: CronTrigger|SimpleTrigger  glob: **/*.java
Glob: **/quartz.properties
Grep: <bean.*SchedulerFactoryBean  glob: **/*.xml
```

記錄 cron expression 或間隔時間、觸發的方法。

---

## Step 8：RMI

```
Grep: extends.*UnicastRemoteObject  glob: **/*.java
Grep: implements.*Remote       glob: **/*.java
Grep: Naming\.rebind|Naming\.bind  glob: **/*.java
```

---

## Step 9：gRPC

```
Glob: **/*.proto
Grep: grpc                     glob: **/pom.xml
Grep: grpc                     glob: **/build.gradle
Grep: extends.*ImplBase        glob: **/*.java
```

---

## Step 10：全域設定掃描

### 10a：建構工具與依賴

```
Glob: **/pom.xml
Glob: **/build.gradle
Glob: **/build.gradle.kts
```

讀取每個 pom.xml / build.gradle，記錄：
- 建構工具（Maven / Gradle）及版本
- Java 版本（source / target / release）
- 所有 dependency（groupId:artifactId:version）
- 應用伺服器相關 plugin（maven-ear-plugin、cargo-maven2-plugin、maven-war-plugin）

### 10b：全域安全設定

```
Grep: <security-constraint|<login-config|<security-role  glob: **/web.xml
Grep: <realm|<login-module               glob: **/*.xml
Grep: SecurityFilterChain|WebSecurityConfigurerAdapter  glob: **/*.java
Grep: AuthenticationManager|UserDetailsService  glob: **/*.java
Grep: jwt|JsonWebToken|oauth             glob: **/*.java
Grep: jwt|oauth                          glob: **/*.properties
Grep: CorsFilter|addCorsMappings|CorsConfiguration  glob: **/*.java
Grep: csrf                               glob: **/*.java
```

記錄：
- 認證機制（Form Login / Basic Auth / JWT / SAML / OAuth2 / LDAP / Certificate）
- 使用者/角色儲存方式（DB table name / LDAP / XML file）
- Session 管理方式（Stateful / Stateless / Cluster replication）
- CORS 設定（允許的 origin、methods）
- CSRF 設定（啟用/停用）

### 10c：Spring XML 設定

```
Glob: **/applicationContext*.xml
Glob: **/spring-*.xml
Glob: **/dispatcher-servlet.xml
Glob: **/beans.xml
Grep: <import\s+resource              glob: **/*.xml
Grep: <bean\s+                        glob: **/*.xml
Grep: <context:component-scan         glob: **/*.xml
Grep: <tx:annotation-driven           glob: **/*.xml
Grep: <mvc:annotation-driven          glob: **/*.xml
```

記錄：XML 設定檔清單、每個檔案定義的 bean 數量、component-scan base-package、import 鏈。

### 10d：Application Server 設定

```
Glob: **/jboss-web.xml
Glob: **/jboss-deployment-structure.xml
Glob: **/weblogic.xml
Glob: **/weblogic-application.xml
Glob: **/ibm-web-bnd.xml
Glob: **/context.xml
Glob: **/server.xml
Grep: <resource-ref                    glob: **/web.xml
Grep: <resource-env-ref                glob: **/web.xml
Grep: <jndi                           glob: **/*.xml
```

記錄：
- App Server 類型（JBoss/WildFly、WebLogic、WebSphere、Tomcat、GlassFish）
- 所有 JNDI resource-ref（DataSource、JMS ConnectionFactory、Mail Session 等）
- 每個 resource-ref 需搬至 Spring Boot `application.properties` 的對應設定

---

## 輸出格式

寫入 `analysis-memory/01-own-protocols.md`：

```markdown
# Phase 1：自身通訊協定掃描結果

## 協定摘要

| 協定類型 | 數量 | 狀態 |
|---|---|---|
| REST API | N 個 endpoint | ✅ / 未偵測到 |
| SOAP Web Service | N 個 service | ✅ / 未偵測到 |
| Servlet | N 個 | ✅ / 未偵測到 |
| EJB | N 個 bean | ✅ / 未偵測到 |
| JMS Consumer | N 個 listener | ✅ / 未偵測到 |
| WebSocket | N 個 | ✅ / 未偵測到 |
| Scheduled Tasks | N 個 | ✅ / 未偵測到 |
| RMI | N 個 | ✅ / 未偵測到 |
| gRPC | N 個 | ✅ / 未偵測到 |

## REST API 端點清單

| # | HTTP Method | URL Path | 方法簽名 | 參數 | 回傳型別 | 檔案位置 |
|---|---|---|---|---|---|---|

## SOAP Web Service 清單

| # | Service Name | Operation | 參數 | 回傳型別 | 檔案位置 |
|---|---|---|---|---|---|

## Servlet 清單

| # | URL Pattern | Class | HTTP Methods | 檔案位置 |
|---|---|---|---|---|

## EJB 清單

| # | Bean Class | Type | Remote Interface | Local Interface | 也曝露為 | JNDI Name | 檔案位置 |
|---|---|---|---|---|---|---|---|

### EJB Remote Interface 方法

| # | Bean | Interface | 方法簽名 | 回傳型別 | 檔案位置 |
|---|---|---|---|---|---|

### EJB Interceptor

| Interceptor Class | 作用範圍 | 檔案位置 |
|---|---|---|

### EJB 交易設定

| Bean | 管理方式 | Class Level | Method Level 覆寫 |
|---|---|---|---|

## JMS / Message Consumer 清單

| # | Queue/Topic | 消息型別 | Handler Method | 檔案位置 |
|---|---|---|---|---|

## 其他協定
（WebSocket、排程、RMI、gRPC，偵測到才列）

## 建構與依賴

| 建構工具 | Java 版本 | App Server |
|---|---|---|

### 依賴清單

| # | groupId | artifactId | version | 用途 | Spring Boot 替代 |
|---|---|---|---|---|---|
> 「Spring Boot 替代」欄在分析階段留空，改寫時填入。

## 全域安全設定

| 項目 | 值 | 來源 |
|---|---|---|
| 認證機制 | Form Login / JWT / ... | web.xml / SecurityConfig.java |
| 使用者儲存 | DB: users table / LDAP | ... |
| Session 管理 | Stateful / Stateless | ... |
| CORS | 允許的 origin | ... |
| CSRF | 啟用 / 停用 | ... |

## Spring XML 設定清單

| # | 設定檔 | Bean 數量 | component-scan 範圍 |
|---|---|---|---|

## App Server 設定

| # | JNDI 名稱 | 資源類型 | 用途 | 需搬至 application.properties |
|---|---|---|---|---|

## Phase 4 建議分析順序

1. {最核心的 endpoint group}
2. {次要的 endpoint group}
3. ...
```

同時寫入 `analysis-memory/endpoint-checklist.md`：

```markdown
# Endpoint 分析清單

> Phase 1 完成時自動產生，Phase 4 每完成一個端點後更新狀態。
> 跨 session 恢復時，讀取此清單確認下一個要分析的端點。

## 統計

- 總計：N 個端點
- ✅ 已完成：0 個
- 🔄 進行中：0 個
- ⬜ 未開始：N 個

## 清單

| # | 協定 | 端點 | 入口方法 | 優先順序 | 狀態 | 分析檔案 |
|---|---|---|---|---|---|---|
| 1 | REST | GET /api/users | UserController.getUsers() | P1 | ⬜ 未開始 | — |
| 2 | REST | POST /api/orders | OrderController.create() | P1 | ⬜ 未開始 | — |
| 3 | EJB | OrderService.createOrder | OrderServiceBean.createOrder() | P1 | ⬜ 未開始 | — |
| 4 | SOAP | getAccount | AccountWS.getAccount() | P2 | ⬜ 未開始 | — |
| ... | | | | | | |

## 多協定 EJB 處理

同一 EJB Bean 透過多種協定曝露（如 @Remote + @WebService + @Path）時：
- 在清單中列為 **1 行**，協定欄列出所有曝露方式（如 `EJB+REST+SOAP`）
- Phase 4 只追蹤 **1 次**，輸出檔名用主要協定（EJB > REST > SOAP）
- 例：`| 5 | EJB+REST+SOAP | OrderService.createOrder | OrderServiceBean.createOrder() | P1 | ⬜ 未開始 | — |`

## 優先順序說明

- **P0**：使用者指定優先分析
- **P1**：核心業務端點（訂單、付款、使用者管理等）
- **P2**：次要業務端點
- **P3**：工具/管理端點
```
