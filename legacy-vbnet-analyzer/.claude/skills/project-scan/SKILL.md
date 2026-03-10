---
name: Project Scan
description: Scans VB.NET WinForms project structure and categorizes all source files
---

# Project Scan

## 描述

掃描 VB.NET WinForms 專案的完整目錄結構，分類所有原始碼檔案，識別專案的組織方式與分層架構。

## 使用者

- Architecture Scout

## 執行步驟

### Step 1：掃描專案根目錄

使用 Glob 工具掃描以下檔案類型：

- `**/*.vb` — 所有 VB.NET 原始碼
- `**/*.designer.vb` — Form Designer 檔案
- `**/*.resx` — 資源檔
- `**/*.vbproj` — 專案檔
- `**/*.sln` — 方案檔
- `**/*.config` — 配置檔（App.config、Web.config）
- `**/*.xsd` — DataSet 定義檔

### Step 2：分類檔案

根據檔案內容將所有 `.vb` 檔案分類為：

| 分類 | 搜尋關鍵字 |
|---|---|
| Form | `Inherits System.Windows.Forms.Form` 或 `Inherits Form` |
| UserControl | `Inherits UserControl` |
| Module | `Module {ModuleName}` 宣告 |
| Class | 一般類別檔案（不符合以上分類） |
| DataSet | `.Designer.vb` 中包含 `Inherits DataSet` |

使用 Grep 工具搜尋各分類的關鍵字。

### Step 3：識別進入點

搜尋應用程式的啟動點：

- 搜尋 `Sub Main` 方法
- 搜尋 `Application.Run` 呼叫
- 檢查專案檔（`.vbproj`）中的 `<StartupObject>` 設定

### Step 4：識別參考與相依性

讀取 `.vbproj` 檔案，提取：

- `<Reference>` 標籤中的所有參考元件
- `<ProjectReference>` 標籤中的專案參考
- NuGet 套件（`packages.config` 或 `<PackageReference>`）

## 輸出格式

```markdown
# 專案結構掃描結果

## 基本資訊
- 專案名稱：{name}
- 方案檔：{.sln path}
- 專案檔：{.vbproj path}
- 啟動物件：{startup form/module}

## 檔案分類統計
| 類型 | 數量 | 檔案清單 |
|---|---|---|
| Form | {n} | {paths} |
| UserControl | {n} | {paths} |
| Module | {n} | {paths} |
| Class | {n} | {paths} |
| DataSet | {n} | {paths} |
| 配置檔 | {n} | {paths} |
| 資源檔 | {n} | {paths} |

## 參考元件
| 元件名稱 | 類型 | 版本 |
|---|---|---|

## 專案目錄結構
{tree structure}
```

## 範例

**輸入**：掃描專案路徑 `/projects/legacy-app/`

**輸出**：

```markdown
# 專案結構掃描結果

## 基本資訊
- 專案名稱：LegacyApp
- 方案檔：/projects/legacy-app/LegacyApp.sln
- 專案檔：/projects/legacy-app/LegacyApp/LegacyApp.vbproj
- 啟動物件：frmMain

## 檔案分類統計
| 類型 | 數量 |
|---|---|
| Form | 45 |
| UserControl | 12 |
| Module | 8 |
| Class | 23 |
| DataSet | 5 |

## 參考元件
| 元件名稱 | 類型 | 版本 |
|---|---|---|
| System.Data | Framework | 4.0 |
| Newtonsoft.Json | NuGet | 12.0.3 |
```
