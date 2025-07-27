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
