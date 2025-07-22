# SQL SELECT INTO vs INSERT INTO  📊

> 從資料複製到效能比較，掌握 SQL 資料操作的精髓！

## 目錄
1. [SELECT INTO 基礎篇](#select-into-基礎篇)
   - 1.1 [什麼是 SELECT INTO？](#什麼是-select-into)
   - 1.2 [基本語法與應用](#基本語法與應用)
   - 1.3 [快速備份與複製](#快速備份與複製)

2. [INSERT INTO 基礎篇](#insert-into-基礎篇)
   - 2.1 [基本 INSERT 語法](#基本-insert-語法)
   - 2.2 [資料插入方式](#資料插入方式)
   - 2.3 [實務應用場景](#實務應用場景)

3. [效能比較實驗](#效能比較實驗)
   - 3.1 [INSERT INTO 效能測試](#insert-into-效能測試)
   - 3.2 [SELECT INTO 效能測試](#select-into-效能測試)
   - 3.3 [兩者效能對比分析](#兩者效能對比分析)

4. [進階技巧篇](#進階技巧篇)
   - 4.1 [大量資料處理策略](#大量資料處理策略)
   - 4.2 [效能優化技巧](#效能優化技巧)
   - 4.3 [最佳實踐指南](#最佳實踐指南)

5. [實務應用場景](#實務應用場景)
   - 5.1 [資料備份策略](#資料備份策略)
   - 5.2 [資料遷移方案](#資料遷移方案)
   - 5.3 [測試環境建立](#測試環境建立)

---

## SELECT INTO 基礎篇

### 什麼是 SELECT INTO？

> 🚀 **核心概念**：SELECT INTO 可直接從查詢結果建立一個新的資料表，不需事先用 CREATE TABLE 定義欄位！

**主要特色**
- ✅ **自動建表**：自動根據查詢結果建立表格結構
- ✅ **快速複製**：可以用來快速複製一張表的結構與資料做為備份
- ✅ **簡化操作**：一個語句完成建表和資料插入
- ⚡ **高效能**：比逐筆 INSERT 快很多

### 基本語法與應用

**基本語法結構**
```sql
SELECT column1, column2, ...
INTO new_table_name
FROM source_table
WHERE conditions;
```

**實際範例**
```sql
-- 基本 SELECT INTO 範例
SELECT *
INTO ABC
FROM RD
WHERE DeptCode = 'R1003';

-- 檢查建立的新表
SELECT * FROM ABC;
```

**帶條件的複製**
```sql
-- 複製特定條件的資料
SELECT EmployeeID, FirstName, LastName, Department
INTO ActiveEmployees
FROM Employees
WHERE Status = 'Active' 
  AND HireDate >= '2020-01-01';

-- 複製表結構（不含資料）
SELECT *
INTO EmptyEmployeeBackup
FROM Employees
WHERE 1 = 0;  -- 永遠為 false 的條件
```

### 快速備份與複製

**完整表格備份**
```sql
-- 完整備份（結構 + 資料）
SELECT *
INTO Employees_Backup_20250715
FROM Employees;

-- 檢查備份結果
SELECT COUNT(*) AS BackupRecordCount FROM Employees_Backup_20250715;
SELECT COUNT(*) AS OriginalRecordCount FROM Employees;
```

**部分欄位備份**
```sql
-- 只備份重要欄位
SELECT EmployeeID, 
       FirstName, 
       LastName, 
       Email, 
       HireDate,
       GETDATE() AS BackupDate
INTO Employees_Essential_Backup
FROM Employees
WHERE Status = 'Active';
```

**跨資料庫複製**
```sql
-- 從其他資料庫複製資料
SELECT *
INTO LocalDB.dbo.ProductBackup
FROM RemoteDB.dbo.Products
WHERE CategoryID IN (1, 2, 3);
```

---

## INSERT INTO 基礎篇

### 基本 INSERT 語法

> 📝 **傳統方式**：INSERT INTO 需要先建立表格，再插入資料

**基本語法**
```sql
-- 插入完整資料
INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...);

-- 插入多筆資料
INSERT INTO table_name (column1, column2, ...)
VALUES (value1a, value2a, ...),
       (value1b, value2b, ...),
       (value1c, value2c, ...);
```

### 資料插入方式

**簡單資料插入範例**
```sql
-- 建立測試表格
DROP TABLE IF EXISTS TESTwow;
CREATE TABLE TESTwow(
    ID INT IDENTITY(1,1),
    Test NVARCHAR(30)
);

-- 插入資料
INSERT INTO TESTwow
VALUES(N'ＡＢＣ１２３');

-- 檢查結果
SELECT * FROM TESTwow;

-- 清理
DROP TABLE IF EXISTS TESTwow;
```

**多種插入方式比較**
```sql
-- 方式一：指定欄位
INSERT INTO TESTwow (Test)
VALUES (N'測試資料1');

-- 方式二：全部欄位（按順序）
INSERT INTO TESTwow
VALUES (N'測試資料2');

-- 方式三：多筆資料
INSERT INTO TESTwow (Test)
VALUES (N'資料1'),
       (N'資料2'),
       (N'資料3');

-- 方式四：從其他表插入
INSERT INTO TESTwow (Test)
SELECT ProductName 
FROM Products 
WHERE CategoryID = 1;
```

### 實務應用場景

**批次資料處理**
```sql
-- 建立日誌表
CREATE TABLE UserActionLog(
    LogID INT IDENTITY(1,1) PRIMARY KEY,
    UserID INT,
    Action NVARCHAR(50),
    ActionTime DATETIME DEFAULT GETDATE(),
    Remark NVARCHAR(100)
);

-- 批次插入日誌
INSERT INTO UserActionLog (UserID, Action, Remark)
VALUES (1001, 'Login', '使用者登入'),
       (1002, 'Purchase', '購買商品'),
       (1003, 'Logout', '使用者登出'),
       (1001, 'ViewProduct', '瀏覽商品頁面');

SELECT * FROM UserActionLog;
```

---

## 效能比較實驗

### INSERT INTO 效能測試

> ⏱️ **效能實驗**：測試 INSERT INTO 處理大量資料的效能

```sql
USE AdventureWorks2022;
SET STATISTICS TIME ON;

-- 建立測試表格
CREATE TABLE #TEST1(
    id INT NOT NULL,
    userName NVARCHAR(20),
    remark NVARCHAR(50)
);

DECLARE @id INT = 0,
        @userName NVARCHAR(20) = '',
        @remark NVARCHAR(50) = '',
        @i INT = 0;

-- 迴圈插入 10 萬筆資料
WHILE @i < 100000
BEGIN
    SET @id = @i;
    IF @i % 2 = 0
        BEGIN
            SET @userName = N'偶';
            SET @remark = N'燈燈燈楞燈';
        END
    ELSE
        BEGIN
            SET @userName = N'姬';
            SET @remark = N'低哩蹦達';
        END
    
    INSERT INTO #TEST1(id, userName, remark)
    VALUES(@id, @userName, @remark);
    
    SET @i = @i + 1;
END

SELECT COUNT(*) AS RecordCount FROM #TEST1;
DROP TABLE #TEST1;
```

### SELECT INTO 效能測試

**完整效能測試腳本**
```sql
USE AdventureWorks2022;

-- 關閉統計資訊以提高效能
SET STATISTICS TIME OFF;
SET NOCOUNT ON;

-- 計時變數
DECLARE @startTime DATETIME2 = SYSDATETIME();

-- 第一階段：INSERT INTO 建立資料
CREATE TABLE #TEST1(
    id INT NOT NULL,
    userName NVARCHAR(20),
    remark NVARCHAR(50)
);

DECLARE @id INT = 0,
        @userName NVARCHAR(20) = '',
        @remark NVARCHAR(50) = '',
        @i INT = 0;

WHILE @i < 100000
BEGIN
    SET @id = @i;
    IF @i % 2 = 0
        BEGIN
            SET @userName = N'偶';
            SET @remark = N'燈燈燈楞燈';
        END
    ELSE
        BEGIN
            SET @userName = N'姬';
            SET @remark = N'低哩蹦達';
        END
    
    INSERT INTO #TEST1(id, userName, remark)
    VALUES(@id, @userName, @remark);
    
    SET @i = @i + 1;
END

-- 第二階段：SELECT INTO 複製資料
DECLARE @copyStartTime DATETIME2 = SYSDATETIME();

SELECT *
INTO #TEST2
FROM #TEST1;

DECLARE @copyEndTime DATETIME2 = SYSDATETIME();

-- 計算並顯示時間
DECLARE @endTime DATETIME2 = SYSDATETIME();
DECLARE @totalTime INT = DATEDIFF(MILLISECOND, @startTime, @endTime);
DECLARE @copyTime INT = DATEDIFF(MILLISECOND, @copyStartTime, @copyEndTime);

PRINT N'總執行時間(ms): ' + CAST(@totalTime AS VARCHAR);
PRINT N'SELECT INTO 複製時間(ms): ' + CAST(@copyTime AS VARCHAR);

-- 驗證資料
SELECT COUNT(*) AS OriginalCount FROM #TEST1;
SELECT COUNT(*) AS CopiedCount FROM #TEST2;

-- 清理
DROP TABLE #TEST1, #TEST2;
```

### 兩者效能對比分析

**詳細效能比較**
```sql
-- 效能比較測試腳本
USE AdventureWorks2022;
SET NOCOUNT ON;

-- 測試資料準備
CREATE TABLE #SourceData(
    ID INT,
    Name NVARCHAR(50),
    Value DECIMAL(10,2),
    CreateDate DATETIME
);

-- 插入測試資料
INSERT INTO #SourceData
SELECT TOP 50000
       ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS ID,
       'TestData' + CAST(ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS VARCHAR) AS Name,
       RAND() * 1000 AS Value,
       GETDATE() AS CreateDate
FROM sys.objects a
CROSS JOIN sys.objects b;

-- 測試 1: INSERT INTO 效能
DECLARE @insertStart DATETIME2 = SYSDATETIME();

CREATE TABLE #InsertTest(
    ID INT,
    Name NVARCHAR(50),
    Value DECIMAL(10,2),
    CreateDate DATETIME
);

INSERT INTO #InsertTest
SELECT * FROM #SourceData;

DECLARE @insertEnd DATETIME2 = SYSDATETIME();
DECLARE @insertTime INT = DATEDIFF(MILLISECOND, @insertStart, @insertEnd);

-- 測試 2: SELECT INTO 效能
DECLARE @selectStart DATETIME2 = SYSDATETIME();

SELECT *
INTO #SelectTest
FROM #SourceData;

DECLARE @selectEnd DATETIME2 = SYSDATETIME();
DECLARE @selectTime INT = DATEDIFF(MILLISECOND, @selectStart, @selectEnd);

-- 結果比較
PRINT N'=== 效能比較結果 ===';
PRINT N'INSERT INTO 時間: ' + CAST(@insertTime AS VARCHAR) + N' ms';
PRINT N'SELECT INTO 時間: ' + CAST(@selectTime AS VARCHAR) + N' ms';
PRINT N'效能提升: ' + CAST((@insertTime - @selectTime) * 100.0 / @insertTime AS VARCHAR(10)) + N'%';

-- 清理
DROP TABLE #SourceData, #InsertTest, #SelectTest;
```

**效能差異分析表**

| 方面 | INSERT INTO | SELECT INTO |
|------|-------------|-------------|
| **建表需求** | 需要先建表 | 自動建表 |
| **語法複雜度** | 較複雜 | 簡單 |
| **大量資料效能** | 較慢 | 較快 |
| **記憶體使用** | 較多 | 較少 |
| **適用場景** | 已存在的表 | 新建表 |

---

## 進階技巧篇

### 大量資料處理策略

**分批處理大量資料**
```sql
-- 大量資料分批 SELECT INTO
DECLARE @BatchSize INT = 10000;
DECLARE @CurrentBatch INT = 1;
DECLARE @TotalRows INT;

-- 取得總筆數
SELECT @TotalRows = COUNT(*) FROM LargeSourceTable;

WHILE (@CurrentBatch - 1) * @BatchSize < @TotalRows
BEGIN
    DECLARE @StartRow INT = (@CurrentBatch - 1) * @BatchSize + 1;
    DECLARE @EndRow INT = @CurrentBatch * @BatchSize;
    
    -- 動態建立表名
    DECLARE @TableName NVARCHAR(50) = N'BatchData_' + CAST(@CurrentBatch AS VARCHAR);
    DECLARE @SQL NVARCHAR(MAX);
    
    SET @SQL = N'
    SELECT *
    INTO ' + @TableName + N'
    FROM (
        SELECT *, ROW_NUMBER() OVER (ORDER BY ID) AS RowNum
        FROM LargeSourceTable
    ) ranked
    WHERE RowNum BETWEEN ' + CAST(@StartRow AS VARCHAR) + N' AND ' + CAST(@EndRow AS VARCHAR);
    
    EXEC sp_executesql @SQL;
    
    PRINT N'批次 ' + CAST(@CurrentBatch AS VARCHAR) + N' 完成';
    SET @CurrentBatch = @CurrentBatch + 1;
END
```

### 效能優化技巧

**優化 SELECT INTO 效能**
```sql
-- 技巧 1: 使用適當的 WHERE 條件
SELECT *
INTO OptimizedTable
FROM LargeTable
WHERE CreateDate >= '2024-01-01'  -- 減少資料量
  AND Status = 'Active';          -- 使用索引欄位

-- 技巧 2: 只選擇需要的欄位
SELECT ID, Name, ImportantField
INTO EssentialData
FROM LargeTable
WHERE Conditions;

-- 技巧 3: 使用 NOLOCK（讀取時）
SELECT *
INTO ReadOnlyBackup
FROM SourceTable WITH (NOLOCK)
WHERE BackupConditions;
```

**優化 INSERT INTO 效能**
```sql
-- 技巧 1: 批次插入
INSERT INTO TargetTable (Col1, Col2, Col3)
VALUES (Val1a, Val2a, Val3a),
       (Val1b, Val2b, Val3b),
       (Val1c, Val2c, Val3c);  -- 一次插入多筆

-- 技巧 2: 使用 INSERT INTO ... SELECT
INSERT INTO TargetTable (Col1, Col2, Col3)
SELECT Col1, Col2, Col3
FROM SourceTable
WHERE Conditions;

-- 技巧 3: 暫時停用約束（大量資料時）
ALTER TABLE TargetTable NOCHECK CONSTRAINT ALL;
-- 執行大量插入
ALTER TABLE TargetTable CHECK CONSTRAINT ALL;
```

### 最佳實踐指南

**選擇決策樹**
```
需要複製資料？
├─ 是新表格？
│  ├─ 是 → 使用 SELECT INTO
│  └─ 否 → 使用 INSERT INTO
└─ 是修改現有表？
   └─ 使用 INSERT INTO
```

**錯誤處理範例**
```sql
BEGIN TRY
    BEGIN TRANSACTION;
    
    -- 檢查目標表是否存在
    IF OBJECT_ID('TargetTable') IS NOT NULL
    BEGIN
        RAISERROR('目標表已存在', 16, 1);
    END
    
    -- 執行 SELECT INTO
    SELECT *
    INTO TargetTable
    FROM SourceTable
    WHERE ValidConditions;
    
    -- 驗證資料完整性
    IF (SELECT COUNT(*) FROM TargetTable) = 0
    BEGIN
        RAISERROR('沒有資料被複製', 16, 1);
    END
    
    COMMIT TRANSACTION;
    PRINT N'資料複製成功';
    
END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION;
    PRINT N'錯誤: ' + ERROR_MESSAGE();
    
    -- 清理可能建立的表
    IF OBJECT_ID('TargetTable') IS NOT NULL
        DROP TABLE TargetTable;
END CATCH
```

---

## 實務應用場景

### 資料備份策略

**定期備份腳本**
```sql
-- 建立定期備份程序
CREATE PROCEDURE sp_CreateDailyBackup
AS
BEGIN
    DECLARE @BackupDate VARCHAR(8) = CONVERT(VARCHAR, GETDATE(), 112);
    DECLARE @TableName NVARCHAR(100) = N'Customers_Backup_' + @BackupDate;
    DECLARE @SQL NVARCHAR(MAX);
    
    -- 檢查今日是否已有備份
    IF OBJECT_ID(@TableName) IS NOT NULL
    BEGIN
        PRINT N'今日備份已存在: ' + @TableName;
        RETURN;
    END
    
    -- 建立備份
    SET @SQL = N'
    SELECT *,
           GETDATE() AS BackupTime,
           ''' + @BackupDate + N''' AS BackupDate
    INTO ' + @TableName + N'
    FROM Customers
    WHERE Status = ''Active''';
    
    EXEC sp_executesql @SQL;
    
    -- 記錄備份資訊
    DECLARE @RecordCount INT;
    SET @SQL = N'SELECT @Count = COUNT(*) FROM ' + @TableName;
    EXEC sp_executesql @SQL, N'@Count INT OUTPUT', @Count = @RecordCount OUTPUT;
    
    PRINT N'備份完成: ' + @TableName + N', 記錄數: ' + CAST(@RecordCount AS VARCHAR);
END
```

### 資料遷移方案

**跨資料庫遷移**
```sql
-- 資料遷移腳本
DECLARE @MigrationLog TABLE (
    StepName VARCHAR(100),
    StartTime DATETIME2,
    EndTime DATETIME2,
    RecordCount INT,
    Status VARCHAR(20)
);

-- 步驟 1: 遷移客戶資料
DECLARE @StartTime DATETIME2 = SYSDATETIME();

SELECT *
INTO NewDB.dbo.Customers
FROM OldDB.dbo.Customers
WHERE LastModified >= '2024-01-01';

DECLARE @CustomerCount INT = @@ROWCOUNT;
DECLARE @EndTime DATETIME2 = SYSDATETIME();

INSERT INTO @MigrationLog VALUES ('客戶資料遷移', @StartTime, @EndTime, @CustomerCount, '完成');

-- 步驟 2: 遷移訂單資料
SET @StartTime = SYSDATETIME();

SELECT o.*
INTO NewDB.dbo.Orders
FROM OldDB.dbo.Orders o
INNER JOIN NewDB.dbo.Customers c ON o.CustomerID = c.CustomerID;

DECLARE @OrderCount INT = @@ROWCOUNT;
SET @EndTime = SYSDATETIME();

INSERT INTO @MigrationLog VALUES ('訂單資料遷移', @StartTime, @EndTime, @OrderCount, '完成');

-- 顯示遷移報告
SELECT StepName,
       RecordCount,
       DATEDIFF(MILLISECOND, StartTime, EndTime) AS DurationMS,
       Status
FROM @MigrationLog;
```

### 測試環境建立

**快速建立測試資料**
```sql
-- 建立測試環境腳本
PRINT N'=== 建立測試環境 ===';

-- 1. 複製生產環境結構（不含資料）
SELECT TOP 0 *
INTO Test_Customers
FROM Production_Customers;

SELECT TOP 0 *
INTO Test_Orders  
FROM Production_Orders;

-- 2. 複製樣本資料
INSERT INTO Test_Customers
SELECT TOP 100 *
FROM Production_Customers
ORDER BY NEWID();  -- 隨機選取

-- 3. 複製對應的訂單資料
INSERT INTO Test_Orders
SELECT o.*
FROM Production_Orders o
INNER JOIN Test_Customers c ON o.CustomerID = c.CustomerID;

-- 4. 產生額外測試資料
WITH NumberSeries AS (
    SELECT 1 AS Num
    UNION ALL
    SELECT Num + 1 FROM NumberSeries WHERE Num < 1000
)
SELECT 
    Num + 100000 AS CustomerID,
    'TestCustomer' + CAST(Num AS VARCHAR) AS CustomerName,
    'test' + CAST(Num AS VARCHAR) + '@test.com' AS Email,
    DATEADD(DAY, -Num, GETDATE()) AS CreateDate
INTO Additional_Test_Customers
FROM NumberSeries
OPTION (MAXRECURSION 1000);

PRINT N'測試環境建立完成';
```

---

## 總結與最佳實踐

### 📋 **使用場景決策表**

| 需求 | 建議方案 | 理由 |
|------|----------|------|
| 快速備份整張表 | SELECT INTO | 一行指令完成 |
| 複製表結構 | SELECT INTO + WHERE 1=0 | 自動產生結構 |
| 新增資料到現有表 | INSERT INTO | 保持原表結構 |
| 大量資料處理 | SELECT INTO | 效能較好 |
| 需要錯誤處理 | INSERT INTO | 較容易控制 |

### 🎯 **效能優化金句**

> "SELECT INTO 一氣呵成，INSERT INTO 穩扎穩打！"

> "新表用 SELECT INTO，舊表用 INSERT INTO！"

> "大量資料 SELECT INTO 快，小量資料差不多！"

### ✅ **最佳實踐檢查清單**

**SELECT INTO 使用前檢查**
- [ ] 確認目標表不存在
- [ ] 評估資料量大小
- [ ] 考慮是否需要索引
- [ ] 確認磁碟空間足夠

**INSERT INTO 使用前檢查**
- [ ] 確認目標表存在且結構正確
- [ ] 檢查約束條件
- [ ] 考慮是否需要暫停約束
- [ ] 評估是否需要批次處理

**效能優化檢查**
- [ ] 使用適當的 WHERE 條件
- [ ] 只選擇必要的欄位
- [ ] 考慮使用 NOLOCK（適當時）
- [ ] 大量資料考慮分批處理

### 🚀 **進階技巧總結**

1. **資料複製**：SELECT INTO 速度快，適合建新表
2. **資料插入**：INSERT INTO 控制性好，適合現有表
3. **效能考量**：大量資料優先考慮 SELECT INTO
4. **錯誤處理**：INSERT INTO 在交易中較容易控制
5. **維護性**：SELECT INTO 語法簡潔，INSERT INTO 功能完整