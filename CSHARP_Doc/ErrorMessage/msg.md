# C# 錯誤訊息指南

## 目錄
- [1. Sequence contains no elements](#1-sequence-contains-no-elements)

---

## 1. Sequence contains no elements

是 .NET 中 LINQ 常見的一個例外（Exception），通常發生在你對一個「空集合」進行不該對空集合做的操作。

### 🚨 **常見發生場景**

| 方法名稱              | 錯誤條件               |
| ----------------- | ------------------ |
| `First()`         | 空集合時沒有第一個元素        |
| `Last()`          | 空集合時沒有最後一個元素       |
| `Single()`        | 空集合時沒有唯一元素，或超過一個元素 |
| `Max()` / `Min()` | 空集合沒辦法比較最大最小值      |

### 💥 **錯誤範例**

```csharp
var numbers = new List<int>(); // 空集合

// ❌ 這些操作會拋出 "Sequence contains no elements" 異常
int first = numbers.First();        // InvalidOperationException
int last = numbers.Last();          // InvalidOperationException  
int single = numbers.Single();      // InvalidOperationException
int max = numbers.Max();            // InvalidOperationException
int min = numbers.Min();            // InvalidOperationException

// ❌ 篩選後變成空集合也會發生
var users = new List<User>();
var adminUser = users.Where(u => u.Role == "Admin").First(); // 如果沒有 Admin 就會出錯
```

### ✅ **解決方案**

**方案一：使用安全版本的方法**
```csharp
var numbers = new List<int>();

// ✅ 使用 OrDefault 版本，空集合時回傳預設值
int first = numbers.FirstOrDefault();        // 回傳 0（int 的預設值）
int last = numbers.LastOrDefault();          // 回傳 0
int single = numbers.SingleOrDefault();      // 回傳 0

// ✅ 對於 Max/Min，可以檢查集合是否為空
int max = numbers.Any() ? numbers.Max() : 0;
int min = numbers.Any() ? numbers.Min() : 0;

// ✅ 或使用 DefaultIfEmpty
int maxSafe = numbers.DefaultIfEmpty().Max(); // 如果空集合，會使用預設值 0
```

**方案二：事前檢查**
```csharp
var numbers = new List<int>();

// ✅ 先檢查集合是否有元素
if (numbers.Any())
{
    int first = numbers.First();
    int max = numbers.Max();
    // 安全地使用
}
else
{
    Console.WriteLine("集合是空的，無法取得元素");
}

// ✅ 使用條件運算子
int result = numbers.Any() ? numbers.First() : -1; // 空集合時回傳 -1
```

**方案三：例外處理**
```csharp
var numbers = new List<int>();

try
{
    int first = numbers.First();
    Console.WriteLine($"第一個元素: {first}");
}
catch (InvalidOperationException)
{
    Console.WriteLine("集合是空的，沒有第一個元素");
}
```

### 🎯 **實際應用範例**

**範例一：用戶查詢**
```csharp
// ❌ 危險寫法
public User GetAdminUser()
{
    return _users.Where(u => u.Role == "Admin").First(); // 可能沒有 Admin
}

// ✅ 安全寫法
public User GetAdminUser()
{
    return _users.Where(u => u.Role == "Admin").FirstOrDefault();
}

// ✅ 更好的寫法（含null檢查）
public User GetAdminUser()
{
    var admin = _users.Where(u => u.Role == "Admin").FirstOrDefault();
    if (admin == null)
    {
        throw new InvalidOperationException("系統中沒有找到管理員帳戶");
    }
    return admin;
}
```

**範例二：統計計算**
```csharp
// ❌ 危險寫法
public decimal GetAverageOrderAmount()
{
    return _orders.Where(o => o.Status == "Completed").Average(o => o.Amount);
}

// ✅ 安全寫法
public decimal GetAverageOrderAmount()
{
    var completedOrders = _orders.Where(o => o.Status == "Completed");
    return completedOrders.Any() ? completedOrders.Average(o => o.Amount) : 0;
}

// ✅ 使用 DefaultIfEmpty
public decimal GetAverageOrderAmount()
{
    return _orders
        .Where(o => o.Status == "Completed")
        .DefaultIfEmpty()
        .Average(o => o?.Amount ?? 0);
}
```

**範例三：最大最小值查詢**
```csharp
// ❌ 危險寫法
public DateTime GetLatestOrderDate()
{
    return _orders.Max(o => o.OrderDate);
}

// ✅ 安全寫法
public DateTime? GetLatestOrderDate()
{
    return _orders.Any() ? _orders.Max(o => o.OrderDate) : (DateTime?)null;
}

// ✅ 使用預設值
public DateTime GetLatestOrderDate()
{
    return _orders.Any() ? _orders.Max(o => o.OrderDate) : DateTime.MinValue;
}
```

### 🏆 **最佳實踐建議**

1. **優先使用 OrDefault 方法**：`FirstOrDefault()`, `LastOrDefault()`, `SingleOrDefault()`
2. **事前檢查**：在使用 `Max()`, `Min()`, `Average()` 前先用 `Any()` 檢查
3. **使用 DefaultIfEmpty()**：為空集合提供預設值
4. **回傳可空型別**：讓呼叫端知道可能沒有結果
5. **適當的例外處理**：在業務邏輯中處理空集合的情況

### 🔍 **除錯技巧**

```csharp
// 除錯時可以這樣檢查
var query = _users.Where(u => u.Role == "Admin");
Console.WriteLine($"找到 {query.Count()} 個管理員"); // 先看看有幾個結果

if (query.Any())
{
    var admin = query.First(); // 確定有結果後再取第一個
}
```

> **🌟 重點提醒**
> 
> - **預防勝於治療**：使用 `OrDefault` 方法比捕捉例外更有效率
> - **明確意圖**：如果業務邏輯上不允許空集合，使用例外處理讓錯誤更明顯
> - **性能考量**：`Any()` 比 `Count() > 0` 更有效率
> - **可空型別**：善用 `?` 來表示可能沒有結果的情況