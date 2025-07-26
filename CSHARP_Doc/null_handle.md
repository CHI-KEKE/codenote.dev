# C# Null 處理完整指南 🎯

> 掌握 C# 中的 null 處理技巧，讓你的程式碼更安全、更優雅！

## 📋 目錄
- [Nullable&lt;T&gt; 基礎概念](#nullable-t-基礎概念)
- [HasValue 屬性詳解](#hasvalue-屬性詳解)
- [GetValueOrDefault 方法](#getvalueordefault-方法)
- [is null 的威力](#is-null-的威力)
- [方法多載與 null 的奧秘](#方法多載與-null-的奧秘)
- [實戰範例與最佳實踐](#實戰範例與最佳實踐)

---

## 🎨 Nullable&lt;T&gt; 基礎概念

### 什麼是 Nullable&lt;T&gt;？

在 C# 中，值型別（如 `int`、`DateTime`）預設不能為 null，但透過 `Nullable<T>` 或簡寫 `T?`，我們可以讓它們「擁抱 null」！

```csharp
// 傳統值型別 - 不能為 null
int normalInt = 10;

// 可空值型別 - 可以為 null
int? nullableInt = null;
DateTime? nullableDate = null;
```

### 📝 參數預設值的優雅寫法

```csharp
// ✅ 推薦：直接在參數中設定預設值
public void ProcessData(int? userId = null, DateTime? startDate = null)
{
    if (userId.HasValue)
    {
        Console.WriteLine($"處理使用者 ID: {userId.Value}");
    }
    
    if (startDate.HasValue)
    {
        Console.WriteLine($"開始日期: {startDate.Value:yyyy-MM-dd}");
    }
}

// 呼叫方式更簡潔
ProcessData(); // 使用預設值
ProcessData(123); // 只指定 userId
ProcessData(123, DateTime.Now); // 指定所有參數
```

---

## 🔍 HasValue 屬性詳解

### 🌟 HasValue 的特色

`HasValue` 是 `Nullable<T>` 的專屬屬性，用來檢查是否有值：

```csharp
int? score = null;

// ✅ 使用 HasValue - 語意清晰
if (score.HasValue)
{
    Console.WriteLine($"分數：{score.Value}");
}
else
{
    Console.WriteLine("尚未評分");
}

// 🤔 與 != null 比較
if (score != null)
{
    Console.WriteLine($"分數：{score.Value}");
}
```

### 🎯 HasValue vs != null

| 特性 | HasValue | != null |
|------|----------|---------|
| **適用範圍** | 僅 `Nullable<T>` | 所有參考型別 |
| **效能** | 內部優化，更快 | 一般比較 |
| **語意** | 明確表示值檢查 | 通用 null 檢查 |
| **型別安全** | 編譯時期檢查 | 執行時期檢查 |

### 🎪 實際應用場景

#### 1️⃣ 資料庫查詢結果
```csharp
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public DateTime? LastLoginTime { get; set; } // 可能為 null
}

public void DisplayUserInfo(User user)
{
    Console.WriteLine($"使用者：{user.Name}");
    
    if (user.LastLoginTime.HasValue)
    {
        var daysSinceLogin = (DateTime.Now - user.LastLoginTime.Value).Days;
        Console.WriteLine($"上次登入：{daysSinceLogin} 天前");
    }
    else
    {
        Console.WriteLine("從未登入過");
    }
}
```

#### 2️⃣ API 參數檢查
```csharp
public IActionResult GetProducts(int? categoryId = null, decimal? minPrice = null)
{
    var query = dbContext.Products.AsQueryable();
    
    if (categoryId.HasValue)
    {
        query = query.Where(p => p.CategoryId == categoryId.Value);
    }
    
    if (minPrice.HasValue)
    {
        query = query.Where(p => p.Price >= minPrice.Value);
    }
    
    return Ok(query.ToList());
}
```

#### 3️⃣ 系統設定與配置
```csharp
public class AppConfig
{
    public int? MaxRetryCount { get; set; }
    public TimeSpan? Timeout { get; set; }
    
    public int GetRetryCount()
    {
        return MaxRetryCount.HasValue ? MaxRetryCount.Value : 3; // 預設 3 次
    }
    
    public TimeSpan GetTimeout()
    {
        return Timeout.HasValue ? Timeout.Value : TimeSpan.FromSeconds(30); // 預設 30 秒
    }
}
```

---

## 🛡️ GetValueOrDefault 方法

### 🎯 避免 InvalidOperationException

當我們嘗試存取 `Nullable<T>.Value` 但實際上為 null 時，會拋出 `InvalidOperationException`。`GetValueOrDefault()` 提供了安全的解決方案：

```csharp
int? score = null;

// ❌ 危險：會拋出 InvalidOperationException
try
{
    int value = score.Value; // 💥 例外！
}
catch (InvalidOperationException ex)
{
    Console.WriteLine("糟糕！嘗試存取 null 值");
}

// ✅ 安全：使用 GetValueOrDefault
int safeValue1 = score.GetValueOrDefault(); // 回傳 0 (int 的預設值)
int safeValue2 = score.GetValueOrDefault(100); // 回傳 100 (自定預設值)
```

### 🎨 實用範例

```csharp
public class ShoppingCart
{
    public decimal? DiscountPercentage { get; set; }
    public int? ItemCount { get; set; }
    
    public decimal CalculateDiscount(decimal totalAmount)
    {
        // 如果沒有折扣，預設為 0%
        var discount = DiscountPercentage.GetValueOrDefault(0m);
        return totalAmount * (discount / 100);
    }
    
    public string GetItemCountDisplay()
    {
        var count = ItemCount.GetValueOrDefault(0);
        return count == 0 ? "購物車是空的" : $"共 {count} 項商品";
    }
}
```

---

## ⚡ is null 的威力

### 🎯 為什麼要使用 is null？

在 C# 中，`==` 運算符可能被重載，導致 null 檢查結果不如預期。`is null` 提供了純粹的參考比較：

```csharp
public class Person
{
    public string Name { get; set; }
    
    // 重載 == 運算符
    public static bool operator ==(Person left, Person right)
    {
        if (ReferenceEquals(left, right)) return true;
        if (left is null || right is null) return false;
        return left.Name == right.Name;
    }
    
    public static bool operator !=(Person left, Person right)
    {
        return !(left == right);
    }
}

// 測試情境
Person person = null;

// ✅ is null - 確保純粹的參考比較
if (person is null)
{
    Console.WriteLine("person 確實為 null"); // 會執行
}

// 🤔 == null - 可能受重載影響
if (person == null)
{
    Console.WriteLine("這也會執行，但不保證所有情況都正確");
}
```

### 🌟 is null 的三大優勢

#### 1️⃣ 確保純粹的參考比較
```csharp
public void SafeNullCheck<T>(T obj) where T : class
{
    // ✅ 不受 == 重載影響
    if (obj is null)
    {
        Console.WriteLine("物件為 null");
        return;
    }
    
    // 安全地使用物件
    Console.WriteLine($"物件型別：{obj.GetType().Name}");
}
```

#### 2️⃣ 減少 NullReferenceException
```csharp
public void ProcessUser(User user)
{
    // ✅ 先檢查，避免例外
    if (user is null)
    {
        throw new ArgumentNullException(nameof(user), "使用者不能為 null");
    }
    
    // 安全地存取屬性
    Console.WriteLine($"處理使用者：{user.Name}");
}
```

#### 3️⃣ 提升可讀性與維護性
```csharp
public class DataProcessor
{
    public void ProcessData(string[] data)
    {
        // ✅ 語意清楚
        if (data is null)
        {
            Console.WriteLine("沒有資料需要處理");
            return;
        }
        
        if (data.Length == 0)
        {
            Console.WriteLine("資料陣列是空的");
            return;
        }
        
        // 處理資料...
    }
}
```

---

## 🎭 方法多載與 null 的奧秘

### 🧩 編譯器如何選擇多載方法？

當傳入 `null` 時，C# 編譯器會根據型別相容性和特定性來選擇最適合的方法：

#### 範例 1：值型別 vs 參考型別
```csharp
public class OverloadDemo
{
    public string ProcessValue(List<int> data)
    {
        return "處理 List<int> 參數";
    }
    
    public string ProcessValue(int value)
    {
        return "處理 int 參數";
    }
}

// 測試
var demo = new OverloadDemo();
string result = demo.ProcessValue(null);
Console.WriteLine(result); // 輸出：處理 List<int> 參數

// 📖 原因：int 是值型別，不能接受 null
// 因此編譯器選擇 List<int> 版本
```

#### 範例 2：參考型別之間的選擇
```csharp
public class AdvancedOverloadDemo
{
    public string HandleData(List<object> data)
    {
        return "處理 List<object> 參數";
    }
    
    public string HandleData(object data)
    {
        return "處理 object 參數";
    }
}

// 測試
var demo = new AdvancedOverloadDemo();
string result = demo.HandleData(null);
Console.WriteLine(result); // 輸出：處理 List<object> 參數

// 📖 原因：在 C# 型別系統中，泛型型別被認為比非泛型型別「更具體」
```

### 🎯 多載解析規則總結

| 情況 | 編譯器選擇邏輯 |
|------|----------------|
| **值型別 vs 參考型別** | 選擇能接受 null 的參考型別 |
| **泛型 vs 非泛型** | 優先選擇泛型型別（更具體） |
| **繼承層次** | 選擇更具體的子類別型別 |
| **相同特定性** | 編譯錯誤（模糊呼叫） |

### 🛠️ 實際應用：設計清晰的 API

```csharp
public class ApiController
{
    // ❌ 避免：容易造成混淆的多載
    public IActionResult GetUser(int? id) { /* ... */ }
    public IActionResult GetUser(string email) { /* ... */ }
    
    // ✅ 推薦：明確的方法名稱
    public IActionResult GetUserById(int? id) 
    {
        if (id is null)
        {
            return BadRequest("使用者 ID 不能為空");
        }
        
        // 查詢邏輯...
        return Ok();
    }
    
    public IActionResult GetUserByEmail(string email)
    {
        if (email is null)
        {
            return BadRequest("電子郵件不能為空");
        }
        
        // 查詢邏輯...
        return Ok();
    }
}
```

---

## 🏆 實戰範例與最佳實踐

### 🎯 綜合範例：訂單處理系統

```csharp
public class OrderService
{
    public class Order
    {
        public int Id { get; set; }
        public DateTime? ShippedDate { get; set; }
        public decimal? DiscountAmount { get; set; }
        public int? CustomerId { get; set; }
    }
    
    public string ProcessOrder(Order order, DateTime? preferredShipDate = null)
    {
        // ✅ 使用 is null 進行參考檢查
        if (order is null)
        {
            throw new ArgumentNullException(nameof(order), "訂單不能為 null");
        }
        
        var result = new List<string>();
        
        // ✅ 使用 HasValue 檢查可空值型別
        if (order.ShippedDate.HasValue)
        {
            var daysSinceShipped = (DateTime.Now - order.ShippedDate.Value).Days;
            result.Add($"已出貨 {daysSinceShipped} 天");
        }
        else
        {
            // ✅ 使用 GetValueOrDefault 提供預設值
            var shipDate = preferredShipDate.GetValueOrDefault(DateTime.Now.AddDays(1));
            result.Add($"預計出貨日期：{shipDate:yyyy-MM-dd}");
        }
        
        // ✅ 處理折扣資訊
        var discount = order.DiscountAmount.GetValueOrDefault(0m);
        if (discount > 0)
        {
            result.Add($"折扣金額：${discount:F2}");
        }
        
        // ✅ 客戶資訊處理
        if (order.CustomerId.HasValue)
        {
            result.Add($"客戶 ID：{order.CustomerId.Value}");
        }
        else
        {
            result.Add("匿名訂單");
        }
        
        return string.Join(" | ", result);
    }
}

// 使用範例
var orderService = new OrderService();

var order1 = new OrderService.Order
{
    Id = 1,
    CustomerId = 123,
    DiscountAmount = 10.50m
};

var order2 = new OrderService.Order
{
    Id = 2,
    ShippedDate = DateTime.Now.AddDays(-3)
};

Console.WriteLine(orderService.ProcessOrder(order1));
// 輸出：預計出貨日期：2025-07-13 | 折扣金額：$10.50 | 客戶 ID：123

Console.WriteLine(orderService.ProcessOrder(order2, DateTime.Now.AddDays(1)));
// 輸出：已出貨 3 天 | 匿名訂單
```

### 🎨 最佳實踐總結

#### ✅ DO (推薦做法)
```csharp
// 1. 使用 is null 進行純粹的 null 檢查
if (obj is null) { /* ... */ }

// 2. 使用 HasValue 檢查 Nullable<T>
if (nullableValue.HasValue) { /* ... */ }

// 3. 使用 GetValueOrDefault 提供安全的預設值
var safeValue = nullableValue.GetValueOrDefault(defaultValue);

// 4. 在方法參數中直接設定預設值
public void Method(int? param = null) { /* ... */ }

// 5. 提供清晰的 null 處理邏輯
public string ProcessData(string input)
{
    if (input is null)
    {
        return "無資料";
    }
    
    return input.Trim().ToUpper();
}
```

#### ❌ DON'T (避免的做法)
```csharp
// 1. 避免直接存取 .Value 而不檢查
var value = nullableInt.Value; // 可能拋出例外

// 2. 避免過度依賴 == null（特別是自定類別）
if (customObject == null) { /* 可能不準確 */ }

// 3. 避免混淆的方法多載
public void Process(int? id) { }
public void Process(string name) { } // 容易混淆

// 4. 避免忽略 null 檢查
public void UnsafeMethod(object obj)
{
    obj.ToString(); // 危險！沒有 null 檢查
}
```

---

## 🎓 總結

掌握 C# 的 null 處理是寫出安全、優雅程式碼的關鍵技能：

1. **Nullable&lt;T&gt;** 讓值型別擁抱 null 的可能性
2. **HasValue** 提供清晰的值檢查語意
3. **GetValueOrDefault** 確保安全的值存取
4. **is null** 保證純粹的參考比較
5. **方法多載** 需要理解編譯器的選擇邏輯

記住：**防禦性程式設計始於良好的 null 處理！** 🛡️

---

> 💡 **小貼士**: 現代 C# 還提供了 null-conditional operators (`?.` 和 `?[]`) 以及 null-coalescing operators (`??` 和 `??=`)，讓 null 處理更加簡潔優雅！

*最後更新：2025年7月12日*
