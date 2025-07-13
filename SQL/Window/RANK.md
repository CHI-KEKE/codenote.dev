## 📖 目錄
- [📖 目錄](#-目錄)
- [🌟 RANK 排名函式概述](#-rank-排名函式概述)
  - [🎨 排名函式分類](#-排名函式分類)
- [☘️ RANK vs DENSE\_RANK 詳解](#️-rank-vs-dense_rank-詳解)
  - [📊 函式比較表](#-函式比較表)
  - [🎯 實際範例比較](#-實際範例比較)
  - [🎪 使用場景建議](#-使用場景建議)
- [🎪 實戰案例：排名查詢](#-實戰案例排名查詢)
  - [💰 每個月消費前兩名](#-每個月消費前兩名)
  - [🎁 取得最新的促銷規則記錄](#-取得最新的促銷規則記錄)
  - [🏃‍♂️ 進行中活動的最新規則記錄](#️-進行中活動的最新規則記錄)
  - [👑 會員最高金額訂單及對應裝置](#-會員最高金額訂單及對應裝置)
- [🎯 進階排名技巧](#-進階排名技巧)
  - [🔄 複雜分組排名](#-複雜分組排名)
  - [📈 多重排序條件](#-多重排序條件)
  - [🎪 條件式排名](#-條件式排名)
---

## 🌟 RANK 排名函式概述

### 🎨 排名函式分類

| 函式類型 | 主要用途 | 特殊特性 |
|----------|----------|----------|
| **🏅 RANK()** | 傳統排名 | 並列時會跳號 |
| **🎯 DENSE_RANK()** | 密集排名 | 並列時不跳號 |
| **🔢 ROW_NUMBER()** | 唯一編號 | 每筆記錄都有唯一序號 |

---

## ☘️ RANK vs DENSE_RANK 詳解

### 📊 函式比較表

| 函式名稱 | 是否跳號 | 相同名次時下一名的編號 | 適用場景 |
|----------|----------|----------------------|----------|
| **RANK()** | ✅ 會跳號 | 跳過與前面相同名次的筆數數量 | 傳統排名（如體育競賽） |
| **DENSE_RANK()** | ❌ 不跳號 | 直接接續下一名次（不跳號） | 密集排名（如分數等級） |
| **ROW_NUMBER()** | ❌ 不跳號 | 每一列都有唯一序號 | 需要唯一識別時 |

### 🎯 實際範例比較

```sql
-- 假設有以下分數資料：95, 95, 90, 85
SELECT 
    Score,
    RANK() OVER (ORDER BY Score DESC) AS Rank_Result,
    DENSE_RANK() OVER (ORDER BY Score DESC) AS Dense_Rank_Result,
    ROW_NUMBER() OVER (ORDER BY Score DESC) AS Row_Number_Result
FROM ScoreTable;

-- 結果：
-- Score | Rank_Result | Dense_Rank_Result | Row_Number_Result
-- 95    | 1           | 1                 | 1
-- 95    | 1           | 1                 | 2  
-- 90    | 3           | 2                 | 3
-- 85    | 4           | 3                 | 4
```

### 🎪 使用場景建議

- **🏅 RANK()** - 當需要傳統排名邏輯時（體育比賽、考試排名）
- **🎯 DENSE_RANK()** - 當需要緊密排名時（等級分類、優先級排序）
- **🔢 ROW_NUMBER()** - 當需要唯一識別碼時（分頁、去重）

---

## 🎪 實戰案例：排名查詢

### 💰 每個月消費前兩名

```sql
USE WebStoreDB;

DECLARE @startDateA DATETIME = DATEFROMPARTS(YEAR(GETDATE())-1, 1, 1);
DECLARE @endDateA DATETIME = DATEFROMPARTS(YEAR(GETDATE()), 1, 1);

WITH MonthlyRanking AS (
    SELECT 
        TradesOrderGroup_MemberId,
        RANK() OVER (
            PARTITION BY YEAR(TradesOrderGroup_DateTime), MONTH(TradesOrderGroup_DateTime)
            ORDER BY TradesOrderGroup_TotalPayment DESC
        ) AS PaymentRank,
        FORMAT(TradesOrderGroup_DateTime, 'yyyy-MM') AS YearMonth,
        TradesOrderGroup_TotalPayment AS Payment
    FROM TradesOrderGroup (NOLOCK)
    WHERE TradesOrderGroup_ValidFlag = 1
        AND TradesOrderGroup_DateTime >= @startDateA
        AND TradesOrderGroup_DateTime < @endDateA
)
SELECT *
FROM MonthlyRanking
WHERE PaymentRank <= 2
ORDER BY YearMonth, PaymentRank;
```

> 🎯 **商業價值**：識別每月的 VIP 客戶，制定個人化行銷策略

### 🎁 取得最新的促銷規則記錄

```sql
USE WebStoreDB;

WITH LatestPromotionRules AS (
    SELECT 
        DENSE_RANK() OVER (
            PARTITION BY PromotionEngineRuleRecord_PromotionEngineId 
            ORDER BY PromotionEngineRuleRecord_PromotionEngineDateTime DESC
        ) AS RecordFreshness,
        *
    FROM PromotionEngineRuleRecord(NOLOCK)
)
SELECT *
FROM LatestPromotionRules
WHERE RecordFreshness = 1;
```

> 🔍 **技術重點**：使用 DENSE_RANK 確保每個促銷活動都能取得最新記錄

### 🏃‍♂️ 進行中活動的最新規則記錄

```sql
USE WebStoreDB;

WITH OngoingPromotionLatestRules AS (
    SELECT 
        DENSE_RANK() OVER (
            PARTITION BY PromotionEngineRuleRecord_PromotionEngineId 
            ORDER BY PromotionEngineRuleRecord_PromotionEngineDateTime DESC
        ) AS RecordOrder,
        *
    FROM PromotionEngineRuleRecord(NOLOCK)
    WHERE PromotionEngineRuleRecord_PromotionEngineId IN (
        SELECT PromotionEngine_Id
        FROM PromotionEngine(NOLOCK)
        WHERE PromotionEngine_EndDateTime > GETDATE()
    )
)
SELECT *
FROM OngoingPromotionLatestRules
WHERE RecordOrder = 1;
```

### 👑 會員最高金額訂單及對應裝置

```sql
USE WebStoreDB;

WITH MemberHighestOrders AS (
    SELECT 
        RANK() OVER (
            PARTITION BY TradesOrderGroup_MemberId 
            ORDER BY TradesOrderGroup_TotalPayment DESC
        ) AS PaymentRank,
        TradesOrderGroup_MemberId,
        TradesOrderGroup_TrackDeviceTypeDef,
        TradesOrderGroup_TotalPayment
    FROM TradesOrderGroup(NOLOCK)
    WHERE TradesOrderGroup_ValidFlag = 1
)
SELECT *
FROM MemberHighestOrders
WHERE PaymentRank = 1;
```

> 📱 **分析價值**：了解高價值訂單傾向使用的裝置類型，優化 UI/UX 設計

---

## 🎯 進階排名技巧

### 🔄 複雜分組排名

```sql
-- 依部門和年份進行薪資排名
SELECT 
    employee_name,
    department,
    salary,
    YEAR(hire_date) as hire_year,
    RANK() OVER (
        PARTITION BY department, YEAR(hire_date)
        ORDER BY salary DESC
    ) AS dept_year_rank
FROM employees;
```

### 📈 多重排序條件

```sql
-- 先依總金額排序，再依訂單日期排序
SELECT 
    order_id,
    customer_id,
    total_amount,
    order_date,
    RANK() OVER (
        PARTITION BY customer_id
        ORDER BY total_amount DESC, order_date DESC
    ) AS customer_order_rank
FROM orders;
```

### 🎪 條件式排名

```sql
-- 只對特定條件的記錄進行排名
SELECT 
    product_name,
    category,
    sales_amount,
    CASE 
        WHEN sales_amount > 10000 THEN
            RANK() OVER (
                PARTITION BY category
                ORDER BY sales_amount DESC
            )
        ELSE NULL
    END AS high_sales_rank
FROM products;
```

---