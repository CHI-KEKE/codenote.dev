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
  - [2.3 多參數委派](#23-多參數委派)
    - [2.3.1 Action 多參數範例](#231-action-多參數範例)
    - [2.3.2 Func 多參數範例](#232-func-多參數範例)
    - [2.3.3 實際應用場景](#233-實際應用場景)
  - [2.4 用資料映射行為](#24-用資料映射行為)
    - [2.4.1 基於委派的映射](#241-基於委派的映射)
    - [2.4.2 策略模式實作](#242-策略模式實作)
    - [2.4.3 設計模式比較](#243-設計模式比較)
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

### 2.3 多參數委派

委派不僅可以處理單一參數，還能靈活地處理多個參數的情況。這在實際開發中非常有用，特別是在事件處理、資料轉換和複雜業務邏輯中。

#### 2.3.1 Action 多參數範例

##### 🎯 **基本多參數用法**

`Action` 委派可以接受最多 16 個參數，非常適合執行不需要回傳值的操作：

```csharp
void Main()
{
    // 🔢 單一參數的 Action
    Action<int> f1 = (i) => Console.WriteLine($"得到 : {i}");
    f1(555);
    
    // 📝 多參數的 Action - 處理位元組陣列和編碼
    Action<byte[], Encoding> f2 = (b, e) => Console.WriteLine($"Encode 這玩意兒 : {e.GetString(b)}");
    f2(new byte[] {11, 34, 56}, Encoding.UTF8);
    
    // 🎨 三參數的 Action - 格式化輸出
    Action<string, int, DateTime> f3 = (name, age, date) => 
        Console.WriteLine($"使用者: {name}, 年齡: {age}, 註冊日期: {date:yyyy-MM-dd}");
    f3("Allen", 30, DateTime.Now);
}
```

##### 📊 **實際業務應用**

```csharp
void Main()
{
    // 📧 郵件發送委派
    Action<string, string, string> sendEmail = (to, subject, body) =>
    {
        Console.WriteLine($"發送郵件到: {to}");
        Console.WriteLine($"主旨: {subject}");
        Console.WriteLine($"內容: {body}");
        Console.WriteLine("郵件發送完成！");
    };
    
    sendEmail("user@example.com", "歡迎使用服務", "感謝您註冊我們的服務！");
    
    // 🛒 購物車操作委派
    Action<string, decimal, int> addToCart = (productName, price, quantity) =>
    {
        var total = price * quantity;
        Console.WriteLine($"商品 '{productName}' 已加入購物車");
        Console.WriteLine($"單價: {price:C}, 數量: {quantity}, 小計: {total:C}");
    };
    
    addToCart("iPhone 15", 32900, 2);
    
    // 📝 日誌記錄委派
    Action<LogLevel, string, Exception> logMessage = (level, message, ex) =>
    {
        var timestamp = DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss");
        Console.WriteLine($"[{timestamp}] [{level}] {message}");
        if (ex != null)
            Console.WriteLine($"例外詳情: {ex.Message}");
    };
    
    logMessage(LogLevel.Info, "系統啟動完成", null);
    logMessage(LogLevel.Error, "資料庫連線失敗", new Exception("Connection timeout"));
}

public enum LogLevel
{
    Info, Warning, Error
}
```

#### 2.3.2 Func 多參數範例

##### 🔄 **帶回傳值的多參數委派**

`Func` 委派可以處理多個輸入參數並回傳結果，非常適合計算和轉換操作：

```csharp
void Main()
{
    // 🧮 數學計算委派
    Func<int, int, int> add = (a, b) => a + b;
    Func<int, int, int, int> multiply3Numbers = (x, y, z) => x * y * z;
    
    Console.WriteLine($"加法結果: {add(10, 20)}");
    Console.WriteLine($"三數相乘: {multiply3Numbers(2, 3, 4)}");
    
    // 💰 商業計算委派
    Func<decimal, decimal, decimal, decimal> calculateTotal = (price, tax, discount) =>
    {
        var taxAmount = price * tax;
        var discountAmount = price * discount;
        return price + taxAmount - discountAmount;
    };
    
    var total = calculateTotal(1000m, 0.1m, 0.05m); // 價格1000, 稅率10%, 折扣5%
    Console.WriteLine($"最終金額: {total:C}");
    
    // 📊 字串處理委派
    Func<string, string, string, string> formatUserInfo = (name, email, role) =>
        $"姓名: {name} | 信箱: {email} | 角色: {role}";
    
    var userInfo = formatUserInfo("Allen", "allen@example.com", "管理員");
    Console.WriteLine(userInfo);
}
```

##### 🏭 **複雜業務邏輯範例**

```csharp
void Main()
{
    // 🔍 資料驗證委派
    Func<string, string, int, bool> validateUser = (name, email, age) =>
    {
        if (string.IsNullOrWhiteSpace(name)) return false;
        if (!email.Contains("@")) return false;
        if (age < 18 || age > 120) return false;
        return true;
    };
    
    Console.WriteLine($"使用者驗證1: {validateUser("Allen", "allen@test.com", 25)}"); // true
    Console.WriteLine($"使用者驗證2: {validateUser("", "invalid-email", 15)}");      // false
    
    // 💳 信用卡處理委派
    Func<string, DateTime, string, PaymentResult> processPayment = (cardNumber, expiry, cvv) =>
    {
        // 模擬信用卡驗證邏輯
        if (cardNumber.Length != 16) 
            return new PaymentResult { Success = false, Message = "信用卡號碼格式錯誤" };
        
        if (expiry < DateTime.Now) 
            return new PaymentResult { Success = false, Message = "信用卡已過期" };
        
        if (cvv.Length != 3) 
            return new PaymentResult { Success = false, Message = "CVV 格式錯誤" };
        
        return new PaymentResult { Success = true, Message = "付款成功", TransactionId = Guid.NewGuid().ToString("N")[..8] };
    };
    
    var result = processPayment("1234567890123456", DateTime.Now.AddYears(2), "123");
    Console.WriteLine($"付款結果: {result.Message}");
    if (result.Success)
        Console.WriteLine($"交易編號: {result.TransactionId}");
}

public class PaymentResult
{
    public bool Success { get; set; }
    public string Message { get; set; }
    public string TransactionId { get; set; }
}
```

#### 2.3.3 實際應用場景

##### 🎯 **事件處理應用**

```csharp
void Main()
{
    var eventManager = new EventManager();
    
    // 註冊多參數事件處理器
    eventManager.RegisterUserAction((userId, action, timestamp) =>
    {
        Console.WriteLine($"使用者 {userId} 在 {timestamp:HH:mm:ss} 執行了 {action}");
    });
    
    eventManager.RegisterSystemAlert((level, service, message, data) =>
    {
        Console.WriteLine($"[{level}] {service}: {message}");
        if (data != null)
            Console.WriteLine($"附加資料: {data}");
    });
    
    // 觸發事件
    eventManager.TriggerUserAction("user123", "登入", DateTime.Now);
    eventManager.TriggerSystemAlert("ERROR", "Database", "連線逾時", "Connection timeout after 30s");
}

public class EventManager
{
    private Action<string, string, DateTime> userActionHandler;
    private Action<string, string, string, string> systemAlertHandler;
    
    public void RegisterUserAction(Action<string, string, DateTime> handler)
    {
        userActionHandler = handler;
    }
    
    public void RegisterSystemAlert(Action<string, string, string, string> handler)
    {
        systemAlertHandler = handler;
    }
    
    public void TriggerUserAction(string userId, string action, DateTime timestamp)
    {
        userActionHandler?.Invoke(userId, action, timestamp);
    }
    
    public void TriggerSystemAlert(string level, string service, string message, string data)
    {
        systemAlertHandler?.Invoke(level, service, message, data);
    }
}
```

##### 🔧 **工廠模式應用**

```csharp
void Main()
{
    var factory = new DelegateFactory();
    
    // 註冊不同的建立函式
    factory.RegisterCreator("user", (name, email, role) => 
        new User { Name = name, Email = email, Role = role });
    
    factory.RegisterCreator("product", (name, price, category) => 
        new Product { Name = name, Price = decimal.Parse(price), Category = category });
    
    // 使用工廠建立物件
    var user = factory.Create("user", "Allen", "allen@test.com", "Admin") as User;
    var product = factory.Create("product", "iPhone", "32900", "Electronics") as Product;
    
    Console.WriteLine($"建立使用者: {user.Name} ({user.Role})");
    Console.WriteLine($"建立產品: {product.Name} - {product.Price:C}");
}

public class DelegateFactory
{
    private Dictionary<string, Func<string, string, string, object>> creators = new();
    
    public void RegisterCreator(string type, Func<string, string, string, object> creator)
    {
        creators[type] = creator;
    }
    
    public object Create(string type, string param1, string param2, string param3)
    {
        if (creators.TryGetValue(type, out var creator))
            return creator(param1, param2, param3);
        
        throw new ArgumentException($"Unknown type: {type}");
    }
}

public class User
{
    public string Name { get; set; }
    public string Email { get; set; }
    public string Role { get; set; }
}

public class Product
{
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string Category { get; set; }
}
```

##### 📈 **資料處理管線**

```csharp
void Main()
{
    var processor = new DataProcessor();
    
    // 建立資料處理管線
    processor.AddStep("validate", (data, context, config) =>
    {
        Console.WriteLine($"驗證資料: {data} (內容: {context}, 設定: {config})");
        return !string.IsNullOrEmpty(data);
    });
    
    processor.AddStep("transform", (data, context, config) =>
    {
        Console.WriteLine($"轉換資料: {data.ToUpper()}");
        return true;
    });
    
    processor.AddStep("store", (data, context, config) =>
    {
        Console.WriteLine($"儲存資料到 {config} 系統");
        return true;
    });
    
    // 執行處理管線
    processor.Process("hello world", "user_session", "database");
}

public class DataProcessor
{
    private List<(string name, Func<string, string, string, bool> step)> steps = new();
    
    public void AddStep(string name, Func<string, string, string, bool> step)
    {
        steps.Add((name, step));
    }
    
    public bool Process(string data, string context, string config)
    {
        foreach (var (name, step) in steps)
        {
            Console.WriteLine($"執行步驟: {name}");
            if (!step(data, context, config))
            {
                Console.WriteLine($"步驟 {name} 失敗，停止處理");
                return false;
            }
        }
        Console.WriteLine("所有步驟完成！");
        return true;
    }
}
```

##### 🎯 **多參數委派的優勢**

**靈活性提升：**
- ✅ **多元輸入**：可以處理複雜的多參數場景
- ✅ **型別安全**：編譯時期確保參數型別正確
- ✅ **可組合性**：容易組合和重用不同的處理邏輯

**程式碼簡潔：**
- 🎨 **減少類別定義**：避免為簡單操作建立額外的類別
- 🎨 **內嵌邏輯**：可以在需要的地方直接定義處理邏輯
- 🎨 **Lambda 表達式**：簡潔的語法表達複雜的邏輯

**效能考量：**
- 🚀 **記憶體效率**：避免建立不必要的物件
- 🚀 **執行效率**：直接的函式呼叫，減少間接性
- 🚀 **編譯器最佳化**：編譯器可以對委派進行最佳化

> **🌟 重點提醒**
> 
> 1. **參數數量限制**：`Action` 和 `Func` 都有參數數量限制（最多 16 個），超過時需要自訂委派型別
> 2. **可讀性考量**：參數過多時可能影響程式碼可讀性，考慮使用物件參數或自訂委派
> 3. **型別推斷**：善用 `var` 和型別推斷來簡化委派的宣告
> 4. **效能最佳化**：在效能關鍵的場景中，注意委派呼叫的開銷

### 2.4 用資料映射行為

在複雜的業務邏輯中，經常需要根據不同的輸入資料執行不同的操作。使用委派來建立資料與行為之間的映射關係，可以讓程式碼更加彈性和可維護。

#### 2.4.1 基於委派的映射

##### 📊 **基本委派映射範例**

透過字典將資料鍵值對應到特定的委派函式：

```csharp
public enum Op { Add, Minus }

public static class Calculator
{
    private static readonly IReadOnlyDictionary<Op, Func<int, int, int>> _ops =
        new Dictionary<Op, Func<int, int, int>>
        {
            [Op.Add] = (a, b) => a + b,
            [Op.Minus] = (a, b) => a - b
        };

    public static int Exec(Op op, int a, int b) => _ops[op](a, b);
}

// 使用方式
void Main()
{
    var result1 = Calculator.Exec(Op.Add, 12, 34);    // 46
    var result2 = Calculator.Exec(Op.Minus, 100, 25); // 75
    
    Console.WriteLine($"加法結果: {result1}");
    Console.WriteLine($"減法結果: {result2}");
}
```

##### 🎯 **進階委派映射應用**

```csharp
void Main()
{
    var processor = new CommandProcessor();
    
    // 註冊不同的命令處理器
    processor.RegisterCommand("greet", name => $"Hello, {name}!");
    processor.RegisterCommand("upper", text => text.ToUpper());
    processor.RegisterCommand("reverse", text => new string(text.Reverse().ToArray()));
    processor.RegisterCommand("length", text => $"Length: {text.Length}");
    
    // 執行不同的命令
    Console.WriteLine(processor.Execute("greet", "Allen"));     // Hello, Allen!
    Console.WriteLine(processor.Execute("upper", "hello"));     // HELLO
    Console.WriteLine(processor.Execute("reverse", "world"));   // dlrow
    Console.WriteLine(processor.Execute("length", "test"));     // Length: 4
}

public class CommandProcessor
{
    private readonly Dictionary<string, Func<string, string>> _commands = new();
    
    public void RegisterCommand(string name, Func<string, string> handler)
    {
        _commands[name] = handler;
    }
    
    public string Execute(string command, string input)
    {
        if (_commands.TryGetValue(command, out var handler))
            return handler(input);
        
        throw new ArgumentException($"Unknown command: {command}");
    }
    
    public IEnumerable<string> GetAvailableCommands() => _commands.Keys;
}
```

##### 🏭 **工廠模式與委派結合**

```csharp
void Main()
{
    var factory = new ObjectFactory();
    
    // 註冊不同型別的建立器
    factory.RegisterBuilder<User>("user", data => new User 
    { 
        Name = data.GetValueOrDefault("name", "Unknown"),
        Email = data.GetValueOrDefault("email", ""),
        Age = int.Parse(data.GetValueOrDefault("age", "0"))
    });
    
    factory.RegisterBuilder<Product>("product", data => new Product
    {
        Name = data.GetValueOrDefault("name", "Unknown Product"),
        Price = decimal.Parse(data.GetValueOrDefault("price", "0")),
        Category = data.GetValueOrDefault("category", "General")
    });
    
    // 使用工廠建立物件
    var userData = new Dictionary<string, string>
    {
        ["name"] = "Allen",
        ["email"] = "allen@example.com",
        ["age"] = "30"
    };
    
    var productData = new Dictionary<string, string>
    {
        ["name"] = "iPhone 15",
        ["price"] = "32900",
        ["category"] = "Electronics"
    };
    
    var user = factory.Create<User>("user", userData);
    var product = factory.Create<Product>("product", productData);
    
    Console.WriteLine($"建立使用者: {user.Name}, {user.Email}");
    Console.WriteLine($"建立產品: {product.Name}, {product.Price:C}");
}

public class ObjectFactory
{
    private readonly Dictionary<string, Func<Dictionary<string, string>, object>> _builders = new();
    
    public void RegisterBuilder<T>(string type, Func<Dictionary<string, string>, T> builder)
    {
        _builders[type] = data => builder(data);
    }
    
    public T Create<T>(string type, Dictionary<string, string> data)
    {
        if (_builders.TryGetValue(type, out var builder))
            return (T)builder(data);
        
        throw new ArgumentException($"Unknown type: {type}");
    }
}

public class User
{
    public string Name { get; set; }
    public string Email { get; set; }
    public int Age { get; set; }
}

public class Product
{
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string Category { get; set; }
}
```

#### 2.4.2 策略模式實作

##### 🎨 **Plan B：策略模式（最通用、擴充/測試友善）**

對於更複雜的場景，策略模式提供了更好的可測試性和可擴展性：

```csharp
public interface IOperation
{
    string Name { get; }            // 或改用 enum/Keyed Service
    int Execute(int a, int b);
}

public sealed class Add : IOperation
{
    public string Name => "Add";
    public int Execute(int a, int b) => a + b;
}

public sealed class Minus : IOperation
{
    public string Name => "Minus";
    public int Execute(int a, int b) => a - b;
}

public sealed class Multiply : IOperation
{
    public string Name => "Multiply";
    public int Execute(int a, int b) => a * b;
}

public sealed class Divide : IOperation
{
    public string Name => "Divide";
    public int Execute(int a, int b) => b != 0 ? a / b : throw new DivideByZeroException();
}

public sealed class OperationRegistry
{
    private readonly IReadOnlyDictionary<string, IOperation> _map;
    
    public OperationRegistry(IEnumerable<IOperation> ops)
        => _map = ops.ToDictionary(o => o.Name, StringComparer.OrdinalIgnoreCase);

    public int Exec(string name, int a, int b)
        => _map.TryGetValue(name, out var op)
           ? op.Execute(a, b)
           : throw new KeyNotFoundException($"Unknown op: {name}");
    
    public IEnumerable<string> GetAvailableOperations() => _map.Keys;
}

// 使用範例
void Main()
{
    var operations = new List<IOperation>
    {
        new Add(),
        new Minus(),
        new Multiply(),
        new Divide()
    };
    
    var registry = new OperationRegistry(operations);
    
    Console.WriteLine($"加法: {registry.Exec("Add", 10, 5)}");        // 15
    Console.WriteLine($"減法: {registry.Exec("Minus", 10, 5)}");      // 5
    Console.WriteLine($"乘法: {registry.Exec("Multiply", 10, 5)}");   // 50
    Console.WriteLine($"除法: {registry.Exec("Divide", 10, 5)}");     // 2
    
    Console.WriteLine("可用的操作:");
    foreach (var op in registry.GetAvailableOperations())
    {
        Console.WriteLine($"- {op}");
    }
}
```

##### 🧪 **測試友善的設計**

策略模式的一個重要優勢是易於測試：

```csharp
void Main()
{
    // 測試個別操作
    TestOperation(new Add(), 5, 3, 8);
    TestOperation(new Minus(), 10, 4, 6);
    TestOperation(new Multiply(), 7, 6, 42);
    
    // 測試註冊表
    TestRegistry();
}

void TestOperation(IOperation operation, int a, int b, int expected)
{
    var result = operation.Execute(a, b);
    var status = result == expected ? "✅ PASS" : "❌ FAIL";
    Console.WriteLine($"{status} {operation.Name}({a}, {b}) = {result} (expected: {expected})");
}

void TestRegistry()
{
    var mockOperations = new List<IOperation>
    {
        new Add(),
        new Minus()
    };
    
    var registry = new OperationRegistry(mockOperations);
    
    // 測試已知操作
    try
    {
        var result = registry.Exec("Add", 1, 2);
        Console.WriteLine($"✅ Registry test passed: Add(1, 2) = {result}");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"❌ Registry test failed: {ex.Message}");
    }
    
    // 測試未知操作
    try
    {
        registry.Exec("Unknown", 1, 2);
        Console.WriteLine("❌ Should have thrown exception for unknown operation");
    }
    catch (KeyNotFoundException)
    {
        Console.WriteLine("✅ Correctly threw exception for unknown operation");
    }
}
```

#### 2.4.3 設計模式比較

##### 📊 **委派 vs 策略模式比較**

| 比較項目 | 委派映射 | 策略模式 |
|----------|----------|----------|
| **實作複雜度** | 🟢 簡單 | 🟡 中等 |
| **效能** | 🟢 高（直接呼叫） | 🟡 中等（介面呼叫） |
| **可測試性** | 🟡 中等 | 🟢 優秀 |
| **可擴展性** | 🟡 中等 | 🟢 優秀 |
| **相依性注入** | 🔴 困難 | 🟢 容易 |
| **程式碼量** | 🟢 少 | 🟡 中等 |
| **型別安全** | 🟢 優秀 | 🟢 優秀 |

##### 🎯 **選擇建議**

**使用委派映射的時機：**
- ✅ 邏輯簡單，只需要簡單的函式呼叫
- ✅ 效能要求高，需要最小的呼叫開銷
- ✅ 不需要複雜的相依性注入
- ✅ 快速原型開發

**使用策略模式的時機：**
- ✅ 需要複雜的業務邏輯封裝
- ✅ 需要進行單元測試和整合測試
- ✅ 需要相依性注入和 IoC 容器支援
- ✅ 需要在執行時期動態添加新的策略
- ✅ 企業級應用開發

##### 🔄 **混合應用範例**

有時候可以結合兩種方法的優點：

```csharp
void Main()
{
    var hybridCalculator = new HybridCalculator();
    
    // 簡單操作使用委派
    Console.WriteLine($"簡單加法: {hybridCalculator.SimpleAdd(5, 3)}");
    
    // 複雜操作使用策略模式
    Console.WriteLine($"複雜計算: {hybridCalculator.ComplexCalculation("PowerOperation", 2, 8)}");
}

public class HybridCalculator
{
    // 簡單操作使用委派映射
    private readonly Dictionary<string, Func<int, int, int>> _simpleOps = new()
    {
        ["add"] = (a, b) => a + b,
        ["subtract"] = (a, b) => a - b
    };
    
    // 複雜操作使用策略模式
    private readonly Dictionary<string, IComplexOperation> _complexOps = new()
    {
        ["PowerOperation"] = new PowerOperation(),
        ["FactorialOperation"] = new FactorialOperation()
    };
    
    public int SimpleAdd(int a, int b) => _simpleOps["add"](a, b);
    
    public double ComplexCalculation(string operation, int a, int b)
    {
        if (_complexOps.TryGetValue(operation, out var op))
            return op.Calculate(a, b);
        
        throw new ArgumentException($"Unknown complex operation: {operation}");
    }
}

public interface IComplexOperation
{
    double Calculate(int a, int b);
}

public class PowerOperation : IComplexOperation
{
    public double Calculate(int a, int b) => Math.Pow(a, b);
}

public class FactorialOperation : IComplexOperation
{
    public double Calculate(int a, int b)
    {
        // 計算 a! * b!
        return Factorial(a) * Factorial(b);
    }
    
    private long Factorial(int n)
    {
        if (n <= 1) return 1;
        long result = 1;
        for (int i = 2; i <= n; i++)
            result *= i;
        return result;
    }
}
```

> **🌟 重點提醒**
> 
> 1. **適材適用**：根據專案需求選擇合適的設計模式，不要過度設計
> 2. **效能考量**：委派映射效能較高，策略模式較靈活但有額外開銷
> 3. **測試策略**：策略模式更容易進行單元測試和模擬測試
> 4. **擴展性規劃**：考慮未來的擴展需求，選擇適合的架構模式