# SQL Window Function AVG 📊

## 📖 目錄
- [SQL Window Function AVG 📊](#sql-window-function-avg-)
  - [📖 目錄](#-目錄)
  - [🌟 AVG 平均值概述](#-avg-平均值概述)
    - [🎨 平均值計算類型](#-平均值計算類型)
  - [📏 基礎平均值範例](#-基礎平均值範例)
    - [🎯 整體平均值比較](#-整體平均值比較)
    - [📊 分組平均值分析](#-分組平均值分析)
    - [📈 移動平均值計算](#-移動平均值計算)
  - [🔍 偏離平均值分析](#-偏離平均值分析)
    - [📏 絕對差異分析](#-絕對差異分析)
    - [📊 百分比差異分析](#-百分比差異分析)
    - [⚡ 異常值檢測](#-異常值檢測)
  - [💼 商業實戰案例](#-商業實戰案例)
    - [💰 員工薪資水平分析](#-員工薪資水平分析)
    - [🛍️ 客戶消費行為分析](#️-客戶消費行為分析)
    - [📈 產品銷售表現分析](#-產品銷售表現分析)
  - [🎯 進階平均值技巧](#-進階平均值技巧)
    - [🔄 加權平均值計算](#-加權平均值計算)
    - [📊 多期間平均比較](#-多期間平均比較)
    - [🎪 條件性平均值](#-條件性平均值)
    - [⚠️ 注意事項](#️-注意事項)

---

## 🌟 AVG 平均值概述

### 🎨 平均值計算類型

| 計算類型 | 語法範例 | 適用場景 |
|----------|----------|----------|
| **🌐 全域平均** | `AVG(amount) OVER()` | 整體水平比較 |
| **🔄 分組平均** | `AVG(amount) OVER(PARTITION BY category)` | 同類比較 |
| **📈 移動平均** | `AVG(amount) OVER(ORDER BY date ROWS 6 PRECEDING)` | 趨勢分析 |
| **📊 累計平均** | `AVG(amount) OVER(ORDER BY date ROWS UNBOUNDED PRECEDING)` | 歷史表現 |

---

## 📏 基礎平均值範例

### 🎯 整體平均值比較

```sql
-- 比較每筆訂單與整體平均的差異
SELECT 
    order_id,
    customer_id,
    order_amount,
    AVG(order_amount) OVER() AS 整體平均金額,
    order_amount - AVG(order_amount) OVER() AS 與平均差異,
    CASE 
        WHEN order_amount > AVG(order_amount) OVER() THEN '🔥 高於平均'
        WHEN order_amount < AVG(order_amount) OVER() THEN '❄️ 低於平均'
        ELSE '📊 等於平均'
    END AS 相對表現
FROM orders
ORDER BY order_amount DESC;
```

### 📊 分組平均值分析

```sql
-- 各部門員工薪資與部門平均比較
SELECT 
    employee_name,
    department,
    salary,
    AVG(salary) OVER(PARTITION BY department) AS 部門平均薪資,
    salary - AVG(salary) OVER(PARTITION BY department) AS 與部門平均差異,
    CAST(
        (salary - AVG(salary) OVER(PARTITION BY department)) * 100.0 / 
        AVG(salary) OVER(PARTITION BY department) 
        AS DECIMAL(5,2)
    ) AS 差異百分比
FROM employees
ORDER BY department, salary DESC;
```

### 📈 移動平均值計算

```sql
-- 7天移動平均銷售額
SELECT 
    sale_date,
    daily_sales,
    AVG(daily_sales) OVER (
        ORDER BY sale_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS 七天移動平均,
    AVG(daily_sales) OVER (
        ORDER BY sale_date
        ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
    ) AS 三十天移動平均
FROM daily_sales
ORDER BY sale_date;
```

---

## 🔍 偏離平均值分析

### 📏 絕對差異分析

```sql
USE WebStoreDB;

-- 訂單金額偏離平均值分析
WITH PaymentDeviation AS (
    SELECT  
        TradesOrderGroup_MemberId,
        TradesOrderGroup_Id,
        TradesOrderGroup_TotalPayment,
        AVG(TradesOrderGroup_TotalPayment) OVER() AS 全體平均金額,
        AVG(TradesOrderGroup_TotalPayment) OVER(
            PARTITION BY TradesOrderGroup_MemberId
        ) AS 會員平均金額,
        ABS(TradesOrderGroup_TotalPayment - AVG(TradesOrderGroup_TotalPayment) OVER()) AS 絕對差異
    FROM TradesOrderGroup(NOLOCK)
    WHERE TradesOrderGroup_ValidFlag = 1
        AND TradesOrderGroup_DateTime >= DATEADD(MONTH, -6, GETDATE())
)
SELECT 
    TradesOrderGroup_MemberId,
    TradesOrderGroup_Id,
    TradesOrderGroup_TotalPayment,
    全體平均金額,
    會員平均金額,
    絕對差異,
    CASE 
        WHEN 絕對差異 > 全體平均金額 THEN '🚨 極度異常'
        WHEN 絕對差異 > 全體平均金額 * 0.5 THEN '⚠️ 明顯異常'
        WHEN 絕對差異 > 全體平均金額 * 0.2 THEN '📊 輕微異常'
        ELSE '✅ 正常範圍'
    END AS 異常等級
FROM PaymentDeviation
ORDER BY 絕對差異 DESC;
```

### 📊 百分比差異分析

```sql
USE WebStoreDB;

-- 計算訂單相對於平均值的百分比差異
WITH PaymentPercentDeviation AS (
    SELECT  
        TradesOrderGroup_TotalPayment,
        AVG(TradesOrderGroup_TotalPayment) OVER() AS 平均金額,
        TradesOrderGroup_TotalPayment - AVG(TradesOrderGroup_TotalPayment) OVER() AS 金額差異,
        CAST(
            (TradesOrderGroup_TotalPayment - AVG(TradesOrderGroup_TotalPayment) OVER()) * 100.0 / 
            AVG(TradesOrderGroup_TotalPayment) OVER() 
            AS DECIMAL(10,2)
        ) AS 百分比差異
    FROM TradesOrderGroup(NOLOCK)
    WHERE TradesOrderGroup_ValidFlag = 1
        AND TradesOrderGroup_DateTime >= DATEADD(MONTH, -3, GETDATE())
)
SELECT 
    TradesOrderGroup_TotalPayment,
    平均金額,
    金額差異,
    百分比差異,
    CASE 
        WHEN 百分比差異 > 200 THEN '🔥 超高消費 (>200%)'
        WHEN 百分比差異 > 100 THEN '⭐ 高消費 (100-200%)'
        WHEN 百分比差異 > 50 THEN '📈 中高消費 (50-100%)'
        WHEN 百分比差異 > 0 THEN '📊 略高於平均'
        WHEN 百分比差異 > -50 THEN '📉 略低於平均'
        ELSE '❄️ 低消費 (<-50%)'
    END AS 消費等級
FROM PaymentPercentDeviation
WHERE 百分比差異 > 50  -- 只顯示超過平均50%的訂單
ORDER BY 百分比差異 DESC;
```

### ⚡ 異常值檢測

```sql
-- 使用標準差檢測異常值
WITH OrderStats AS (
    SELECT 
        order_id,
        amount,
        AVG(amount) OVER() AS 平均值,
        STDEV(amount) OVER() AS 標準差
    FROM orders
),
AnomalyDetection AS (
    SELECT 
        order_id,
        amount,
        平均值,
        標準差,
        ABS(amount - 平均值) / 標準差 AS Z分數
    FROM OrderStats
)
SELECT 
    order_id,
    amount,
    平均值,
    Z分數,
    CASE 
        WHEN Z分數 > 3 THEN '🚨 極端異常'
        WHEN Z分數 > 2 THEN '⚠️ 明顯異常'
        WHEN Z分數 > 1.5 THEN '📊 輕微異常'
        ELSE '✅ 正常'
    END AS 異常狀態
FROM AnomalyDetection
ORDER BY Z分數 DESC;
```

---

## 💼 商業實戰案例

### 💰 員工薪資水平分析

```sql
-- 員工薪資相對位置分析
WITH SalaryAnalysis AS (
    SELECT 
        employee_id,
        employee_name,
        department,
        position,
        salary,
        AVG(salary) OVER() AS 公司平均薪資,
        AVG(salary) OVER(PARTITION BY department) AS 部門平均薪資,
        AVG(salary) OVER(PARTITION BY position) AS 職位平均薪資,
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) OVER() AS 公司薪資中位數
    FROM employees
),
SalaryComparison AS (
    SELECT 
        *,
        CAST((salary - 公司平均薪資) * 100.0 / 公司平均薪資 AS DECIMAL(5,2)) AS 相對公司平均百分比,
        CAST((salary - 部門平均薪資) * 100.0 / 部門平均薪資 AS DECIMAL(5,2)) AS 相對部門平均百分比,
        CAST((salary - 職位平均薪資) * 100.0 / 職位平均薪資 AS DECIMAL(5,2)) AS 相對職位平均百分比
    FROM SalaryAnalysis
)
SELECT 
    employee_name,
    department,
    position,
    salary,
    公司平均薪資,
    部門平均薪資,
    職位平均薪資,
    相對公司平均百分比,
    相對部門平均百分比,
    相對職位平均百分比,
    CASE 
        WHEN 相對部門平均百分比 > 20 THEN '🏆 部門頂尖'
        WHEN 相對部門平均百分比 > 10 THEN '⭐ 部門優秀'
        WHEN 相對部門平均百分比 > -10 THEN '📊 部門平均'
        ELSE '📉 需關注'
    END AS 部門表現等級
FROM SalaryComparison
ORDER BY department, salary DESC;
```

### 🛍️ 客戶消費行為分析

```sql
USE WebStoreDB;

-- 客戶消費行為相對分析
WITH CustomerBehaviorAnalysis AS (
    SELECT 
        TradesOrderGroup_MemberId,
        COUNT(*) AS 訂單次數,
        SUM(TradesOrderGroup_TotalPayment) AS 總消費金額,
        AVG(TradesOrderGroup_TotalPayment) AS 平均訂單金額,
        AVG(AVG(TradesOrderGroup_TotalPayment)) OVER() AS 全體客戶平均訂單金額,
        AVG(SUM(TradesOrderGroup_TotalPayment)) OVER() AS 全體客戶平均總金額,
        AVG(COUNT(*)) OVER() AS 全體客戶平均訂單次數
    FROM TradesOrderGroup(NOLOCK)
    WHERE TradesOrderGroup_ValidFlag = 1
        AND TradesOrderGroup_DateTime >= DATEADD(YEAR, -1, GETDATE())
    GROUP BY TradesOrderGroup_MemberId
),
CustomerSegmentation AS (
    SELECT 
        TradesOrderGroup_MemberId,
        訂單次數,
        總消費金額,
        平均訂單金額,
        全體客戶平均訂單金額,
        全體客戶平均總金額,
        全體客戶平均訂單次數,
        CASE 
            WHEN 總消費金額 > 全體客戶平均總金額 * 2 AND 訂單次數 > 全體客戶平均訂單次數 * 1.5 
            THEN '💎 鑽石客戶'
            WHEN 總消費金額 > 全體客戶平均總金額 * 1.5 
            THEN '🥇 金牌客戶'
            WHEN 平均訂單金額 > 全體客戶平均訂單金額 * 1.3 
            THEN '⭐ 高價值客戶'
            WHEN 訂單次數 > 全體客戶平均訂單次數 * 1.5 
            THEN '🔄 高頻客戶'
            ELSE '👤 一般客戶'
        END AS 客戶分級
    FROM CustomerBehaviorAnalysis
)
SELECT 
    客戶分級,
    COUNT(*) AS 客戶數量,
    AVG(總消費金額) AS 分級平均總消費,
    AVG(平均訂單金額) AS 分級平均訂單金額,
    AVG(訂單次數) AS 分級平均訂單次數
FROM CustomerSegmentation
GROUP BY 客戶分級
ORDER BY 分級平均總消費 DESC;
```

### 📈 產品銷售表現分析

```sql
-- 產品銷售表現相對分析
WITH ProductPerformance AS (
    SELECT 
        product_id,
        product_name,
        category,
        SUM(quantity) AS 總銷售量,
        SUM(amount) AS 總銷售額,
        AVG(price) AS 平均售價,
        AVG(SUM(quantity)) OVER() AS 全體平均銷售量,
        AVG(SUM(amount)) OVER() AS 全體平均銷售額,
        AVG(AVG(price)) OVER() AS 全體平均售價,
        AVG(SUM(quantity)) OVER(PARTITION BY category) AS 類別平均銷售量,
        AVG(SUM(amount)) OVER(PARTITION BY category) AS 類別平均銷售額
    FROM sales
    WHERE sale_date >= DATEADD(MONTH, -3, GETDATE())
    GROUP BY product_id, product_name, category
),
ProductRanking AS (
    SELECT 
        *,
        CAST((總銷售量 - 全體平均銷售量) * 100.0 / 全體平均銷售量 AS DECIMAL(5,2)) AS 銷售量偏離百分比,
        CAST((總銷售額 - 全體平均銷售額) * 100.0 / 全體平均銷售額 AS DECIMAL(5,2)) AS 銷售額偏離百分比,
        CAST((總銷售量 - 類別平均銷售量) * 100.0 / 類別平均銷售量 AS DECIMAL(5,2)) AS 類別銷售量偏離百分比
    FROM ProductPerformance
)
SELECT 
    product_name,
    category,
    總銷售量,
    總銷售額,
    平均售價,
    銷售量偏離百分比,
    銷售額偏離百分比,
    類別銷售量偏離百分比,
    CASE 
        WHEN 銷售額偏離百分比 > 100 THEN '🚀 明星產品'
        WHEN 銷售額偏離百分比 > 50 THEN '⭐ 熱賣產品'
        WHEN 銷售額偏離百分比 > 0 THEN '📈 表現良好'
        WHEN 銷售額偏離百分比 > -30 THEN '📊 表現平平'
        ELSE '📉 需改進'
    END AS 產品表現評級
FROM ProductRanking
ORDER BY 銷售額偏離百分比 DESC;
```

---

## 🎯 進階平均值技巧

### 🔄 加權平均值計算

```sql
-- 以銷售量為權重的加權平均價格
SELECT 
    product_category,
    AVG(price) AS 簡單平均價格,
    SUM(price * quantity) / SUM(quantity) AS 加權平均價格,
    SUM(price * quantity) / SUM(quantity) - AVG(price) AS 價格差異
FROM sales
GROUP BY product_category;
```

### 📊 多期間平均比較

```sql
-- 比較不同期間的平均表現
WITH MonthlyAverage AS (
    SELECT 
        YEAR(sale_date) AS 年份,
        MONTH(sale_date) AS 月份,
        AVG(amount) AS 月平均金額
    FROM sales
    WHERE sale_date >= DATEADD(MONTH, -12, GETDATE())
    GROUP BY YEAR(sale_date), MONTH(sale_date)
),
PeriodComparison AS (
    SELECT 
        年份,
        月份,
        月平均金額,
        LAG(月平均金額) OVER (ORDER BY 年份, 月份) AS 上月平均,
        LAG(月平均金額, 12) OVER (ORDER BY 年份, 月份) AS 去年同月平均,
        AVG(月平均金額) OVER (
            ORDER BY 年份, 月份 
            ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
        ) AS 三個月移動平均
    FROM MonthlyAverage
)
SELECT 
    年份,
    月份,
    月平均金額,
    上月平均,
    去年同月平均,
    三個月移動平均,
    CASE 
        WHEN 上月平均 IS NOT NULL THEN 
            CAST((月平均金額 - 上月平均) * 100.0 / 上月平均 AS DECIMAL(5,2))
    END AS 月增長率,
    CASE 
        WHEN 去年同月平均 IS NOT NULL THEN 
            CAST((月平均金額 - 去年同月平均) * 100.0 / 去年同月平均 AS DECIMAL(5,2))
    END AS 年增長率
FROM PeriodComparison
ORDER BY 年份, 月份;
```

### 🎪 條件性平均值

```sql
-- 條件性平均值計算
SELECT 
    customer_id,
    order_date,
    amount,
    -- 只計算大額訂單的平均
    AVG(CASE WHEN amount > 1000 THEN amount END) OVER(
        PARTITION BY customer_id
    ) AS 大額訂單平均,
    -- 工作日與假日分別計算
    AVG(CASE WHEN DATEPART(WEEKDAY, order_date) BETWEEN 2 AND 6 
        THEN amount END) OVER() AS 工作日平均,
    AVG(CASE WHEN DATEPART(WEEKDAY, order_date) IN (1, 7) 
        THEN amount END) OVER() AS 假日平均
FROM orders
ORDER BY customer_id, order_date;
```

---

### ⚠️ 注意事項

1. **NULL 值處理**
   ```sql
   -- AVG 會自動忽略 NULL 值
   AVG(ISNULL(amount, 0)) -- 如果希望 NULL 當作 0
   AVG(amount) -- 忽略 NULL 值
   ```

2. **資料類型精度**
   ```sql
   -- 避免整數除法的精度問題
   AVG(CAST(amount AS DECIMAL(10,2))) OVER()
   ```

---