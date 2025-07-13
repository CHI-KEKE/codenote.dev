## 📖 目錄
- [📖 目錄](#-目錄)
- [🌟 SUM 累計總和概述](#-sum-累計總和概述)
  - [🎨 Frame 範圍選項](#-frame-範圍選項)
- [🎮 基礎累計範例](#-基礎累計範例)
  - [📊 累計遊玩時數](#-累計遊玩時數)
  - [💰 累計營收統計](#-累計營收統計)
  - [📈 移動總和計算](#-移動總和計算)
- [🎯 進階累計應用](#-進階累計應用)
  - [🎁 首次突破特定金額](#-首次突破特定金額)
  - [📊 推播類型百分比統計](#-推播類型百分比統計)
  - [🔄 重置累計計算](#-重置累計計算)
- [💼 商業實戰案例](#-商業實戰案例)
  - [📈 會員消費成長軌跡](#-會員消費成長軌跡)
  - [🎪 季度累計業績追蹤](#-季度累計業績追蹤)
  - [💎 VIP 等級晉升追蹤](#-vip-等級晉升追蹤)
- [⚡ 效能優化技巧](#-效能優化技巧)
  - [🚀 與傳統方法比較](#-與傳統方法比較)
  - [📊 索引策略](#-索引策略)

---

## 🌟 SUM 累計總和概述

### 🎨 Frame 範圍選項

| Frame 選項 | 說明 | 適用場景 |
|------------|------|----------|
| **UNBOUNDED PRECEDING** | 從分組開始到當前列 | 累計總和 |
| **CURRENT ROW** | 僅當前列 | 單筆計算 |
| **n PRECEDING** | 前 n 列到當前列 | 移動平均 |
| **n FOLLOWING** | 當前列到後 n 列 | 預測分析 |

---

## 🎮 基礎累計範例

### 📊 累計遊玩時數

```sql
-- 🚀 方法一：Window Function (推薦)
SELECT 
    player_id,
    event_date,
    games_played,
    SUM(games_played) OVER (
        PARTITION BY player_id
        ORDER BY event_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS 累計遊玩時數
FROM Activity
ORDER BY player_id, event_date;

-- 📊 結果展示
-- player_id | event_date | games_played | 累計遊玩時數
-- 1         | 2021-01-01 | 5           | 5
-- 1         | 2021-01-02 | 3           | 8
-- 1         | 2021-01-03 | 7           | 15
```

### 💰 累計營收統計

```sql
-- 每日累計營收追蹤
SELECT 
    sale_date,
    daily_revenue,
    SUM(daily_revenue) OVER (
        ORDER BY sale_date
        ROWS UNBOUNDED PRECEDING
    ) AS 年度累計營收
FROM daily_sales
ORDER BY sale_date;
```

### 📈 移動總和計算

```sql
-- 最近7天移動總和
SELECT 
    sale_date,
    daily_amount,
    SUM(daily_amount) OVER (
        ORDER BY sale_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS 七天移動總和
FROM sales_data
ORDER BY sale_date;
```

---

## 🎯 進階累計應用

### 🎁 首次突破特定金額

```sql
USE ERPDB;

-- 找出會員首次累計消費突破 10,000 元的日期
WITH AccumulatedPayments AS (
    SELECT  
        SalesOrderGroup_MemberId,
        SalesOrderGroup_DateTime,
        SalesOrderGroup_TotalPayment,
        SUM(SalesOrderGroup_TotalPayment) OVER (
            PARTITION BY SalesOrderGroup_MemberId 
            ORDER BY SalesOrderGroup_DateTime
            ROWS UNBOUNDED PRECEDING
        ) AS 累計消費金額
    FROM SalesOrderGroup(NOLOCK)
    WHERE SalesOrderGroup_ValidFlag = 1
),
MilestoneReached AS (
    SELECT 
        SalesOrderGroup_MemberId,
        SalesOrderGroup_DateTime,
        累計消費金額,
        CASE 
            WHEN 累計消費金額 >= 10000 
            AND LAG(累計消費金額) OVER (
                PARTITION BY SalesOrderGroup_MemberId 
                ORDER BY SalesOrderGroup_DateTime
            ) < 10000 
            THEN 1 
            ELSE 0 
        END AS 首次突破標記
    FROM AccumulatedPayments
)
SELECT 
    SalesOrderGroup_MemberId,
    SalesOrderGroup_DateTime AS 首次突破10000元日期,
    累計消費金額
FROM MilestoneReached
WHERE 首次突破標記 = 1;
```

> 🎉 **商業應用**：識別達到里程碑的會員，觸發獎勵機制

### 📊 推播類型百分比統計

```sql
USE NotificationDB;

-- 計算各推播類型佔比與累計佔比
WITH NotificationStats AS (
    SELECT   
        PushNotification_TypeDef,
        COUNT(*) AS 類型數量,
        SUM(COUNT(*)) OVER() AS 總通知數
    FROM PushNotification(NOLOCK)
    WHERE PushNotification_ValidFlag = 1
    GROUP BY PushNotification_TypeDef
),
PercentageStats AS (
    SELECT 
        PushNotification_TypeDef,
        類型數量,
        總通知數,
        CAST(類型數量 * 100.0 / 總通知數 AS DECIMAL(10,2)) AS 類型百分比,
        SUM(CAST(類型數量 * 100.0 / 總通知數 AS DECIMAL(10,2))) OVER (
            ORDER BY 類型數量 DESC
            ROWS UNBOUNDED PRECEDING
        ) AS 累計百分比
    FROM NotificationStats
)
SELECT 
    PushNotification_TypeDef,
    類型數量,
    類型百分比,
    累計百分比,
    CASE 
        WHEN 累計百分比 <= 80 THEN '🔥 核心類型'
        WHEN 累計百分比 <= 95 THEN '⭐ 重要類型'
        ELSE '📋 其他類型'
    END AS 重要性分類
FROM PercentageStats
ORDER BY 類型數量 DESC;
```

### 🔄 重置累計計算

```sql
-- 按季度重置的累計銷售額
WITH QuarterlySales AS (
    SELECT 
        sale_date,
        amount,
        YEAR(sale_date) AS sale_year,
        DATEPART(QUARTER, sale_date) AS sale_quarter
    FROM sales
),
QuarterlyRunningTotal AS (
    SELECT 
        sale_date,
        amount,
        sale_year,
        sale_quarter,
        SUM(amount) OVER (
            PARTITION BY sale_year, sale_quarter
            ORDER BY sale_date
            ROWS UNBOUNDED PRECEDING
        ) AS 季度累計銷售額
    FROM QuarterlySales
)
SELECT * 
FROM QuarterlyRunningTotal
ORDER BY sale_date;
```

---

## 💼 商業實戰案例

### 📈 會員消費成長軌跡

```sql
USE WebStoreDB;

-- 追蹤會員消費成長軌跡與達成目標進度
WITH MemberGrowthTrack AS (
    SELECT 
        TradesOrderGroup_MemberId,
        TradesOrderGroup_DateTime,
        TradesOrderGroup_TotalPayment,
        SUM(TradesOrderGroup_TotalPayment) OVER (
            PARTITION BY TradesOrderGroup_MemberId
            ORDER BY TradesOrderGroup_DateTime
            ROWS UNBOUNDED PRECEDING
        ) AS 累計消費金額,
        ROW_NUMBER() OVER (
            PARTITION BY TradesOrderGroup_MemberId
            ORDER BY TradesOrderGroup_DateTime
        ) AS 購買次數
    FROM TradesOrderGroup(NOLOCK)
    WHERE TradesOrderGroup_ValidFlag = 1
        AND TradesOrderGroup_DateTime >= DATEADD(YEAR, -1, GETDATE())
)
SELECT 
    TradesOrderGroup_MemberId,
    TradesOrderGroup_DateTime,
    TradesOrderGroup_TotalPayment,
    累計消費金額,
    購買次數,
    CASE 
        WHEN 累計消費金額 >= 50000 THEN '💎 鑽石會員'
        WHEN 累計消費金額 >= 20000 THEN '🥇 金牌會員'
        WHEN 累計消費金額 >= 10000 THEN '🥈 銀牌會員'
        WHEN 累計消費金額 >= 5000 THEN '🥉 銅牌會員'
        ELSE '👤 一般會員'
    END AS 會員等級,
    CASE 
        WHEN 累計消費金額 < 5000 THEN 5000 - 累計消費金額
        WHEN 累計消費金額 < 10000 THEN 10000 - 累計消費金額
        WHEN 累計消費金額 < 20000 THEN 20000 - 累計消費金額
        WHEN 累計消費金額 < 50000 THEN 50000 - 累計消費金額
        ELSE 0
    END AS 距離下一等級差額
FROM MemberGrowthTrack
ORDER BY TradesOrderGroup_MemberId, TradesOrderGroup_DateTime;
```

### 🎪 季度累計業績追蹤

```sql
-- 業務人員季度業績追蹤與目標達成率
WITH QuarterlyPerformance AS (
    SELECT 
        salesperson_id,
        sale_date,
        amount,
        YEAR(sale_date) AS sale_year,
        DATEPART(QUARTER, sale_date) AS quarter,
        SUM(amount) OVER (
            PARTITION BY salesperson_id, YEAR(sale_date), DATEPART(QUARTER, sale_date)
            ORDER BY sale_date
            ROWS UNBOUNDED PRECEDING
        ) AS 季度累計業績
    FROM sales
    WHERE sale_date >= DATEADD(YEAR, -1, GETDATE())
),
PerformanceWithTarget AS (
    SELECT 
        *,
        100000 AS 季度目標,  -- 假設季度目標為 10 萬
        CAST(季度累計業績 * 100.0 / 100000 AS DECIMAL(5,2)) AS 目標達成率
    FROM QuarterlyPerformance
)
SELECT 
    salesperson_id,
    sale_year,
    quarter,
    季度累計業績,
    季度目標,
    目標達成率,
    CASE 
        WHEN 目標達成率 >= 100 THEN '🎉 已達成'
        WHEN 目標達成率 >= 80 THEN '⚡ 接近達成'
        WHEN 目標達成率 >= 60 THEN '📈 穩步進行'
        ELSE '🚨 需要加油'
    END AS 達成狀態
FROM PerformanceWithTarget
WHERE sale_date = (
    SELECT MAX(sale_date) 
    FROM PerformanceWithTarget p2 
    WHERE p2.salesperson_id = PerformanceWithTarget.salesperson_id
    AND p2.sale_year = PerformanceWithTarget.sale_year 
    AND p2.quarter = PerformanceWithTarget.quarter
);
```

### 💎 VIP 等級晉升追蹤

```sql
-- VIP 會員等級晉升歷程追蹤
WITH VIPLevelHistory AS (
    SELECT 
        member_id,
        purchase_date,
        amount,
        SUM(amount) OVER (
            PARTITION BY member_id
            ORDER BY purchase_date
            ROWS UNBOUNDED PRECEDING
        ) AS 累計金額,
        CASE 
            WHEN SUM(amount) OVER (
                PARTITION BY member_id
                ORDER BY purchase_date
                ROWS UNBOUNDED PRECEDING
            ) >= 100000 THEN 'Diamond'
            WHEN SUM(amount) OVER (
                PARTITION BY member_id
                ORDER BY purchase_date
                ROWS UNBOUNDED PRECEDING
            ) >= 50000 THEN 'Platinum'
            WHEN SUM(amount) OVER (
                PARTITION BY member_id
                ORDER BY purchase_date
                ROWS UNBOUNDED PRECEDING
            ) >= 20000 THEN 'Gold'
            WHEN SUM(amount) OVER (
                PARTITION BY member_id
                ORDER BY purchase_date
                ROWS UNBOUNDED PRECEDING
            ) >= 10000 THEN 'Silver'
            ELSE 'Bronze'
        END AS 當前等級
    FROM purchases
),
LevelChanges AS (
    SELECT 
        *,
        LAG(當前等級) OVER (
            PARTITION BY member_id 
            ORDER BY purchase_date
        ) AS 前一等級,
        CASE 
            WHEN 當前等級 != LAG(當前等級) OVER (
                PARTITION BY member_id 
                ORDER BY purchase_date
            ) THEN 1 
            ELSE 0 
        END AS 等級變更標記
    FROM VIPLevelHistory
)
SELECT 
    member_id,
    purchase_date AS 晉升日期,
    前一等級,
    當前等級,
    累計金額
FROM LevelChanges
WHERE 等級變更標記 = 1
ORDER BY member_id, purchase_date;
```

---

## ⚡ 效能優化技巧

### 🚀 與傳統方法比較

```sql
-- ❌ 傳統自連接方法 (效能較差)
SELECT 
    a.player_id,
    a.event_date,
    SUM(b.games_played) AS 累計時數
FROM Activity a
LEFT JOIN Activity b
    ON a.player_id = b.player_id
    AND a.event_date >= b.event_date
GROUP BY a.player_id, a.event_date
ORDER BY a.player_id, a.event_date;

-- ✅ Window Function 方法 (推薦)
SELECT 
    player_id,
    event_date,
    SUM(games_played) OVER (
        PARTITION BY player_id
        ORDER BY event_date
        ROWS UNBOUNDED PRECEDING
    ) AS 累計時數
FROM Activity
ORDER BY player_id, event_date;
```

### 📊 索引策略

```sql
-- 針對累計計算優化的索引
CREATE INDEX IX_Activity_PlayerDate 
ON Activity (player_id, event_date)
INCLUDE (games_played);

-- 針對大量資料的分區索引
CREATE INDEX IX_Sales_DateAmount 
ON sales (sale_date, amount)
WHERE sale_date >= DATEADD(YEAR, -2, GETDATE());
```