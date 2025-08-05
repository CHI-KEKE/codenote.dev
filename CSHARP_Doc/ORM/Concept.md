# 🗄️ C# ORM 概念完整指南

## 📖 目錄
- [一、什麼是 ORM](#一什麼是-orm)
  - [1. 基本定義](#1-基本定義)
    - [1.1 Object-Relational Mapping](#11-object-relational-mapping)
    - [1.2 抽象層概念](#12-抽象層概念)
    - [1.3 效能考量](#13-效能考量)
- [二、EF 的好處](#二ef-的好處)
  - [1. 開發優勢](#1-開發優勢)
    - [1.1 型別安全](#11-型別安全)
    - [1.2 跨資料庫支援](#12-跨資料庫支援)
    - [1.3 架構設計](#13-架構設計)
  - [2. 功能特色](#2-功能特色)
    - [2.1 交易管理](#21-交易管理)
    - [2.2 進階功能](#22-進階功能)
    - [2.3 效能優化](#23-效能優化)
    - [2.4 預存程序支援](#24-預存程序支援)
- [三、EF 的物件導向特性](#三ef-的物件導向特性)
  - [1. OO 優勢](#1-oo-優勢)
    - [1.1 類別映射](#11-類別映射)
    - [1.2 程式碼品質](#12-程式碼品質)
- [四、Lazy Loading 延遲載入](#四lazy-loading-延遲載入)
  - [1. 基本概念](#1-基本概念)
    - [1.1 延遲載入機制](#11-延遲載入機制)
    - [1.2 效能優化](#12-效能優化)
  - [2. N+1 查詢問題](#2-n1-查詢問題)
    - [2.1 問題描述](#21-問題描述)
    - [2.2 解決方案](#22-解決方案)
- [五、高頻交易適合 EF 嗎](#五高頻交易適合-ef-嗎)
  - [1. 基本判斷](#1-基本判斷)
    - [1.1 EF Core 定位](#11-ef-core-定位)
    - [1.2 高頻交易需求](#12-高頻交易需求)
  - [2. EF Core 限制](#2-ef-core-限制)
    - [2.1 記憶體追蹤機制](#21-記憶體追蹤機制)
    - [2.2 SQL 生成控制](#22-sql-生成控制)
    - [2.3 連線管理瓶頸](#23-連線管理瓶頸)
  - [3. 應用場景分析](#3-應用場景分析)
    - [3.1 不適合場景](#31-不適合場景)
    - [3.2 適合場景](#32-適合場景)
- [六、DbContext 管理](#六dbcontext-管理)
  - [1. 基本概念](#1-基本概念-1)
    - [1.1 核心功能](#11-核心功能)
    - [1.2 主要組件](#12-主要組件)
  - [2. 生命週期問題](#2-生命週期問題)
    - [2.1 記憶體洩漏](#21-記憶體洩漏)
    - [2.2 資料一致性問題](#22-資料一致性問題)
- [七、ORM 防範 SQL Injection](#七orm-防範-sql-injection)
  - [1. SQL Injection 威脅](#1-sql-injection-威脅)
    - [1.1 攻擊原理](#11-攻擊原理)
    - [1.2 威脅本質](#12-威脅本質)
    - [1.3 危險範例](#13-危險範例)
  - [2. ORM 防護機制](#2-orm-防護機制)
    - [2.1 物件導向操作](#21-物件導向操作)
    - [2.2 參數化查詢](#22-參數化查詢)
    - [2.3 原始 SQL 安全](#23-原始-sql-安全)

---

## 一、什麼是 ORM

### 1. 基本定義

#### 1.1 Object-Relational Mapping

📋 **ORM 全名**

ORM 代表 **Object-Relational Mapping**（物件關聯式映射）

#### 1.2 抽象層概念

🔗 **核心概念**

ORM 在物件導向程式設計（OOP）與關聯式資料庫（RDBMS）之間建立一個抽象層，讓開發者可以透過物件操作資料，而不是直接操作資料表。

#### 1.3 效能考量

⚠️ **效能影響**

ORM 會帶來一定的效能開銷，因為它需要轉換物件與 SQL 之間的關聯。

## 二、EF 的好處

### 1. 開發優勢

#### 1.1 型別安全

✅ **強型別與 IntelliSense 支援**

- 編譯時期檢查錯誤
- 提供完整的程式碼提示功能
- 減少執行時期錯誤

#### 1.2 跨資料庫支援

🔄 **資料庫可攜性**

EF 提供跨資料庫的可攜性，可以切換不同的資料庫系統：
- SQL Server
- MySQL  
- PostgreSQL

#### 1.3 架構設計

🏗️ **三層架構**

EF 透過以下方式運作：
- **模型 (Model)** → **映射 (Mapping)** → **資料存取 (Data Access)**

### 2. 功能特色

#### 2.1 交易管理

🔒 **資料一致性**

- 支援交易與資料一致性
- 透過 Unit of Work 機制
- 內建 Transaction 管理

#### 2.2 進階功能

🚀 **自動化功能**

**EF 自動提供：**
- Lazy Loading（延遲載入）
- Change Tracking（變更追蹤）

**對比 ADO.NET：**
- EF 自動處理 SQL 轉換與交易
- ADO.NET 需要開發者手動控制：
  - SqlCommand
  - DataReader
  - Transaction 等細節

#### 2.3 效能優化

⚡ **效能改善方式**

可透過以下方式來改善效能：
- 避免 Lazy Loading
- 使用 AsNoTracking()
- 批次查詢
- 手動優化 SQL 查詢

#### 2.4 預存程序支援

💾 **Stored Procedure**

EF 可以使用 Stored Procedure，提供更靈活的資料庫操作方式。

## 三、EF 的物件導向特性

### 1. OO 優勢

#### 1.1 類別映射

🏷️ **物件對應**

ORM 的優勢是我們可以透過 OO 的方式來處理資料庫：
- 可以使用 **類別 (Class)** 來表示資料庫的 **表 (Table)**

#### 1.2 程式碼品質

📝 **開發效益**

- **無需直接操作 SQL 語句**
- **更具可讀性**：程式碼更容易理解
- **封裝性**：隱藏資料庫操作細節
- **可維護性**：降低維護成本和複雜度

## 四、Lazy Loading 延遲載入

### 1. 基本概念

#### 1.1 延遲載入機制

🔄 **Lazy Loading 運作原理**

當你第一次 `customer.Orders` 還沒用到時，EF 其實不會去查 Orders 資料表。只有當你真的去用到它時，EF 才去資料庫發出 SQL。

**觸發時機：**
- `customer.Orders.ToList()`
- `foreach (var o in customer.Orders)`

#### 1.2 效能優化

⚡ **為什麼使用 Lazy Loading**

因為有時候你只是想要 Customer 的名字，根本不需要 Orders。如果馬上載入 Orders（這叫 Eager Loading），就浪費了效能。

### 2. N+1 查詢問題

#### 2.1 問題描述

🚨 **N+1 查詢問題**

這個問題常常是 Lazy Loading 的副作用。假設你要列出所有 Customer 以及他們的 Orders：

```csharp
var customers = context.Customers.ToList();

foreach (var customer in customers)
{
    Console.WriteLine(customer.Name);

    foreach (var order in customer.Orders)
    {
        Console.WriteLine(order.OrderDate);
    }
}
```

**查詢次數分析：**

1. **第一次查詢：** `context.Customers.ToList()`
   - 這是第 1 次查詢，抓所有客戶

2. **後續查詢：** 當程式執行到 `customer.Orders` 時，因為是 Lazy Loading，每一個 customer 都會再跑一次 SQL 去查 Orders
   - 如果有 100 個客戶，就再查 100 次

**總查詢次數：** 1 + N = 1 + 100 = 101 次查詢

#### 2.2 解決方案

✅ **使用 Eager Loading (Include)**

如果你知道一開始就需要 Orders，可以使用 Eager Loading：

```csharp
var customers = context.Customers
    .Include(c => c.Orders)
    .ToList();
```

🎯 **優勢**
- **減少查詢次數**：從 N+1 次減少到 1 次
- **提升效能**：避免多次資料庫往返
- **明確意圖**：清楚表達需要載入的關聯資料

## 五、高頻交易適合 EF 嗎

### 1. 基本判斷

#### 1.1 EF Core 定位

📋 **溫和派選手**

EF Core 是溫和派選手，適合大多數系統。

#### 1.2 高頻交易需求

⚡ **極限挑戰**

高頻交易是極限挑戰，需要更裸機、更精簡、更預測性的操作。

### 2. EF Core 限制

#### 2.1 記憶體追蹤機制

❌ **ChangeTracker 負擔**

EF Core 有記憶體追蹤機制（ChangeTracker）會追蹤你每個查出來的 Entity 狀態，增加記憶體與 CPU 消耗。

**高頻寫入問題：**
- GC 負擔大幅上升
- 延遲不穩定
- 記憶體使用量增加

#### 2.2 SQL 生成控制

❌ **自動生成限制**

自動生成 SQL 難掌控效能細節，高頻交易需要手寫精準控制的 SQL 與索引使用。

**問題分析：**
- EF Core 雖然支援 raw SQL，但那樣就失去 ORM 的意義了
- 無法精確控制查詢計劃
- 難以優化到毫秒級效能

#### 2.3 連線管理瓶頸

❌ **DbContext 限制**

資料庫連線控管會成為瓶頸，EF Core 通常透過 DbContext 進行連線，而它是「非執行緒安全」的。

**衝突問題：**
- 高頻交易中常用連線池與持久連線優化機制
- 這與 EF 的設計相衝

### 3. 應用場景分析

#### 3.1 不適合場景

🚫 **避開 EF Core 的場景**

如果在打造類似以下系統，建議避開 EF Core：

- **電商秒殺**
- **即時報價**
- **金流配對引擎**

👉 **替代方案**：Dapper + 手寫 SQL + 精準控制資料庫索引與結構

#### 3.2 適合場景

✅ **EF Core 很適合**

- **一般業務系統**：CRUD-based 應用
- **複雜查詢但非毫秒級要求**：查詢複雜但效能不是毫秒級要求
- **快速開發需求**：想省時間開發、模型跟資料庫一一對應
- **可接受追蹤機制**：可接受記憶體中 tracking、Cache 的語意

## 六、DbContext 管理

### 1. 基本概念

#### 1.1 核心功能

🎯 **管理資料存取**

DbContext 是 Entity Framework 的核心元件，用來**管理資料存取**。

#### 1.2 主要組件

📋 **包含功能**

DbContext 包含以下核心功能：
- **資料庫連線 (Database Connection)**
- **快取 (Caching)**
- **變更追蹤 (Change Tracking)**

### 2. 生命週期問題

#### 2.1 記憶體洩漏

⚠️ **Change Tracking 問題**

DbContext 生命週期管理不當，可能會造成記憶體洩漏（因為 Change Tracking 會保留大量物件）。

📦 **情境描述**

```csharp
public class ProductService
{
    private readonly DbContext _db;

    public ProductService(DbContext db)
    {
        _db = db;
    }

    public void ProcessManyProducts()
    {
        foreach (var id in GetProductIds())
        {
            var product = _db.Products.Find(id);
            product.Price *= 1.1m;
        }

        _db.SaveChanges();
    }
}
```

🚨 **問題分析**

如果這個 foreach 處理了幾萬筆資料，EF Core 會：

1. **把每一筆 Entity 加進 Change Tracker 裡**（記憶體追蹤池）
2. **永遠不會清除**，直到整個 `_db` 被釋放

**記憶體洩漏的本質：**
- 你以為你只有一個變數，但 EF 幫你默默留了一堆物件引用
- 你的記憶體使用會飆升
- 長時間執行下會造成 GC 頻繁、甚至 OOM（Out of Memory）
- Debug 時你會看到大量「DbContext keeps object alive」的記憶體引用

✅ **解決方案**

**方案一：使用非追蹤查詢**
```csharp
var product = _db.Products.AsNoTracking().First(x => x.Id == id);
```

**方案二：處理一筆就 SaveChanges，再 Detach**
```csharp
_db.Entry(product).State = EntityState.Detached;
```

**方案三：考慮短生命週期的 DbContext**
- 或一批處理用 Dapper 替代

#### 2.2 資料一致性問題

⚠️ **交易範圍問題**

不同 DbContext 可能造成資料一致性問題（不同 DbContext 可能造成交易範圍問題）。

📦 **情境描述**

你有一個服務 OrderService 和 PaymentService，每個服務都使用各自的 DbContext（雖然都連到同一個資料庫）。

```csharp
public class OrderService
{
    public void CreateOrder()
    {
        using var db = new AppDbContext(); // DbContext A
        db.Orders.Add(new Order { ... });
        db.SaveChanges();
    }
}

public class PaymentService
{
    public void ChargeCustomer()
    {
        using var db = new AppDbContext(); // DbContext B
        db.Payments.Add(new Payment { ... });
        db.SaveChanges();
    }
}
```

**Controller 呼叫：**
```csharp
CreateOrder();
ChargeCustomer();
```

🚨 **問題分析**

1. **OrderService 的交易**在 DbContext A 裡完成了
2. **PaymentService 是另一個 DbContext B**，它的交易是新的

**結果：**
- ✅ Order 成功，但 ❌ Payment 失敗 → **資料不一致！**

✅ **解決方案**

**方案一：使用單一 DbContext 管理所有交易單位（Unit of Work）**

**方案二：使用 TransactionScope 跨 DbContext 明確管理交易**
```csharp
using var scope = new TransactionScope();
// ... 執行多個 DbContext 操作
scope.Complete();
```

## 七、ORM 防範 SQL Injection

### 1. SQL Injection 威脅

#### 1.1 攻擊原理

🚨 **SQL Injection 定義**

SQL Injection 是駭客在「輸入欄位」中輸入惡意的 SQL 語法，試圖控制或竄改資料庫。

#### 1.2 威脅本質

⚠️ **核心問題**

當你把使用者的輸入直接「拼接」到 SQL 指令裡，資料庫根本分不清楚哪些是資料、哪些是指令，這就會出事。

📋 **問題分析**

- **資料與指令混淆**：使用者輸入被誤認為 SQL 指令
- **直接字串拼接**：缺乏輸入驗證與參數化處理
- **安全性漏洞**：可能導致資料洩漏或資料庫被破壞

#### 1.3 危險範例

🚫 **不安全的做法（直接拼接）**

```csharp
string userInput = "'; DROP TABLE Users; --";
string sql = "SELECT * FROM Users WHERE Name = '" + userInput + "'";
// sql 最後會變成：SELECT * FROM Users WHERE Name = ''; DROP TABLE Users; --
// 然後 Users 資料表就被刪了！
```

🚨 **攻擊結果分析**

1. **原始 SQL**：`SELECT * FROM Users WHERE Name = ''; DROP TABLE Users; --`
2. **第一段**：`SELECT * FROM Users WHERE Name = ''` （空查詢）
3. **第二段**：`DROP TABLE Users` （刪除整個資料表！）
4. **第三段**：`--` （註解掉後續內容）

### 2. ORM 防護機制

#### 2.1 物件導向操作

✅ **使用 Entity Framework 的安全做法**

```csharp
using (var db = new MyDbContext())
{
    var userInput = "'; DROP TABLE Users; --";
    var users = db.Users.Where(u => u.Name == userInput).ToList();
}
```

🛡️ **防護原理**

ORM 會幫你把 `userInput` 當成**資料（data）**，而不是 SQL 的一部分：

- **資料庫**根本不會理會裡面那串奇怪的字
- **只當成查詢條件**來處理
- **完全隔離**指令與資料

#### 2.2 參數化查詢

🔒 **背後機制**

ORM 背後其實是把你的查詢轉換成像這樣的參數化 SQL：

```sql
SELECT * FROM Users WHERE Name = @p0
-- @p0 = "'; DROP TABLE Users; --"
```

📋 **參數化優勢**

- **@p0 是參數**：明確標示為資料值
- **資料庫會把它當成純文字資料**：不會解析為 SQL 指令
- **不會執行裡面的指令**：完全分離資料與邏輯
- **分離資料與邏輯的設計**：避免資料被誤當成命令來執行

#### 2.3 原始 SQL 安全

🛡️ **即使要寫原始 SQL，ORM 還是能保護你**

```csharp
var users = db.Users
    .FromSqlRaw("SELECT * FROM Users WHERE Name = {0}", userInput)
    .ToList();
```

✅ **安全保證**

這樣寫 ORM 還是會幫你用參數化方式包裝 `{0}`，一樣安全：

- **自動參數化**：ORM 自動將 `{0}` 轉換為安全參數
- **保持靈活性**：可以寫原始 SQL 但不失安全性
- **最佳實踐**：結合手寫 SQL 的靈活性與 ORM 的安全性