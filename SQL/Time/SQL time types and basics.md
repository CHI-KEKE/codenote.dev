## 📖 目錄
1. [時間資料類型比較](#時間資料類型比較)
2. [基本時間操作](#基本時間操作)
3. [時間格式轉換](#時間格式轉換)
4. [時間計算秘訣](#時間計算秘訣)
5. [特定時間點組合](#特定時間點組合)

---

## 🎯 時間資料類型比較

### DATE 類型 📆
```sql
-- 範圍：0001-01-01 到 9999-12-31
-- 精度：僅包含日期部分（年、月、日）
-- 存儲大小：3 個位元組

DECLARE @warrantyPeriod DATE = GETDATE() - @warrantyPeriodDays; -- 超過鑑賞期N天
```

### DATETIME 類型 🕐
```sql
-- 範圍：1753-01-01 到 9999-12-31
-- 精度：包含日期和時間部分（年、月、日、時、分、秒和毫秒）
-- 存儲大小：8 個位元組

-- 標準時間格式範例
DECLARE @sampleTime DATETIME = '2024-01-30T09:40:00';
```

---

## ⚡ 基本時間操作

### 🧮 時間加減 - DATEADD
```sql
-- 語法：DATEADD(時間單位, 數值, 日期)
-- 常用時間單位對照表
/*
year    年      yy 或 yyyy
month   月      mm 或 m
day     日      dd 或 d
hour    小時    hh
minute  分鐘    mi 或 n
second  秒      ss 或 s
*/

-- 📝 實用範例
SELECT DATEADD(DAY, -185, SalesOrderThirdPartyPayment_DateTime) AS '185天前';
SELECT DATEADD(mi, -6, GETDATE()) AS '6分鐘前'; -- mi = minute

-- 💡 假設現在是 3:00，結果會是 2:54
DECLARE @startDateTime DATETIME = DATEADD(mi, -6, GETDATE());
```

### 📏 時間差計算 - DATEDIFF
```sql
-- 語法：DATEDIFF(時間單位, 開始時間, 結束時間)

-- 計算執行時間（毫秒）
DECLARE @startTime DATETIME = GETDATE();
-- ... 您的程式碼 ...
DECLARE @endTime DATETIME = GETDATE();
SELECT DATEDIFF(MILLISECOND, @startTime, @endTime) AS '執行時間(毫秒)';

-- 計算天數差
SELECT DATEDIFF(DAY, '2024-01-01', GETDATE()) AS '距離新年天數';
```

---

## 🔄 時間格式轉換

### CONVERT 函式的神奇用法
```sql
-- CONVERT(VARCHAR(10), GETDATE(), 120) 
-- 120 格式：YYYY-MM-DD

-- 📅 取得年月格式
SELECT CONVERT(CHAR(7), GETDATE(), 120) AS '年月'; -- 結果：2025-01

-- 📊 分組統計常用技巧
SELECT 
    CONVERT(VARCHAR(7), TradesOrderGroup_DateTime, 120) AS 月份,
    COUNT(*) AS 訂單數量
FROM TradesOrderGroup(NOLOCK)
WHERE TradesOrderGroup_ValidFlag = 1
    AND TradesOrderGroup_DateTime >= '2025-01-01'
GROUP BY CONVERT(VARCHAR(7), TradesOrderGroup_DateTime, 120);
```

---

## 🎨 特定時間點組合

### 🌅 轉換成當天特定時間
> ✅ **用途**：將目前日期與自訂時間組合為 DATETIME，常見於：
> - 計算某天特定時間的資料（例如：每天中午12點前的訂單）
> - 設定排程時間點
> - 比對時間段範圍

#### 方法一：CONVERT + CAST 組合技
```sql
USE WebStoreDB;

-- 🎯 步驟解析
-- 1. CONVERT(VARCHAR(10), GETDATE(), 120) → 取得日期字串 '2025-01-13'
-- 2. + ' 12:00:00' → 組合時間 '2025-01-13 12:00:00'  
-- 3. CAST(...AS DATETIME) → 轉換回 DATETIME 格式

SELECT CAST(CONVERT(VARCHAR(10), GETDATE(), 120) + ' 12:00:00' AS DATETIME) AS 中午;
SELECT CAST(CONVERT(VARCHAR(10), GETDATE(), 120) + ' 00:01:00' AS DATETIME) AS 凌晨;
SELECT CAST(CONVERT(VARCHAR(10), GETDATE(), 120) + ' 21:45:00' AS DATETIME) AS 夜晚;
```

#### 方法二：DATEDIFF + DATEADD 組合技
```sql
-- 🧩 原理解析
-- 1. DATEDIFF(DAY, 0, GETDATE()) → 計算距離1900-01-01的天數
-- 2. DATEADD(HOUR, 12, ...) → 在午夜基礎上加12小時

SELECT DATEADD(HOUR, 12, DATEDIFF(DAY, 0, GETDATE())) AS 中午12點;
```

---

## 🔧 實用時間函式組合

### DATEFROMPARTS - 組合年月日
```sql
-- 📅 建立指定年月日
DECLARE @year INT = YEAR(GETDATE());
DECLARE @month INT = 1;

SELECT DATEFROMPARTS(@year, @month, 1) AS '今年一月一日';
SELECT DATEFROMPARTS(@year + 1, 1, 1) AS '明年一月一日';
```

### 時間單位提取函式
```sql
-- 🗓️ 提取各種時間單位
SELECT 
    YEAR(GETDATE()) AS 年份,
    MONTH(GETDATE()) AS 月份,
    DAY(GETDATE()) AS 日期,
    DATEPART(WEEK, GETDATE()) AS 週數;
```

---

## 💡 專業小貼士

### ⚠️ 效能注意事項
```sql
-- ❌ 效能較差的寫法（Non-SARGable）
WHERE DAY(VipMember_CreatedDateTime) = 1

-- ✅ 效能較佳的寫法（可以使用索引）
WHERE VipMember_CreatedDateTime >= @startDate 
    AND VipMember_CreatedDateTime < @endDate
```

### 🎯 常用變數宣告模板
```sql
-- 時間範圍設定模板
DECLARE @year INT = YEAR(GETDATE()),
        @month INT = 1,
        @startDate DATETIME,
        @endDate DATETIME;

SET @startDate = DATEFROMPARTS(@year, @month, 1);
SET @endDate = DATEADD(DAY, 1, @startDate);
```

---

## 🚀 下一步
想了解更多進階應用嗎？請參考：
- [SQL 時間查詢與統計實戰指南](./SQL_時間查詢與統計實戰指南.md)

---
*📝 提示：記得在正式環境中謹慎使用 NOLOCK 提示，並考慮索引優化！*
