## 📖 目錄
- [📖 目錄](#-目錄)
- [🌟 COUNT 計數概述](#-count-計數概述)
  - [🔄 分組計數分析](#-分組計數分析)
  - [📈 累計計數追蹤](#-累計計數追蹤)
- [👥 會員訂單總數統計](#-會員訂單總數統計)
  - [📊 基礎訂單計數](#-基礎訂單計數)
  - [🎯 深度客戶分析](#-深度客戶分析)
  - [📈 客戶活躍度追蹤](#-客戶活躍度追蹤)
- [💼 商業實戰案例](#-商業實戰案例)
  - [🛍️ 購買行為分析](#️-購買行為分析)
  - [📱 用戶活躍度統計](#-用戶活躍度統計)
  - [🎪 事件觸發次數分析](#-事件觸發次數分析)
- [🎯 進階計數技巧](#-進階計數技巧)
  - [🔄 條件計數](#-條件計數)
  - [📊 滾動計數](#-滾動計數)
  - [🎪 去重計數](#-去重計數)
- [⚡ 特殊應用場景](#-特殊應用場景)
  - [🏃‍♂️ 連續登入統計](#️-連續登入統計)
  - [📈 成長軌跡追蹤](#-成長軌跡追蹤)
  - [🎁 里程碑檢測](#-里程碑檢測)
- [⚡ 效能優化建議](#-效能優化建議)
  - [📊 索引策略](#-索引策略)
  - [🚀 查詢優化](#-查詢優化)
  - [⚠️ 注意事項](#️-注意事項)
---

## 🌟 COUNT 計數概述

```

### 🎨 計數統計類型

| 計數類型 | 語法範例 | 適用場景 |
|----------|----------|----------|
| **🌐 全域計數** | `COUNT(*) OVER()` | 總體規模統計 |
| **🔄 分組計數** | `COUNT(*) OVER(PARTITION BY category)` | 類別規模比較 |
| **📈 累計計數** | `COUNT(*) OVER(ORDER BY date ROWS UNBOUNDED PRECEDING)` | 成長軌跡追蹤 |
| **🎪 滾動計數** | `COUNT(*) OVER(ORDER BY date ROWS 6 PRECEDING)` | 短期活躍度 |

---

## 👥 基礎計數範例

### 📊 全域計數統計

```sql
-- 每筆記錄顯示總體統計資訊
SELECT 
    order_id,
    customer_id,
    order_date,
    amount,
    COUNT(*) OVER() AS 總訂單數,
    COUNT(DISTINCT customer_id) OVER() AS 總客戶數,
    CAST(COUNT(*) OVER(PARTITION BY customer_id) * 100.0 / COUNT(*) OVER() AS DECIMAL(5,2)) AS 該客戶訂單佔比
FROM orders
ORDER BY customer_id, order_date;
```

### 🔄 分組計數分析

```sql
-- 各部門員工數量統計
SELECT 
    employee_name,
    department,
    position,
    salary,
    COUNT(*) OVER(PARTITION BY department) AS 部門總人數,
    COUNT(*) OVER(PARTITION BY department, position) AS 相同職位人數,
    COUNT(*) OVER() AS 全公司總人數,
    CAST(COUNT(*) OVER(PARTITION BY department) * 100.0 / COUNT(*) OVER() AS DECIMAL(5,2)) AS 部門人數佔比
FROM employees
ORDER BY department, position;
```

### 📈 累計計數追蹤

```sql
-- 累計用戶註冊數追蹤
SELECT 
    registration_date,
    user_count_daily,
    COUNT(*) OVER (
        ORDER BY registration_date 
        ROWS UNBOUNDED PRECEDING
    ) AS 累計註冊天數,
    SUM(user_count_daily) OVER (
        ORDER BY registration_date 
        ROWS UNBOUNDED PRECEDING
    ) AS 累計用戶數,
    COUNT(*) OVER (
        ORDER BY registration_date 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS 七天內活躍天數
FROM daily_registrations
ORDER BY registration_date;
```

---

## 👥 會員訂單總數統計

### 📊 基礎訂單計數

```sql
USE WebStoreDB;

-- 會員訂單統計基礎分析
SELECT 
    CrmSalesOrder_CrmMemberId,
    CrmSalesOrder_Id,
    CrmSalesOrder_TotalAmount,
    CrmSalesOrder_DateTime,
    COUNT(*) OVER (PARTITION BY CrmSalesOrder_CrmMemberId) AS 會員總訂單數,
    COUNT(*) OVER() AS 平台總訂單數,
    ROW_NUMBER() OVER (
        PARTITION BY CrmSalesOrder_CrmMemberId 
        ORDER BY CrmSalesOrder_DateTime
    ) AS 會員訂單序號,
    CAST(
        COUNT(*) OVER (PARTITION BY CrmSalesOrder_CrmMemberId) * 100.0 / 
        COUNT(*) OVER() 
        AS DECIMAL(5,2)
    ) AS 訂單佔比
FROM CrmSalesOrder(NOLOCK)
WHERE CrmSalesOrder_ValidFlag = 1
    AND CrmSalesOrder_DateTime >= DATEADD(YEAR, -1, GETDATE())
ORDER BY 會員總訂單數 DESC, CrmSalesOrder_CrmMemberId, CrmSalesOrder_DateTime;
```

### 🎯 深度客戶分析

```sql
USE WebStoreDB;

-- 深度客戶行為計數分析
WITH CustomerOrderAnalysis AS (
    SELECT 
        TradesOrderGroup_MemberId,
        TradesOrderGroup_DateTime,
        TradesOrderGroup_TotalPayment,
        COUNT(*) OVER (PARTITION BY TradesOrderGroup_MemberId) AS 會員總訂單數,
        COUNT(*) OVER (
            PARTITION BY TradesOrderGroup_MemberId, YEAR(TradesOrderGroup_DateTime)
        ) AS 年度訂單數,
        COUNT(*) OVER (
            PARTITION BY TradesOrderGroup_MemberId, YEAR(TradesOrderGroup_DateTime), MONTH(TradesOrderGroup_DateTime)
        ) AS 月度訂單數,
        COUNT(*) OVER() AS 平台總訂單數,
        COUNT(DISTINCT TradesOrderGroup_MemberId) OVER() AS 平台總會員數
    FROM TradesOrderGroup(NOLOCK)
    WHERE TradesOrderGroup_ValidFlag = 1
        AND TradesOrderGroup_DateTime >= DATEADD(YEAR, -2, GETDATE())
),
CustomerSegmentation AS (
    SELECT 
        TradesOrderGroup_MemberId,
        會員總訂單數,
        MAX(年度訂單數) AS 年度最高訂單數,
        AVG(CAST(月度訂單數 AS FLOAT)) AS 平均月度訂單數,
        平台總訂單數,
        平台總會員數,
        CASE 
            WHEN 會員總訂單數 >= 50 THEN '💎 超級VIP (50+單)'
            WHEN 會員總訂單數 >= 20 THEN '👑 鑽石VIP (20-49單)'
            WHEN 會員總訂單數 >= 10 THEN '🥇 金牌會員 (10-19單)'
            WHEN 會員總訂單數 >= 5 THEN '🥈 銀牌會員 (5-9單)'
            WHEN 會員總訂單數 >= 3 THEN '🥉 銅牌會員 (3-4單)'
            ELSE '👤 一般會員 (1-2單)'
        END AS 會員等級
    FROM CustomerOrderAnalysis
    GROUP BY TradesOrderGroup_MemberId, 會員總訂單數, 平台總訂單數, 平台總會員數
)
SELECT 
    會員等級,
    COUNT(*) AS 該等級會員數,
    AVG(會員總訂單數) AS 平均訂單數,
    MIN(會員總訂單數) AS 最少訂單數,
    MAX(會員總訂單數) AS 最多訂單數,
    SUM(會員總訂單數) AS 該等級總訂單數,
    CAST(COUNT(*) * 100.0 / MAX(平台總會員數) AS DECIMAL(5,2)) AS 會員數佔比,
    CAST(SUM(會員總訂單數) * 100.0 / MAX(平台總訂單數) AS DECIMAL(5,2)) AS 訂單數佔比
FROM CustomerSegmentation
GROUP BY 會員等級
ORDER BY 平均訂單數 DESC;
```

### 📈 客戶活躍度追蹤

```sql
USE WebStoreDB;

-- 客戶活躍度週期性統計
WITH CustomerActivityTracking AS (
    SELECT 
        TradesOrderGroup_MemberId,
        TradesOrderGroup_DateTime,
        YEAR(TradesOrderGroup_DateTime) AS 訂單年份,
        MONTH(TradesOrderGroup_DateTime) AS 訂單月份,
        DATEPART(QUARTER, TradesOrderGroup_DateTime) AS 訂單季度,
        DATEPART(WEEK, TradesOrderGroup_DateTime) AS 訂單週數,
        COUNT(*) OVER (
            PARTITION BY TradesOrderGroup_MemberId, YEAR(TradesOrderGroup_DateTime)
        ) AS 年度總訂單,
        COUNT(*) OVER (
            PARTITION BY TradesOrderGroup_MemberId, DATEPART(QUARTER, TradesOrderGroup_DateTime)
        ) AS 季度總訂單,
        COUNT(*) OVER (
            PARTITION BY TradesOrderGroup_MemberId, YEAR(TradesOrderGroup_DateTime), MONTH(TradesOrderGroup_DateTime)
        ) AS 月度總訂單,
        COUNT(*) OVER (
            PARTITION BY TradesOrderGroup_MemberId, DATEPART(WEEK, TradesOrderGroup_DateTime)
        ) AS 週度總訂單
    FROM TradesOrderGroup(NOLOCK)
    WHERE TradesOrderGroup_ValidFlag = 1
        AND TradesOrderGroup_DateTime >= DATEADD(MONTH, -6, GETDATE())
),
ActivityInsights AS (
    SELECT 
        TradesOrderGroup_MemberId,
        COUNT(DISTINCT 訂單年份) AS 活躍年數,
        COUNT(DISTINCT 訂單季度) AS 活躍季數,
        COUNT(DISTINCT 訂單月份) AS 活躍月數,
        COUNT(DISTINCT 訂單週數) AS 活躍週數,
        MAX(年度總訂單) AS 年度最高訂單數,
        MAX(季度總訂單) AS 季度最高訂單數,
        MAX(月度總訂單) AS 月度最高訂單數,
        MAX(週度總訂單) AS 週度最高訂單數,
        AVG(CAST(月度總訂單 AS FLOAT)) AS 平均月度訂單數
    FROM CustomerActivityTracking
    GROUP BY TradesOrderGroup_MemberId
)
SELECT 
    TradesOrderGroup_MemberId,
    活躍月數,
    年度最高訂單數,
    月度最高訂單數,
    平均月度訂單數,
    CASE 
        WHEN 活躍月數 >= 6 AND 平均月度訂單數 >= 5 THEN '🔥 高頻忠實客戶'
        WHEN 活躍月數 >= 4 AND 平均月度訂單數 >= 3 THEN '⭐ 穩定活躍客戶'
        WHEN 活躍月數 >= 3 AND 平均月度訂單數 >= 2 THEN '📈 成長中客戶'
        WHEN 活躍月數 >= 2 THEN '📊 偶爾活躍客戶'
        ELSE '📉 低活躍客戶'
    END AS 活躍度等級,
    CASE 
        WHEN 月度最高訂單數 >= 10 THEN '💥 爆發型'
        WHEN 平均月度訂單數 >= 5 THEN '🚀 高頻型'
        WHEN 平均月度訂單數 >= 2 THEN '📊 穩定型'
        ELSE '🐌 低頻型'
    END AS 消費模式
FROM ActivityInsights
ORDER BY 平均月度訂單數 DESC, 活躍月數 DESC;
```

---

## 💼 商業實戰案例

### 🛍️ 購買行為分析

```sql
-- 購買行為深度計數分析
WITH PurchaseBehaviorAnalysis AS (
    SELECT 
        customer_id,
        product_category,
        purchase_date,
        amount,
        COUNT(*) OVER (PARTITION BY customer_id) AS 客戶總購買次數,
        COUNT(*) OVER (PARTITION BY customer_id, product_category) AS 客戶類別購買次數,
        COUNT(DISTINCT product_category) OVER (PARTITION BY customer_id) AS 客戶購買類別數,
        COUNT(*) OVER (PARTITION BY product_category) AS 類別總銷售次數,
        COUNT(DISTINCT customer_id) OVER (PARTITION BY product_category) AS 類別客戶數,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY purchase_date) AS 客戶購買序號,
        ROW_NUMBER() OVER (PARTITION BY customer_id, product_category ORDER BY purchase_date) AS 類別內購買序號
    FROM purchases
    WHERE purchase_date >= DATEADD(YEAR, -1, GETDATE())
),
CustomerProfile AS (
    SELECT 
        customer_id,
        客戶總購買次數,
        客戶購買類別數,
        COUNT(DISTINCT product_category) AS 涉及類別數,
        AVG(CAST(客戶類別購買次數 AS FLOAT)) AS 平均類別購買次數,
        MAX(客戶類別購買次數) AS 最愛類別購買次數,
        CASE 
            WHEN 客戶購買類別數 >= 5 AND 客戶總購買次數 >= 20 THEN '🌟 全品類達人'
            WHEN 客戶購買類別數 >= 3 AND 客戶總購買次數 >= 15 THEN '🎯 多元消費者'
            WHEN 最愛類別購買次數 >= 10 THEN '❤️ 專一愛好者'
            WHEN 客戶總購買次數 >= 10 THEN '🔄 頻繁購買者'
            ELSE '👤 一般消費者'
        END AS 消費者類型
    FROM PurchaseBehaviorAnalysis
    GROUP BY customer_id, 客戶總購買次數, 客戶購買類別數
)
SELECT 
    消費者類型,
    COUNT(*) AS 客戶數量,
    AVG(客戶總購買次數) AS 平均購買次數,
    AVG(客戶購買類別數) AS 平均涉及類別數,
    AVG(平均類別購買次數) AS 平均單類別購買次數
FROM CustomerProfile
GROUP BY 消費者類型
ORDER BY 平均購買次數 DESC;
```

### 📱 用戶活躍度統計

```sql
-- 用戶多維度活躍度統計
WITH UserActivityMetrics AS (
    SELECT 
        user_id,
        activity_date,
        activity_type,
        COUNT(*) OVER (PARTITION BY user_id) AS 總活動次數,
        COUNT(*) OVER (PARTITION BY user_id, activity_type) AS 該類型活動次數,
        COUNT(DISTINCT activity_type) OVER (PARTITION BY user_id) AS 活動類型多樣性,
        COUNT(*) OVER (
            PARTITION BY user_id 
            ORDER BY activity_date 
            ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
        ) AS 七天內活動次數,
        COUNT(DISTINCT activity_date) OVER (PARTITION BY user_id) AS 活躍天數,
        DATEDIFF(DAY, MIN(activity_date) OVER (PARTITION BY user_id), MAX(activity_date) OVER (PARTITION BY user_id)) AS 活躍跨度天數
    FROM user_activities
    WHERE activity_date >= DATEADD(MONTH, -3, GETDATE())
),
ActivitySegmentation AS (
    SELECT 
        user_id,
        總活動次數,
        活動類型多樣性,
        活躍天數,
        活躍跨度天數,
        CASE 
            WHEN 活躍跨度天數 > 0 THEN CAST(活躍天數 AS FLOAT) / 活躍跨度天數 
            ELSE 1 
        END AS 活躍密度,
        CASE 
            WHEN 總活動次數 >= 100 AND 活動類型多樣性 >= 5 THEN '🔥 超級活躍用戶'
            WHEN 總活動次數 >= 50 AND 活動類型多樣性 >= 3 THEN '⭐ 高活躍用戶'
            WHEN 總活動次數 >= 20 THEN '📈 中活躍用戶'
            WHEN 總活動次數 >= 10 THEN '📊 低活躍用戶'
            ELSE '😴 沉睡用戶'
        END AS 活躍等級
    FROM UserActivityMetrics
    GROUP BY user_id, 總活動次數, 活動類型多樣性, 活躍天數, 活躍跨度天數
)
SELECT 
    活躍等級,
    COUNT(*) AS 用戶數量,
    AVG(總活動次數) AS 平均活動次數,
    AVG(活動類型多樣性) AS 平均活動類型數,
    AVG(活躍天數) AS 平均活躍天數,
    AVG(活躍密度) AS 平均活躍密度
FROM ActivitySegmentation
GROUP BY 活躍等級
ORDER BY 平均活動次數 DESC;
```

### 🎪 事件觸發次數分析

```sql
-- 系統事件觸發統計與分析
WITH EventTriggerAnalysis AS (
    SELECT 
        event_type,
        user_id,
        trigger_datetime,
        COUNT(*) OVER (PARTITION BY event_type) AS 事件總觸發次數,
        COUNT(*) OVER (PARTITION BY user_id, event_type) AS 用戶該事件觸發次數,
        COUNT(DISTINCT user_id) OVER (PARTITION BY event_type) AS 事件影響用戶數,
        COUNT(*) OVER (
            PARTITION BY event_type 
            ORDER BY trigger_datetime 
            ROWS BETWEEN INTERVAL '1' HOUR PRECEDING AND CURRENT ROW
        ) AS 一小時內該事件次數,
        COUNT(*) OVER (
            PARTITION BY user_id, event_type 
            ORDER BY trigger_datetime 
            ROWS BETWEEN INTERVAL '1' DAY PRECEDING AND CURRENT ROW
        ) AS 用戶24小時內該事件次數
    FROM system_events
    WHERE trigger_datetime >= DATEADD(DAY, -7, GETDATE())
),
EventInsights AS (
    SELECT 
        event_type,
        事件總觸發次數,
        事件影響用戶數,
        事件總觸發次數 / 事件影響用戶數 AS 平均每用戶觸發次數,
        MAX(用戶該事件觸發次數) AS 單用戶最高觸發次數,
        MAX(一小時內該事件次數) AS 小時內最高頻率,
        CASE 
            WHEN MAX(一小時內該事件次數) > 100 THEN '🚨 高頻事件'
            WHEN 事件總觸發次數 / 事件影響用戶數 > 10 THEN '⚡ 活躍事件'
            WHEN 事件影響用戶數 > 1000 THEN '📈 普及事件'
            ELSE '📊 一般事件'
        END AS 事件等級
    FROM EventTriggerAnalysis
    GROUP BY event_type, 事件總觸發次數, 事件影響用戶數
)
SELECT 
    event_type,
    事件等級,
    事件總觸發次數,
    事件影響用戶數,
    平均每用戶觸發次數,
    單用戶最高觸發次數,
    小時內最高頻率
FROM EventInsights
ORDER BY 事件總觸發次數 DESC;
```

---

## 🎯 進階計數技巧

### 🔄 條件計數

```sql
-- 條件性計數統計
SELECT 
    customer_id,
    order_date,
    amount,
    status,
    -- 條件計數
    COUNT(CASE WHEN amount > 1000 THEN 1 END) OVER (
        PARTITION BY customer_id
    ) AS 高額訂單次數,
    COUNT(CASE WHEN status = 'completed' THEN 1 END) OVER (
        PARTITION BY customer_id
    ) AS 完成訂單次數,
    COUNT(CASE WHEN DATEPART(WEEKDAY, order_date) IN (1, 7) THEN 1 END) OVER (
        PARTITION BY customer_id
    ) AS 假日下單次數,
    -- 計算比例
    CAST(
        COUNT(CASE WHEN amount > 1000 THEN 1 END) OVER (PARTITION BY customer_id) * 100.0 /
        COUNT(*) OVER (PARTITION BY customer_id)
        AS DECIMAL(5,2)
    ) AS 高額訂單比例
FROM orders
ORDER BY customer_id, order_date;
```

### 📊 滾動計數

```sql
-- 滾動視窗計數分析
SELECT 
    user_id,
    login_date,
    COUNT(*) OVER (
        PARTITION BY user_id
        ORDER BY login_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS 七天內登入次數,
    COUNT(*) OVER (
        PARTITION BY user_id
        ORDER BY login_date
        ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
    ) AS 三十天內登入次數,
    CASE 
        WHEN COUNT(*) OVER (
            PARTITION BY user_id
            ORDER BY login_date
            ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
        ) >= 5 THEN '🔥 活躍用戶'
        WHEN COUNT(*) OVER (
            PARTITION BY user_id
            ORDER BY login_date
            ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
        ) >= 10 THEN '📈 穩定用戶'
        ELSE '📉 不活躍用戶'
    END AS 活躍等級
FROM user_logins
ORDER BY user_id, login_date;
```

### 🎪 去重計數

```sql
-- 去重計數統計 (使用 COUNT(DISTINCT))
WITH DistinctCountAnalysis AS (
    SELECT 
        department,
        employee_id,
        project_id,
        work_date,
        COUNT(DISTINCT project_id) OVER (PARTITION BY employee_id) AS 員工參與專案數,
        COUNT(DISTINCT employee_id) OVER (PARTITION BY project_id) AS 專案參與人數,
        COUNT(DISTINCT employee_id) OVER (PARTITION BY department) AS 部門總人數,
        COUNT(DISTINCT project_id) OVER (PARTITION BY department) AS 部門專案數
    FROM employee_projects
)
SELECT 
    department,
    employee_id,
    員工參與專案數,
    部門總人數,
    部門專案數,
    CAST(員工參與專案數 * 100.0 / 部門專案數 AS DECIMAL(5,2)) AS 專案參與率
FROM DistinctCountAnalysis
GROUP BY department, employee_id, 員工參與專案數, 部門總人數, 部門專案數
ORDER BY department, 專案參與率 DESC;
```

---

## ⚡ 特殊應用場景

### 🏃‍♂️ 連續登入統計

```sql
-- 連續登入天數統計
WITH LoginStreaks AS (
    SELECT 
        user_id,
        login_date,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) AS login_sequence,
        DATEADD(DAY, -ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date), login_date) AS streak_group
    FROM user_daily_logins
),
StreakCounts AS (
    SELECT 
        user_id,
        streak_group,
        COUNT(*) AS 連續登入天數,
        MIN(login_date) AS 連續開始日,
        MAX(login_date) AS 連續結束日
    FROM LoginStreaks
    GROUP BY user_id, streak_group
),
UserStreakAnalysis AS (
    SELECT 
        user_id,
        MAX(連續登入天數) AS 最長連續登入天數,
        COUNT(*) AS 連續登入段數,
        CASE 
            WHEN MAX(連續結束日) = CAST(GETDATE() AS DATE) AND MAX(連續登入天數) >= 7 THEN '🔥 當前7天+'
            WHEN MAX(連續結束日) = CAST(GETDATE() AS DATE) AND MAX(連續登入天數) >= 3 THEN '⭐ 當前3天+'
            WHEN MAX(連續登入天數) >= 30 THEN '👑 歷史30天+'
            WHEN MAX(連續登入天數) >= 7 THEN '🏆 歷史7天+'
            ELSE '📊 一般'
        END AS 連續登入等級
    FROM StreakCounts
    GROUP BY user_id
)
SELECT 
    連續登入等級,
    COUNT(*) AS 用戶數量,
    AVG(最長連續登入天數) AS 平均最長連續天數,
    MAX(最長連續登入天數) AS 紀錄保持天數
FROM UserStreakAnalysis
GROUP BY 連續登入等級
ORDER BY 平均最長連續天數 DESC;
```

### 📈 成長軌跡追蹤

```sql
-- 用戶成長軌跡計數追蹤
WITH UserGrowthTracking AS (
    SELECT 
        user_id,
        achievement_date,
        achievement_type,
        COUNT(*) OVER (
            PARTITION BY user_id 
            ORDER BY achievement_date
            ROWS UNBOUNDED PRECEDING
        ) AS 累計成就數,
        COUNT(DISTINCT achievement_type) OVER (
            PARTITION BY user_id 
            ORDER BY achievement_date
            ROWS UNBOUNDED PRECEDING
        ) AS 累計成就類型數,
        DATEDIFF(DAY, MIN(achievement_date) OVER (PARTITION BY user_id), achievement_date) AS 成長天數
    FROM user_achievements
    WHERE achievement_date >= DATEADD(YEAR, -1, GETDATE())
),
GrowthMilestones AS (
    SELECT 
        user_id,
        achievement_date,
        累計成就數,
        累計成就類型數,
        成長天數,
        CASE 
            WHEN 累計成就數 = 1 THEN '🌟 首次成就'
            WHEN 累計成就數 = 10 THEN '🏆 十項達成'
            WHEN 累計成就數 = 50 THEN '👑 五十大關'
            WHEN 累計成就數 = 100 THEN '💎 百項傳說'
            WHEN 累計成就類型數 = 5 THEN '🎯 多元發展'
            WHEN 累計成就類型數 = 10 THEN '🌈 全面開花'
            ELSE NULL
        END AS 里程碑標記
    FROM UserGrowthTracking
)
SELECT 
    里程碑標記,
    COUNT(*) AS 達成用戶數,
    AVG(成長天數) AS 平均達成天數,
    MIN(成長天數) AS 最快達成天數,
    MAX(成長天數) AS 最慢達成天數
FROM GrowthMilestones
WHERE 里程碑標記 IS NOT NULL
GROUP BY 里程碑標記
ORDER BY 平均達成天數;
```

### 🎁 里程碑檢測

```sql
-- 智能里程碑檢測系統
WITH MilestoneDetection AS (
    SELECT 
        customer_id,
        purchase_date,
        amount,
        COUNT(*) OVER (
            PARTITION BY customer_id 
            ORDER BY purchase_date
            ROWS UNBOUNDED PRECEDING
        ) AS 累計購買次數,
        SUM(amount) OVER (
            PARTITION BY customer_id 
            ORDER BY purchase_date
            ROWS UNBOUNDED PRECEDING
        ) AS 累計消費金額,
        -- 檢測里程碑突破
        CASE 
            WHEN COUNT(*) OVER (
                PARTITION BY customer_id 
                ORDER BY purchase_date
                ROWS UNBOUNDED PRECEDING
            ) IN (1, 5, 10, 25, 50, 100) THEN '📊 購買次數里程碑'
            WHEN SUM(amount) OVER (
                PARTITION BY customer_id 
                ORDER BY purchase_date
                ROWS UNBOUNDED PRECEDING
            ) >= 10000 
            AND LAG(SUM(amount) OVER (
                PARTITION BY customer_id 
                ORDER BY purchase_date
                ROWS UNBOUNDED PRECEDING
            )) OVER (PARTITION BY customer_id ORDER BY purchase_date) < 10000 
            THEN '💰 萬元俱樂部'
            WHEN SUM(amount) OVER (
                PARTITION BY customer_id 
                ORDER BY purchase_date
                ROWS UNBOUNDED PRECEDING
            ) >= 50000 
            AND LAG(SUM(amount) OVER (
                PARTITION BY customer_id 
                ORDER BY purchase_date
                ROWS UNBOUNDED PRECEDING
            )) OVER (PARTITION BY customer_id ORDER BY purchase_date) < 50000 
            THEN '👑 五萬VIP'
            ELSE NULL
        END AS 里程碑類型
    FROM purchases
),
MilestoneRewards AS (
    SELECT 
        customer_id,
        purchase_date,
        累計購買次數,
        累計消費金額,
        里程碑類型,
        CASE 
            WHEN 里程碑類型 = '📊 購買次數里程碑' AND 累計購買次數 = 100 THEN '🎁 百單獎勵: 1000元購物金'
            WHEN 里程碑類型 = '📊 購買次數里程碑' AND 累計購買次數 = 50 THEN '🎁 五十單獎勵: 500元購物金'
            WHEN 里程碑類型 = '📊 購買次數里程碑' AND 累計購買次數 = 25 THEN '🎁 二十五單獎勵: 250元購物金'
            WHEN 里程碑類型 = '💰 萬元俱樂部' THEN '🎁 萬元獎勵: VIP資格 + 200元購物金'
            WHEN 里程碑類型 = '👑 五萬VIP' THEN '🎁 VIP獎勵: 專屬客服 + 1000元購物金'
            ELSE NULL
        END AS 獎勵內容
    FROM MilestoneDetection
    WHERE 里程碑類型 IS NOT NULL
)
SELECT 
    里程碑類型,
    COUNT(*) AS 觸發次數,
    COUNT(DISTINCT customer_id) AS 獲得獎勵客戶數,
    MAX(累計購買次數) AS 最高購買次數,
    MAX(累計消費金額) AS 最高消費金額
FROM MilestoneRewards
GROUP BY 里程碑類型
ORDER BY 觸發次數 DESC;
```

---

## ⚡ 效能優化建議

### 📊 索引策略

```sql
-- 針對計數查詢的索引優化
CREATE INDEX IX_Orders_CustomerDate 
ON orders (customer_id, order_date)
INCLUDE (amount, status);

-- 針對時間範圍查詢的索引
CREATE INDEX IX_UserLogins_UserDate 
ON user_logins (user_id, login_date)
WHERE login_date >= DATEADD(MONTH, -6, GETDATE());
```

### 🚀 查詢優化

```sql
-- 使用 CTE 避免重複計算
WITH CustomerStats AS (
    SELECT 
        customer_id,
        COUNT(*) AS total_orders,
        COUNT(DISTINCT YEAR(order_date)) AS active_years
    FROM orders
    WHERE order_date >= DATEADD(YEAR, -2, GETDATE())
    GROUP BY customer_id
)
SELECT 
    o.customer_id,
    o.order_date,
    o.amount,
    cs.total_orders,
    cs.active_years
FROM orders o
JOIN CustomerStats cs ON o.customer_id = cs.customer_id
ORDER BY cs.total_orders DESC;
```

### ⚠️ 注意事項

1. **COUNT(*) vs COUNT(column)**
   ```sql
   COUNT(*) -- 計算所有行（包括 NULL）
   COUNT(column) -- 計算非 NULL 值
   COUNT(DISTINCT column) -- 計算去重後的非 NULL 值
   ```

2. **效能考量**
   ```sql
   -- 限制計算範圍避免全表掃描
   COUNT(*) OVER(PARTITION BY customer_id) -- 而非 COUNT(*) OVER()
   ```

---