---
name: Project Scan
description: 掃描 VB.NET WinForms 專案結構，分類所有原始碼檔案，識別分層架構與相依性
role: VB.NET 專案架構分析師
---

# 專案結構掃描

## 前置輸入

讀取 `memory-bank/progress.md` 確認本 skill 尚未執行。

## 執行步驟

### Step 1：掃描專案檔案

使用檔案搜尋找到以下類型的檔案：

- `**/*.vb` — 所有 VB.NET 原始碼
- `**/*.Designer.vb` — Form Designer 檔案
- `**/*.resx` — 資源檔
- `**/*.vbproj` — 專案檔
- `**/*.sln` — 方案檔
- `**/*.config` — 配置檔（App.config、Web.config）
- `**/*.xsd` — DataSet 定義檔

### Step 2：分類所有 .vb 檔案

根據檔案內容搜尋關鍵字分類：

| 分類 | 搜尋關鍵字 |
|---|---|
| Form | `Inherits System.Windows.Forms.Form` 或 `Inherits Form` |
| UserControl | `Inherits UserControl` |
| Module | `Module ` 宣告（注意空格） |
| Class | 一般類別（不符合以上分類） |
| DataSet | `.Designer.vb` 中包含 `Inherits DataSet` |

### Step 3：識別進入點

- 搜尋 `Sub Main` 方法
- 搜尋 `Application.Run` 呼叫
- 讀取 `.vbproj` 中的 `<StartupObject>` 設定

### Step 4：識別參考與相依性

讀取 `.vbproj` 檔案，提取：

- `<Reference>` 標籤中的所有參考元件
- `<ProjectReference>` 標籤中的專案參考
- NuGet 套件（`packages.config` 或 `<PackageReference>`）

### Step 5：識別目錄分層

根據目錄名稱和檔案分布，判斷專案是否有分層架構（例如 DAL、BLL、UI、Services、Models 等資料夾）。

## 輸出

寫入 `memory-bank/01-project-structure.md`：

```markdown
# 專案結構掃描結果

## 基本資訊
- 專案名稱：{name}
- 方案檔：{.sln path}
- 專案檔：{.vbproj path}
- 啟動物件：{startup form/module}
- .NET Framework 版本：{version}

## 檔案分類統計
| 類型 | 數量 | 檔案清單 |
|---|---|---|

## 分層架構
{描述專案的分層方式}

## 參考元件
| 元件名稱 | 類型 | 版本 |
|---|---|---|

## 專案目錄結構
{tree structure}

## 初步觀察
{重要發現}
```

## 更新進度

更新 `memory-bank/progress.md`：將「Phase 1a」狀態改為 `✅ 完成`，填入完成日期、專案名稱和路徑。
