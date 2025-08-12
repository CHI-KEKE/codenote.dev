# 🎯 Delegate

## 📖 目錄

- [一、概念](#一概念)
  - [1.1 TargetObject & MethodPointer](#11-targetobject--methodpointer)
    - [1.1.1 基本概念](#111-基本概念)
    - [1.1.2 底層運作原理](#112-底層運作原理)
  - [1.2 <>c 類別](#12-c-類別)
    - [1.2.1 編譯器生成機制](#121-編譯器生成機制)
    - [1.2.2 內部結構](#122-內部結構)
- [二、應用案例](#二應用案例)
  - [2.1 簡單的驗證邏輯](#21-簡單的驗證邏輯)
    - [2.1.1 使用 Func 委派](#211-使用-func-委派)
    - [2.1.2 使用 Predicate 委派](#212-使用-predicate-委派)
    - [2.1.3 實際測試結果](#213-實際測試結果)
  - [2.2 Retry 重試機制](#22-retry-重試機制)
    - [2.2.1 擴展方法實作](#221-擴展方法實作)
    - [2.2.2 使用範例](#222-使用範例)
    - [2.2.3 實際應用場景](#223-實際應用場景)
---

## 一、概念

### 1.1 TargetObject & MethodPointer

#### 1.1.1 基本概念

🎯 **Delegate 的核心組成**

每個 delegate 都由兩個重要部分組成：
- **TargetObject**：目標物件實例
- **MethodPointer**：方法在記憶體中的位置

#### 1.1.2 底層運作原理

```csharp
using System;

class Program
{
    static void Main()
    {
        var obj = new Calculator();

        // 建立 delegate（裝一個實例方法）
        Func<int, int> myDelegate = obj.MultiplyBy2;

        // 呼叫 delegate
        int result = myDelegate(10);

        Console.WriteLine($"結果: {result}"); // 輸出 20

        // 額外觀察 Target 與 Method
        Console.WriteLine($"TargetObject: {myDelegate.Target}");  // Calculator 實例
        Console.WriteLine($"MethodPointer: {myDelegate.Method}"); // MultiplyBy2 方法資訊
    }
}

class Calculator
{
    public int MultiplyBy2(int x) => x * 2;
}
```

##### 🔧 **執行步驟分析**

**建立 myDelegate 時：**
```csharp
myDelegate = new Func<int, int>(obj.MultiplyBy2);
```

底層其實做了：
- **TargetObject** = obj（Calculator 類別的一個實例）
- **MethodPointer** = MultiplyBy2 這個方法在記憶體的位置

**呼叫 myDelegate(10) 時：**
1. .NET 先檢查 TargetObject（這裡是 obj）
2. 用 MethodPointer 找到 MultiplyBy2 的位置
3. 執行這個方法，並把 10 傳進去
4. 回傳 20

### 1.2 <>c 類別

#### 1.2.1 編譯器生成機制

🏭 **編譯器的自動化處理**

當你在程式中寫 Lambda 或匿名方法時，編譯器會想：「欸，你這個方法需要被記住，而且可能還要存變數，那我幫你找個地方放。」

於是它偷偷生成一個編譯器專用的類別（`<>c`），專門用來裝這些匿名方法。

**命名規則：**
- `Program.<>c.<>9__0_0` 其實就是編譯器幫你放好的一個「方法存放位置」
- `Program.<>c` 是這個「藏方法的倉庫」的類別名稱（用奇怪的命名確保你平常不會去碰它）

#### 1.2.2 內部結構

🔧 **編譯器生成的類別特性**

這個類別通常是 **sealed**（不讓你繼承），因為它是自動生成、固定用途的輔助類別。

**內部方法範例：**
```csharp
// 編譯器自動生成的方法，對應你的 Lambda 表達式
internal decimal <Main>b__0_0(decimal d) => d * 5;
internal decimal <Main>b__0_1(decimal d) => d * 7;
```

這就是編譯器把你的 `d => d * 5` 和 `d => d * 7` 變成的普通方法。

**轉換過程：**
```csharp
// 你寫的程式碼
Func<decimal, decimal> multiply5 = d => d * 5;
Func<decimal, decimal> multiply7 = d => d * 7;

// 編譯器自動轉換成
// 1. 生成 <>c 類別
// 2. 把 Lambda 轉成普通方法
// 3. 建立 delegate 指向這些方法
```

**優勢分析：**
- ✅ **記憶體效率**：避免重複建立相同的匿名方法
- ✅ **效能最佳化**：編譯器可以對這些方法進行最佳化
- ✅ **型別安全**：保持強型別檢查的優勢
- ✅ **除錯支援**：仍然可以在除錯時追蹤到原始 Lambda

## 二、應用案例

### 2.1 簡單的驗證邏輯

當我們需要執行簡單的驗證邏輯，但又不想為此獨立寫一個函式時，委派是一個很好的選擇。

#### 2.1.1 使用 Func 委派

```csharp
void Main()
{
    // 🔍 測試字串長度差異
    "Ａ".Length.Dump();  // 全形字元 A
    "A".Length.Dump();   // 半形字元 A
    
    // 🎯 使用 Func 委派建立產品 SKU 驗證邏輯
    Func<string, bool> _isValidProductSkuOuterId = productSkuOuterId => 
        System.Text.RegularExpressions.Regex.IsMatch(productSkuOuterId, @"^[a-zA-Z0-9ａ-ｚＡ-Ｚ０-９]+$");
    
    // 📝 測試不同的輸入
    _isValidProductSkuOuterId("Ａ０").Dump();   // true - 全形字元
    _isValidProductSkuOuterId("a9ｚ").Dump();   // true - 混合半形全形
    _isValidProductSkuOuterId(",").Dump();      // false - 包含特殊字元
}
```

#### 2.1.2 使用 Predicate 委派

```csharp
void Main()
{
    // 🎯 使用 Predicate 委派（專門用於 bool 回傳值）
    Predicate<string> isValidOuterId = x => 
        Regex.IsMatch(x, @"^[a-zA-Z0-9ａ-ｚＡ-Ｚ０-９]+$");
    
    // 📝 測試相同的驗證邏輯
    isValidOuterId("Ａ０").Dump();   // true
    isValidOuterId("a9ｚ").Dump();   // true  
    isValidOuterId(",").Dump();      // false
}
```

### 2.2 Retry 重試機制

在處理網路請求、資料庫操作或其他可能暫時失敗的操作時，重試機制是一個常見且重要的模式。透過 delegate 的威力，我們可以建立通用的重試擴展方法。

#### 2.2.1 擴展方法實作

🔄 **通用 Retry 擴展方法**

以下實作展示如何為 `Func<Task>` 和 `Func<Task<TResult>>` 建立重試機制：

```csharp
namespace Nine1.Promotion.Common.Utils.Extension
{
    public static class FuncRetryExtensions
    {
        /// <summary>
        /// 無回傳值的 RetryAsync
        /// </summary>
        /// <param name="func">要重試的非同步函式</param>
        /// <param name="retryCount">重試次數，預設 3 次</param>
        /// <param name="delayMilliseconds">重試間隔時間（毫秒），預設 200ms</param>
        /// <param name="onRetry">重試時的回呼函式，可用於記錄或通知</param>
        public static async Task RetryAsync(this Func<Task> func, int retryCount = 3, int delayMilliseconds = 200, Action<Exception, int> onRetry = null)
        {
            if (func == null) throw new ArgumentNullException(nameof(func));
            if (retryCount < 1) throw new ArgumentOutOfRangeException(nameof(retryCount));
            if (delayMilliseconds < 0) throw new ArgumentOutOfRangeException(nameof(delayMilliseconds));

            int attempt = 0;
            var exceptions = new List<Exception>();

            while (true)
            {
                try
                {
                    await func();
                    return; // 成功執行，跳出迴圈
                }
                catch (Exception ex)
                {
                    exceptions.Add(ex);
                    if (++attempt >= retryCount)
                        throw new AggregateException($"Operation failed after {retryCount} attempts.", exceptions);

                    onRetry?.Invoke(ex, attempt);
                    await Task.Delay(delayMilliseconds);
                }
            }
        }

        /// <summary>
        /// 有回傳值的 RetryAsync
        /// </summary>
        /// <typeparam name="TResult">回傳值型別</typeparam>
        /// <param name="func">要重試的非同步函式</param>
        /// <param name="retryCount">重試次數，預設 3 次</param>
        /// <param name="delayMilliseconds">重試間隔時間（毫秒），預設 200ms</param>
        /// <param name="onRetry">重試時的回呼函式，可用於記錄或通知</param>
        /// <returns>函式執行的結果</returns>
        public static async Task<TResult> RetryAsync<TResult>(this Func<Task<TResult>> func, int retryCount = 3, int delayMilliseconds = 200, Action<Exception, int> onRetry = null)
        {
            if (func == null) throw new ArgumentNullException(nameof(func));
            if (retryCount < 1) throw new ArgumentOutOfRangeException(nameof(retryCount));
            if (delayMilliseconds < 0) throw new ArgumentOutOfRangeException(nameof(delayMilliseconds));

            int attempt = 0;
            var exceptions = new List<Exception>();

            while (true)
            {
                try
                {
                    return await func();
                }
                catch (Exception ex)
                {
                    exceptions.Add(ex);
                    if (++attempt >= retryCount)
                        throw new AggregateException($"Operation failed after {retryCount} attempts.", exceptions);

                    onRetry?.Invoke(ex, attempt);
                    await Task.Delay(delayMilliseconds);
                }
            }
        }
    }
}
```

#### 2.2.2 使用範例

##### 🌐 **網路 API 呼叫重試**

```csharp
async Task Main()
{
    var httpClient = new HttpClient();
    
    // 無回傳值的重試範例
    await new Func<Task>(async () =>
    {
        var response = await httpClient.PostAsync("https://api.example.com/data", 
            new StringContent("test data"));
        
        if (!response.IsSuccessStatusCode)
            throw new HttpRequestException($"HTTP Error: {response.StatusCode}");
            
    }).RetryAsync(
        retryCount: 5,
        delayMilliseconds: 1000,
        onRetry: (ex, attempt) => Console.WriteLine($"重試第 {attempt} 次，錯誤: {ex.Message}")
    );
    
    Console.WriteLine("API 呼叫成功！");
}
```

##### 📊 **資料庫查詢重試**

```csharp
async Task Main()
{
    // 有回傳值的重試範例
    var userData = await new Func<Task<UserInfo>>(async () =>
    {
        // 模擬資料庫查詢
        await Task.Delay(100); // 模擬網路延遲
        
        // 30% 機率失敗，用來測試重試機制
        if (new Random().Next(100) < 30)
            throw new SqlException("Database connection timeout");
            
        return new UserInfo { Id = 123, Name = "Allen" };
        
    }).RetryAsync(
        retryCount: 3,
        delayMilliseconds: 500,
        onRetry: (ex, attempt) => 
        {
            Console.WriteLine($"🔄 資料庫查詢失敗，第 {attempt} 次重試");
            Console.WriteLine($"   錯誤訊息: {ex.Message}");
        }
    );
    
    Console.WriteLine($"✅ 成功取得使用者資料: {userData.Name}");
}

public class UserInfo
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

##### 🔧 **檔案操作重試**

```csharp
async Task Main()
{
    string filePath = @"C:\temp\important_file.txt";
    
    // 檔案寫入重試
    await new Func<Task>(async () =>
    {
        // 可能因為檔案被其他程式鎖定而失敗
        await File.WriteAllTextAsync(filePath, "重要資料內容");
        
    }).RetryAsync(
        retryCount: 5,
        delayMilliseconds: 300,
        onRetry: (ex, attempt) => 
        {
            Console.WriteLine($"📁 檔案寫入失敗，{attempt} 秒後重試...");
            if (ex is UnauthorizedAccessException)
                Console.WriteLine("   可能是檔案被其他程式使用中");
        }
    );
    
    Console.WriteLine("✅ 檔案寫入成功！");
}
```

#### 2.2.3 實際應用場景

##### 🎯 **重試機制的優勢**

**穩定性提升：**
- ✅ **暫時性錯誤恢復**：網路抖動、資料庫連線暫時中斷等
- ✅ **系統韌性增強**：減少因偶發錯誤導致的操作失敗
- ✅ **使用者體驗改善**：減少錯誤訊息，提高操作成功率

**靈活性設計：**
- 🔧 **可配置重試次數**：根據不同操作的重要性調整
- 🔧 **可調整重試間隔**：避免對系統造成過大壓力
- 🔧 **自訂重試回呼**：記錄、通知或其他自訂邏輯

**企業級應用：**
- 🏢 **微服務通訊**：服務間 API 呼叫的重試機制
- 🏢 **資料同步**：分散式系統間的資料同步重試
- 🏢 **外部整合**：第三方 API 整合的穩定性保障

##### ⚠️ **使用注意事項**

**避免無限重試：**
```csharp
// ❌ 不要這樣做 - 可能導致無限重試
await func.RetryAsync(retryCount: int.MaxValue);

// ✅ 應該設置合理的重試次數
await func.RetryAsync(retryCount: 3);
```

**冪等性考量：**
```csharp
// ✅ 適合重試的操作（冪等）
await GetUserDataAsync().RetryAsync();

// ⚠️ 需要謹慎的操作（非冪等）
await CreateNewOrderAsync().RetryAsync(); // 可能重複建立訂單
```

**重試間隔策略：**
```csharp
// 🎯 指數退避策略的進階實作
public static async Task RetryWithExponentialBackoff<T>(
    this Func<Task<T>> func, 
    int maxRetries = 3)
{
    for (int i = 0; i < maxRetries; i++)
    {
        try
        {
            return await func();
        }
        catch when (i < maxRetries - 1)
        {
            var delay = TimeSpan.FromMilliseconds(Math.Pow(2, i) * 1000);
            await Task.Delay(delay);
        }
    }
}
```

> **🌟 重點提醒**
> 
> 1. **冪等性檢查**：確保重試的操作是安全的，不會造成重複效果
> 2. **錯誤類型判斷**：某些錯誤（如驗證錯誤）不應該重試
> 3. **監控與記錄**：記錄重試次數和失敗原因，用於系統監控
> 4. **資源管理**：避免重試機制消耗過多系統資源