## 📖 目錄
- [📖 目錄](#-目錄)
- [🌟 MAX/MIN 極值概述](#-maxmin-極值概述)
  - [🎨 極值分析類型](#-極值分析類型)
- [💎 基礎極值範例](#-基礎極值範例)
  - [🏆 全域最大最小值](#-全域最大最小值)
  - [🔄 分組極值分析](#-分組極值分析)
  - [📈 歷史極值追蹤](#-歷史極值追蹤)
- [💎 相對於最高值的百分比](#-相對於最高值的百分比)
  - [📊 基礎百分比計算](#-基礎百分比計算)
  - [🎪 分組百分比分析](#-分組百分比分析)
  - [📈 極值達成率分析](#-極值達成率分析)
- [💼 商業實戰案例](#-商業實戰案例)
  - [💰 業績表現分析](#-業績表現分析)
  - [🛍️ 產品價格定位分析](#️-產品價格定位分析)
  - [👑 客戶價值分層](#-客戶價值分層)
- [🎯 進階極值技巧](#-進階極值技巧)
  - [🔄 滾動極值計算](#-滾動極值計算)
  - [📊 多維度極值比較](#-多維度極值比較)
  - [⚡ 極值變化追蹤](#-極值變化追蹤)
- [🎪 特殊應用場景](#-特殊應用場景)
  - [🏃‍♂️ 即時排行榜](#️-即時排行榜)
  - [📈 績效基準設定](#-績效基準設定)
  - [🎁 獎勵機制觸發](#-獎勵機制觸發)
  - [⚠️ 注意事項](#️-注意事項)
---

## 🌟 MAX/MIN 極值概述

### 🎨 極值分析類型

| 分析類型 | 語法範例 | 商業應用 |
|----------|----------|----------|
| **🌐 全域極值** | `MAX(amount) OVER()` | 整體標竿比較 |
| **🔄 分組極值** | `MAX(amount) OVER(PARTITION BY category)` | 同類最佳表現 |
| **📈 歷史極值** | `MAX(amount) OVER(ORDER BY date ROWS UNBOUNDED PRECEDING)` | 歷史最高紀錄 |
| **🎪 滾動極值** | `MAX(amount) OVER(ORDER BY date ROWS 6 PRECEDING)` | 短期高點低點 |

---

## 💎 基礎極值範例

### 🏆 全域最大最小值

```sql
-- 分析每筆訂單相對於全域極值的位置
SELECT 
    order_id,
    customer_id,
    order_amount,
    MAX(order_amount) OVER() AS 歷史最高金額,
    MIN(order_amount) OVER() AS 歷史最低金額,
    order_amount - MIN(order_amount) OVER() AS 超越最低金額,
    MAX(order_amount) OVER() - order_amount AS 距離最高金額,
    CASE 
        WHEN order_amount = MAX(order_amount) OVER() THEN '🏆 歷史最高'
        WHEN order_amount = MIN(order_amount) OVER() THEN '📉 歷史最低'
        ELSE '📊 一般範圍'
    END AS 極值狀態
FROM orders
ORDER BY order_amount DESC;
```

### 🔄 分組極值分析

```sql
-- 各類別產品的價格極值分析
SELECT 
    product_name,
    category,
    price,
    MAX(price) OVER(PARTITION BY category) AS 類別最高價,
    MIN(price) OVER(PARTITION BY category) AS 類別最低價,
    MAX(price) OVER(PARTITION BY category) - MIN(price) OVER(PARTITION BY category) AS 類別價格區間,
    CASE 
        WHEN price = MAX(price) OVER(PARTITION BY category) THEN '👑 類別之王'
        WHEN price = MIN(price) OVER(PARTITION BY category) THEN '💰 經濟選擇'
        ELSE '📊 中間價位'
    END AS 價格定位
FROM products
ORDER BY category, price DESC;
```

### 📈 歷史極值追蹤

```sql
-- 追蹤每日銷售的歷史極值記錄
SELECT 
    sale_date,
    daily_sales,
    MAX(daily_sales) OVER (
        ORDER BY sale_date 
        ROWS UNBOUNDED PRECEDING
    ) AS 截至當日最高銷售,
    MIN(daily_sales) OVER (
        ORDER BY sale_date 
        ROWS UNBOUNDED PRECEDING
    ) AS 截至當日最低銷售,
    CASE 
        WHEN daily_sales = MAX(daily_sales) OVER (
            ORDER BY sale_date 
            ROWS UNBOUNDED PRECEDING
        ) THEN '🚀 創新高!'
        WHEN daily_sales = MIN(daily_sales) OVER (
            ORDER BY sale_date 
            ROWS UNBOUNDED PRECEDING
        ) THEN '📉 創新低'
        ELSE '📊 一般'
    END AS 紀錄狀態
FROM daily_sales
ORDER BY sale_date;
```

---

## 💎 相對於最高值的百分比

### 📊 基礎百分比計算

```sql
USE WebStoreDB;

-- 計算每筆訂單相對於最高金額的百分比
SELECT   
    TradesOrderGroup_Id,
    TradesOrderGroup_MemberId,
    TradesOrderGroup_TotalPayment,
    MAX(TradesOrderGroup_TotalPayment) OVER() AS 最高訂單金額,
    MIN(TradesOrderGroup_TotalPayment) OVER() AS 最低訂單金額,
    CAST(
        TradesOrderGroup_TotalPayment * 100.0 / 
        MAX(TradesOrderGroup_TotalPayment) OVER() 
        AS DECIMAL(10,2)
    ) AS 相對最高值百分比,
    CAST(
        (TradesOrderGroup_TotalPayment - MIN(TradesOrderGroup_TotalPayment) OVER()) * 100.0 / 
        (MAX(TradesOrderGroup_TotalPayment) OVER() - MIN(TradesOrderGroup_TotalPayment) OVER())
        AS DECIMAL(10,2)
    ) AS 區間內百分位
FROM TradesOrderGroup(NOLOCK)
WHERE TradesOrderGroup_ValidFlag = 1
    AND TradesOrderGroup_DateTime > '2025-01-01'
ORDER BY 相對最高值百分比 DESC;
```

### 🎪 分組百分比分析

```sql
-- 各會員相對於個人最高消費的百分比
WITH MemberMaxAnalysis AS (
    SELECT 
        TradesOrderGroup_MemberId,
        TradesOrderGroup_DateTime,
        TradesOrderGroup_TotalPayment,
        MAX(TradesOrderGroup_TotalPayment) OVER(
            PARTITION BY TradesOrderGroup_MemberId
        ) AS 會員最高消費,
        MIN(TradesOrderGroup_TotalPayment) OVER(
            PARTITION BY TradesOrderGroup_MemberId
        ) AS 會員最低消費,
        COUNT(*) OVER(
            PARTITION BY TradesOrderGroup_MemberId
        ) AS 會員總訂單數
    FROM TradesOrderGroup(NOLOCK)
    WHERE TradesOrderGroup_ValidFlag = 1
)
SELECT 
    TradesOrderGroup_MemberId,
    TradesOrderGroup_DateTime,
    TradesOrderGroup_TotalPayment,
    會員最高消費,
    會員最低消費,
    會員總訂單數,
    CAST(
        TradesOrderGroup_TotalPayment * 100.0 / 會員最高消費 
        AS DECIMAL(5,2)
    ) AS 相對個人最高百分比,
    CASE 
        WHEN TradesOrderGroup_TotalPayment = 會員最高消費 THEN '🏆 個人最高'
        WHEN TradesOrderGroup_TotalPayment >= 會員最高消費 * 0.8 THEN '⭐ 高消費'
        WHEN TradesOrderGroup_TotalPayment >= 會員最高消費 * 0.5 THEN '📊 中等消費'
        WHEN TradesOrderGroup_TotalPayment >= 會員最高消費 * 0.3 THEN '📉 低消費'
        ELSE '❄️ 最低消費'
    END AS 消費等級
FROM MemberMaxAnalysis
WHERE 會員總訂單數 >= 5  -- 只分析有足夠訂單數的會員
ORDER BY TradesOrderGroup_MemberId, TradesOrderGroup_DateTime;
```

### 📈 極值達成率分析

```sql
-- 分析各部門相對於公司最高薪資的達成率
WITH SalaryExtremesAnalysis AS (
    SELECT 
        employee_name,
        department,
        position,
        salary,
        MAX(salary) OVER() AS 公司最高薪資,
        MAX(salary) OVER(PARTITION BY department) AS 部門最高薪資,
        MIN(salary) OVER() AS 公司最低薪資,
        MIN(salary) OVER(PARTITION BY department) AS 部門最低薪資
    FROM employees
)
SELECT 
    department,
    employee_name,
    position,
    salary,
    公司最高薪資,
    部門最高薪資,
    CAST(salary * 100.0 / 公司最高薪資 AS DECIMAL(5,2)) AS 公司最高達成率,
    CAST(salary * 100.0 / 部門最高薪資 AS DECIMAL(5,2)) AS 部門最高達成率,
    CASE 
        WHEN salary = 公司最高薪資 THEN '👑 公司之王'
        WHEN salary = 部門最高薪資 THEN '🏆 部門冠軍'
        WHEN salary >= 部門最高薪資 * 0.9 THEN '⭐ 頂級薪資'
        WHEN salary >= 部門最高薪資 * 0.7 THEN '📈 高級薪資'
        ELSE '📊 標準薪資'
    END AS 薪資等級
FROM SalaryExtremesAnalysis
ORDER BY department, salary DESC;
```

---

## 💼 商業實戰案例

### 💰 業績表現分析

```sql
-- 業務員績效相對分析
WITH SalesPerformance AS (
    SELECT 
        salesperson_id,
        salesperson_name,
        region,
        YEAR(sale_date) AS sale_year,
        MONTH(sale_date) AS sale_month,
        SUM(amount) AS 月度業績
    FROM sales
    WHERE sale_date >= DATEADD(YEAR, -1, GETDATE())
    GROUP BY salesperson_id, salesperson_name, region, YEAR(sale_date), MONTH(sale_date)
),
PerformanceExtremesAnalysis AS (
    SELECT 
        *,
        MAX(月度業績) OVER() AS 全公司最高月度業績,
        MAX(月度業績) OVER(PARTITION BY region) AS 區域最高月度業績,
        MAX(月度業績) OVER(PARTITION BY salesperson_id) AS 個人最高月度業績,
        MIN(月度業績) OVER(PARTITION BY salesperson_id) AS 個人最低月度業績,
        AVG(月度業績) OVER(PARTITION BY salesperson_id) AS 個人平均月度業績
    FROM SalesPerformance
)
SELECT 
    salesperson_name,
    region,
    sale_year,
    sale_month,
    月度業績,
    個人最高月度業績,
    個人平均月度業績,
    CAST(月度業績 * 100.0 / 全公司最高月度業績 AS DECIMAL(5,2)) AS 全公司最高達成率,
    CAST(月度業績 * 100.0 / 區域最高月度業績 AS DECIMAL(5,2)) AS 區域最高達成率,
    CAST(月度業績 * 100.0 / 個人最高月度業績 AS DECIMAL(5,2)) AS 個人最高達成率,
    CASE 
        WHEN 月度業績 = 個人最高月度業績 THEN '🚀 個人巔峰'
        WHEN 月度業績 >= 個人平均月度業績 * 1.2 THEN '⭐ 超常發揮'
        WHEN 月度業績 >= 個人平均月度業績 THEN '📈 正常表現'
        WHEN 月度業績 >= 個人平均月度業績 * 0.8 THEN '📊 略低於平均'
        ELSE '📉 需要改進'
    END AS 表現評級
FROM PerformanceExtremesAnalysis
ORDER BY salesperson_name, sale_year, sale_month;
```

### 🛍️ 產品價格定位分析

```sql
-- 產品價格定位與市場區間分析
WITH ProductPriceAnalysis AS (
    SELECT 
        product_id,
        product_name,
        category,
        brand,
        price,
        MAX(price) OVER() AS 全市場最高價,
        MIN(price) OVER() AS 全市場最低價,
        MAX(price) OVER(PARTITION BY category) AS 類別最高價,
        MIN(price) OVER(PARTITION BY category) AS 類別最低價,
        AVG(price) OVER(PARTITION BY category) AS 類別平均價,
        MAX(price) OVER(PARTITION BY brand) AS 品牌最高價,
        MIN(price) OVER(PARTITION BY brand) AS 品牌最低價
    FROM products
),
PricePositioning AS (
    SELECT 
        *,
        CAST((price - 類別最低價) * 100.0 / (類別最高價 - 類別最低價) AS DECIMAL(5,2)) AS 類別價格百分位,
        CAST(price * 100.0 / 類別最高價 AS DECIMAL(5,2)) AS 類別最高價達成率,
        CASE 
            WHEN price >= 類別平均價 * 1.5 THEN '💎 奢華級'
            WHEN price >= 類別平均價 * 1.2 THEN '⭐ 高端級'
            WHEN price >= 類別平均價 * 0.8 THEN '📊 主流級'
            WHEN price >= 類別平均價 * 0.6 THEN '💰 經濟級'
            ELSE '🏷️ 入門級'
        END AS 價格定位
    FROM ProductPriceAnalysis
)
SELECT 
    category,
    品牌,
    product_name,
    price,
    類別最高價,
    類別最低價,
    類別平均價,
    類別價格百分位,
    類別最高價達成率,
    價格定位,
    CASE 
        WHEN price = 類別最高價 THEN '👑 類別價格之王'
        WHEN price = 品牌最高價 THEN '🏆 品牌旗艦'
        WHEN 類別價格百分位 >= 90 THEN '💎 頂級產品'
        WHEN 類別價格百分位 >= 70 THEN '⭐ 高端產品'
        WHEN 類別價格百分位 >= 30 THEN '📊 主流產品'
        ELSE '💰 經濟產品'
    END AS 市場地位
FROM PricePositioning
ORDER BY category, price DESC;
```

### 👑 客戶價值分層

```sql
USE WebStoreDB;

-- 客戶價值分層與極值分析
WITH CustomerValueAnalysis AS (
    SELECT 
        TradesOrderGroup_MemberId,
        COUNT(*) AS 訂單總數,
        SUM(TradesOrderGroup_TotalPayment) AS 累計消費,
        AVG(TradesOrderGroup_TotalPayment) AS 平均訂單金額,
        MAX(TradesOrderGroup_TotalPayment) AS 最高單筆消費,
        MIN(TradesOrderGroup_TotalPayment) AS 最低單筆消費,
        DATEDIFF(DAY, MIN(TradesOrderGroup_DateTime), MAX(TradesOrderGroup_DateTime)) AS 客戶生命週期天數
    FROM TradesOrderGroup(NOLOCK)
    WHERE TradesOrderGroup_ValidFlag = 1
        AND TradesOrderGroup_DateTime >= DATEADD(YEAR, -2, GETDATE())
    GROUP BY TradesOrderGroup_MemberId
),
ValueTierAnalysis AS (
    SELECT 
        *,
        MAX(累計消費) OVER() AS 全平台最高累計消費,
        MAX(最高單筆消費) OVER() AS 全平台最高單筆,
        MAX(訂單總數) OVER() AS 全平台最多訂單數,
        CAST(累計消費 * 100.0 / MAX(累計消費) OVER() AS DECIMAL(5,2)) AS 消費力百分位,
        CAST(最高單筆消費 * 100.0 / MAX(最高單筆消費) OVER() AS DECIMAL(5,2)) AS 單筆消費力百分位,
        CAST(訂單總數 * 100.0 / MAX(訂單總數) OVER() AS DECIMAL(5,2)) AS 活躍度百分位
    FROM CustomerValueAnalysis
    WHERE 客戶生命週期天數 > 30  -- 排除新註冊用戶
)
SELECT 
    TradesOrderGroup_MemberId,
    累計消費,
    訂單總數,
    平均訂單金額,
    最高單筆消費,
    客戶生命週期天數,
    消費力百分位,
    單筆消費力百分位,
    活躍度百分位,
    CASE 
        WHEN 消費力百分位 >= 95 AND 單筆消費力百分位 >= 90 THEN '💎 鑽石VIP'
        WHEN 消費力百分位 >= 85 OR 單筆消費力百分位 >= 80 THEN '👑 白金VIP'
        WHEN 消費力百分位 >= 70 OR 活躍度百分位 >= 80 THEN '🥇 金牌會員'
        WHEN 消費力百分位 >= 50 OR 活躍度百分位 >= 60 THEN '🥈 銀牌會員'
        WHEN 消費力百分位 >= 30 OR 活躍度百分位 >= 40 THEN '🥉 銅牌會員'
        ELSE '👤 一般會員'
    END AS 會員等級,
    CASE 
        WHEN 累計消費 = 全平台最高累計消費 THEN '🏆 消費王者'
        WHEN 最高單筆消費 = 全平台最高單筆 THEN '💰 單筆之王'
        WHEN 訂單總數 = 全平台最多訂單數 THEN '🔄 活躍之王'
        ELSE NULL
    END AS 特殊榮譽
FROM ValueTierAnalysis
ORDER BY 消費力百分位 DESC;
```

---

## 🎯 進階極值技巧

### 🔄 滾動極值計算

```sql
-- 30天滾動高點低點分析
WITH RollingExtremes AS (
    SELECT 
        sale_date,
        daily_sales,
        MAX(daily_sales) OVER (
            ORDER BY sale_date 
            ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
        ) AS 三十天高點,
        MIN(daily_sales) OVER (
            ORDER BY sale_date 
            ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
        ) AS 三十天低點,
        AVG(daily_sales) OVER (
            ORDER BY sale_date 
            ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
        ) AS 三十天平均
    FROM daily_sales
)
SELECT 
    sale_date,
    daily_sales,
    三十天高點,
    三十天低點,
    三十天平均,
    CAST((daily_sales - 三十天低點) * 100.0 / (三十天高點 - 三十天低點) AS DECIMAL(5,2)) AS 區間百分位,
    CASE 
        WHEN daily_sales = 三十天高點 THEN '🔥 創30天新高'
        WHEN daily_sales = 三十天低點 THEN '❄️ 創30天新低'
        WHEN daily_sales >= 三十天平均 * 1.2 THEN '📈 表現優異'
        WHEN daily_sales <= 三十天平均 * 0.8 THEN '📉 表現不佳'
        ELSE '📊 正常範圍'
    END AS 表現狀態
FROM RollingExtremes
ORDER BY sale_date;
```

### 📊 多維度極值比較

```sql
-- 多維度銷售極值分析
WITH MultiDimensionExtremes AS (
    SELECT 
        product_name,
        category,
        region,
        month_year,
        sales_amount,
        -- 不同維度的極值
        MAX(sales_amount) OVER() AS 全域最高,
        MAX(sales_amount) OVER(PARTITION BY category) AS 類別最高,
        MAX(sales_amount) OVER(PARTITION BY region) AS 區域最高,
        MAX(sales_amount) OVER(PARTITION BY month_year) AS 月份最高,
        MIN(sales_amount) OVER() AS 全域最低,
        MIN(sales_amount) OVER(PARTITION BY category) AS 類別最低,
        MIN(sales_amount) OVER(PARTITION BY region) AS 區域最低
    FROM product_sales
)
SELECT 
    product_name,
    category,
    region,
    month_year,
    sales_amount,
    CASE 
        WHEN sales_amount = 全域最高 THEN '🌟 全球冠軍'
        WHEN sales_amount = 類別最高 THEN '🏆 類別冠軍'
        WHEN sales_amount = 區域最高 THEN '👑 區域王者'
        WHEN sales_amount = 月份最高 THEN '📅 月度之星'
        ELSE '📊 一般表現'
    END AS 冠軍稱號,
    CONCAT(
        CAST(sales_amount * 100.0 / 全域最高 AS DECIMAL(5,1)), '% (全域), ',
        CAST(sales_amount * 100.0 / 類別最高 AS DECIMAL(5,1)), '% (類別), ',
        CAST(sales_amount * 100.0 / 區域最高 AS DECIMAL(5,1)), '% (區域)'
    ) AS 多維度達成率
FROM MultiDimensionExtremes
ORDER BY sales_amount DESC;
```

### ⚡ 極值變化追蹤

```sql
-- 追蹤極值的變化軌跡
WITH ExtremeChanges AS (
    SELECT 
        sale_date,
        daily_sales,
        MAX(daily_sales) OVER (
            ORDER BY sale_date 
            ROWS UNBOUNDED PRECEDING
        ) AS 歷史最高,
        LAG(MAX(daily_sales) OVER (
            ORDER BY sale_date 
            ROWS UNBOUNDED PRECEDING
        )) OVER (ORDER BY sale_date) AS 前日歷史最高,
        MIN(daily_sales) OVER (
            ORDER BY sale_date 
            ROWS UNBOUNDED PRECEDING
        ) AS 歷史最低,
        LAG(MIN(daily_sales) OVER (
            ORDER BY sale_date 
            ROWS UNBOUNDED PRECEDING
        )) OVER (ORDER BY sale_date) AS 前日歷史最低
    FROM daily_sales
)
SELECT 
    sale_date,
    daily_sales,
    歷史最高,
    歷史最低,
    CASE 
        WHEN 歷史最高 > 前日歷史最高 THEN '🚀 突破歷史新高!'
        WHEN 歷史最低 < 前日歷史最低 THEN '📉 創歷史新低'
        ELSE '📊 無新紀錄'
    END AS 紀錄變化,
    DATEDIFF(DAY, 
        LAG(sale_date) OVER (ORDER BY sale_date), 
        sale_date
    ) AS 距離上次紀錄天數
FROM ExtremeChanges
WHERE 歷史最高 > 前日歷史最高 OR 歷史最低 < 前日歷史最低
ORDER BY sale_date;
```

---

## 🎪 特殊應用場景

### 🏃‍♂️ 即時排行榜

```sql
-- 動態更新的即時排行榜
WITH RealTimeLeaderboard AS (
    SELECT 
        player_id,
        player_name,
        current_score,
        MAX(current_score) OVER() AS 最高分,
        RANK() OVER (ORDER BY current_score DESC) AS 排名,
        CAST(current_score * 100.0 / MAX(current_score) OVER() AS DECIMAL(5,1)) AS 與第一名差距百分比
    FROM player_scores
    WHERE last_updated >= DATEADD(HOUR, -1, GETDATE())  -- 最近1小時活躍
)
SELECT 
    排名,
    player_name,
    current_score,
    與第一名差距百分比,
    CASE 
        WHEN 排名 = 1 THEN '👑 王者'
        WHEN 排名 <= 3 THEN '🏆 前三強'
        WHEN 排名 <= 10 THEN '⭐ 十強'
        WHEN 與第一名差距百分比 >= 80 THEN '🔥 競爭激烈'
        ELSE '📊 努力追趕'
    END AS 位置狀態
FROM RealTimeLeaderboard
ORDER BY 排名;
```

### 📈 績效基準設定

```sql
-- 基於極值設定績效基準
WITH PerformanceBenchmarks AS (
    SELECT 
        department,
        AVG(performance_score) AS 部門平均,
        MAX(performance_score) AS 部門最高,
        MIN(performance_score) AS 部門最低,
        MAX(performance_score) OVER() AS 全公司最高,
        -- 設定不同等級的基準線
        MAX(performance_score) * 0.9 AS 優秀基準,
        MAX(performance_score) * 0.7 AS 良好基準,
        MAX(performance_score) * 0.5 AS 及格基準
    FROM employee_performance
    GROUP BY department
)
SELECT 
    department,
    部門平均,
    部門最高,
    優秀基準,
    良好基準,
    及格基準,
    CASE 
        WHEN 部門最高 >= 優秀基準 THEN '🌟 卓越部門'
        WHEN 部門平均 >= 良好基準 THEN '⭐ 優秀部門'
        WHEN 部門平均 >= 及格基準 THEN '📊 標準部門'
        ELSE '📈 待提升部門'
    END AS 部門評級
FROM PerformanceBenchmarks
ORDER BY 部門最高 DESC;
```

### 🎁 獎勵機制觸發

```sql
-- 基於極值達成的獎勵觸發機制
WITH RewardTriggers AS (
    SELECT 
        employee_id,
        employee_name,
        monthly_sales,
        MAX(monthly_sales) OVER(PARTITION BY YEAR(sale_month)) AS 年度最高月銷售,
        MAX(monthly_sales) OVER() AS 歷史最高月銷售,
        AVG(monthly_sales) OVER(PARTITION BY employee_id) AS 個人平均月銷售
    FROM monthly_employee_sales
    WHERE sale_month >= DATEADD(MONTH, -12, GETDATE())
)
SELECT 
    employee_name,
    monthly_sales,
    個人平均月銷售,
    年度最高月銷售,
    歷史最高月銷售,
    CASE 
        WHEN monthly_sales = 歷史最高月銷售 THEN '💎 歷史紀錄獎 (10萬獎金)'
        WHEN monthly_sales = 年度最高月銷售 THEN '👑 年度冠軍獎 (5萬獎金)'
        WHEN monthly_sales >= 個人平均月銷售 * 2 THEN '🚀 突破獎 (2萬獎金)'
        WHEN monthly_sales >= 個人平均月銷售 * 1.5 THEN '⭐ 進步獎 (1萬獎金)'
        ELSE '📊 基本獎勵 (5千獎金)'
    END AS 獎勵等級
FROM RewardTriggers
ORDER BY monthly_sales DESC;
```

---

### ⚠️ 注意事項

1. **NULL 值處理**
   ```sql
   -- MAX/MIN 會忽略 NULL 值
   MAX(ISNULL(amount, 0)) OVER()  -- 如果希望 NULL 當作 0
   ```

2. **大量資料的效能考量**
   ```sql
   -- 限制計算範圍
   MAX(amount) OVER(PARTITION BY category)  -- 而非 MAX(amount) OVER()
   ```

