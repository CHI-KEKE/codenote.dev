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