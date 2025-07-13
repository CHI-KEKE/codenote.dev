## 📖 目錄
- [📖 目錄](#-目錄)
- [💳 金流統計進階分析](#-金流統計進階分析)
  - [💎 每種金流每年月的平均消費（需大於1000）](#-每種金流每年月的平均消費需大於1000)
    - [📅 按日期分組](#-按日期分組)
    - [📈 按年月分組](#-按年月分組)
    - [🗓️ 按年-月字串分組](#️-按年-月字串分組)
- [📈 複雜統計分析](#-複雜統計分析)
  - [💎 每小時付款成功率](#-每小時付款成功率)
    - [🎯 基本版本](#-基本版本)
    - [🚀 進階版本：自訂時間校正](#-進階版本自訂時間校正)
- [🎯 效能優化進階技巧](#-效能優化進階技巧)
  - [⚡ 索引策略與 SARGable 查詢](#-索引策略與-sargable-查詢)
    - [什麼是 SARGable？](#什麼是-sargable)
    - [📈 效能優化實例](#-效能優化實例)
  - [🔍 暫存表與批次處理](#-暫存表與批次處理)
    - [暫存表的有效運用](#暫存表的有效運用)
  - [📊 CTE 與子查詢優化](#-cte-與子查詢優化)
    - [Common Table Expression (CTE) 的最佳實踐](#common-table-expression-cte-的最佳實踐)
- [🛠️ 實戰效能調校案例](#️-實戰效能調校案例)
  - [📋 案例一：大量資料的時間範圍查詢優化](#-案例一大量資料的時間範圍查詢優化)
    - [問題場景](#問題場景)
    - [優化方案](#優化方案)
  - [📋 案例二：複雜統計查詢的分解優化](#-案例二複雜統計查詢的分解優化)
    - [問題場景](#問題場景-1)
    - [優化方案](#優化方案-1)
- [🏆 效能優化總結](#-效能優化總結)
  - [🎯 關鍵原則](#-關鍵原則)
  - [📊 效能檢查清單](#-效能檢查清單)
- [🔗 相關文件](#-相關文件)

---

## 💳 金流統計進階分析

### 💎 每種金流每年月的平均消費（需大於1000）

這個案例展示了如何使用不同的分組策略來分析金流資料，每種方法都有其適用場景：

#### 📅 按日期分組
```sql
USE WebStoreDB;

-- 🎯 分頁設定（第一頁顯示今年）
DECLARE @pageNum INT = 1;
DECLARE @targetYear INT = YEAR(GETDATE()) - (@pageNum - 1);
DECLARE @startDate DATETIME = DATEFROMPARTS(@targetYear, 1, 1);
DECLARE @endDate DATETIME = DATEFROMPARTS(@targetYear + 1, 1, 1);

SELECT 
    TradesOrderThirdPartyPayment_TypeDef AS 金流類型,
    CAST(TradesOrderThirdPartyPayment_DateTime AS DATE) AS 付款日期,
    AVG(TradesOrderThirdPartyPayment_TotalPayment) AS 平均消費
FROM TradesOrderThirdPartyPayment(NOLOCK)
WHERE TradesOrderThirdPartyPayment_ValidFlag = 1
    AND TradesOrderThirdPartyPayment_DateTime >= @startDate
    AND TradesOrderThirdPartyPayment_DateTime < @endDate
GROUP BY 
    TradesOrderThirdPartyPayment_TypeDef, 
    CAST(TradesOrderThirdPartyPayment_DateTime AS DATE)
HAVING AVG(TradesOrderThirdPartyPayment_TotalPayment) > 1000
ORDER BY 付款日期;
```

> 📊 **適用場景**：需要查看每日詳細數據的趨勢分析

#### 📈 按年月分組
```sql
SELECT 
    TradesOrderThirdPartyPayment_TypeDef AS 金流類型,
    YEAR(TradesOrderThirdPartyPayment_DateTime) AS 年份,
    MONTH(TradesOrderThirdPartyPayment_DateTime) AS 月份,
    AVG(TradesOrderThirdPartyPayment_TotalPayment) AS 平均消費
FROM TradesOrderThirdPartyPayment(NOLOCK)
WHERE TradesOrderThirdPartyPayment_ValidFlag = 1
    AND TradesOrderThirdPartyPayment_DateTime >= @startDate
    AND TradesOrderThirdPartyPayment_DateTime < @endDate
GROUP BY 
    TradesOrderThirdPartyPayment_TypeDef,
    YEAR(TradesOrderThirdPartyPayment_DateTime),
    MONTH(TradesOrderThirdPartyPayment_DateTime)
HAVING AVG(TradesOrderThirdPartyPayment_TotalPayment) > 1000
ORDER BY 年份, 月份;
```

> 📊 **適用場景**：需要在報表中分別顯示年份和月份欄位

#### 🗓️ 按年-月字串分組
```sql
SELECT 
    TradesOrderThirdPartyPayment_TypeDef AS 金流類型,
    CONVERT(CHAR(7), TradesOrderThirdPartyPayment_DateTime, 120) AS 年月,
    AVG(TradesOrderThirdPartyPayment_TotalPayment) AS 平均消費
FROM TradesOrderThirdPartyPayment(NOLOCK)
WHERE TradesOrderThirdPartyPayment_ValidFlag = 1
    AND TradesOrderThirdPartyPayment_DateTime >= @startDate
    AND TradesOrderThirdPartyPayment_DateTime < @endDate
GROUP BY 
    TradesOrderThirdPartyPayment_TypeDef, 
    CONVERT(CHAR(7), TradesOrderThirdPartyPayment_DateTime, 120)
HAVING AVG(TradesOrderThirdPartyPayment_TotalPayment) > 1000
ORDER BY 年月;
```

> 📊 **適用場景**：需要緊湊格式顯示，適合圖表展示

---

## 📈 複雜統計分析

### 💎 每小時付款成功率

這是一個複雜的商業分析案例，用於監控支付系統的穩定性和成功率。

#### 🎯 基本版本
```sql
USE WebStoreDB;

SELECT 
    DATEADD(HOUR, DATEDIFF(HOUR, 0, TradesOrderThirdPartyPayment_DateTime), 0) AS 小時區間,
    COUNT(*) AS 總筆數,
    SUM(CASE WHEN TradesOrderThirdPartyPayment_StatusDef = 'Success' THEN 1 ELSE 0 END) AS 成功筆數,
    CAST(1.0 * SUM(CASE WHEN TradesOrderThirdPartyPayment_StatusDef = 'Success' THEN 1 ELSE 0 END) / COUNT(*) 
         AS DECIMAL(10,2)) AS 成功率
FROM TradesOrderThirdPartyPayment(NOLOCK)
WHERE TradesOrderThirdPartyPayment_ValidFlag = 1
    AND TradesOrderThirdPartyPayment_TypeDef = 'CreditCardOnce_Stripe'
    AND TradesOrderThirdPartyPayment_DateTime > '2025-01-01 09:00'
GROUP BY DATEADD(HOUR, DATEDIFF(HOUR, 0, TradesOrderThirdPartyPayment_DateTime), 0)
ORDER BY 小時區間;
```

> 🔍 **技術解析**：
> - `DATEADD(HOUR, DATEDIFF(HOUR, 0, datetime), 0)` 是將時間截斷到小時的經典技巧
> - 使用 `1.0 *` 確保除法結果為小數而非整數

#### 🚀 進階版本：自訂時間校正
```sql
-- 🎛️ 可調整的金流類型
-- EWallet_PayMe, CreditCardOnce_Stripe
DECLARE @payType VARCHAR(30) = 'CreditCardOnce_Stripe';

WITH PaymentStats AS (
    SELECT 
        -- ⏰ 自訂時間校正：減去23分35秒
        DATEADD(HOUR, DATEDIFF(HOUR, 0, 
                DATEADD(SECOND, -35, DATEADD(MINUTE, -23, TradesOrderThirdPartyPayment_CreatedDateTime))
               ), 0) AS CustomHourly,
        SUM(CASE WHEN TradesOrderThirdPartyPayment_StatusDef NOT IN ('WaitingToPay','Hidden') 
                 THEN 1 ELSE 0 END) AS Total_Count,
        SUM(CASE WHEN TradesOrderThirdPartyPayment_StatusDef IN ('Success','RePaySuccess','AuthSuccess','CancelAfterSuccess') 
                 THEN 1 ELSE 0 END) AS Success_Count,
        SUM(CASE WHEN TradesOrderThirdPartyPayment_StatusDef = 'Fail' 
                 THEN 1 ELSE 0 END) AS Fail_Count,
        SUM(CASE WHEN TradesOrderThirdPartyPayment_StatusDef = 'Timeout' 
                 THEN 1 ELSE 0 END) AS Timeout_Count,
        SUM(CASE WHEN TradesOrderThirdPartyPayment_StatusDef = 'CancelRequest' 
                 THEN 1 ELSE 0 END) AS CancelRequest_Count
    FROM TradesOrderThirdPartyPayment (NOLOCK)
    WHERE TradesOrderThirdPartyPayment_TypeDef = @payType
        AND TradesOrderThirdPartyPayment_ValidFlag = 1
        AND TradesOrderThirdPartyPayment_CreatedDateTime >= '2025-03-25'
        AND TradesOrderThirdPartyPayment_CreatedDateTime < '2025-06-25'
    GROUP BY DATEADD(HOUR, DATEDIFF(HOUR, 0, 
                    DATEADD(SECOND, -35, DATEADD(MINUTE, -23, TradesOrderThirdPartyPayment_CreatedDateTime))
                   ), 0)
)
SELECT *,
       CAST(CAST(Success_Count * 100.0 / Total_Count AS DECIMAL(10,2)) AS VARCHAR(30)) + '%' AS 成功率,
       CAST(CAST(Fail_Count * 100.0 / Total_Count AS DECIMAL(10,2)) AS VARCHAR(30)) + '%' AS 失敗率,
       DATEADD(SECOND, 35, DATEADD(MINUTE, 23, CustomHourly)) AS 開始時間,
       DATEADD(SECOND, 35, DATEADD(MINUTE, 23 + 60, CustomHourly)) AS 結束時間
FROM PaymentStats
WHERE Total_Count > 10 
    -- 📊 可選的成功率篩選條件
    -- AND (Success_Count * 1.0 / NULLIF(Total_Count,0)) < 0.6
    -- AND (Success_Count * 1.0 / NULLIF(Total_Count,0)) > 0.5
ORDER BY CustomHourly;
```

> 🎯 **進階特性**：
> 1. **時間校正**：考慮到系統時區或延遲，可以對時間進行微調
> 2. **多狀態統計**：詳細分析各種付款狀態的分布
> 3. **彈性篩選**：可根據交易量或成功率進行過濾
> 4. **可讀性優化**：將百分比格式化為易讀的字串格式

---

## 🎯 效能優化進階技巧

### ⚡ 索引策略與 SARGable 查詢

#### 什麼是 SARGable？
**SARGable** = **S**earch **ARG**ument **able**，意思是查詢條件能夠有效利用索引。

```sql
-- ✅ SARGable 查詢 - 可以有效利用索引
WHERE CreatedDateTime >= '2025-01-01' 
    AND CreatedDateTime < '2025-02-01'
WHERE UserId = 12345
WHERE Status IN ('Active', 'Pending')

-- ❌ Non-SARGable 查詢 - 無法有效利用索引
WHERE YEAR(CreatedDateTime) = 2025          -- 在欄位上使用函式
WHERE MONTH(CreatedDateTime) = 1            -- 在欄位上使用函式
WHERE UserName LIKE '%John%'               -- 前置萬用字元
WHERE UserId + 10 = 12355                  -- 在欄位上進行運算
```

#### 📈 效能優化實例
```sql
-- ❌ 效能較差：Non-SARGable
SELECT * FROM Orders 
WHERE YEAR(OrderDate) = 2025 AND MONTH(OrderDate) = 7;

-- ✅ 效能較佳：SARGable  
DECLARE @startDate DATE = '2025-07-01';
DECLARE @endDate DATE = '2025-08-01';

SELECT * FROM Orders 
WHERE OrderDate >= @startDate AND OrderDate < @endDate;
```

### 🔍 暫存表與批次處理

#### 暫存表的有效運用
```sql
-- 💡 處理大量資料時的最佳實踐
DROP TABLE IF EXISTS #MonthlyStats;
CREATE TABLE #MonthlyStats (
    YearMonth CHAR(7),
    TotalAmount DECIMAL(18,2),
    OrderCount INT,
    AvgAmount DECIMAL(18,2),
    INDEX IX_YearMonth (YearMonth)  -- 建立索引提升查詢效能
);

-- 分批處理避免長時間鎖定
DECLARE @batchSize INT = 10000;
DECLARE @offset INT = 0;
DECLARE @rowCount INT = 1;

WHILE @rowCount > 0
BEGIN
    INSERT INTO #MonthlyStats (YearMonth, TotalAmount, OrderCount, AvgAmount)
    SELECT 
        CONVERT(CHAR(7), OrderDate, 120),
        SUM(Amount),
        COUNT(*),
        AVG(Amount)
    FROM Orders
    WHERE OrderId BETWEEN @offset + 1 AND @offset + @batchSize
    GROUP BY CONVERT(CHAR(7), OrderDate, 120);
    
    SET @rowCount = @@ROWCOUNT;
    SET @offset = @offset + @batchSize;
END;
```

### 📊 CTE 與子查詢優化

#### Common Table Expression (CTE) 的最佳實踐
```sql
-- 🎯 複雜分析使用 CTE 提升可讀性
WITH MonthlyRevenue AS (
    -- 第一層：計算每月收入
    SELECT 
        YEAR(OrderDate) AS OrderYear,
        MONTH(OrderDate) AS OrderMonth,
        SUM(Amount) AS MonthlyTotal,
        COUNT(*) AS OrderCount
    FROM Orders 
    WHERE OrderDate >= '2024-01-01'
    GROUP BY YEAR(OrderDate), MONTH(OrderDate)
),
RevenueWithGrowth AS (
    -- 第二層：計算成長率
    SELECT *,
        LAG(MonthlyTotal) OVER (ORDER BY OrderYear, OrderMonth) AS PrevMonthTotal,
        MonthlyTotal - LAG(MonthlyTotal) OVER (ORDER BY OrderYear, OrderMonth) AS GrowthAmount
    FROM MonthlyRevenue
)
-- 第三層：最終結果
SELECT *,
    CASE 
        WHEN PrevMonthTotal IS NULL THEN NULL
        WHEN PrevMonthTotal = 0 THEN NULL
        ELSE CAST(GrowthAmount * 100.0 / PrevMonthTotal AS DECIMAL(5,2))
    END AS GrowthPercentage
FROM RevenueWithGrowth
ORDER BY OrderYear, OrderMonth;
```

---

## 🛠️ 實戰效能調校案例

### 📋 案例一：大量資料的時間範圍查詢優化

#### 問題場景
```sql
-- ❌ 原始查詢：效能較差
SELECT COUNT(*) 
FROM LargeTable 
WHERE YEAR(CreateDate) = 2025 AND MONTH(CreateDate) BETWEEN 1 AND 6;
```

#### 優化方案
```sql
-- ✅ 優化後：使用範圍查詢
DECLARE @startDate DATE = '2025-01-01';
DECLARE @endDate DATE = '2025-07-01';

SELECT COUNT(*) 
FROM LargeTable 
WHERE CreateDate >= @startDate AND CreateDate < @endDate;

-- 📊 建議的索引
-- CREATE INDEX IX_LargeTable_CreateDate ON LargeTable(CreateDate) INCLUDE (OtherColumns);
```

### 📋 案例二：複雜統計查詢的分解優化

#### 問題場景
```sql
-- ❌ 原始查詢：複雜且難以優化
SELECT 
    YEAR(OrderDate) AS Year,
    MONTH(OrderDate) AS Month,
    COUNT(*) AS OrderCount,
    SUM(Amount) AS TotalAmount,
    AVG(Amount) AS AvgAmount,
    COUNT(DISTINCT CustomerId) AS UniqueCustomers,
    MAX(Amount) AS MaxAmount,
    MIN(Amount) AS MinAmount
FROM Orders o
WHERE OrderDate >= '2024-01-01' 
    AND EXISTS (SELECT 1 FROM Customers c WHERE c.Id = o.CustomerId AND c.Status = 'Active')
GROUP BY YEAR(OrderDate), MONTH(OrderDate)
ORDER BY Year, Month;
```

#### 優化方案
```sql
-- ✅ 優化後：分解成多個步驟
-- 步驟一：先篩選有效客戶的訂單
WITH ValidOrders AS (
    SELECT o.OrderDate, o.Amount, o.CustomerId
    FROM Orders o
    INNER JOIN Customers c ON c.Id = o.CustomerId 
    WHERE o.OrderDate >= '2024-01-01'
        AND c.Status = 'Active'
)
-- 步驟二：進行統計分析
SELECT 
    YEAR(OrderDate) AS Year,
    MONTH(OrderDate) AS Month,
    COUNT(*) AS OrderCount,
    SUM(Amount) AS TotalAmount,
    AVG(Amount) AS AvgAmount,
    COUNT(DISTINCT CustomerId) AS UniqueCustomers,
    MAX(Amount) AS MaxAmount,
    MIN(Amount) AS MinAmount
FROM ValidOrders
GROUP BY YEAR(OrderDate), MONTH(OrderDate)
ORDER BY Year, Month;
```

---

## 🏆 效能優化總結

### 🎯 關鍵原則
1. **SARGable 查詢**：避免在 WHERE 條件中對索引欄位使用函式
2. **索引策略**：合適的索引是效能的基礎
3. **批次處理**：大量資料處理時使用暫存表和批次操作
4. **查詢分解**：複雜查詢分解成多個簡單步驟

### 📊 效能檢查清單
- [ ] 查詢是否使用了 SARGable 條件？
- [ ] 是否有適當的索引支援查詢？
- [ ] 大量資料是否使用批次處理？
- [ ] 複雜邏輯是否適合用 CTE 分解？
- [ ] 是否避免了不必要的函式呼叫？

---

## 🔗 相關文件

- [SQL 時間類型與基本操作完整指南](./SQL_時間類型與基本操作完整指南.md)
- [SQL 時間查詢基礎與統計技巧](./SQL_時間查詢基礎與統計技巧.md)

---

*🚀 掌握這些進階技巧，讓您的 SQL 查詢不僅功能強大，效能也更加卓越！*
