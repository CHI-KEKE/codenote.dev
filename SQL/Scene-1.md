# SQL 業務場景實戰指南 📊

> 從會員分析到風險控制，掌握電商業務中的 SQL 核心應用！

## 目錄
1. [會員分析場景](#會員分析場景)
   - 1.1 [活躍會員識別](#活躍會員識別)
   - 1.2 [好會員定義](#好會員定義)
   - 1.3 [會員等級分類](#會員等級分類)

2. [商品風險控制](#商品風險控制)
   - 2.1 [退貨率分析](#退貨率分析)
   - 2.2 [高風險商品識別](#高風險商品識別)
   - 2.3 [商品銷量區間分析](#商品銷量區間分析)

3. [訂單風險管理](#訂單風險管理)
   - 3.1 [風險訂單偵測](#風險訂單偵測)
   - 3.2 [訂單等級分類](#訂單等級分類)
   - 3.3 [金額區間統計](#金額區間統計)

4. [資料存在性檢查](#資料存在性檢查)
   - 4.1 [COUNT vs EXISTS 比較](#count-vs-exists-比較)
   - 4.2 [效能優化技巧](#效能優化技巧)
   - 4.3 [實務應用場景](#實務應用場景)

5. [進階邏輯處理](#進階邏輯處理)
   - 5.1 [三角形類型判斷](#三角形類型判斷)
   - 5.2 [複雜條件邏輯](#複雜條件邏輯)
   - 5.3 [數學邏輯應用](#數學邏輯應用)

---

## 會員分析場景

### 活躍會員識別

> 💡 **業務需求**：判斷每個會員在最近 3 個月內是否下單超過 5 次，若是，則標記為活躍會員

**實務場景**
- 會員行銷策略制定
- VIP 會員篩選
- 活動推廣對象選擇

```sql
USE WebStoreDB;

-- 方法一：使用 CTE 統計訂單次數
WITH MemberOrderStats AS (
    SELECT TradesOrderGroup_MemberId,
           COUNT(*) AS ORDER_COUNT
    FROM TradesOrderGroup WITH (NOLOCK)
    WHERE TradesOrderGroup_DateTime >= DATEADD(MONTH, -3, GETDATE())
    GROUP BY TradesOrderGroup_MemberId
)
SELECT TradesOrderGroup_MemberId,
       ORDER_COUNT,
       CASE
           WHEN ORDER_COUNT > 5 THEN 1 
           ELSE 0
       END AS IsActiveMember
FROM MemberOrderStats
ORDER BY ORDER_COUNT DESC;
```

**優化版本 - 加入會員基本資訊**
```sql
WITH MemberOrderStats AS (
    SELECT TradesOrderGroup_MemberId,
           COUNT(*) AS ORDER_COUNT,
           SUM(TradesOrderGroup_TotalPayment) AS TOTAL_AMOUNT
    FROM TradesOrderGroup WITH (NOLOCK)
    WHERE TradesOrderGroup_DateTime >= DATEADD(MONTH, -3, GETDATE())
      AND TradesOrderGroup_ValidFlag = 1
    GROUP BY TradesOrderGroup_MemberId
)
SELECT m.Member_Id,
       m.Member_Name,
       ISNULL(s.ORDER_COUNT, 0) AS ORDER_COUNT,
       ISNULL(s.TOTAL_AMOUNT, 0) AS TOTAL_AMOUNT,
       CASE
           WHEN s.ORDER_COUNT > 5 THEN 1 
           ELSE 0
       END AS IsActiveMember
FROM Member m
LEFT JOIN MemberOrderStats s ON m.Member_Id = s.TradesOrderGroup_MemberId;
```

### 好會員定義

> 🏆 **業務邏輯**：每個會員算出消費總額，超過 20,000 元記為好會員

```sql
USE WebStoreDB;

SELECT TradesOrderGroup_MemberId,
       SUM(TradesOrderGroup_TotalPayment) AS total_pay,
       CASE 
           WHEN SUM(TradesOrderGroup_TotalPayment) > 20000 THEN 1 
           ELSE 0 
       END AS Good_Member
FROM TradesOrderGroup WITH (NOLOCK)
WHERE TradesOrderGroup_ValidFlag = 1
GROUP BY TradesOrderGroup_MemberId
ORDER BY total_pay DESC;
```

**進階版本 - 多維度會員分析**
```sql
WITH MemberStats AS (
    SELECT TradesOrderGroup_MemberId,
           COUNT(*) AS OrderCount,
           SUM(TradesOrderGroup_TotalPayment) AS TotalPay,
           AVG(TradesOrderGroup_TotalPayment) AS AvgPay,
           MAX(TradesOrderGroup_TotalPayment) AS MaxPay,
           MIN(TradesOrderGroup_DateTime) AS FirstOrderDate,
           MAX(TradesOrderGroup_DateTime) AS LastOrderDate
    FROM TradesOrderGroup WITH (NOLOCK)
    WHERE TradesOrderGroup_ValidFlag = 1
    GROUP BY TradesOrderGroup_MemberId
)
SELECT *,
       DATEDIFF(DAY, FirstOrderDate, LastOrderDate) AS CustomerLifeDays,
       CASE 
           WHEN TotalPay > 20000 THEN 1 
           ELSE 0 
       END AS Good_Member,
       CASE
           WHEN OrderCount >= 10 AND TotalPay > 20000 THEN 'VIP'
           WHEN TotalPay > 20000 THEN 'Good'
           WHEN OrderCount >= 5 THEN 'Regular'
           ELSE 'New'
       END AS MemberCategory
FROM MemberStats
ORDER BY TotalPay DESC;
```

### 會員等級分類

> 📊 **分級策略**：根據總消費金額區分會員等級

```sql
USE WebStoreDB;

WITH MemberTotalPay AS (
    SELECT SUM(TradesOrderGroup_TotalPayment) AS Total_Pay,
           Member_Id
    FROM Member
    INNER JOIN TradesOrderGroup WITH (NOLOCK)
        ON Member_Id = TradesOrderGroup_MemberId
    WHERE TradesOrderGroup_ValidFlag = 1
    GROUP BY Member_Id
)
SELECT CASE 
           WHEN Total_Pay > 10000 THEN 'VipMember'
           WHEN Total_Pay > 5000 AND Total_Pay <= 10000 THEN 'NORMAL'
           ELSE 'Fresh'
       END AS Member_Level,    
       Member_Id,
       Total_Pay,
       FORMAT(Total_Pay, 'C0') AS FormattedAmount
FROM MemberTotalPay
ORDER BY Total_Pay DESC;
```

**統計各等級會員數量**
```sql
WITH MemberLevels AS (
    SELECT CASE 
               WHEN SUM(TradesOrderGroup_TotalPayment) > 10000 THEN 'VipMember'
               WHEN SUM(TradesOrderGroup_TotalPayment) > 5000 THEN 'NORMAL'
               ELSE 'Fresh'
           END AS Member_Level,
           Member_Id,
           SUM(TradesOrderGroup_TotalPayment) AS Total_Pay
    FROM Member
    INNER JOIN TradesOrderGroup WITH (NOLOCK)
        ON Member_Id = TradesOrderGroup_MemberId
    WHERE TradesOrderGroup_ValidFlag = 1
    GROUP BY Member_Id
)
SELECT Member_Level,
       COUNT(*) AS MemberCount,
       AVG(Total_Pay) AS AvgSpending,
       SUM(Total_Pay) AS TotalRevenue,
       FORMAT(AVG(Total_Pay), 'C0') AS FormattedAvg
FROM MemberLevels
GROUP BY Member_Level
ORDER BY AVG(Total_Pay) DESC;
```

---

## 商品風險控制

### 退貨率分析

> ⚠️ **風險控制**：計算每個商品的退貨率，若超過 30%，則標記為高風險商品

**資料關聯邏輯**
```
退貨表 => SalepageId : 退貨總數
訂單表 => SalepageId : 訂單總數
                    ↓ JOIN
              計算退貨率
```

```sql
-- 完整退貨率分析
WITH TotalOrders AS (
    SELECT SalesOrderSlave_SalePageId, 
           COUNT(*) AS TotalCount
    FROM SalesOrderSlave WITH (NOLOCK)
    WHERE SalesOrderSlave_ValidFlag = 1
    GROUP BY SalesOrderSlave_SalePageId
),
ReturnOrders AS (
    SELECT SalesOrderSlave_SalePageId AS ReturnSalepageId, 
           COUNT(*) AS Salepage_Return_Count
    FROM ReturnGoodsOrderSlave WITH (NOLOCK)
    JOIN SalesOrderSlave WITH (NOLOCK)
        ON ReturnGoodsOrderSlave_SalesOrderSlaveId = SalesOrderSlave.SalesOrderSlave_Id
    GROUP BY SalesOrderSlave_SalePageId
)
SELECT A.SalesOrderSlave_SalePageId AS 商品頁ID,
       A.TotalCount AS 總訂單數,
       ISNULL(B.Salepage_Return_Count, 0) AS 退貨數,
       CASE 
           WHEN B.Salepage_Return_Count IS NULL THEN '0.00 %'
           ELSE FORMAT(CAST(B.Salepage_Return_Count AS FLOAT) * 100.0 / A.TotalCount, 'N2') + ' %'
       END AS 退貨率,
       CASE 
           WHEN B.Salepage_Return_Count IS NULL THEN 0
           WHEN (CAST(B.Salepage_Return_Count AS FLOAT) * 100.0 / A.TotalCount) > 30 THEN 1 
           ELSE 0 
       END AS IsHighReturnRisk
FROM TotalOrders A
LEFT JOIN ReturnOrders B ON A.SalesOrderSlave_SalePageId = B.ReturnSalepageId
ORDER BY ISNULL(B.Salepage_Return_Count, 0) * 100.0 / A.TotalCount DESC;
```

### 高風險商品識別

**進階分析 - 多維度風險評估**
```sql
WITH ProductRiskAnalysis AS (
    -- 基礎退貨率計算
    SELECT sp.SalePage_Id,
           sp.SalePage_Title,
           COUNT(sos.SalesOrderSlave_Id) AS TotalOrders,
           COUNT(rgs.ReturnGoodsOrderSlave_Id) AS ReturnCount,
           CASE 
               WHEN COUNT(sos.SalesOrderSlave_Id) = 0 THEN 0
               ELSE CAST(COUNT(rgs.ReturnGoodsOrderSlave_Id) AS FLOAT) * 100.0 / COUNT(sos.SalesOrderSlave_Id)
           END AS ReturnRate,
           AVG(sos.SalesOrderSlave_Price) AS AvgPrice,
           SUM(sos.SalesOrderSlave_Qty) AS TotalQty
    FROM SalePage sp
    LEFT JOIN SalesOrderSlave sos ON sp.SalePage_Id = sos.SalesOrderSlave_SalePageId
    LEFT JOIN ReturnGoodsOrderSlave rgs ON sos.SalesOrderSlave_Id = rgs.ReturnGoodsOrderSlave_SalesOrderSlaveId
    WHERE sos.SalesOrderSlave_ValidFlag = 1
    GROUP BY sp.SalePage_Id, sp.SalePage_Title
)
SELECT *,
       CASE
           WHEN ReturnRate > 50 THEN '極高風險'
           WHEN ReturnRate > 30 THEN '高風險'
           WHEN ReturnRate > 15 THEN '中風險'
           WHEN ReturnRate > 5 THEN '低風險'
           ELSE '安全'
       END AS RiskLevel,
       CASE
           WHEN ReturnRate > 30 THEN 1
           ELSE 0
       END AS IsHighReturnRisk
FROM ProductRiskAnalysis
WHERE TotalOrders > 10  -- 只分析有足夠樣本的商品
ORDER BY ReturnRate DESC;
```

### 商品銷量區間分析

> 📈 **銷量分析**：列出購買數量區間的商品分佈

```sql
USE WebStoreDB;

-- 建立暫存表統計銷量
DROP TABLE IF EXISTS #salepageWithCount;
CREATE TABLE #salepageWithCount(
    salepageid BIGINT,
    buyQty INT
);

INSERT INTO #salepageWithCount
SELECT TradesOrderSlave_SalePageId,
       SUM(TradesOrderSlave_Qty)
FROM TradesOrderSlave WITH (NOLOCK)
WHERE TradesOrderSlave_DateTime > '2025-05-01'
  AND TradesOrderSlave_ValidFlag = 1
GROUP BY TradesOrderSlave_SalePageId;

-- 分析銷量區間分佈
SELECT CASE
           WHEN buyQty <= 50 THEN '0-50'
           WHEN buyQty <= 100 THEN '51-100'
           WHEN buyQty <= 200 THEN '101-200'
           ELSE '200+'
       END AS Qty_Range,
       COUNT(*) AS ProductCount,
       STRING_AGG(CAST(salepageid AS VARCHAR), ',') AS SalepageList,
       AVG(buyQty) AS AvgQtyInRange,
       SUM(buyQty) AS TotalQtyInRange
FROM #salepageWithCount
GROUP BY CASE
             WHEN buyQty <= 50 THEN '0-50'
             WHEN buyQty <= 100 THEN '51-100'
             WHEN buyQty <= 200 THEN '101-200'
             ELSE '200+'
         END
ORDER BY MIN(buyQty);

DROP TABLE IF EXISTS #salepageWithCount;
```

---

## 訂單風險管理

### 風險訂單偵測

> 🚨 **風險控制**：找出會員單筆訂單金額超過該會員平均訂單 3 倍者

**業務邏輯**
- 計算會員歷史平均訂單金額
- 比較當前訂單與歷史平均值
- 標記異常高額訂單

```sql
USE ERPDB;

WITH MemberAvgPayment AS (
    SELECT SalesOrderGroup_MemberId,
           AVG(SalesOrderGroup_TotalPayment) AS AVG_Pay_Before_This_year
    FROM SalesOrderGroup WITH (NOLOCK)
    WHERE SalesOrderGroup_DateTime < '2025-01-01'
      AND SalesOrderGroup_ValidFlag = 1
    GROUP BY SalesOrderGroup_MemberId
)
SELECT s.SalesOrderGroup_MemberId,
       s.SalesOrderGroup_TradesOrderGroupId,
       s.SalesOrderGroup_TotalPayment,
       a.AVG_Pay_Before_This_year,
       FORMAT(s.SalesOrderGroup_TotalPayment / a.AVG_Pay_Before_This_year, 'N2') AS PaymentRatio,
       CASE
           WHEN s.SalesOrderGroup_TotalPayment > a.AVG_Pay_Before_This_year * 3 THEN 1 
           ELSE 0
       END AS High_Risk_Order
FROM SalesOrderGroup s WITH (NOLOCK)
INNER JOIN MemberAvgPayment a ON a.SalesOrderGroup_MemberId = s.SalesOrderGroup_MemberId
WHERE s.SalesOrderGroup_DateTime >= '2025-01-01'
  AND s.SalesOrderGroup_ValidFlag = 1
ORDER BY PaymentRatio DESC;
```

**進階風險分析**
```sql
WITH RiskAnalysis AS (
    SELECT s.SalesOrderGroup_MemberId,
           s.SalesOrderGroup_TradesOrderGroupId,
           s.SalesOrderGroup_TotalPayment,
           a.AVG_Pay_Before_This_year,
           s.SalesOrderGroup_TotalPayment / a.AVG_Pay_Before_This_year AS PaymentRatio,
           COUNT(*) OVER (PARTITION BY s.SalesOrderGroup_MemberId) AS MemberOrderCount
    FROM SalesOrderGroup s WITH (NOLOCK)
    INNER JOIN (
        SELECT SalesOrderGroup_MemberId,
               AVG(SalesOrderGroup_TotalPayment) AS AVG_Pay_Before_This_year
        FROM SalesOrderGroup WITH (NOLOCK)
        WHERE SalesOrderGroup_DateTime < '2025-01-01'
        GROUP BY SalesOrderGroup_MemberId
        HAVING COUNT(*) >= 3  -- 至少要有3筆歷史訂單才計算平均
    ) a ON a.SalesOrderGroup_MemberId = s.SalesOrderGroup_MemberId
    WHERE s.SalesOrderGroup_DateTime >= '2025-01-01'
)
SELECT *,
       CASE
           WHEN PaymentRatio > 5 THEN '極高風險'
           WHEN PaymentRatio > 3 THEN '高風險'
           WHEN PaymentRatio > 2 THEN '中風險'
           ELSE '正常'
       END AS RiskLevel,
       CASE
           WHEN PaymentRatio > 3 THEN 1
           ELSE 0
       END AS High_Risk_Order
FROM RiskAnalysis
ORDER BY PaymentRatio DESC;
```

### 訂單等級分類

> 📊 **訂單分級**：根據訂單金額劃分不同等級

```sql
USE WebStoreDB;

SELECT CASE
           WHEN TradesOrderGroup_TotalPayment > 10000 THEN 'CoolOrder'
           WHEN TradesOrderGroup_TotalPayment >= 5000 AND TradesOrderGroup_TotalPayment <= 10000 THEN 'General Order'
           WHEN TradesOrderGroup_TotalPayment < 5000 THEN 'NormalOrder'
       END AS OrderLevel,
       TradesOrderGroup_Id,
       TradesOrderGroup_MemberId,
       TradesOrderGroup_TotalPayment,
       FORMAT(TradesOrderGroup_TotalPayment, 'C0') AS FormattedAmount,
       TradesOrderGroup_DateTime
FROM TradesOrderGroup WITH (NOLOCK)
WHERE TradesOrderGroup_ValidFlag = 1
ORDER BY TradesOrderGroup_TotalPayment DESC;
```

**訂單等級統計分析**
```sql
WITH OrderLevels AS (
    SELECT CASE
               WHEN TradesOrderGroup_TotalPayment > 10000 THEN 'CoolOrder'
               WHEN TradesOrderGroup_TotalPayment >= 5000 THEN 'General Order'
               ELSE 'NormalOrder'
           END AS OrderLevel,
           TradesOrderGroup_TotalPayment
    FROM TradesOrderGroup WITH (NOLOCK)
    WHERE TradesOrderGroup_ValidFlag = 1
)
SELECT OrderLevel,
       COUNT(*) AS OrderCount,
       FORMAT(AVG(TradesOrderGroup_TotalPayment), 'C0') AS AvgAmount,
       FORMAT(SUM(TradesOrderGroup_TotalPayment), 'C0') AS TotalAmount,
       FORMAT(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER(), 'N2') + '%' AS Percentage
FROM OrderLevels
GROUP BY OrderLevel
ORDER BY AVG(TradesOrderGroup_TotalPayment) DESC;
```

### 金額區間統計

> 💰 **統計分析**：統計過去 90 天內不同金額區間的訂單筆數

```sql
USE WebStoreDB;

SELECT COUNT(CASE WHEN TradesOrderGroup_TotalPayment > 1000 THEN 1 END) AS '高額訂單(>1000)',
       COUNT(CASE WHEN TradesOrderGroup_TotalPayment BETWEEN 100 AND 1000 THEN 1 END) AS '中等訂單(100-1000)',
       COUNT(CASE WHEN TradesOrderGroup_TotalPayment < 100 THEN 1 END) AS '小額訂單(<100)',
       COUNT(*) AS '總訂單數',
       FORMAT(AVG(TradesOrderGroup_TotalPayment), 'C0') AS '平均訂單金額'
FROM TradesOrderGroup WITH (NOLOCK)
WHERE TradesOrderGroup_DateTime > DATEADD(DAY, -90, GETDATE())
  AND TradesOrderGroup_ValidFlag = 1;
```

**詳細金額區間分析**
```sql
WITH AmountRanges AS (
    SELECT CASE
               WHEN TradesOrderGroup_TotalPayment >= 5000 THEN '5000+'
               WHEN TradesOrderGroup_TotalPayment >= 1000 THEN '1000-4999'
               WHEN TradesOrderGroup_TotalPayment >= 500 THEN '500-999'
               WHEN TradesOrderGroup_TotalPayment >= 100 THEN '100-499'
               ELSE '<100'
           END AS AmountRange,
           TradesOrderGroup_TotalPayment
    FROM TradesOrderGroup WITH (NOLOCK)
    WHERE TradesOrderGroup_DateTime > DATEADD(DAY, -90, GETDATE())
      AND TradesOrderGroup_ValidFlag = 1
)
SELECT AmountRange,
       COUNT(*) AS OrderCount,
       FORMAT(SUM(TradesOrderGroup_TotalPayment), 'C0') AS TotalRevenue,
       FORMAT(AVG(TradesOrderGroup_TotalPayment), 'C0') AS AvgAmount,
       FORMAT(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER(), 'N1') + '%' AS OrderPercentage,
       FORMAT(SUM(TradesOrderGroup_TotalPayment) * 100.0 / SUM(SUM(TradesOrderGroup_TotalPayment)) OVER(), 'N1') + '%' AS RevenuePercentage
FROM AmountRanges
GROUP BY AmountRange
ORDER BY MIN(CASE 
                WHEN AmountRange = '<100' THEN 1
                WHEN AmountRange = '100-499' THEN 2
                WHEN AmountRange = '500-999' THEN 3
                WHEN AmountRange = '1000-4999' THEN 4
                WHEN AmountRange = '5000+' THEN 5
            END);
```

---

## 資料存在性檢查

### COUNT vs EXISTS 比較

> ⚡ **效能優化**：不同的存在性檢查方法及其效能差異

```sql
USE WebStoreDB;
SET STATISTICS IO ON;
SET STATISTICS TIME ON;

-- 方法一：使用 COUNT(*) - 較慢
IF (SELECT COUNT(*) FROM TradesOrderGroup WITH (NOLOCK) 
    WHERE TradesOrderGroup_DateTime > '2025-05-29') > 0
BEGIN
    PRINT(N'方法一：有訂單');
END

-- 方法二：使用 EXISTS - 推薦
IF EXISTS (SELECT 1 FROM TradesOrderGroup WITH (NOLOCK) 
           WHERE TradesOrderGroup_DateTime > '2025-05-29')
BEGIN
    PRINT(N'方法二：有訂單');
END

-- 方法三：使用 TOP 1 - 中等效能
IF (SELECT TOP 1 1 FROM TradesOrderGroup WITH (NOLOCK) 
    WHERE TradesOrderGroup_DateTime > '2025-05-29') IS NOT NULL
BEGIN
    PRINT(N'方法三：有訂單');
END
```

### 效能優化技巧

**EXISTS 的優勢**
```sql
-- ✅ 推薦：使用 EXISTS
IF EXISTS (SELECT 1 FROM Orders WHERE CustomerId = @CustomerId)
BEGIN
    -- EXISTS 找到第一筆就停止
    PRINT('客戶有訂單');
END

-- ❌ 不推薦：使用 COUNT
IF (SELECT COUNT(*) FROM Orders WHERE CustomerId = @CustomerId) > 0
BEGIN
    -- COUNT 會掃描所有符合條件的記錄
    PRINT('客戶有訂單');
END
```

**複雜條件的存在性檢查**
```sql
-- 檢查會員是否為 VIP（多條件）
DECLARE @MemberId INT = 12345;

IF EXISTS (
    SELECT 1 
    FROM Member m
    INNER JOIN TradesOrderGroup t ON m.Member_Id = t.TradesOrderGroup_MemberId
    WHERE m.Member_Id = @MemberId
      AND t.TradesOrderGroup_DateTime >= DATEADD(YEAR, -1, GETDATE())
    GROUP BY m.Member_Id
    HAVING SUM(t.TradesOrderGroup_TotalPayment) > 50000
       AND COUNT(*) > 20
)
BEGIN
    PRINT('VIP 會員');
END
ELSE
BEGIN
    PRINT('一般會員');
END
```

### 實務應用場景

**資料完整性檢查**
```sql
-- 檢查訂單是否有對應的會員資料
SELECT o.TradesOrderGroup_Id,
       o.TradesOrderGroup_MemberId,
       CASE
           WHEN EXISTS (SELECT 1 FROM Member m WHERE m.Member_Id = o.TradesOrderGroup_MemberId)
           THEN '會員存在'
           ELSE '會員不存在'
       END AS MemberStatus
FROM TradesOrderGroup o WITH (NOLOCK)
WHERE o.TradesOrderGroup_DateTime >= DATEADD(DAY, -7, GETDATE());
```

**條件式資料處理**
```sql
-- 根據資料存在與否執行不同邏輯
DECLARE @ProcessDate DATE = GETDATE();

IF EXISTS (SELECT 1 FROM DailyReport WHERE ReportDate = @ProcessDate)
BEGIN
    -- 更新現有報告
    UPDATE DailyReport 
    SET LastUpdated = GETDATE()
    WHERE ReportDate = @ProcessDate;
    
    PRINT('報告已更新');
END
ELSE
BEGIN
    -- 建立新報告
    INSERT INTO DailyReport (ReportDate, CreatedDate)
    VALUES (@ProcessDate, GETDATE());
    
    PRINT('新報告已建立');
END
```

---

## 進階邏輯處理

### 三角形類型判斷

> 📐 **邏輯應用**：根據三邊長度判斷三角形類型

**業務場景**
- 邏輯判斷能力測試
- 複雜條件處理
- 數學邏輯在 SQL 中的應用

```sql
-- 三角形分類邏輯
SELECT A, B, C,
       CASE
           -- 首先檢查是否能構成三角形
           WHEN A + B <= C OR A + C <= B OR B + C <= A THEN 'Not a Legal Triangle'
           -- 等邊三角形：三邊相等
           WHEN A = B AND B = C AND A = C THEN 'Equilateral'
           -- 等腰三角形：任意兩邊相等
           WHEN A = B OR A = C OR B = C THEN 'Isosceles'
           -- 不等邊三角形：三邊都不相等
           ELSE 'Scalene'
       END AS TriangleType
FROM Triangles
ORDER BY A, B, C;
```

**帶詳細說明的版本**
```sql
SELECT A, B, C,
       CASE
           WHEN A + B <= C OR A + C <= B OR B + C <= A THEN 'Not a Legal Triangle'
           WHEN A = B AND B = C THEN 'Equilateral'
           WHEN A = B OR A = C OR B = C THEN 'Isosceles'
           ELSE 'Scalene'
       END AS TriangleType,
       CASE
           WHEN A + B <= C OR A + C <= B OR B + C <= A THEN '不符合三角形定理'
           WHEN A = B AND B = C THEN '三邊相等'
           WHEN A = B OR A = C OR B = C THEN '兩邊相等'
           ELSE '三邊都不相等'
       END AS Description,
       A + B + C AS Perimeter,
       CASE
           WHEN A + B > C AND A + C > B AND B + C > A 
           THEN SQRT((A + B + C) * (-A + B + C) * (A - B + C) * (A + B - C)) / 4.0
           ELSE NULL
       END AS Area
FROM Triangles;
```

### 複雜條件邏輯

**多層條件判斷範例**
```sql
-- 客戶價值分級（複雜邏輯）
WITH CustomerAnalysis AS (
    SELECT m.Member_Id,
           m.Member_Name,
           COUNT(t.TradesOrderGroup_Id) AS OrderCount,
           SUM(t.TradesOrderGroup_TotalPayment) AS TotalSpend,
           AVG(t.TradesOrderGroup_TotalPayment) AS AvgOrderValue,
           DATEDIFF(DAY, MIN(t.TradesOrderGroup_DateTime), MAX(t.TradesOrderGroup_DateTime)) AS CustomerLifeDays,
           MAX(t.TradesOrderGroup_DateTime) AS LastOrderDate
    FROM Member m
    LEFT JOIN TradesOrderGroup t ON m.Member_Id = t.TradesOrderGroup_MemberId
    WHERE t.TradesOrderGroup_ValidFlag = 1
    GROUP BY m.Member_Id, m.Member_Name
)
SELECT *,
       CASE
           -- 鑽石級客戶
           WHEN TotalSpend > 100000 AND OrderCount > 50 AND DATEDIFF(DAY, LastOrderDate, GETDATE()) <= 30
           THEN '鑽石客戶'
           
           -- 白金級客戶
           WHEN TotalSpend > 50000 AND OrderCount > 20 AND DATEDIFF(DAY, LastOrderDate, GETDATE()) <= 60
           THEN '白金客戶'
           
           -- 黃金級客戶
           WHEN TotalSpend > 20000 AND OrderCount > 10 AND DATEDIFF(DAY, LastOrderDate, GETDATE()) <= 90
           THEN '黃金客戶'
           
           -- 流失客戶
           WHEN DATEDIFF(DAY, LastOrderDate, GETDATE()) > 365
           THEN '流失客戶'
           
           -- 新客戶
           WHEN OrderCount <= 3 AND DATEDIFF(DAY, LastOrderDate, GETDATE()) <= 30
           THEN '新客戶'
           
           -- 一般客戶
           ELSE '一般客戶'
       END AS CustomerLevel,
       
       CASE
           WHEN DATEDIFF(DAY, LastOrderDate, GETDATE()) <= 30 THEN '活躍'
           WHEN DATEDIFF(DAY, LastOrderDate, GETDATE()) <= 90 THEN '一般'
           WHEN DATEDIFF(DAY, LastOrderDate, GETDATE()) <= 180 THEN '不活躍'
           ELSE '流失'
       END AS ActivityLevel
FROM CustomerAnalysis
WHERE OrderCount > 0
ORDER BY TotalSpend DESC;
```

### 數學邏輯應用

**商業邏輯中的數學運算**
```sql
-- 商品定價策略分析
WITH ProductPricing AS (
    SELECT sp.SalePage_Id,
           sp.SalePage_Title,
           sp.SalePage_Price,
           AVG(sos.SalesOrderSlave_Price) AS ActualAvgPrice,
           COUNT(sos.SalesOrderSlave_Id) AS SalesCount,
           SUM(sos.SalesOrderSlave_Qty) AS TotalQty,
           SUM(sos.SalesOrderSlave_Price * sos.SalesOrderSlave_Qty) AS Revenue
    FROM SalePage sp
    LEFT JOIN SalesOrderSlave sos ON sp.SalePage_Id = sos.SalesOrderSlave_SalePageId
    WHERE sos.SalesOrderSlave_ValidFlag = 1
    GROUP BY sp.SalePage_Id, sp.SalePage_Title, sp.SalePage_Price
)
SELECT *,
       -- 價格差異分析
       CASE
           WHEN ABS(SalePage_Price - ActualAvgPrice) / SalePage_Price > 0.1 
           THEN '價格差異大'
           ELSE '價格一致'
       END AS PriceConsistency,
       
       -- 銷售效率分級
       CASE
           WHEN Revenue / SalesCount > 1000 THEN '高效商品'
           WHEN Revenue / SalesCount > 500 THEN '中效商品'
           ELSE '低效商品'
       END AS EfficiencyLevel,
       
       -- 計算幾何平均價格（防止極值影響）
       POWER(Revenue / TotalQty, 0.5) AS GeometricMeanPrice,
       
       -- 價格彈性指標
       CASE
           WHEN SalePage_Price > 0 
           THEN (ActualAvgPrice - SalePage_Price) / SalePage_Price * 100
           ELSE 0
       END AS PriceVariancePercent
FROM ProductPricing
WHERE SalesCount > 0
ORDER BY Revenue DESC;
```

---

## 最佳實踐與效能優化

### 📋 **SQL 業務場景最佳實踐**

| 場景類型 | 最佳做法 | 注意事項 |
|----------|----------|----------|
| 會員分析 | 使用 CTE 分階段處理 | 注意會員資料完整性 |
| 風險控制 | 結合歷史資料比較 | 設定合理的風險閾值 |
| 存在性檢查 | 優先使用 EXISTS | 避免使用 COUNT(*) |
| 複雜邏輯 | 善用 CASE WHEN | 注意條件順序 |

### 🎯 **效能優化金句**

> "EXISTS 找到就停，COUNT 數到底！"

> "業務邏輯要清晰，SQL 條件要簡潔！"

> "WITH 讓複雜查詢變簡單，分階段處理更清楚！"

### ✅ **檢查清單**

- [ ] 是否使用了適當的索引？
- [ ] 條件過濾是否夠精確？
- [ ] 是否避免了不必要的資料掃描？
- [ ] 業務邏輯是否正確實現？
- [ ] 是否考慮了資料的完整性？

這份指南涵蓋了電商業務中最常見的 SQL 應用場景，從會員分析到風險控制，讓您能夠快速應對各種業務需求！🚀✨