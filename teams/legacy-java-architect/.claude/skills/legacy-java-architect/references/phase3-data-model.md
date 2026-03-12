# Phase 3：Data Model 與 Entity 關聯分析

建立專案完整的資料模型全貌：Entity 清單、欄位結構、關聯關係、ER 圖。

前置：讀取 `analysis-memory/01-own-protocols.md` 和 `analysis-memory/02-upstream-downstream.md`。

---

## Step 1：JPA / Hibernate Entity 掃描

```
Grep: @Entity                   glob: **/*.java
Grep: @Table\(name              glob: **/*.java
Grep: @MappedSuperclass         glob: **/*.java
Grep: @Embeddable               glob: **/*.java
Grep: @Inheritance               glob: **/*.java
```

**對每個 @Entity class**：

1. 讀取完整檔案
2. 記錄：class 名稱、`@Table(name=...)` 的實體 table 名稱、繼承策略
3. 列出所有欄位：
   - `@Column(name=...)` → 欄位名稱、型別、nullable、unique
   - `@Id` / `@GeneratedValue` → 主鍵策略
   - `@Enumerated` → Enum 映射方式
   - `@Temporal` → 日期型別
   - `@Lob` → 大物件
4. 記錄 `@MappedSuperclass` 的共用欄位（如 createTime、updateBy）

---

## Step 2：Entity 關聯映射

```
Grep: @ManyToOne               glob: **/*.java
Grep: @OneToMany               glob: **/*.java
Grep: @OneToOne                glob: **/*.java
Grep: @ManyToMany              glob: **/*.java
Grep: @JoinColumn              glob: **/*.java
Grep: @JoinTable               glob: **/*.java
Grep: mappedBy                 glob: **/*.java
```

**對每個關聯**：

| 記錄項目 | 說明 |
|---|---|
| 來源 Entity | 擁有關聯的 Entity class |
| 目標 Entity | 被關聯的 Entity class |
| 關聯類型 | ManyToOne / OneToMany / OneToOne / ManyToMany |
| 外鍵欄位 | `@JoinColumn(name=...)` |
| fetch 策略 | LAZY / EAGER |
| cascade 設定 | CascadeType 值 |
| 方向性 | 單向 / 雙向（有無 mappedBy） |

---

## Step 3：EJB 2.x CMP Entity Bean

```
Grep: implements\s+EntityBean   glob: **/*.java
Grep: <entity>                  glob: **/ejb-jar.xml
Grep: <cmp-field>               glob: **/ejb-jar.xml
Grep: <cmr-field>               glob: **/ejb-jar.xml
Grep: <ejb-relation>            glob: **/ejb-jar.xml
```

讀取 `ejb-jar.xml` 中的 `<entity>` 定義，提取：
- `<ejb-name>` 與 `<abstract-schema-name>`
- 所有 `<cmp-field>` 欄位
- `<cmr-field>` Container-Managed Relationship（等同 JPA 關聯）
- `<ejb-relation>` 定義的關聯類型與目標

---

## Step 4：SQL 中的 Table 補充

從 Phase 2 已偵測到的 SQL 語句和 MyBatis/iBatis mapper 中，補充 Entity 未涵蓋的 Table：

```
Grep: @Table\(name              glob: **/*.java
Grep: FROM\s+\w+|INSERT\s+INTO\s+\w+|UPDATE\s+\w+|DELETE\s+FROM\s+\w+  glob: **/*.java
Grep: FROM\s+\w+|INSERT\s+INTO\s+\w+|UPDATE\s+\w+|DELETE\s+FROM\s+\w+  glob: **/*.xml
Grep: CREATE\s+TABLE            glob: **/*.sql
```

交叉比對：
- 有 Entity 但沒被 SQL 存取的 Table → 可能是遺留未使用
- 有 SQL 存取但沒有 Entity 的 Table → 直接 JDBC 存取，記錄為「無 Entity 映射」

---

## Step 5：Hibernate 映射檔（XML 式 ORM）

老舊專案可能不用 annotation，改用 XML 映射：

```
Glob: **/*.hbm.xml
Grep: <hibernate-mapping        glob: **/*.xml
Grep: <class\s+name             glob: **/*.hbm.xml
Grep: <many-to-one|<one-to-many|<set|<bag  glob: **/*.hbm.xml
```

讀取每個 `.hbm.xml`，提取 class-to-table 映射、property 欄位、關聯定義。

---

## Step 6：DTO / VO / Request / Response 結構

```
Grep: class\s+\w*(DTO|Dto|VO|Vo)\b        glob: **/*.java
Grep: class\s+\w*(Request|Response)\b      glob: **/*.java
Grep: class\s+\w*(Form|Command|Param)\b    glob: **/*.java
```

**對每個 DTO/VO class**：

1. 讀取完整檔案
2. 記錄所有欄位（名稱、型別）
3. 若有 `@JsonProperty`、`@XmlElement`、`@SerializedName` 等序列化 annotation → 記錄實際序列化名稱
4. 找出對應的 Entity（同名或欄位高度相似）

**交叉比對**：
- 有 Entity 但沒有 DTO → API 直接回傳 Entity（改寫時需建立 DTO 隔離）
- 有 DTO 但欄位與 Entity 差異大 → 有複雜轉換邏輯，記錄轉換所在位置

---

## 輸出格式

寫入 `analysis-memory/03-data-model.md`：

```markdown
# Phase 3：Data Model 分析結果

## Entity 清單

| # | Entity Class | Table Name | 主鍵 | 欄位數 | 繼承 | 檔案位置 |
|---|---|---|---|---|---|---|

## 關聯清單

| # | 來源 Entity | 關聯類型 | 目標 Entity | 外鍵 | Fetch | Cascade |
|---|---|---|---|---|---|---|

## 無 Entity 映射的 Table

| # | Table Name | 存取方式 | 存取位置 |
|---|---|---|---|

## 共用父類別（@MappedSuperclass）

| # | Class | 共用欄位 | 繼承的 Entity |
|---|---|---|---|

## DTO / VO 清單

| # | Class | 類型 | 對應 Entity | 欄位數 | 序列化格式 | 檔案位置 |
|---|---|---|---|---|---|---|

## Entity ↔ DTO 映射

| Entity | DTO/VO | 共同欄位 | 差異欄位 | 轉換邏輯位置 |
|---|---|---|---|---|

## ER 圖（Mermaid erDiagram）

erDiagram
    ORDER ||--o{ ORDER_ITEM : contains
    ORDER }o--|| CUSTOMER : belongs_to
    ORDER_ITEM }o--|| PRODUCT : references

## 備註

- ⚠ {待確認項目}
```
