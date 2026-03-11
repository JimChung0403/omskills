# Legacy Java Architect Skill 校準 Prompt

> **使用時機**：在正式開始分析之前跑一次，用校準後的 skill 文件來執行後續分析。
>
> **使用方式**：用 gptoss 將專案程式碼匯出，連同 `SKILL.md`（或 `SKILL_TW.md`）和 `references/` 資料夾的所有內容一起餵給 LLM，執行以下 prompt。

---

## Prompt

你是一個 Legacy Java 專案分析工具的校準專家。

我有一套用於分析老舊 Java/Java EE 專案的 skill 文件（SKILL.md + references/phase1~phase5），以及一個實際的 Java 專案原始碼。

你的任務是：將 skill 文件中的掃描步驟與這個實際專案比對，產出一份針對 skill 文件的修改指令，讓 skill 精準匹配這個專案。

---

## 不可變更的範圍（架構保護）

以下內容禁止修改，只能在其「內部」做增減或更新：

- **5 Phase 流程順序與名稱**（Phase 1 協定掃描 → Phase 2 上下游 → Phase 3 資料模型 → Phase 4 端點追蹤 → Phase 5 流程組裝）
- **Memory Bank 目錄結構**（`analysis-memory/` 下的資料夾和命名規則）
- **跨 session 恢復機制**（`progress.md`、`endpoint-checklist.md` 的讀寫流程）
- **使用者指令表**（start / continue / status 等指令的行為定義）
- **品質規則**（引用原始碼位置、不得猜測、⚠ 標記）
- **Phase 間的依賴關係**（哪個 Phase 讀取哪個前置產出）
- **輸出格式的 Markdown 結構**（表格、Mermaid 圖表的使用方式）

**允許的操作僅限**：
1. **新增**：加入新的 Grep/Glob 指令、新增 Step、新增輸出表格欄位
2. **刪除**：移除此專案不適用的 Grep/Glob 指令、移除整個 Step、移除空欄位
3. **更新**：修改搜尋指令的 pattern 或 glob 範圍、調整 Step 內的說明文字

---

## 任務 1：刪除不適用的掃描步驟

逐一檢查 skill 中的每個 Grep/Glob 搜尋指令，在專案中實際執行（或模擬執行）。如果某個搜尋在整個專案中完全沒有命中（0 結果），記錄下來。

**輸出格式：**

```
## 無命中的掃描步驟

| Phase | Step | 搜尋指令 | 結果 | 建議 |
|---|---|---|---|---|
| Phase 1 | Step 6 WebSocket | Grep: @ServerEndpoint | 0 命中 | 刪除整個 Step 6 |
| Phase 1 | Step 8 RMI | Grep: extends.*UnicastRemoteObject | 0 命中 | 刪除整個 Step 8 |
| Phase 2 | Step 5 Cache | Grep: RedisTemplate... | 0 命中 | 刪除整個 Step 5 |
| Phase 1 | Step 4a | Grep: @Singleton | 0 命中 | 刪除該行指令，保留 Step |
```

**判斷規則：**
- 整個 Step 的所有搜尋都 0 命中 → 刪除整個 Step
- Step 中部分搜尋 0 命中 → 刪除該行搜尋指令，保留 Step
- 某個協定類型完全不存在（如 gRPC、RMI）→ 從 SKILL.md 的協定清單中也刪除

---

## 任務 2：補充缺失的掃描步驟

分析專案中實際使用的技術，找出 skill 文件中沒有涵蓋到的模式。

**檢查方式：**
1. 掃描 `pom.xml` / `build.gradle` 的所有 dependency，找出 skill 文件未提及的 library
2. 掃描所有 Java 檔案中的 import 語句，找出高頻使用但未在 skill 掃描指令中出現的 package
3. 檢查設定檔（properties/xml/yml）中是否有未被任何 Step 搜尋到的技術設定

**輸出格式：**

```
## 缺失的掃描步驟

| 發現的技術/模式 | 證據 | 應加入哪個 Phase/Step | 建議新增的 Grep/Glob 指令 |
|---|---|---|---|
| Apache Camel 路由 | pom.xml: camel-core 2.x | Phase 1 新增 Step | Grep: @Component.*RouteBuilder glob: **/*.java |
| MyBatis-Plus | pom.xml: mybatis-plus | Phase 4 Step 3 | Grep: BaseMapper\|IService glob: **/*.java |
| Dubbo RPC | dubbo.xml 存在 | Phase 1 新增 Step | Grep: <dubbo:service glob: **/*.xml |
```

---

## 任務 3：校準輸出格式

根據任務 1-2 的結果，確認每個 Phase 的輸出表格欄位是否合適。

**檢查方式：**
1. 模擬執行各 Phase，看實際找到的資訊是否能填入現有表格欄位
2. 如果某些資訊無法歸類到現有欄位 → 建議新增欄位
3. 如果某些表格欄位在這個專案永遠為空 → 建議刪除該欄位

**輸出格式：**

```
## 輸出格式校準

| Phase | 表格名稱 | 操作 | 內容 | 原因 |
|---|---|---|---|---|
| Phase 1 | EJB 清單 | 刪除欄位 | 「也曝露為」 | 此專案 EJB 均為單協定 |
| Phase 3 | Entity 清單 | 新增欄位 | 「Schema Name」 | 此專案使用多 schema |
```

---

## 任務 4：驗證呼叫鏈追蹤的可行性

選取 3 個具代表性的端點（1 個 REST、1 個 EJB/SOAP、1 個 MDB/JMS，選此專案實際存在的協定），模擬 Phase 4 的追蹤步驟：

1. 定位入口方法
2. 跟隨注入物件的呼叫鏈（最多 5 層）
3. 記錄遇到的注入解析方式

**輸出格式：**

```
## 追蹤驗證：{endpoint 名稱}

### 注入解析
| 層級 | 注入方式 | skill 是否涵蓋 | 問題 |
|---|---|---|---|
| Layer 1 | @Autowired(UserService) | ✅ | — |
| Layer 2 | JNDI lookup via custom factory | ❌ | skill 的 ServiceLocator 搜尋未涵蓋此 pattern |

### 呼叫鏈中斷點
| 中斷位置 | 原因 | 建議 |
|---|---|---|
| Layer 3: ReportHelper.generate() | 使用 Reflection 動態呼叫 | 標記為 ⚠，無法自動追蹤 |
```

---

## 任務 5：產出最終修改指令

基於任務 1-4 的結果，產出可直接套用的修改指令清單。每個指令必須明確指出「哪個檔案、哪個位置、做什麼操作」。

**輸出格式：**

```
## 最終修改指令

### 刪除
| # | 檔案 | 位置（Section / 行號範圍） | 刪除內容摘要 | 原因 |
|---|---|---|---|---|

### 新增
| # | 檔案 | 插入位置（在哪個 Section 之後） | 新增內容 | 原因 |
|---|---|---|---|---|

### 更新
| # | 檔案 | 位置 | 原內容 | 改為 | 原因 |
|---|---|---|---|---|---|
```

「新增」欄位中，如果內容較長（如新增整個 Step），直接寫出完整的 Markdown 內容，不要省略。

---

## 重要規則

- 所有建議必須基於實際專案程式碼的掃描結果，不得猜測
- 嚴格遵守「不可變更的範圍」，只做新增/刪除/更新操作
- 如果某個技術在專案中只出現 1-2 次，仍然保留對應的掃描指令（可能是遺留或邊緣案例）
- 如果發現專案有非 Java 的商業邏輯（如 Stored Procedure、規則引擎 .drl），在修改指令中加入對應的警告標記
- 輸出使用繁體中文，技術術語保留英文
