# C# 程式碼品質提升指南

## 目錄
  - [1.1 抽取共用驗證邏輯](#11-抽取共用驗證邏輯)
  - [1.2 關於靜態與實例方法](#12-關於靜態與實例方法)
    - [1.2.1 舉例 Substring() vs char.ToUpper()](#121-舉例-substring-vs-chartoupper)
  - [1.3 適時的考慮把想法包成物件](#13-適時的考慮把想法包成物件)
    - [1.3.1 參與者物件範例](#131-參與者物件範例)
  - [1.4 自製管理器,可註冊可執行](#14-自製管理器可註冊可執行)
    - [1.4.1 Plugin 管理器範例](#141-plugin-管理器範例)
  - [1.5 考慮使用泛型](#15-考慮使用泛型)
    - [1.5.1 泛型計算器範例](#151-泛型計算器範例)
    - [1.5.2 Wrapper](#152-wrapper)
    - [1.5.3 泛型與快取](#153-泛型與快取)
    - [1.5.4 泛型快取 - 進階版](#154-泛型快取-進階版)
    - [1.5.5 快取反序列化泛型](#155-快取反序列化泛型)
  - [1.6 實體與介面](#16-實體與介面)
    - [1.6.1 介面變數與實體物件](#161-介面變數與實體物件)
    - [1.6.2 GetType() 與型別判斷](#162-gettype-與型別判斷)
  - [1.7 Closure 閉包](#17-closure-閉包)
    - [1.7.1 延後執行的陷阱](#171-延後執行的陷阱)
    - [1.7.2 點擊次數 Counter](#172-點擊次數-counter)
    - [1.7.3 記住外部變數的威力](#173-記住外部變數的威力)
  - [1.8 count / flag / batch + while loop 使用](#18-count--flag--batch--while-loop-使用)
    - [1.8.1 基本計數與標記模式](#181-基本計數與標記模式)
    - [1.8.2 批次處理模式](#182-批次處理模式)
    - [1.8.3 複合條件控制](#183-複合條件控制)
---

### 1.1 抽取共用驗證邏輯9

在促銷引擎的開發中，常常需要對「指定商品類型」和「排除商品類型」進行相同的驗證邏輯。如果每次都重複寫相同的驗證程式碼，會造成：

透過建立共用的驗證函式 `ValidateSalePageTypeList`，我們可以優雅地解決上述問題：

```csharp
/// <summary>
/// 指定排除類型 / 指定排除商品 對應驗證
/// </summary>
/// <param name="entity">The entity.</param>
/// <returns></returns>
private ValidationFailure ValidateTargetTypeProductList(PromotionEngineBaseEntity entity)
{
    return ValidateSalePageTypeList(
        entity.TargetTypeDef, 
        entity.IncludeTargetIdList, 
        entity.IncludeProductSkuOuterIdList, 
        nameof(PromotionEngineBaseEntity.TargetTypeDef), 
        "指定商品類型, 請指定商品或料號")
        ?? ValidateSalePageTypeList(
        entity.ExcludeTargetTypeDef, 
        entity.ExcludeTargetIdList, 
        entity.ExcludeProductSkuOuterIdList, 
        nameof(PromotionEngineBaseEntity.ExcludeTargetTypeDef), 
        "排除商品類型, 請排除商品或料號");
}

/// <summary>
/// Validates the type list.
/// </summary>
/// <param name="typeDef">The type definition.</param>
/// <param name="targetIdList">The target identifier list.</param>
/// <param name="productSkuOuterIdList">The product sku outer identifier list.</param>
/// <param name="propertyName">Name of the property.</param>
/// <param name="errorMessage">The error message.</param>
/// <returns> ValidationFailure </returns>
private ValidationFailure ValidateSalePageTypeList(
    string typeDef, 
    List<long> targetIdList, 
    List<string> productSkuOuterIdList, 
    string propertyName, 
    string errorMessage)
{
    var noTargetIdSelected = targetIdList == null || targetIdList.Any() == false;
    var noProductSkuOuterIdSelected = productSkuOuterIdList == null || productSkuOuterIdList.Any() == false;
    
    if (typeDef == nameof(PromotionEngineTargetTypeDefEnum.SalePage) && 
        noTargetIdSelected && 
        noProductSkuOuterIdSelected)
    {
        return new ValidationFailure(propertyName, errorMessage);
    }

    return null;
}
```

### 1.2 關於靜態與實例方法

在 C# 程式開發中，理解靜態方法與實例方法的差異是非常重要的。這兩種方法的呼叫方式和設計理念完全不同，讓我們透過具體範例來深入理解。

#### 1.2.1 舉例 Substring() vs char.ToUpper()

透過比較 `Substring()` 和 `char.ToUpper()` 這兩個常用方法，我們可以清楚看出靜態與實例方法的本質差異。

##### 🔍 Substring() - 實例方法範例

`Substring()` 定義在 `System.String` class 裡面（實際型別是 `sealed class String`）

**原始方法定義（.NET BCL）：**
```csharp
namespace System
{
    public sealed class String : IComparable, ICloneable, IEnumerable<char>
    {
        public string Substring(int startIndex);
        public string Substring(int startIndex, int length);
        // 其他方法 ...
    }
}
```

**呼叫方式：**
```csharp
string name = "hello";
var part = name.Substring(1);  // 這裡是「name 這個物件」自己執行 Substring
Console.WriteLine(part);       // 輸出：ello
```

> **💡 重點**：`Substring()` 是「字串物件自己的方法」，呼叫形式是 `物件.方法()`。

##### ⚡ char.ToUpper() - 靜態方法範例

`char.ToUpper()` 定義在 `System.Char` struct 裡面（型別是 `public readonly struct Char`）

**原始方法定義（.NET BCL）：**
```csharp
namespace System
{
    public readonly struct Char : IComparable, IEquatable<char>
    {
        public static char ToUpper(char c);
        public static char ToUpper(char c, CultureInfo culture);
        // 其他方法 ...
    }
}
```

**呼叫方式：**
```csharp
char c = 'a';
char upper = char.ToUpper(c); // 這裡是用「類別名」呼叫靜態方法
Console.WriteLine(upper);     // 輸出：A
```

> **💡 重點**：`ToUpper()` 是一個靜態方法 (static)，呼叫形式是 `類別.方法()`，而不是靠物件。

##### 📊 兩者差異重點整理

| 方法名稱 | 定義類別 | 方法型態 | 呼叫方式 | 記憶體配置 |
|----------|----------|----------|----------|------------|
| `Substring()` | `System.String` | 實例方法 | `myString.Substring()` | 需要物件實例 |
| `ToUpper()` | `System.Char` | 靜態方法 | `char.ToUpper(myChar)` | 不需要物件實例 |

##### 🤔 為什麼呼叫方式看起來不一樣？

**實例方法：**
- 必須先有一個「物件」(string)，方法才能用
- 方法會用到這個物件內的資料
- 呼叫時透過 `物件.方法()` 的形式

```csharp
string text = "Hello World";
// text 物件包含了字元陣列資料
string result = text.Substring(6); // 方法需要存取 text 物件的內部資料
```

**靜態方法：**
- 不需要物件，方法跟類別本身綁在一起
- 只要給參數即可，不依賴任何物件的狀態
- 呼叫時透過 `類別.方法()` 的形式

```csharp
char letter = 'a';
// ToUpper 不需要依賴任何物件的狀態，純粹是功能性的轉換
char upperLetter = char.ToUpper(letter); // 只是一個純函式轉換
```

### 1.3 適時的考慮把想法包成物件

以下範例展示如何將相關的概念（參與者的名稱和時區）包裝成一個有意義的物件：

```csharp
void Main()
{
    var participants = new List<Participant>
    {
        new Participant("倫敦總部", TimeZoneInfo.FindSystemTimeZoneById("GMT Standard Time")),
        new Participant("Tokyo 分公司", TimeZoneInfo.FindSystemTimeZoneById("Tokyo Standard Time")),
        new Participant("Eastern 分公司", TimeZoneInfo.FindSystemTimeZoneById("Eastern Standard Time")),
        new Participant("AUS 分公司", TimeZoneInfo.FindSystemTimeZoneById("AUS Eastern Standard Time"))
    };

    // 示範印出每個參與者的名稱和時區
    foreach (var p in participants)
    {
        Console.WriteLine($"{p.Name} - {p.TimeZoneInfo.DisplayName}");
    }
}

public class Participant
{
    public string Name { get; }
    public TimeZoneInfo TimeZoneInfo { get; }

    public Participant(string name, TimeZoneInfo timeZoneInfo)
    {
        Name = name;
        TimeZoneInfo = timeZoneInfo;
    }
}
```

### 1.4 自製管理器,可註冊可執行

#### 1.4.1 Plugin 管理器範例

以下範例展示如何建立一個簡單但功能完整的 Plugin 管理器：

```csharp
void Main()
{
    var manager = new PluginManager();
    manager.RegisterPlugin("burnout", () => Console.WriteLine("Burnout..."));
    manager.RegisterPlugin("tired", () => Console.WriteLine("tired..."));
    manager.ExecutePlugin("burnout");
    manager.ExecutePlugin("tired");
    manager.ListPlugins();
}

public class PluginManager
{
    Dictionary<string, Action> plugins = new Dictionary<string, Action>();
    
    public void RegisterPlugin(string name, Action action)
    {
        if (plugins.ContainsKey(name) == false)
        {
            plugins.Add(name, action);
        }
        else
        {
            Console.WriteLine($"plugin : {name}, 已存在!");
        }
    }
    
    public void ExecutePlugin(string name)
    {
        if (plugins.TryGetValue(name, out var plugin))
        {
            plugin(); // 執行該 plugin
        }
        else
        {
            Console.WriteLine($"找不到名稱為 '{name}' 的 Plugin。");
        }
    }

    public void ListPlugins()
    {
        Console.WriteLine("目前可用的 Plugins：");
        foreach (var name in plugins.Keys)
        {
            Console.WriteLine($"- {name}");
        }
    }
}
```

### 1.5 考慮使用泛型

#### 1.5.1 泛型計算器範例

```csharp
void Main()
{
    // 使用 Lambda 表達式進行計算
    Calculator.Operate<decimal>(1m, 2m, (x,y) => x + y).Dump();  // 輸出：3
    
    // 使用預定義的運算函式字典
    Calculator.Operate<decimal>(1m, 2m, Calculator.OpDict<decimal>()["Add"]).Dump();  // 輸出：3
}

public static class Calculator
{
    /// <summary>
    /// 泛型運算方法，接受任何型別 T 和對應的運算函式
    /// </summary>
    /// <typeparam name="T">數值型別</typeparam>
    /// <param name="a">第一個運算數</param>
    /// <param name="b">第二個運算數</param>
    /// <param name="doSomething">運算函式</param>
    /// <returns>運算結果</returns>
    public static T Operate<T>(T a, T b, Func<T, T, T> doSomething) => doSomething(a, b);
    
    /// <summary>
    /// 提供預定義的運算函式字典，限制 T 必須實作 INumber 介面
    /// </summary>
    /// <typeparam name="T">必須實作 System.Numerics.INumber 的數值型別</typeparam>
    /// <returns>包含基本運算的函式字典</returns>
    public static Dictionary<string, Func<T, T, T>> OpDict<T>() where T : System.Numerics.INumber<T> => new()
    {
        { "Add", (x, y) => x + y },
        { "Sub", (x, y) => x - y },
        { "Mul", (x, y) => x * y },
        { "Div", (x, y) => x / y }
    };
}
```

##### 🎯 泛型的優勢

**型別安全：**
- ✅ 編譯時期檢查型別正確性
- ✅ 避免執行時期的型別轉換錯誤
- ✅ IntelliSense 提供更好的程式碼提示

**效能提升：**
- 🚀 避免裝箱 (Boxing) 和拆箱 (Unboxing) 操作
- 🚀 減少型別轉換的效能損耗
- 🚀 編譯器優化更有效率

**程式碼重用：**
- 🔄 一次編寫，多種型別適用
- 🔄 減少重複程式碼
- 🔄 提高維護性

##### 📊 使用範例比較

**傳統做法（沒有泛型）：**
```csharp
// 需要為每種型別寫不同的方法
public static int AddInt(int a, int b) => a + b;
public static decimal AddDecimal(decimal a, decimal b) => a + b;
public static double AddDouble(double a, double b) => a + b;
public static float AddFloat(float a, float b) => a + b;
```

**泛型做法：**
```csharp
// 一個方法支援所有數值型別
public static T Add<T>(T a, T b) where T : System.Numerics.INumber<T> => a + b;

// 使用方式
var intResult = Add<int>(1, 2);           // 3
var decimalResult = Add<decimal>(1.5m, 2.3m);  // 3.8
var doubleResult = Add<double>(1.1, 2.2);      // 3.3
```

#### 1.5.2 Wrapper

Wrapper 模式是一種常見的設計模式，它可以用來包裝現有的物件，提供額外的功能或改變其行為。在泛型的幫助下，我們可以建立通用的 Wrapper 類別來處理任何型別的物件。

##### 🎯 Wrapper 基本概念

Wrapper 類別的主要目的是：
- 🔐 **封裝原始物件**：提供對底層物件的受控存取
- 🎨 **增強功能**：在不修改原始類別的情況下添加新功能
- 🛡️ **保護資料**：控制對包裝物件的存取方式
- 🔄 **轉換介面**：為不相容的介面提供適配

##### 📝 實際應用範例

以下範例展示了泛型 Wrapper 的使用和一些需要注意的問題：

```csharp
void Main()
{
    var items = new List<int> { 1, 2, 3 };
    var wrappers = CreateWrapper2<int>(items);

    var store = new List<Wrapper<int>>();
    store.AddRange(wrappers);

    // ❌ 這會回傳 false，因為 Wrapper 沒有實作相等性比較
    var storeContainsAnyWrappers = wrappers
        .Any(wrapper => store.Contains(wrapper)).Dump(); // = false	
}

public IEnumerable<Wrapper<T>> CreateWrapper<T>(IEnumerable<T> items)
{
    return items.Select(item => new Wrapper<T>(item));
}

public IEnumerable<Wrapper<T>> CreateWrapper2<T>(IEnumerable<T> items)
{
    return items.Select(item =>
    {
        Console.WriteLine($"Create wrapper for {item}");
        return new Wrapper<T>(item);
    }).ToList();
}

public class Wrapper<T>
{
    private readonly T _item;

    public Wrapper(T item)
    {
        _item = item;
    }
}
```

##### 🔍 問題分析

在上述範例中，`storeContainsAnyWrappers` 會回傳 `false`，這是因為：

**🚨 物件參考比較問題：**
1. `CreateWrapper2` 建立了新的 `Wrapper<T>` 實例
2. `store.AddRange(wrappers)` 將這些實例加入到 store 中
3. `Contains()` 使用預設的參考相等性比較
4. 即使包裝的值相同，但 `Wrapper` 物件是不同的實例

##### ✅ 解決方案

**方案一：實作 IEquatable<T> 介面**
```csharp
public class Wrapper<T> : IEquatable<Wrapper<T>>
{
    private readonly T _item;

    public Wrapper(T item)
    {
        _item = item;
    }

    public T Item => _item;

    public bool Equals(Wrapper<T> other)
    {
        if (other is null) return false;
        if (ReferenceEquals(this, other)) return true;
        return EqualityComparer<T>.Default.Equals(_item, other._item);
    }

    public override bool Equals(object obj)
    {
        return obj is Wrapper<T> wrapper && Equals(wrapper);
    }

    public override int GetHashCode()
    {
        return EqualityComparer<T>.Default.GetHashCode(_item);
    }

    public static bool operator ==(Wrapper<T> left, Wrapper<T> right)
    {
        return EqualityComparer<Wrapper<T>>.Default.Equals(left, right);
    }

    public static bool operator !=(Wrapper<T> left, Wrapper<T> right)
    {
        return !(left == right);
    }
}
```

**方案二：使用記錄類型 (Record)**
```csharp
public record Wrapper<T>(T Item);

// 使用方式
void Main()
{
    var items = new List<int> { 1, 2, 3 };
    var wrappers = items.Select(item => new Wrapper<int>(item)).ToList();
    
    var store = new List<Wrapper<int>>();
    store.AddRange(wrappers);
    
    // ✅ 現在會回傳 true，因為 record 自動實作了相等性比較
    var storeContainsAnyWrappers = wrappers
        .Any(wrapper => store.Contains(wrapper)); // = true
}
```

> **🌟 重點提醒**
> 
> 1. **相等性問題**：Wrapper 類別需要適當實作相等性比較，否則 `Contains()` 等方法可能無法正常工作
> 2. **記錄類型優勢**：C# 9+ 的 record 類型自動提供值相等性，非常適合作為簡單的 Wrapper
> 3. **延遲執行**：注意 LINQ 的延遲執行特性對 Wrapper 建立時機的影響
> 4. **功能擴展**：Wrapper 模式可以用來添加日誌、快取、驗證等橫切關注點

#### 1.5.3 泛型與快取

泛型不僅在計算和包裝上有用，在快取機制中也能發揮強大的作用。透過泛型快取類別，我們可以建立一個通用的快取解決方案，適用於任何型別的資料。

##### 📝 泛型快取範例

```csharp
void Main()
{
    var cache = new SimpleCache<string>();
    var userIntro = cache.GetOrCreate("123", () => DB.GetUserInfo("123"));
    userIntro.Dump();
}

public class SimpleCache<T>
{
    public MemoryCache cache = new MemoryCache(new MemoryCacheOptions());
    
    public T GetOrCreate(string key, Func<T> createItem)
    {
        T cacheEntry;
        if(cache.TryGetValue(key, out cacheEntry) == false)
        {
            cacheEntry = createItem();
            cache.Set(key, cacheEntry);
        }
        
        return cacheEntry;
    }
}

public static class DB
{
    public static string GetUserInfo(string number)
    {
        return "ya";
    }
}
```

##### 🎯 泛型快取的優勢

**型別安全快取：**
- ✅ 確保快取存取的型別正確性
- ✅ 避免型別轉換錯誤
- ✅ 編譯時期檢查型別一致性

**通用性強：**
- 🔄 可快取任何型別的資料（字串、物件、集合等）
- 🔄 一次實作，多種場景適用
- 🔄 提供統一的快取存取介面

**效能提升：**
- 🚀 避免重複的資料庫查詢或計算
- 🚀 減少不必要的物件建立
- 🚀 提供快速的記憶體存取

##### 💡 實際應用場景

**使用者資訊快取：**
```csharp
var userCache = new SimpleCache<UserInfo>();
var user = userCache.GetOrCreate("user_123", () => UserService.GetUserById(123));
```

**計算結果快取：**
```csharp
var calculationCache = new SimpleCache<decimal>();
var result = calculationCache.GetOrCreate("complex_calc_abc", () => PerformComplexCalculation());
```

**查詢結果快取：**
```csharp
var queryCache = new SimpleCache<List<Product>>();
var products = queryCache.GetOrCreate("category_electronics", () => ProductService.GetByCategory("Electronics"));
```

> **🌟 重點提醒**
> 
> 1. **快取策略**：考慮快取過期時間和清理機制
> 2. **記憶體管理**：監控快取大小，避免記憶體洩漏
> 3. **執行緒安全**：在多執行緒環境中確保快取的執行緒安全性
> 4. **資料一致性**：考慮快取更新和失效的策略

#### 1.5.4 泛型快取 - 進階版

在前面的簡單快取範例基礎上，我們可以建立一個更完整的快取解決方案，具備過期時間控制和執行緒安全特性。這個進階版本適合在實際專案中使用。

##### 🔧 **進階快取實作**

以下實作展示了一個具備過期機制和執行緒安全的泛型快取：

```csharp
public static class MemoryHelper
{
    /// <summary>
    /// 執行緒安全的快取字典，儲存所有快取項目
    /// </summary>
    public static ConcurrentDictionary<string, object> dict = new();
    
    /// <summary>
    /// 泛型快取擴展方法，支援過期時間控制
    /// </summary>
    /// <typeparam name="T">快取資料型別</typeparam>
    /// <param name="cacheKey">快取索引鍵</param>
    /// <param name="getData">取得資料的函式</param>
    /// <param name="expiry">快取過期時間</param>
    /// <returns>快取或新建立的資料</returns>
    public static T GetOrSetCache<T>(this string cacheKey, Func<T> getData, TimeSpan expiry)
    {
        // 嘗試從快取中取得資料
        if (dict.TryGetValue(cacheKey, out var obj) && obj is CacheItem<T> item)
        {
            // 檢查快取是否仍然有效
            if (DateTime.Now < item.Expiration)
            {
                Console.WriteLine($"✅ 快取命中: {cacheKey}");
                return item.Data;
            }
            
            // 快取已過期，移除舊資料
            dict.TryRemove(cacheKey, out _);
            Console.WriteLine($"⏰ 快取過期已移除: {cacheKey}");
        }
        
        // 建立新的快取項目
        Console.WriteLine($"🔄 建立新快取: {cacheKey}");
        var newData = getData();
        var newItem = new CacheItem<T>
        {
            Data = newData,
            Expiration = DateTime.Now.Add(expiry)
        };

        dict[cacheKey] = newItem;
        return newData;
    }
}

/// <summary>
/// 泛型快取項目容器
/// </summary>
/// <typeparam name="T">快取資料型別</typeparam>
public class CacheItem<T>
{
    /// <summary>
    /// 快取的資料
    /// </summary>
    public T Data { get; set; }
    
    /// <summary>
    /// 快取過期時間
    /// </summary>
    public DateTime Expiration { get; set; }
}
```

##### 📝 **使用範例**

**基本使用方式：**
```csharp
void Main()
{
    // 快取使用者資訊，5 分鐘過期
    var userInfo = "user_123".GetOrSetCache(() =>
    {
        Console.WriteLine("正在從資料庫查詢使用者資訊...");
        Thread.Sleep(1000); // 模擬資料庫查詢延遲
        return new UserInfo { Id = 123, Name = "Allen", Email = "allen@example.com" };
    }, TimeSpan.FromMinutes(5));
    
    Console.WriteLine($"取得使用者: {userInfo.Name}");
    
    // 再次呼叫，應該會使用快取
    var cachedUserInfo = "user_123".GetOrSetCache(() =>
    {
        Console.WriteLine("這行不應該出現，因為應該使用快取");
        return new UserInfo();
    }, TimeSpan.FromMinutes(5));
    
    Console.WriteLine($"快取的使用者: {cachedUserInfo.Name}");
}

public class UserInfo
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}
```

**複雜資料快取範例：**
```csharp
void Main()
{
    // 快取產品清單，10 分鐘過期
    var products = "products_electronics".GetOrSetCache(() =>
    {
        Console.WriteLine("正在查詢電子產品清單...");
        return new List<Product>
        {
            new() { Id = 1, Name = "iPhone 15", Price = 32900 },
            new() { Id = 2, Name = "MacBook Pro", Price = 89900 },
            new() { Id = 3, Name = "iPad Air", Price = 19900 }
        };
    }, TimeSpan.FromMinutes(10));
    
    Console.WriteLine($"取得 {products.Count} 個產品");
    
    // 快取計算結果，1 小時過期
    var expensiveCalculation = "complex_formula_abc".GetOrSetCache(() =>
    {
        Console.WriteLine("執行複雜計算...");
        Thread.Sleep(2000); // 模擬複雜計算
        return Math.PI * Math.E * 12345.67m;
    }, TimeSpan.FromHours(1));
    
    Console.WriteLine($"計算結果: {expensiveCalculation:F4}");
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}
```

**短期快取範例：**
```csharp
void Main()
{
    Console.WriteLine("測試短期快取過期機制：");
    
    // 設定 3 秒過期的快取
    var shortCache = "short_term_data".GetOrSetCache(() =>
    {
        return $"建立時間: {DateTime.Now:HH:mm:ss}";
    }, TimeSpan.FromSeconds(3));
    
    Console.WriteLine($"第一次取得: {shortCache}");
    
    // 立即再次取得（應該使用快取）
    var cachedData = "short_term_data".GetOrSetCache(() =>
    {
        return $"新建立時間: {DateTime.Now:HH:mm:ss}";
    }, TimeSpan.FromSeconds(3));
    
    Console.WriteLine($"快取取得: {cachedData}");
    
    // 等待 4 秒後再次取得（快取應該已過期）
    Thread.Sleep(4000);
    var expiredData = "short_term_data".GetOrSetCache(() =>
    {
        return $"過期後建立: {DateTime.Now:HH:mm:ss}";
    }, TimeSpan.FromSeconds(3));
    
    Console.WriteLine($"過期後取得: {expiredData}");
}
```

##### 🎯 **進階功能特性**

**執行緒安全性：**
- ✅ **ConcurrentDictionary**：提供執行緒安全的字典操作
- ✅ **原子性操作**：`TryGetValue` 和 `TryRemove` 都是原子性的
- ✅ **無鎖設計**：避免鎖定造成的效能瓶頸

**過期機制：**
- ⏰ **自動過期檢查**：每次存取時都會檢查是否過期
- 🗑️ **自動清理**：過期的項目會自動從快取中移除
- 🔄 **智慧更新**：過期後會自動執行 getData 函式取得新資料

**型別安全：**
- 🔒 **強型別檢查**：確保快取和取出的資料型別一致
- 🛡️ **型別轉換安全**：使用 `is` 模式匹配避免轉型錯誤
- ✨ **泛型支援**：支援任何型別的資料快取

##### 🚀 **效能最佳化建議**

**快取索引鍵設計：**
```csharp
// ✅ 良好的索引鍵命名
"user_profile_123"
"product_list_category_electronics"
"weather_data_taipei_2024-08-13"

// ❌ 避免的索引鍵設計
"data"
"temp"
"cache1"
```

**適當的過期時間：**
```csharp
// 根據資料特性設定合適的過期時間
TimeSpan.FromMinutes(5)    // 使用者會話資料
TimeSpan.FromHours(1)      // 產品資訊
TimeSpan.FromDays(1)       // 靜態配置資料
TimeSpan.FromSeconds(30)   // 即時資料
```

**記憶體管理：**
```csharp
// 定期清理過期項目（可選的背景任務）
public static void CleanupExpiredItems()
{
    var keysToRemove = new List<string>();
    
    foreach (var kvp in MemoryHelper.dict)
    {
        if (kvp.Value is CacheItem<object> item && DateTime.Now >= item.Expiration)
        {
            keysToRemove.Add(kvp.Key);
        }
    }
    
    foreach (var key in keysToRemove)
    {
        MemoryHelper.dict.TryRemove(key, out _);
    }
    
    Console.WriteLine($"清理了 {keysToRemove.Count} 個過期快取項目");
}
```

##### ⚠️ **使用注意事項**

**記憶體考量：**
- 📊 監控快取大小，避免記憶體使用過多
- 🔄 考慮實作 LRU（最近最少使用）清理策略
- 🗑️ 定期清理過期項目，釋放記憶體

**併發考量：**
- 🔒 雖然 `ConcurrentDictionary` 是執行緒安全的，但 `getData()` 函式可能不是
- ⚡ 避免在 `getData()` 中執行長時間的操作
- 🎯 考慮使用 `SemaphoreSlim` 來控制併發的 `getData()` 呼叫

**快取策略：**
```csharp
// 考慮根據不同場景使用不同的快取策略
public enum CacheStrategy
{
    WriteThrough,    // 寫穿透
    WriteBack,       // 寫回
    WriteAround      // 寫繞過
}
```

> **🌟 重點提醒**
> 
> 1. **執行緒安全**：`ConcurrentDictionary` 提供基本的執行緒安全，但要注意 `getData()` 函式的執行緒安全性
> 2. **記憶體管理**：定期監控和清理快取，避免記憶體洩漏
> 3. **過期策略**：根據資料特性設定合適的過期時間，平衡效能和資料新鮮度
> 4. **錯誤處理**：在 `getData()` 函式中加入適當的錯誤處理機制

#### 1.5.5 快取反序列化泛型

在前面的快取範例基礎上，我們經常需要處理物件的序列化和反序列化。建立一個專門處理 Redis 快取並支援泛型反序列化的服務，可以讓快取操作更加簡潔和型別安全。

##### 🎯 **ICacheService 介面設計**

```csharp
public interface ICacheService
{
    T Get<T>(string cacheKey);
    bool Set<T>(string cacheKey, T value, DateTimeOffset expirationTime);
    object Remove(string cacheKey);
}
```

##### 🔧 **RedisCacheService 實作**

以下實作展示了一個完整的 Redis 快取服務，具備泛型支援和自動序列化功能：

```csharp
public class RedisCacheService : ICacheService
{
    private readonly IDatabase _database;

    public RedisCacheService()
    {
        var redis = ConnectionMultiplexer.Connect("localhost:6379");
        this._database = redis.GetDatabase();
    }

    public T Get<T>(string cacheKey)
    {
        var value = this._database.StringGet(cacheKey);
        if (string.IsNullOrEmpty(value) == false)
        {
            return JsonSerializer.Deserialize<T>(value);
        }

        return default;
    }

    public object Remove(string cacheKey)
    {
        var exists = this._database.KeyExists(cacheKey);
        if (exists)
            return this._database.KeyDelete(cacheKey);

        return false;
    }

    public bool Set<T>(string cacheKey, T value, DateTimeOffset expirationTime)
    {
        var expirtyTime = expirationTime.DateTime.Subtract(DateTime.Now);
        return this._database.StringSet(cacheKey, JsonSerializer.Serialize(value), expirtyTime);
    }
}
```

##### 📝 **使用範例**

**基本物件快取：**
```csharp
[HttpGet("drivers")]
public async Task<IActionResult> Get()
{
		var cacheData = _cacheService.GetData<IEnumerable<Driver>>("drivers");

		if(cacheData != null && cacheData.Count() > 0)  // prevent undefined and null may be deem as an object that > 0
				return Ok(cacheData);

		cacheData =  await _DbContext.Drivers.ToListAsync();

		var expiryDate = DataTimeOffset.DataTime.AddSeconds(30)

		_cacheService.SetData<IEnumarable<Driver>>("drivers",cacheData,expiryDate);

		return Ok(cacheData)
		
}


[HttpPost("AddDriver")]

public async Task<IActionResult> Post(Driver driver)
{
		var AddedObj = _DbContext.Drivers.AddASync(driver);
		
		var expiryTime = DataTimeOffset.DateTime.AddSeconds(30);

		_cacheService.SetData<Driver>($"driver{driver.Id}",AddObj.Entity,expiryTime)

		await _context.SaveChangesAsync();

		return Ok(Addobj.Entity);
}


[HttpDelete("DeleteDriver")]

public async Task<IActionResult> Delete(int id)
{
		var existEntity = _DbContext.Drivers.FirstOrDefault(x => x.id == id)

		if(existEntity != null)
		{
				_context.Driver.Remove(existEntity);
				_cacheService.RemoveData($"driver{id}");

				_context.SaveChanges()

				return NoContent();
		}

		return NotFound();
}
---

### 1.6 實體與介面

在 C# 程式設計中，理解介面變數與實體物件之間的關係是非常重要的。介面提供了抽象層，讓我們可以透過相同的介面來操作不同的實作類別。

#### 1.6.1 介面變數與實體物件

##### 📝 基本範例

以下範例展示了介面變數如何指向實體物件：

```csharp
void Main()
{
    IPaymentMiddlewareHttpClient test = new PaymentMiddlewareHttpClient();
    test.GetType().Name.Dump(); // 輸出：PaymentMiddlewareHttpClient
}

public class PaymentMiddlewareHttpClient : IPaymentMiddlewareHttpClient
{
    public string GetRequestId()
    {
        throw new NotImplementedException();
    }
}

public interface IPaymentMiddlewareHttpClient
{
    /// <summary>
    /// 取得 Payment Middleware Request ID
    /// </summary>
    /// <returns>Request ID</returns>
    string GetRequestId();
}
```

#### 1.6.2 GetType() 與型別判斷

##### 🔍 核心概念理解

在上述範例中，`test` 是一個「指向介面型別」的變數，但變數裡面實際放的物件是 `PaymentMiddlewareHttpClient`。

**重要觀念：**
- 🎯 **變數型別**：`test` 的宣告型別是 `IPaymentMiddlewareHttpClient`（介面）
- 🏗️ **物件型別**：實際建立的物件是 `PaymentMiddlewareHttpClient`（實作類別）
- ⚡ **執行期行為**：方法呼叫時，會使用實際物件的實作，不是介面

##### 💡 為什麼 GetType() 回傳實作類別？

因為執行時要知道「這個物件的實際類型是誰」，真正決定方法邏輯的是物件本身的類型（實作類別），不是介面。

`GetType()` 是 `object` 類別的方法，用來在執行期取得物件的真實型別。它不會理會變數表面宣告型別，因為執行期需要知道真正的類別才能調用方法、執行程式邏輯。

##### 📊 型別檢查比較表

| 檢查方式 | 檢查對象 | 結果 | 說明 |
|----------|----------|------|------|
| `test.GetType()` | 實際物件型別 | `PaymentMiddlewareHttpClient` | 執行期的真實型別 |
| `typeof(IPaymentMiddlewareHttpClient)` | 介面型別 | `IPaymentMiddlewareHttpClient` | 編譯期的介面型別 |
| `test is IPaymentMiddlewareHttpClient` | 型別相容性檢查 | `true` | 物件是否實作該介面 |
| `test is PaymentMiddlewareHttpClient` | 具體型別檢查 | `true` | 物件是否為該具體類別 |

##### 🔄 多型與介面的實際應用

```csharp
void Main()
{
    // 建立不同的實作
    IPaymentMiddlewareHttpClient client1 = new PaymentMiddlewareHttpClient();
    IPaymentMiddlewareHttpClient client2 = new MockPaymentMiddlewareHttpClient();
    IPaymentMiddlewareHttpClient client3 = new TestPaymentMiddlewareHttpClient();
    
    // 雖然都宣告為介面型別，但 GetType() 會顯示實際類別
    Console.WriteLine(client1.GetType().Name); // PaymentMiddlewareHttpClient
    Console.WriteLine(client2.GetType().Name); // MockPaymentMiddlewareHttpClient
    Console.WriteLine(client3.GetType().Name); // TestPaymentMiddlewareHttpClient
    
    // 統一處理，但各自執行不同的實作
    ProcessPayment(client1);
    ProcessPayment(client2);
    ProcessPayment(client3);
}

void ProcessPayment(IPaymentMiddlewareHttpClient client)
{
    // 這裡會根據實際物件型別呼叫對應的方法實作
    var requestId = client.GetRequestId();
    Console.WriteLine($"處理付款，Request ID: {requestId}");
}

// 不同的實作類別
public class PaymentMiddlewareHttpClient : IPaymentMiddlewareHttpClient
{
    public string GetRequestId()
    {
        return "PROD-" + Guid.NewGuid().ToString("N")[..8];
    }
}

public class MockPaymentMiddlewareHttpClient : IPaymentMiddlewareHttpClient
{
    public string GetRequestId()
    {
        return "MOCK-12345678";
    }
}

public class TestPaymentMiddlewareHttpClient : IPaymentMiddlewareHttpClient
{
    public string GetRequestId()
    {
        return "TEST-" + DateTime.Now.Ticks.ToString()[..8];
    }
}
```

##### 🎯 實務應用場景

**場景一：依賴注入與型別檢查**
```csharp
void Main()
{
    // 模擬依賴注入容器返回的物件
    IPaymentMiddlewareHttpClient injectedClient = GetPaymentClient();
    
    // 檢查實際注入的是哪一個實作
    Console.WriteLine($"注入的實作類別: {injectedClient.GetType().Name}");
    
    // 根據實際型別進行特殊處理
    if (injectedClient.GetType() == typeof(MockPaymentMiddlewareHttpClient))
    {
        Console.WriteLine("偵測到 Mock 實作，啟用測試模式");
    }
}

IPaymentMiddlewareHttpClient GetPaymentClient()
{
    // 在實際系統中，這可能由 DI 容器決定
    return new PaymentMiddlewareHttpClient();
}
```

**場景二：介面轉型與型別安全**
```csharp
void Main()
{
    IPaymentMiddlewareHttpClient client = new PaymentMiddlewareHttpClient();
    
    // 安全的型別轉換
    if (client is PaymentMiddlewareHttpClient concreteClient)
    {
        // 可以存取具體類別的額外方法
        Console.WriteLine("成功轉型為具體實作");
    }
    
    // 使用模式匹配進行型別檢查
    var result = client switch
    {
        PaymentMiddlewareHttpClient => "正式環境客戶端",
        MockPaymentMiddlewareHttpClient => "測試環境客戶端",
        TestPaymentMiddlewareHttpClient => "開發環境客戶端",
        _ => "未知的客戶端實作"
    };
    
    Console.WriteLine($"客戶端類型: {result}");
}
```

**場景三：除錯和日誌記錄**
```csharp
public class PaymentService
{
    private readonly IPaymentMiddlewareHttpClient _client;
    
    public PaymentService(IPaymentMiddlewareHttpClient client)
    {
        _client = client;
        
        // 記錄實際注入的實作類別，有助於除錯
        Console.WriteLine($"PaymentService 使用的實作: {client.GetType().FullName}");
    }
    
    public void ProcessPayment()
    {
        try
        {
            var requestId = _client.GetRequestId();
            Console.WriteLine($"付款處理完成，ID: {requestId}");
        }
        catch (Exception ex)
        {
            // 在錯誤日誌中包含實際的實作類別資訊
            Console.WriteLine($"付款處理失敗 - 實作類別: {_client.GetType().Name}, 錯誤: {ex.Message}");
        }
    }
}
```

> **🌟 重點提醒**
> 
> 1. **介面 vs 實作**：介面定義契約，實作類別提供具體邏輯，`GetType()` 永遠回傳實際物件的型別
> 2. **多型的威力**：相同介面可以有不同實作，讓程式具有高度彈性和可擴展性
> 3. **型別檢查工具**：善用 `GetType()`、`typeof()`、`is` 和 `as` 來進行型別檢查和轉換
> 4. **除錯友善**：在日誌中記錄實際型別資訊，有助於問題排查和系統監控

### 1.7 Closure 閉包

Closure 就是 lambda 把用到的外部變數「包起來、記住」的能力。

#### 1.7.1 延後執行的陷阱

##### 🎯 核心概念理解

**Lambda 是「延後執行」的，而方法參數是「當下就執行」的。**

- **方法呼叫（傳參數）** → 立刻執行 → 當下的值是什麼就拿什麼
- **Lambda（或閉包）** → 先存下來、之後才執行 → 那時變數可能早就變了！

##### 📝 案例探討

**方法傳參數 = 「立刻做這件事」**

```csharp
void Show(int n) 
{
    Console.WriteLine(n);
}

for (int i = 0; i < 5; i++)
{
    Show(i); // 當下的 i 是多少就傳進去
}
```

**執行結果：**
```
0
1
2
3
4
```

這裡是「現在馬上」把 i 的值傳進 `Show()`，所以每次呼叫都是當下的值。

**Lambda = 「先記下做法，之後才做」**

```csharp
var actions = new List<Action>();

for (int i = 0; i < 5; i++)
{
    actions.Add(() => Console.WriteLine(i)); // 只是記下「到時候要印 i」, 這時候把 i 參考包近來
}

actions.ForEach(a => a.Invoke()); // 現在才真正做, 此時的 i 已經都是 5
```

**執行結果：**
```
5
5
5
5
5
```

> **🚨 問題分析**：Lambda 表達式捕獲的是變數 `i` 的**參考**，而不是值的副本。當 Lambda 實際執行時，`i` 的值已經是迴圈結束後的 5。

##### ✅ 解決方案

**在 Lambda 裡面用 loop 變數，要複製一份出來再用！**

```csharp
var actions = new List<Action>();

for (int i = 0; i < 5; i++)
{
    int copy = i; // 建立區域變數副本
    actions.Add(() => Console.WriteLine(copy)); // 捕獲 copy 而不是 i, 每次的參考都不一樣
}

actions.ForEach(a => a.Invoke());
```

**執行結果：**
```
0
1
2
3
4
```

##### 📊 比較表格

| 比較項目 | 傳參數方法 | Lambda 閉包 |
|----------|------------|-------------|
| **存的東西** | 傳進來時的「值副本」 | 外部變數的「參考」 |
| **是否獨立** | ✅ 是 | ❌ 否，共用 |
| **是否會被改變** | ❌ 不會 | ✅ 會（如果外部變了） |
| **解法** | 無需處理 | 要複製變數成 `copy = i` |

##### 🔍 深入理解

**為什麼會發生這種情況？**

1. **編譯器行為**：編譯器會將 Lambda 表達式轉換為匿名方法和類別
2. **變數捕獲**：Lambda 捕獲的是變數的參考，而不是值
3. **延遲執行**：Lambda 直到被呼叫時才真正執行

**編譯器背後的轉換（概念性）：**
```csharp
// 原始程式碼
for (int i = 0; i < 5; i++)
{
    actions.Add(() => Console.WriteLine(i));
}

// 概念上編譯器產生的程式碼
var displayClass = new ClosureDisplayClass();
for (displayClass.i = 0; displayClass.i < 5; displayClass.i++)
{
    actions.Add(displayClass.Lambda);
}

class ClosureDisplayClass
{
    public int i; // 共用的變數
    public void Lambda() => Console.WriteLine(i);
}
```

#### 1.7.2 點擊次數 Counter

##### 🎯 實際應用範例

以下範例展示了閉包如何保持狀態，建立一個會記住點擊次數的計數器：

```csharp
public static Action CreateClickCounter()
{
    int count = 0;
    return () => {
        count++;
        Console.WriteLine($"你已經點了 : {count} 次!!@@@");
    };
}
```

##### 🔍 閉包機制分析

**關鍵概念理解：**

`return () => { ... }` 是一個**匿名函式（lambda）**，它「捕捉了 count」，所以 `count` 被放進編譯器隱藏生成的物件裡。

##### 📋 編譯器轉換過程

**你寫的版本：**
```csharp
Func<int> CreateCounter()
{
    int count = 0;
    return () => ++count;
}
```

**編譯器轉換版本：**
```csharp
class Closure
{
    public int count = 0;
    public int GetNext()
    {
        return ++count;
    }
}

Func<int> CreateCounter()
{
    Closure closure = new Closure();
    return closure.GetNext;
}
```

##### ⚡ 重要轉換說明

**變數提升：**
- `count` 原本是區域變數 → 被提升為 `Closure` 類別的欄位
- lambda 不再只是一個方法，而是「帶著記憶體（狀態）的 delegate」
- 即使 `CreateCounter()` 執行完畢，closure 物件還活著（因為 delegate 持有參考）

##### 💻 執行結果

```csharp
var counter = CreateClickCounter();
counter(); // 你已經點了 : 1 次!!@@@
counter(); // 你已經點了 : 2 次!!@@@
counter(); // 你已經點了 : 3 次!!@@@
```

##### 🎯 實際應用場景

**場景一：事件計數器**
```csharp
public static Action CreateEventCounter(string eventName)
{
    int count = 0;
    DateTime firstCall = DateTime.Now;
    
    return () => {
        count++;
        var elapsed = DateTime.Now - firstCall;
        Console.WriteLine($"{eventName} 觸發第 {count} 次，距離首次觸發已過 {elapsed.TotalSeconds:F1} 秒");
    };
}

// 使用方式
var buttonClickCounter = CreateEventCounter("按鈕點擊");
var apiCallCounter = CreateEventCounter("API 呼叫");

buttonClickCounter(); // 按鈕點擊 觸發第 1 次，距離首次觸發已過 0.0 秒
buttonClickCounter(); // 按鈕點擊 觸發第 2 次，距離首次觸發已過 1.2 秒
apiCallCounter();     // API 呼叫 觸發第 1 次，距離首次觸發已過 0.0 秒
```

**場景二：累積計算器**
```csharp
public static Func<int, int> CreateAccumulator(int initialValue = 0)
{
    int total = initialValue;
    
    return (int value) => {
        total += value;
        Console.WriteLine($"累積值: {total}");
        return total;
    };
}

// 使用方式
var accumulator = CreateAccumulator(100);
accumulator(10); // 累積值: 110
accumulator(20); // 累積值: 130
accumulator(-5); // 累積值: 125
```

**場景三：狀態機模擬**
```csharp
public static Action<string> CreateStateMachine()
{
    string currentState = "待機";
    var stateHistory = new List<string>();
    
    return (string newState) => {
        Console.WriteLine($"狀態轉換: {currentState} → {newState}");
        stateHistory.Add(currentState);
        currentState = newState;
        Console.WriteLine($"歷史記錄: [{string.Join(" → ", stateHistory)}] → {currentState}");
    };
}

// 使用方式
var stateMachine = CreateStateMachine();
stateMachine("運行中");  // 狀態轉換: 待機 → 運行中
stateMachine("暫停");    // 狀態轉換: 運行中 → 暫停
stateMachine("停止");    // 狀態轉換: 暫停 → 停止
```

##### 🚨 注意事項

**記憶體生命週期：**
- 閉包會延長被捕獲變數的生命週期
- 即使方法執行完畢，被捕獲的變數仍然存在於記憶體中
- 需要注意潛在的記憶體洩漏風險

**執行緒安全：**
- 多個執行緒同時呼叫同一個閉包時需要考慮執行緒安全
- 如有需要，應該加入適當的鎖定機制

> **🌟 重點提醒**
> 
> 1. **狀態保持**：閉包能夠保持區域變數的狀態，即使外層方法已經執行完畢
> 2. **編譯器魔法**：編譯器會自動將閉包轉換為類別，並處理變數提升
> 3. **記憶體影響**：閉包會影響變數的生命週期，需要注意記憶體管理
> 4. **實用模式**：閉包是實現計數器、累積器、狀態機等模式的強大工具

#### 1.7.3 記住外部變數的威力

##### 🎯 實際業務場景範例

以下範例展示了閉包如何捕獲外部變數，在實際業務開發中的應用：

```csharp
var promotionTagIds = GetPromotionTagIds();
var outerIds = GetProductOuterIds();
var targetTypeEnum = PromotionTargetType.Product;

Action action = () => this.UpdateProductSkuOuterIdTag(promotionTagIds, outerIds, targetTypeEnum);
```

##### 🔍 閉包變數捕獲分析

**關鍵概念理解：**

這裡的 lambda `() => ...` 就是個閉包。它**沒有參數**，但它裡面用了「外部的變數」：

- `promotionTagIds`
- `outerIds` 
- `targetTypeEnum`

##### 📋 閉包特性解析

**無參數閉包：**
- Lambda 表達式沒有接收任何參數 `()`
- 但能夠存取定義時的外部變數
- 這些變數被「捕獲」到閉包中

**變數生命週期延長：**
- 即使原本的方法執行完畢，這些變數依然可以被 `action` 存取
- 編譯器會將這些變數提升到一個隱藏的類別中

##### 🔧 編譯器轉換機制

**原始程式碼：**
```csharp
public void ProcessPromotion()
{
    var promotionTagIds = GetPromotionTagIds();
    var outerIds = GetProductOuterIds(); 
    var targetTypeEnum = PromotionTargetType.Product;
    
    Action action = () => this.UpdateProductSkuOuterIdTag(promotionTagIds, outerIds, targetTypeEnum);
    
    // 可能在稍後的某個時間點執行
    ExecuteLater(action);
}
```

**編譯器概念性轉換：**
```csharp
class ClosureDisplayClass
{
    public List<int> promotionTagIds;
    public List<string> outerIds;
    public PromotionTargetType targetTypeEnum;
    public YourClass instance; // 保存 this 參考
    
    public void CapturedMethod()
    {
        instance.UpdateProductSkuOuterIdTag(promotionTagIds, outerIds, targetTypeEnum);
    }
}

public void ProcessPromotion()
{
    var closure = new ClosureDisplayClass();
    closure.instance = this;
    closure.promotionTagIds = GetPromotionTagIds();
    closure.outerIds = GetProductOuterIds();
    closure.targetTypeEnum = PromotionTargetType.Product;
    
    Action action = closure.CapturedMethod;
    ExecuteLater(action);
}
```

##### 💡 實際應用場景

**場景一：非同步任務排程**
```csharp
public void SchedulePromotionUpdate()
{
    var promotionId = GetCurrentPromotionId();
    var updateTime = DateTime.Now.AddHours(1);
    var userEmail = GetCurrentUserEmail();
    
    // 排程任務，稍後執行
    Task.Delay(TimeSpan.FromHours(1)).ContinueWith(_ =>
    {
        // 這個 lambda 捕獲了外部變數
        LogPromotionUpdate(promotionId, updateTime, userEmail);
        SendNotificationEmail(userEmail, $"促銷 {promotionId} 已更新");
    });
}
```

**場景二：事件處理器設定**
```csharp
public void SetupEventHandlers()
{
    var connectionString = GetDatabaseConnectionString();
    var logLevel = GetCurrentLogLevel();
    var userId = GetCurrentUserId();
    
    // 設定各種事件處理器，都捕獲了相同的外部變數
    OnDataUpdated += () => LogDatabaseChange(connectionString, userId, "Data Updated", logLevel);
    OnUserLogin += () => LogDatabaseChange(connectionString, userId, "User Login", logLevel);
    OnError += () => LogDatabaseChange(connectionString, userId, "Error Occurred", logLevel);
}
```

**場景三：條件式操作佇列**
```csharp
public List<Action> BuildOperationQueue()
{
    var operations = new List<Action>();
    var batchId = Guid.NewGuid();
    var timestamp = DateTime.Now;
    var currentUser = GetCurrentUser();
    
    // 根據不同條件建立操作，每個都記住相同的上下文
    if (ShouldUpdateProducts())
    {
        operations.Add(() => UpdateProducts(batchId, timestamp, currentUser));
    }
    
    if (ShouldSendNotifications())
    {
        operations.Add(() => SendBatchNotifications(batchId, timestamp, currentUser));
    }
    
    if (ShouldGenerateReport())
    {
        operations.Add(() => GenerateProcessingReport(batchId, timestamp, currentUser));
    }
    
    return operations;
}
```

##### ⚡ 使用優勢

**程式碼簡潔：**
- 不需要建立額外的類別來傳遞參數
- 避免繁瑣的參數傳遞
- 保持程式碼的可讀性

**上下文保持：**
- 自動捕獲執行環境的狀態
- 確保稍後執行時能存取到正確的變數值
- 適合非同步或延遲執行場景

**彈性設計：**
- 可以動態決定要捕獲哪些變數
- 支援條件式的操作建構
- 方便實現策略模式和命令模式

##### 🚨 注意事項

**變數修改風險：**
```csharp
var items = new List<string> { "A", "B", "C" };
Action action = () => Console.WriteLine($"Items: {string.Join(",", items)}");

items.Add("D"); // 修改了被捕獲的變數
action(); // 輸出：Items: A,B,C,D （包含了後來新增的 "D"）
```

**記憶體洩漏考量：**
- 閉包會保持對捕獲變數的參考
- 大型物件可能無法被垃圾收集
- 需要注意變數的生命週期管理

> **🌟 重點提醒**
> 
> 1. **變數捕獲**：閉包能夠捕獲外部作用域的任何變數，包括區域變數、參數和類別成員
> 2. **無參數威力**：即使 lambda 沒有參數，也能透過變數捕獲存取外部狀態
> 3. **延遲執行優勢**：非常適合非同步操作、事件處理和延遲執行場景
> 4. **記憶體管理**：要注意被捕獲變數的生命週期，避免意外的記憶體洩漏

### 1.8 count / flag / batch + while loop 使用

在處理大量資料或需要複雜條件控制的場景中，合理使用計數器 (count)、標記 (flag) 和批次處理 (batch) 搭配 while 迴圈，可以讓程式碼更有效率且更容易維護。

#### 1.8.1 基本計數與標記模式

##### 📊 計數器控制範例

以下範例展示如何使用計數器來控制迴圈執行：

```csharp
void Main()
{
    // 模擬處理大量資料，每次處理固定數量
    var dataList = Enumerable.Range(1, 100).ToList();
    ProcessDataWithCounter(dataList);
}

public void ProcessDataWithCounter(List<int> dataList)
{
    int processedCount = 0;
    int batchSize = 10;
    int index = 0;
    
    while (index < dataList.Count)
    {
        // 處理當前項目
        var currentItem = dataList[index];
        Console.WriteLine($"處理項目: {currentItem}");
        
        processedCount++;
        index++;
        
        // 每處理 10 個項目就暫停一下
        if (processedCount % batchSize == 0)
        {
            Console.WriteLine($"已處理 {processedCount} 個項目，暫停 100ms...");
            Thread.Sleep(100);
        }
    }
    
    Console.WriteLine($"總共處理了 {processedCount} 個項目");
}
```

##### 🚩 標記控制範例

使用布林標記來控制複雜的迴圈條件：

```csharp
void Main()
{
    var numbers = new List<int> { 1, 5, 8, 12, 15, 20, 25, 30 };
    FindTargetWithFlag(numbers, 15);
}

public void FindTargetWithFlag(List<int> numbers, int target)
{
    bool found = false;
    bool shouldContinue = true;
    int attempts = 0;
    int maxAttempts = 5;
    int index = 0;
    
    while (shouldContinue && !found && index < numbers.Count)
    {
        attempts++;
        var current = numbers[index];
        
        Console.WriteLine($"嘗試 {attempts}: 檢查數字 {current}");
        
        if (current == target)
        {
            found = true;
            Console.WriteLine($"✅ 找到目標數字 {target} 在位置 {index}");
        }
        else if (attempts >= maxAttempts)
        {
            shouldContinue = false;
            Console.WriteLine($"❌ 超過最大嘗試次數 {maxAttempts}，停止搜尋");
        }
        
        index++;
    }
    
    if (!found && shouldContinue)
    {
        Console.WriteLine($"❌ 在清單中找不到目標數字 {target}");
    }
}
```

#### 1.8.2 批次處理模式

##### 📦 資料庫批次操作範例

在處理大量資料庫操作時，批次處理可以顯著提升效能：

```csharp
void Main()
{
    var userIds = Enumerable.Range(1, 1000).ToList();
    ProcessUsersBatch(userIds);
}

public void ProcessUsersBatch(List<int> userIds)
{
    int batchSize = 50;
    int currentIndex = 0;
    int totalProcessed = 0;
    
    while (currentIndex < userIds.Count)
    {
        // 取得當前批次
        var currentBatch = userIds
            .Skip(currentIndex)
            .Take(batchSize)
            .ToList();
        
        // 模擬批次資料庫操作
        ProcessBatch(currentBatch);
        
        totalProcessed += currentBatch.Count;
        currentIndex += batchSize;
        
        Console.WriteLine($"已處理 {totalProcessed}/{userIds.Count} 個使用者");
        
        // 批次間的延遲，避免對資料庫造成過大壓力
        if (currentIndex < userIds.Count)
        {
            Thread.Sleep(50);
        }
    }
    
    Console.WriteLine($"✅ 批次處理完成，總共處理 {totalProcessed} 個使用者");
}

private void ProcessBatch(List<int> userBatch)
{
    // 模擬資料庫批次操作
    Console.WriteLine($"正在處理批次：使用者 ID {userBatch.First()} 到 {userBatch.Last()}");
    
    // 這裡可以是實際的資料庫操作
    // 例如：批次更新、批次插入等
    foreach (var userId in userBatch)
    {
        // 模擬處理每個使用者
        // UpdateUserInDatabase(userId);
    }
}
```

##### 🔄 檔案批次讀取範例

處理大型檔案時的批次讀取策略：

```csharp
void Main()
{
    string filePath = @"C:\temp\large_file.txt";
    ReadFileInBatches(filePath);
}

public void ReadFileInBatches(string filePath)
{
    int batchSize = 1000; // 每次讀取 1000 行
    int lineCount = 0;
    int batchNumber = 1;
    bool hasMoreData = true;
    
    using (var reader = new StreamReader(filePath))
    {
        while (hasMoreData && !reader.EndOfStream)
        {
            var currentBatch = new List<string>();
            int linesInCurrentBatch = 0;
            
            // 讀取一個批次的資料
            while (linesInCurrentBatch < batchSize && !reader.EndOfStream)
            {
                string line = reader.ReadLine();
                if (line != null)
                {
                    currentBatch.Add(line);
                    linesInCurrentBatch++;
                    lineCount++;
                }
            }
            
            // 處理當前批次
            if (currentBatch.Count > 0)
            {
                ProcessLineBatch(currentBatch, batchNumber);
                Console.WriteLine($"批次 {batchNumber}: 處理了 {currentBatch.Count} 行資料");
                batchNumber++;
            }
            else
            {
                hasMoreData = false;
            }
            
            // 記憶體管理：清理當前批次
            currentBatch.Clear();
        }
    }
    
    Console.WriteLine($"檔案讀取完成，總共處理 {lineCount} 行資料，分成 {batchNumber - 1} 個批次");
}

private void ProcessLineBatch(List<string> lines, int batchNumber)
{
    // 模擬批次處理邏輯
    foreach (var line in lines)
    {
        // 處理每一行資料
        // 例如：解析、轉換、驗證等
    }
}
```

#### 1.8.3 複合條件控制

##### 🎯 多重條件組合範例

結合計數器、標記和批次處理的複合控制邏輯：

```csharp
void Main()
{
    var dataProcessor = new AdvancedDataProcessor();
    dataProcessor.ProcessDataWithComplexControl();
}

public class AdvancedDataProcessor
{
    private int maxRetries = 3;
    private int batchSize = 20;
    private int maxErrorsPerBatch = 5;
    
    public void ProcessDataWithComplexControl()
    {
        var dataSource = GenerateTestData(200);
        
        int totalProcessed = 0;
        int totalErrors = 0;
        int currentIndex = 0;
        bool shouldContinue = true;
        int globalRetryCount = 0;
        
        while (shouldContinue && currentIndex < dataSource.Count)
        {
            // 取得當前批次
            var currentBatch = dataSource
                .Skip(currentIndex)
                .Take(batchSize)
                .ToList();
            
            // 批次處理結果
            var batchResult = ProcessBatchWithErrorHandling(currentBatch);
            
            totalProcessed += batchResult.SuccessCount;
            totalErrors += batchResult.ErrorCount;
            
            // 檢查是否需要重試整個批次
            if (batchResult.ErrorCount > maxErrorsPerBatch)
            {
                globalRetryCount++;
                Console.WriteLine($"⚠️ 批次錯誤過多，進行第 {globalRetryCount} 次重試...");
                
                if (globalRetryCount >= maxRetries)
                {
                    shouldContinue = false;
                    Console.WriteLine($"❌ 達到最大重試次數 {maxRetries}，停止處理");
                }
                // 不增加 currentIndex，重新處理同一批次
            }
            else
            {
                // 批次成功，移動到下一批次
                currentIndex += batchSize;
                globalRetryCount = 0; // 重置重試計數器
                
                Console.WriteLine($"✅ 批次處理成功 - 成功: {batchResult.SuccessCount}, 錯誤: {batchResult.ErrorCount}");
            }
            
            // 安全檢查：避免無窮迴圈
            if (totalErrors > dataSource.Count * 0.5) // 如果錯誤率超過 50%
            {
                shouldContinue = false;
                Console.WriteLine($"❌ 錯誤率過高，停止處理");
            }
        }
        
        Console.WriteLine($"處理完成 - 總計處理: {totalProcessed}, 總計錯誤: {totalErrors}");
    }
    
    private BatchResult ProcessBatchWithErrorHandling(List<string> batch)
    {
        int successCount = 0;
        int errorCount = 0;
        
        foreach (var item in batch)
        {
            try
            {
                // 模擬處理邏輯，隨機產生錯誤
                if (ProcessSingleItem(item))
                {
                    successCount++;
                }
                else
                {
                    errorCount++;
                }
            }
            catch (Exception ex)
            {
                errorCount++;
                Console.WriteLine($"處理項目 {item} 時發生錯誤: {ex.Message}");
            }
        }
        
        return new BatchResult { SuccessCount = successCount, ErrorCount = errorCount };
    }
    
    private bool ProcessSingleItem(string item)
    {
        // 模擬處理邏輯，30% 機率失敗
        var random = new Random();
        return random.Next(100) > 30;
    }
    
    private List<string> GenerateTestData(int count)
    {
        return Enumerable.Range(1, count)
            .Select(i => $"Item_{i:D3}")
            .ToList();
    }
    
    private class BatchResult
    {
        public int SuccessCount { get; set; }
        public int ErrorCount { get; set; }
    }
}
```

##### 🎯 複合條件的優勢

**提升可靠性：**
- ✅ 多重檢查機制確保程式穩定性
- ✅ 錯誤處理和重試機制
- ✅ 避免無窮迴圈的安全檢查

**效能最佳化：**
- 🚀 批次處理減少 I/O 操作次數
- 🚀 計數器控制資源使用
- 🚀 標記避免不必要的計算

**程式碼可維護性：**
- 🔧 清晰的條件邏輯
- 🔧 易於調整的參數設定
- 🔧 詳細的執行狀態回饋

> **🌟 重點提醒**
> 
> 1. **避免無窮迴圈**：始終設置適當的退出條件和安全檢查
> 2. **記憶體管理**：在批次處理中及時清理不需要的物件
> 3. **錯誤處理**：為每個可能失敗的操作添加適當的例外處理
> 4. **效能監控**：記錄處理進度和效能指標，便於最佳化和除錯