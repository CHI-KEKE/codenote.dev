## 📖 目錄
- [📖 目錄](#-目錄)
- [🗓️ 月份查詢最佳實踐](#️-月份查詢最佳實踐)
  - [🎯 查詢每個月初註冊的會員](#-查詢每個月初註冊的會員)
    - [❌ 可讀性佳但效能差的寫法](#-可讀性佳但效能差的寫法)
    - [✅ 效能優化的寫法：迴圈批次查詢](#-效能優化的寫法迴圈批次查詢)
- [📊 時間統計分析](#-時間統計分析)
  - [💰 今年每個月總金額](#-今年每個月總金額)
  - [📦 平均出貨處理天數](#-平均出貨處理天數)
- [🎯 進階時間計算](#-進階時間計算)
  - [👴 退休日計算（65歲生日）](#-退休日計算65歲生日)
  - [👥 超過30天未更新資料的客戶](#-超過30天未更新資料的客戶)
- [📈 週期性統計分析](#-週期性統計分析)
  - [🏪 今年每個月的訂單數量](#-今年每個月的訂單數量)
    - [🔢 方法一：MONTH 函式](#-方法一month-函式)
    - [📅 方法二：CONVERT 函式](#-方法二convert-函式)
  - [📊 週平均消費](#-週平均消費)
- [🎯 基礎效能提醒](#-基礎效能提醒)
  - [⚡ 重要原則](#-重要原則)
  - [💡 實用技巧](#-實用技巧)
- [🚀 下一步學習](#-下一步學習)

---

## 🗓️ 月份查詢最佳實踐

### 🎯 查詢每個月初註冊的會員

#### ❌ 可讀性佳但效能差的寫法
```sql
-- 問題：Non-SARGable，無法有效利用索引
USE WebStoreDB;

SELECT VipMember_CreatedDateTime, 
       DAY(VipMember_CreatedDateTime), 
       VipMember_Id
FROM VipMember(NOLOCK)
WHERE VipMember_ValidFlag = 1
    AND DAY(VipMember_CreatedDateTime) = 1;
```

> ⚠️ **效能問題**：這種寫法會導致全表掃描，因為 `DAY()` 函式讓 SQL Server 無法有效利用索引。

#### ✅ 效能優化的寫法：迴圈批次查詢
```sql
USE WebStoreDB;

-- 🏗️ 建立暫存表
DROP TABLE IF EXISTS #VipMember_FirstDay;
CREATE TABLE #VipMember_FirstDay(
    CreateDate DATETIME,
    Id BIGINT
);

-- 🔄 批次處理每個月
DECLARE @year INT = YEAR(GETDATE()),
        @month INT = 1,
        @startDate DATETIME,
        @endDate DATETIME;

WHILE @month <= 12
BEGIN
    SET @startDate = DATEFROMPARTS(@year, @month, 1);
    SET @endDate = DATEADD(DAY, 1, @startDate);

    INSERT INTO #VipMember_FirstDay(CreateDate, Id)
    SELECT VipMember_CreatedDateTime, VipMember_Id
    FROM VipMember(NOLOCK)
    WHERE VipMember_CreatedDateTime >= @startDate
        AND VipMember_CreatedDateTime < @endDate;

    SET @month = @month + 1;
END

SELECT * FROM #VipMember_FirstDay;
```

> ✅ **效能優勢**：使用範圍查詢可以有效利用日期欄位上的索引，大幅提升查詢效能。

---

## 📊 時間統計分析

### 💰 今年每個月總金額
```sql
USE WebStoreDB;

SELECT 
    CONVERT(CHAR(7), TradesOrderGroup_DateTime, 120) AS 月份,
    SUM(TradesOrderGroup_TotalPayment) AS 總金額
FROM TradesOrderGroup(NOLOCK)
WHERE TradesOrderGroup_DateTime >= DATEFROMPARTS(YEAR(GETDATE()), 1, 1)
    AND TradesOrderGroup_DateTime < DATEFROMPARTS(YEAR(GETDATE()) + 1, 1, 1)
GROUP BY CONVERT(CHAR(7), TradesOrderGroup_DateTime, 120)
ORDER BY CONVERT(CHAR(7), TradesOrderGroup_DateTime, 120);
```

> 💡 **小技巧**：`CONVERT(CHAR(7), datetime, 120)` 會產生 'YYYY-MM' 格式，非常適合按月分組。

### 📦 平均出貨處理天數
```sql
USE WebStoreDB;

-- 📈 方法一：按年月分組
SELECT CONVERT(VARCHAR(7), OrderSlaveFlow_CreatedDateTime, 120) AS 年月,
       AVG(DATEDIFF(DAY, OrderSlaveFlow_CreatedDateTime, OrderSlaveFlow_ShippingOrderSlaveDateTime)) AS 平均出貨天數
FROM OrderSlaveFlow(NOLOCK)
WHERE OrderSlaveFlow_ValidFlag = 1
GROUP BY CONVERT(VARCHAR(7), OrderSlaveFlow_CreatedDateTime, 120)
ORDER BY CONVERT(VARCHAR(7), OrderSlaveFlow_CreatedDateTime, 120);

-- 📊 方法二：年份和月份分別顯示
SELECT YEAR(OrderSlaveFlow_CreatedDateTime) AS 年份,
       MONTH(OrderSlaveFlow_CreatedDateTime) AS 月份,
       AVG(DATEDIFF(DAY, OrderSlaveFlow_CreatedDateTime, OrderSlaveFlow_ShippingOrderSlaveDateTime)) AS 平均出貨天數
FROM OrderSlaveFlow(NOLOCK)
WHERE OrderSlaveFlow_ValidFlag = 1
GROUP BY YEAR(OrderSlaveFlow_CreatedDateTime), MONTH(OrderSlaveFlow_CreatedDateTime)
ORDER BY YEAR(OrderSlaveFlow_CreatedDateTime), MONTH(OrderSlaveFlow_CreatedDateTime);
```

---

## 🎯 進階時間計算

### 👴 退休日計算（65歲生日）
```sql
USE WebStoreDB;

SELECT VipMemberInfo_Id,
       VipMemberInfo_FullName,
       VipMemberInfo_Birthday,
       DATEDIFF(DAY, GETDATE(), DATEADD(YEAR, 65, VipMemberInfo_Birthday)) AS 距離退休天數,
       CASE 
           WHEN DATEDIFF(YEAR, GETDATE(), DATEADD(YEAR, 65, VipMemberInfo_Birthday)) > 30 
           THEN '還有超過30年，很可憐 😢' 
           ELSE '即將退休 🎉' 
       END AS 退休狀態
FROM VipMemberInfo(NOLOCK);
```

> 🎯 **商業應用**：這類查詢常用於 HR 系統，幫助規劃人力資源和退休金準備。

### 👥 超過30天未更新資料的客戶
```sql
SELECT VipMemberInfo_FullName,
       VipMemberInfo_UpdatedDateTime,
       DATEDIFF(DAY, VipMemberInfo_UpdatedDateTime, GETDATE()) AS 未更新天數
FROM VipMemberInfo(NOLOCK)
WHERE DATEDIFF(DAY, VipMemberInfo_UpdatedDateTime, GETDATE()) > 30
ORDER BY DATEDIFF(DAY, VipMemberInfo_UpdatedDateTime, GETDATE()) DESC;
```

> 🔍 **資料品質管控**：定期執行此查詢可以找出長期未維護的客戶資料，提醒進行資料清理。

---

## 📈 週期性統計分析

### 🏪 今年每個月的訂單數量

#### 🔢 方法一：MONTH 函式
```sql
USE WebStoreDB;

SELECT MONTH(TradesOrderGroup_DateTime) AS 月份,
       COUNT(*) AS 訂單數量
FROM TradesOrderGroup(NOLOCK)
WHERE TradesOrderGroup_ValidFlag = 1
    AND TradesOrderGroup_DateTime >= '2025-01-01'
GROUP BY MONTH(TradesOrderGroup_DateTime)
ORDER BY MONTH(TradesOrderGroup_DateTime);
```

#### 📅 方法二：CONVERT 函式
```sql
SELECT CONVERT(VARCHAR(7), TradesOrderGroup_DateTime, 120) AS 年月,
       COUNT(*) AS 訂單數量
FROM TradesOrderGroup(NOLOCK)
WHERE TradesOrderGroup_ValidFlag = 1
    AND TradesOrderGroup_DateTime >= '2025-01-01'
GROUP BY CONVERT(VARCHAR(7), TradesOrderGroup_DateTime, 120)
ORDER BY CONVERT(VARCHAR(7), TradesOrderGroup_DateTime, 120);
```

> 🤔 **選擇建議**：
> - **方法一**：適合單一年份分析，結果僅顯示月份數字
> - **方法二**：適合跨年份分析，結果包含年份和月份

### 📊 週平均消費
```sql
USE ERPDB;

WITH A AS (
    SELECT SalesOrderGroup_DateTime, SalesOrderGroup_TotalPayment
    FROM SalesOrderGroup(NOLOCK)
    WHERE SalesOrderGroup_ValidFlag = 1
        AND SalesOrderGroup_DateTime >= '2025-06-01'
        AND SalesOrderGroup_DateTime < '2025-07-01'
)
SELECT 
    DATEPART(WEEK, SalesOrderGroup_DateTime) - 
    DATEPART(WEEK, DATEADD(MONTH, DATEDIFF(MONTH, 0, SalesOrderGroup_DateTime), 0)) + 1 AS 當月第幾週,
    AVG(SalesOrderGroup_TotalPayment) AS 週平均消費
FROM A
GROUP BY DATEPART(WEEK, SalesOrderGroup_DateTime) - 
         DATEPART(WEEK, DATEADD(MONTH, DATEDIFF(MONTH, 0, SalesOrderGroup_DateTime), 0)) + 1;
```

> 🧮 **複雜計算解析**：
> 1. `DATEPART(WEEK, SalesOrderGroup_DateTime)` - 取得該日期是年度第幾週
> 2. `DATEADD(MONTH, DATEDIFF(MONTH, 0, SalesOrderGroup_DateTime), 0)` - 取得該月第一天
> 3. `DATEPART(WEEK, 月初)` - 取得該月第一天是年度第幾週
> 4. 相減後加1 = 該日期在當月是第幾週

---

## 🎯 基礎效能提醒

### ⚡ 重要原則
```sql
-- ✅ 好的寫法：使用範圍查詢
WHERE datetime_column >= @startDate 
    AND datetime_column < @endDate

-- ❌ 避免的寫法：在欄位上使用函式
WHERE YEAR(datetime_column) = 2025
WHERE DAY(datetime_column) = 1
```

### 💡 實用技巧
1. **索引友善**：避免在 WHERE 條件中對日期欄位使用函式
2. **範圍查詢**：使用 `>=` 和 `<` 進行日期範圍篩選
3. **暫存表**：處理大量資料時考慮使用暫存表分批處理

---

## 🚀 下一步學習

想要學習更複雜的分析技巧嗎？請參考：
- [SQL 進階時間分析與效能優化指南](./SQL_進階時間分析與效能優化指南.md)

---

*📝 提示：這份指南涵蓋了日常開發中80%的時間查詢需求，熟練掌握這些技巧將大幅提升您的 SQL 開發效率！*
