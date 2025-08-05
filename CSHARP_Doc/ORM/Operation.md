# ⚡ ORM 操作

## 📖 目錄
- [一、BulkInsert、BulkUpdate 批次操作](#一bulkinsertbulkupdate-批次操作)
  - [1. 基本概念](#1-基本概念)
    - [1.1 效能問題](#11-效能問題)
    - [1.2 批次處理優勢](#12-批次處理優勢)
  - [2. EF Core 原生限制](#2-ef-core-原生限制)
    - [2.1 預設行為](#21-預設行為)
    - [2.2 效能瓶頸](#22-效能瓶頸)
  - [3. 實際應用範例](#3-實際應用範例)
    - [3.1 BulkInsert 實作](#31-bulkinsert-實作)
    - [3.2 運作原理](#32-運作原理)
    - [3.3 Change Tracker 整合](#33-change-tracker-整合)
- [二、複雜查詢 - fromSqlRaw](#二複雜查詢---fromsqlraw)
  - [1. 基本概念](#1-基本概念-1)
    - [1.1 使用時機](#11-使用時機)
    - [1.2 避免 EF 轉換](#12-避免-ef-轉換)
- [三、確保索引可以被有效利用](#三確保索引可以被有效利用)
  - [1. 索引優化](#1-索引優化)
    - [1.1 索引設計原則](#11-索引設計原則)

---

## 一、BulkInsert、BulkUpdate 批次操作

### 1. 基本概念

#### 1.1 效能問題

⚠️ **SaveChanges() 的效能瓶頸**

每當你呼叫 `SaveChanges()`，預設都會為每一筆實體執行一次資料庫操作（例如一條 INSERT、UPDATE、DELETE）。

**每次操作的成本：**
- 開啟交易
- 寫入 Log
- 網路往返

#### 1.2 批次處理優勢

🚀 **批次處理機制**

批次處理將多筆操作集中在同一筆交易裡，降低交易啟動與鎖定（Lock）管理的成本。

**優勢：**
- 減少交易次數
- 降低網路往返成本
- 提升整體效能

### 2. EF Core 原生限制

#### 2.1 預設行為

📋 **原生 API 限制**

在純粹的 EF Core 原生 API 裡，確實並沒有像 BulkInsert 這樣一次性大批量寫入的方法。

#### 2.2 效能瓶頸

⚡ **底層實作問題**

即使 EF Core 幫你把 200 條 INSERT 包成一個封包，底層還是 200 條 `INSERT INTO … VALUES (…)`，而不是一次多筆值的 `INSERT INTO … VALUES (…),(…),(…)`。

**問題分析：**
- 每條 INSERT 還是要各自解析參數
- 資料庫只能把它們當成相似但不同的陳述式來重用計畫
- 效率不如真正的批量語句

### 3. 實際應用範例

#### 3.1 BulkInsert 實作

```csharp
public void BulkInsert(List<PromotionTagSlave> items, int batchSize = 0)
{
    foreach (var i in items)
    {
        this._webStoreReadWriteDBContext.PromotionTagSlave.Add(i);
    }

    this._webStoreReadWriteDBContext.BulkSaveChanges(bulk => bulk.BatchSize = batchSize);
}
```

#### 3.2 運作原理

🔧 **底層機制**

這個方法底層做了兩件事：

1. **批次拆包：** 它會把 Tracker 裡所有「Added」、「Modified」、「Deleted」的實體，按你指定的 BatchSize 分成多批次

2. **直接走 Bulk 路徑：** 每一批都呼叫一次真正的批量 SQL（像是 `INSERT INTO … VALUES (…), (…), …` 或底層 SqlBulkCopy），而不是一筆一筆執行

#### 3.3 Change Tracker 整合

🎯 **關鍵優勢**

因為所有實體都先經過了 EF Core 的 Change Tracker，任何你在 OnModelCreating、或是你程式碼裡為 CreatedUser、UpdatedUser 設定的預設值或 ValueGenerator，都已經在記憶體中被填好。

**重要特性：**
- 送到資料庫時，欄位值會「照你在程式裡設的」寫入
- 不會被資料庫的 default constraint 蓋掉
- 保持 EF Core 的所有驗證和轉換邏輯

## 二、複雜查詢 - fromSqlRaw

### 1. 基本概念

#### 1.1 使用時機

📋 **適用場景**

`fromSqlRaw` 叫用 CSP（Compound SQL Procedure）是一種方法，用於處理複雜的查詢需求。

#### 1.2 避免 EF 轉換

⚡ **核心目的**

避免 EF 轉成能以預測的 SQL Query，提供更精確的查詢控制。

**主要優勢：**
- 直接控制 SQL 語句
- 避免 EF 的查詢轉換
- 確保查詢效能的可預測性
- 支援複雜的查詢邏輯

## 三、確保索引可以被有效利用

### 1. 索引優化

#### 1.1 索引設計原則

🎯 **效能關鍵**

確保索引可以被有效利用是資料庫效能優化的關鍵要素。

**重要考量：**
- 分析查詢模式
- 設計適當的索引策略
- 監控索引使用情況
- 定期檢視索引效能