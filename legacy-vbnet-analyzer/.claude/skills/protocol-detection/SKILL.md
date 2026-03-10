---
name: Protocol Detection
description: Detects communication protocols and data access patterns in VB.NET projects
---

# Protocol Detection

## 描述

偵測 VB.NET WinForms 專案中使用的通訊協定、資料庫連線方式，以及外部服務通訊方式。

## 使用者

- Architecture Scout

## 偵測目標與搜尋模式

### 資料庫連線

使用 Grep 搜尋以下關鍵字：

| 搜尋模式 | 對應技術 |
|---|---|
| `SqlConnection`, `SqlCommand`, `SqlDataAdapter` | ADO.NET（SQL Server） |
| `OracleConnection`, `OracleCommand` | ADO.NET（Oracle） |
| `OleDbConnection`, `OleDbCommand` | OLE DB |
| `OdbcConnection` | ODBC |
| `MySqlConnection` | MySQL Connector |
| `DbContext`, `ObjectContext` | Entity Framework |
| `DataSet`, `TableAdapter`, `.Fill(` | 強型別 DataSet / TableAdapter |

### 外部通訊

| 搜尋模式 | 對應技術 |
|---|---|
| `WebRequest`, `HttpWebRequest`, `WebClient` | HTTP 通訊 |
| `HttpClient` | 現代 HTTP 通訊 |
| `ServiceReference`, `WebReference` | Web Service / WCF 用戶端 |
| `SoapHttpClientProtocol` | SOAP Web Service |
| `ChannelFactory`, `ServiceHost` | WCF |
| `RemotingConfiguration`, `MarshalByRefObject` | .NET Remoting |
| `TcpClient`, `UdpClient`, `Socket` | 原生 Socket |
| `SerialPort` | 串列埠通訊 |

### 連線字串搜尋位置（優先順序）

1. `App.config` 或 `Web.config` 中的 `<connectionStrings>` 區段
2. 程式碼中硬編碼的 `ConnectionString` 賦值
3. `.settings` 檔案中的連線設定
4. Registry 讀取（搜尋 `Registry.GetValue`、`RegistryKey`）

### 資料序列化

| 搜尋模式 | 對應技術 |
|---|---|
| `XmlSerializer`, `XmlDocument`, `XDocument` | XML |
| `JsonConvert`, `JavaScriptSerializer` | JSON |
| `BinaryFormatter`, `SoapFormatter` | 二進位 / SOAP 序列化 |

## 執行步驟

1. 對每個偵測目標，使用 Grep 搜尋所有 `.vb` 和 `.config` 檔案
2. 記錄每個發現的檔案路徑和行號
3. 讀取相關程式碼上下文（前後 5 行），確認用途
4. 彙整為結構化報告

## 輸出格式

```markdown
# 通訊協定偵測結果

## 資料庫連線
- 方式：{ADO.NET / Entity Framework / ...}
- 資料庫類型：{SQL Server / Oracle / ...}
- 連線字串位置：{file:line}
- 連線字串內容：{sanitized connection string}

## 外部通訊
| 協定 | 用途 | 使用位置 |
|---|---|---|

## 資料序列化
| 格式 | 用途 | 使用位置 |
|---|---|---|
```

## 範例

**輸入**：偵測專案 `/projects/legacy-app/` 的通訊協定

**輸出**：

```markdown
# 通訊協定偵測結果

## 資料庫連線
- 方式：ADO.NET（SqlConnection + SqlCommand）
- 資料庫類型：SQL Server
- 連線字串位置：App.config:12
- 連線字串內容：Server=192.168.1.100;Database=MainDB;Integrated Security=True

## 外部通訊
| 協定 | 用途 | 使用位置 |
|---|---|---|
| SOAP Web Service | 呼叫 ERP 系統 | ServiceRef/ERPService.vb:1 |
| HttpWebRequest | 傳送報表至伺服器 | Modules/ReportUploader.vb:45 |

## 資料序列化
| 格式 | 用途 | 使用位置 |
|---|---|---|
| XML | 匯出設定檔 | Utils/ConfigHelper.vb:23 |
```
