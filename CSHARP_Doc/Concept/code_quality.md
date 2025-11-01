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
    - [1.5.3 泛型與快取](#153-泛型與快取)
    - [1.5.4 泛型快取 - 進階版](#154-泛型快取-進階版)
    - [1.5.5 快取反序列化泛型](#155-快取反序列化泛型)
    - [1.5.6 Response Entity 泛型應用](#156-response-entity-泛型應用)
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
  - [1.9 多墊一層](#19-多墊一層)
- [2. 傳 delegate](#2-傳-delegate)
- [3. abstract class](#3-abstract-class)
- [4. try catch](#4-try-catch)
- [5. 方法直接寫在 建立的 entity裡面](#5-方法直接寫在-建立的-entity裡面)
- [6. 關於建構子](#6-關於建構子)
  - [6.1 string 不要放在建構子注入的位置](#61-string-不要放在建構子注入的位置)
---

### 1.1 抽取共用驗證邏輯

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

### 1.9 多墊一層

在軟體開發中，我們經常需要在「不破壞原本系統」的前提下，導入新功能或架構。這是「轉接頭」的概念，保持雙方都可以正常工作。我們要在不動舊的 Service，不直接綁死新的 Service 的情況下，提供一個緩衝區、轉換區、測試平台。

##### 🏗️ **架構概念**

```
Client --> 中介層 (AdapterService) --> 舊Service 
                                    \--> 新Service（逐步導入）
```

##### 🔧 **Adapter Service 實作範例**

**介面定義：**
```csharp
public interface IOrderService
{
    void PlaceOrder(Order order);
}
```

**舊的服務實作：**
```csharp
// 舊的服務
public class LegacyOrderService : IOrderService
{
    public void PlaceOrder(Order order)
    {
        Console.WriteLine("使用舊系統下單");
        // 原本的下單邏輯
    }
}
```

**新的服務實作：**
```csharp
// 新的服務
public class NewOrderService : IOrderService
{
    public void PlaceOrder(Order order)
    {
        Console.WriteLine("使用新系統下單");
        // 新的下單邏輯
    }
}
```

**Adapter Service - 墊一層：**
```csharp
// Adapter Service - 墊一層
public class OrderServiceAdapter : IOrderService
{
    private readonly IOrderService _legacyService;
    private readonly IOrderService _newService;
    private readonly bool _useNew;

    public OrderServiceAdapter(IOrderService legacy, IOrderService newer, bool useNew)
    {
        _legacyService = legacy;
        _newService = newer;
        _useNew = useNew;
    }

    public void PlaceOrder(Order order)
    {
        if (_useNew)
            _newService.PlaceOrder(order);
        else
            _legacyService.PlaceOrder(order);
    }
}
```

##### 💡 **使用優勢**

**平滑過渡：**
- ✅ 不管外面怎麼呼叫，只用 `OrderServiceAdapter`
- ✅ 未來想切換到新系統，只要改 `_useNew = true`
- ✅ 可以逐步測試新功能而不影響現有系統

**風險控制：**
- 🛡️ 新舊系統並存，降低切換風險
- 🛡️ 可以快速回滾到舊系統
- 🛡️ 提供 A/B 測試的基礎架構

**系統隔離：**
- 🔧 舊系統和新系統完全獨立
- 🔧 不需要修改現有的客戶端程式碼
- 🔧 中介層可以加入額外的邏輯（如日誌、監控）

##### 🎯 **進階應用範例**

**基於條件的智慧路由：**
```csharp
public class SmartOrderServiceAdapter : IOrderService
{
    private readonly IOrderService _legacyService;
    private readonly IOrderService _newService;
    private readonly IConfiguration _config;

    public SmartOrderServiceAdapter(
        IOrderService legacy, 
        IOrderService newer, 
        IConfiguration config)
    {
        _legacyService = legacy;
        _newService = newer;
        _config = config;
    }

    public void PlaceOrder(Order order)
    {
        // 根據不同條件決定使用哪個服務
        if (ShouldUseNewService(order))
        {
            try
            {
                _newService.PlaceOrder(order);
            }
            catch (Exception ex)
            {
                // 新服務失敗時回退到舊服務
                Console.WriteLine($"新服務失敗，回退到舊服務: {ex.Message}");
                _legacyService.PlaceOrder(order);
            }
        }
        else
        {
            _legacyService.PlaceOrder(order);
        }
    }

    private bool ShouldUseNewService(Order order)
    {
        // 可以基於多種條件判斷
        var useNewPercentage = _config.GetValue<int>("NewServicePercentage", 0);
        var random = new Random();
        
        // 根據訂單金額決定
        if (order.Amount > 1000)
            return useNewPercentage > random.Next(100);
            
        // 根據客戶類型決定
        if (order.CustomerType == "VIP")
            return true;
            
        return false;
    }
}
```

**帶有監控和日誌的 Adapter：**
```csharp
public class MonitoredOrderServiceAdapter : IOrderService
{
    private readonly IOrderService _legacyService;
    private readonly IOrderService _newService;
    private readonly ILogger<MonitoredOrderServiceAdapter> _logger;
    private readonly IMetrics _metrics;
    private readonly bool _useNew;

    public MonitoredOrderServiceAdapter(
        IOrderService legacy,
        IOrderService newer,
        ILogger<MonitoredOrderServiceAdapter> logger,
        IMetrics metrics,
        bool useNew)
    {
        _legacyService = legacy;
        _newService = newer;
        _logger = logger;
        _metrics = metrics;
        _useNew = useNew;
    }

    public void PlaceOrder(Order order)
    {
        var stopwatch = Stopwatch.StartNew();
        var serviceType = _useNew ? "New" : "Legacy";
        
        try
        {
            _logger.LogInformation($"開始使用 {serviceType} 服務處理訂單 {order.Id}");
            
            if (_useNew)
                _newService.PlaceOrder(order);
            else
                _legacyService.PlaceOrder(order);
                
            stopwatch.Stop();
            
            _logger.LogInformation($"{serviceType} 服務處理訂單 {order.Id} 成功，耗時 {stopwatch.ElapsedMilliseconds}ms");
            _metrics.Counter($"order_success_{serviceType.ToLower()}").Increment();
            _metrics.Timer($"order_duration_{serviceType.ToLower()}").Record(stopwatch.Elapsed);
        }
        catch (Exception ex)
        {
            stopwatch.Stop();
            
            _logger.LogError(ex, $"{serviceType} 服務處理訂單 {order.Id} 失敗，耗時 {stopwatch.ElapsedMilliseconds}ms");
            _metrics.Counter($"order_failure_{serviceType.ToLower()}").Increment();
            
            throw;
        }
    }
}
```

##### 📦 **依賴注入設定**

**在 Startup.cs 或 Program.cs 中設定：**
```csharp
public void ConfigureServices(IServiceCollection services)
{
    // 註冊舊的和新的服務
    services.AddSingleton<LegacyOrderService>();
    services.AddSingleton<NewOrderService>();
    
    // 註冊 Adapter，並決定使用策略
    services.AddSingleton<IOrderService>(provider =>
    {
        var legacy = provider.GetRequiredService<LegacyOrderService>();
        var newer = provider.GetRequiredService<NewOrderService>();
        var config = provider.GetRequiredService<IConfiguration>();
        var logger = provider.GetRequiredService<ILogger<MonitoredOrderServiceAdapter>>();
        var metrics = provider.GetRequiredService<IMetrics>();
        
        var useNew = config.GetValue<bool>("UseNewOrderService", false);
        
        return new MonitoredOrderServiceAdapter(legacy, newer, logger, metrics, useNew);
    });
}
```

##### 🎯 **實際應用場景**

**場景一：系統升級**
- 從舊的支付系統升級到新的支付閘道
- 逐步將使用者流量轉移到新系統
- 確保在出問題時能快速回退

**場景二：第三方服務切換**
- 從舊的物流 API 切換到新的物流服務
- 在切換過程中比較兩個服務的效能
- 根據地區或客戶類型選擇不同的服務

**場景三：演算法升級**
- 推薦演算法的新舊版本比較
- 價格計算引擎的升級
- 搜尋演算法的 A/B 測試

> **🌟 重點提醒**
> 
> 1. **漸進式切換**：透過 Adapter 模式可以實現平滑的系統升級
> 2. **風險降低**：新舊系統並存，降低切換失敗的風險
> 3. **監控重要性**：在 Adapter 中加入監控和日誌，便於比較新舊系統的效能
> 4. **配置驅動**：使用配置檔案控制切換策略，無需重新部署程式碼

## 2. 傳 delegate

Delegate 是 C# 中一個強大的功能，允許我們將方法作為參數傳遞，實現高度彈性和可重用的程式碼設計。透過傳遞 delegate，我們可以將具體的執行邏輯延遲到呼叫時才決定，這種模式在函式程式設計和事件處理中非常常見。

## 3. abstract class

在物件導向程式設計中，當我們明確知道有多個子類別會重複用到相同的邏輯，並且會同時共用欄位與邏輯時，使用 abstract class（抽象類別）是一個絕佳的選擇。抽象類別提供了一個介於完全抽象的介面和具體實作類別之間的平衡點。

### 🎯 **核心概念**

Abstract class 的主要特色：
- 🏗️ **共用實作**：可以包含具體的方法實作，供子類別繼承使用
- 🔒 **強制實作**：可以定義抽象方法，強制子類別必須實作
- 💾 **共用狀態**：可以包含欄位和屬性，讓子類別共用資料
- 🛡️ **無法實例化**：抽象類別本身無法被直接實例化

### 📝 **基本範例**

**基礎抽象類別定義：**
```csharp
public abstract class Animal
{
    // 共用欄位
    protected string Name { get; set; }
    protected int Age { get; set; }
    
    // 建構函式
    protected Animal(string name, int age)
    {
        Name = name;
        Age = age;
    }
    
    // 共用的具體方法
    public virtual void Sleep()
    {
        Console.WriteLine($"{Name} 正在睡覺...");
    }
    
    public void ShowInfo()
    {
        Console.WriteLine($"名字: {Name}, 年齡: {Age}歲");
    }
    
    // 抽象方法 - 強制子類別實作
    public abstract void MakeSound();
    public abstract void Move();
}
```

**具體子類別實作：**
```csharp
public class Dog : Animal
{
    public string Breed { get; set; }
    
    public Dog(string name, int age, string breed) : base(name, age)
    {
        Breed = breed;
    }
    
    public override void MakeSound()
    {
        Console.WriteLine($"{Name} 汪汪叫!");
    }
    
    public override void Move()
    {
        Console.WriteLine($"{Name} 正在跑步!");
    }
    
    // 可以覆寫共用方法
    public override void Sleep()
    {
        Console.WriteLine($"{Name} 蜷縮成一團睡覺...");
    }
    
    // 子類別特有的方法
    public void Fetch()
    {
        Console.WriteLine($"{Name} 正在撿球!");
    }
}

public class Bird : Animal
{
    public double WingSpan { get; set; }
    
    public Bird(string name, int age, double wingSpan) : base(name, age)
    {
        WingSpan = wingSpan;
    }
    
    public override void MakeSound()
    {
        Console.WriteLine($"{Name} 啾啾叫!");
    }
    
    public override void Move()
    {
        Console.WriteLine($"{Name} 正在飛翔!");
    }
    
    // 子類別特有的方法
    public void BuildNest()
    {
        Console.WriteLine($"{Name} 正在築巢!");
    }
}
```

**使用範例：**
```csharp
void Main()
{
    var animals = new List<Animal>
    {
        new Dog("小白", 3, "拉布拉多"),
        new Bird("小藍", 1, 15.5),
        new Dog("小黑", 5, "柯基")
    };
    
    foreach (var animal in animals)
    {
        animal.ShowInfo();        // 使用共用方法
        animal.MakeSound();       // 使用抽象方法的實作
        animal.Move();            // 使用抽象方法的實作
        animal.Sleep();           // 使用共用方法（可能被覆寫）
        Console.WriteLine("---");
    }
    
    // 型別檢查和特有方法呼叫
    foreach (var animal in animals)
    {
        if (animal is Dog dog)
        {
            dog.Fetch();
        }
        else if (animal is Bird bird)
        {
            bird.BuildNest();
        }
    }
}
```

### 🔧 **實際應用場景**

**場景一：檔案處理器抽象類別**
```csharp
public abstract class FileProcessor
{
    protected string FilePath { get; private set; }
    protected DateTime ProcessStartTime { get; private set; }
    
    protected FileProcessor(string filePath)
    {
        FilePath = filePath;
    }
    
    // 通用的檔案處理流程
    public void ProcessFile()
    {
        try
        {
            ProcessStartTime = DateTime.Now;
            
            LogStart();
            ValidateFile();
            
            var content = ReadFile();
            var processedContent = ProcessContent(content);
            WriteResult(processedContent);
            
            LogCompletion();
        }
        catch (Exception ex)
        {
            LogError(ex);
            throw;
        }
    }
    
    // 共用的具體方法
    protected virtual void LogStart()
    {
        Console.WriteLine($"開始處理檔案: {FilePath} at {ProcessStartTime}");
    }
    
    protected virtual void LogCompletion()
    {
        var duration = DateTime.Now - ProcessStartTime;
        Console.WriteLine($"檔案處理完成，耗時: {duration.TotalSeconds:F2} 秒");
    }
    
    protected virtual void LogError(Exception ex)
    {
        Console.WriteLine($"處理檔案時發生錯誤: {ex.Message}");
    }
    
    protected virtual void ValidateFile()
    {
        if (!File.Exists(FilePath))
            throw new FileNotFoundException($"檔案不存在: {FilePath}");
    }
    
    // 抽象方法 - 強制子類別實作
    protected abstract string ReadFile();
    protected abstract string ProcessContent(string content);
    protected abstract void WriteResult(string processedContent);
}
```

**具體實作類別：**
```csharp
public class CsvProcessor : FileProcessor
{
    private readonly char _delimiter;
    
    public CsvProcessor(string filePath, char delimiter = ',') : base(filePath)
    {
        _delimiter = delimiter;
    }
    
    protected override string ReadFile()
    {
        Console.WriteLine("讀取 CSV 檔案...");
        return File.ReadAllText(FilePath);
    }
    
    protected override string ProcessContent(string content)
    {
        Console.WriteLine("處理 CSV 內容...");
        var lines = content.Split('\n');
        var processedLines = new List<string>();
        
        foreach (var line in lines)
        {
            if (!string.IsNullOrWhiteSpace(line))
            {
                var columns = line.Split(_delimiter);
                // 將每一列轉換為大寫
                var processedColumns = columns.Select(col => col.Trim().ToUpper());
                processedLines.Add(string.Join(_delimiter, processedColumns));
            }
        }
        
        return string.Join('\n', processedLines);
    }
    
    protected override void WriteResult(string processedContent)
    {
        var outputPath = Path.ChangeExtension(FilePath, ".processed.csv");
        File.WriteAllText(outputPath, processedContent);
        Console.WriteLine($"結果已寫入: {outputPath}");
    }
}

public class JsonProcessor : FileProcessor
{
    public JsonProcessor(string filePath) : base(filePath)
    {
    }
    
    protected override string ReadFile()
    {
        Console.WriteLine("讀取 JSON 檔案...");
        return File.ReadAllText(FilePath);
    }
    
    protected override string ProcessContent(string content)
    {
        Console.WriteLine("處理 JSON 內容...");
        // 格式化 JSON
        var jsonObject = JsonSerializer.Deserialize<object>(content);
        return JsonSerializer.Serialize(jsonObject, new JsonSerializerOptions 
        { 
            WriteIndented = true 
        });
    }
    
    protected override void WriteResult(string processedContent)
    {
        var outputPath = Path.ChangeExtension(FilePath, ".formatted.json");
        File.WriteAllText(outputPath, processedContent);
        Console.WriteLine($"格式化結果已寫入: {outputPath}");
    }
    
    // 覆寫驗證方法，加入 JSON 特定驗證
    protected override void ValidateFile()
    {
        base.ValidateFile();
        
        var content = File.ReadAllText(FilePath);
        try
        {
            JsonSerializer.Deserialize<object>(content);
        }
        catch (JsonException)
        {
            throw new InvalidDataException("檔案不是有效的 JSON 格式");
        }
    }
}
```

**使用範例：**
```csharp
void Main()
{
    var processors = new List<FileProcessor>
    {
        new CsvProcessor("data.csv"),
        new JsonProcessor("config.json")
    };
    
    foreach (var processor in processors)
    {
        processor.ProcessFile();
        Console.WriteLine("========");
    }
}
```

### 🎯 **進階應用：遊戲角色系統**

**抽象角色基類：**
```csharp
public abstract class GameCharacter
{
    // 共用屬性
    public string Name { get; protected set; }
    public int Level { get; protected set; }
    public int Health { get; protected set; }
    public int MaxHealth { get; protected set; }
    public int Mana { get; protected set; }
    public int MaxMana { get; protected set; }
    
    protected GameCharacter(string name, int level)
    {
        Name = name;
        Level = level;
        MaxHealth = CalculateMaxHealth();
        MaxMana = CalculateMaxMana();
        Health = MaxHealth;
        Mana = MaxMana;
    }
    
    // 共用方法
    public virtual void TakeDamage(int damage)
    {
        Health = Math.Max(0, Health - damage);
        Console.WriteLine($"{Name} 受到 {damage} 點傷害，剩餘血量: {Health}/{MaxHealth}");
        
        if (Health == 0)
        {
            OnDeath();
        }
    }
    
    public virtual void RestoreHealth(int amount)
    {
        Health = Math.Min(MaxHealth, Health + amount);
        Console.WriteLine($"{Name} 恢復 {amount} 點血量，目前血量: {Health}/{MaxHealth}");
    }
    
    public virtual void ConsumeMana(int amount)
    {
        if (Mana >= amount)
        {
            Mana -= amount;
            Console.WriteLine($"{Name} 消耗 {amount} 點魔力，剩餘魔力: {Mana}/{MaxMana}");
        }
        else
        {
            Console.WriteLine($"{Name} 魔力不足!");
        }
    }
    
    protected virtual void OnDeath()
    {
        Console.WriteLine($"{Name} 倒下了!");
    }
    
    // 抽象方法 - 每個角色類型都必須實作
    public abstract void Attack(GameCharacter target);
    public abstract void UseSpecialAbility(GameCharacter target);
    protected abstract int CalculateMaxHealth();
    protected abstract int CalculateMaxMana();
    
    // 虛擬方法 - 可選擇性覆寫
    public virtual void LevelUp()
    {
        Level++;
        var oldMaxHealth = MaxHealth;
        var oldMaxMana = MaxMana;
        
        MaxHealth = CalculateMaxHealth();
        MaxMana = CalculateMaxMana();
        
        Health += (MaxHealth - oldMaxHealth);
        Mana += (MaxMana - oldMaxMana);
        
        Console.WriteLine($"{Name} 升級到 {Level} 級!");
        Console.WriteLine($"血量上限: {oldMaxHealth} -> {MaxHealth}");
        Console.WriteLine($"魔力上限: {oldMaxMana} -> {MaxMana}");
    }
}
```

**具體角色實作：**
```csharp
public class Warrior : GameCharacter
{
    public int Armor { get; private set; }
    
    public Warrior(string name, int level) : base(name, level)
    {
        Armor = Level * 2;
    }
    
    protected override int CalculateMaxHealth()
    {
        return 100 + (Level * 15); // 戰士血量較高
    }
    
    protected override int CalculateMaxMana()
    {
        return 30 + (Level * 3); // 戰士魔力較低
    }
    
    public override void Attack(GameCharacter target)
    {
        var damage = 20 + (Level * 3);
        Console.WriteLine($"{Name} 用劍攻擊 {target.Name}!");
        target.TakeDamage(damage);
    }
    
    public override void UseSpecialAbility(GameCharacter target)
    {
        var manaCost = 15;
        if (Mana >= manaCost)
        {
            ConsumeMana(manaCost);
            var damage = 35 + (Level * 5);
            Console.WriteLine($"{Name} 使用 [重擊] 攻擊 {target.Name}!");
            target.TakeDamage(damage);
        }
    }
    
    // 戰士特有方法
    public void DefensiveStance()
    {
        Console.WriteLine($"{Name} 進入防禦姿態，護甲增加!");
        // 暫時增加護甲值
    }
}

public class Mage : GameCharacter
{
    public int SpellPower { get; private set; }
    
    public Mage(string name, int level) : base(name, level)
    {
        SpellPower = Level * 4;
    }
    
    protected override int CalculateMaxHealth()
    {
        return 60 + (Level * 8); // 法師血量較低
    }
    
    protected override int CalculateMaxMana()
    {
        return 80 + (Level * 12); // 法師魔力較高
    }
    
    public override void Attack(GameCharacter target)
    {
        var manaCost = 5;
        if (Mana >= manaCost)
        {
            ConsumeMana(manaCost);
            var damage = 15 + (Level * 2) + SpellPower;
            Console.WriteLine($"{Name} 施放火球術攻擊 {target.Name}!");
            target.TakeDamage(damage);
        }
    }
    
    public override void UseSpecialAbility(GameCharacter target)
    {
        var manaCost = 25;
        if (Mana >= manaCost)
        {
            ConsumeMana(manaCost);
            var damage = 40 + (Level * 6) + SpellPower;
            Console.WriteLine($"{Name} 施放 [閃電風暴] 攻擊 {target.Name}!");
            target.TakeDamage(damage);
        }
    }
    
    // 法師特有方法
    public void Heal(GameCharacter target)
    {
        var manaCost = 20;
        if (Mana >= manaCost)
        {
            ConsumeMana(manaCost);
            var healAmount = 30 + (Level * 4);
            Console.WriteLine($"{Name} 對 {target.Name} 施放治療術!");
            target.RestoreHealth(healAmount);
        }
    }
}
```

### 🏆 **Abstract Class vs Interface 比較**

| 特性 | Abstract Class | Interface |
|------|----------------|-----------|
| **方法實作** | ✅ 可以有具體實作 | ❌ 僅定義契約 (C# 8+ 可有預設實作) |
| **欄位** | ✅ 可以有欄位 | ❌ 不能有欄位 |
| **建構函式** | ✅ 可以有建構函式 | ❌ 不能有建構函式 |
| **繼承** | ❌ 單一繼承 | ✅ 多重實作 |
| **存取修飾詞** | ✅ 支援所有修飾詞 | ⚠️ 預設 public |
| **適用場景** | 有共用實作和狀態 | 定義行為契約 |

### 🎯 **使用時機指南**

**選擇 Abstract Class 的情況：**
- ✅ 有共用的欄位或屬性需要被繼承
- ✅ 有共用的方法實作可以被重用
- ✅ 需要定義建構函式來初始化共用狀態
- ✅ 子類別之間有 "is-a" 的關係

**選擇 Interface 的情況：**
- ✅ 只需要定義行為契約，不需要共用實作
- ✅ 需要支援多重繼承
- ✅ 想要實現松耦合的設計
- ✅ 不同類別族群需要共同的行為

> **🌟 重點提醒**
> 
> 1. **共用邏輯**：Abstract class 最大的優勢是能夠提供共用的實作，避免程式碼重複
> 2. **強制實作**：透過抽象方法可以強制子類別實作特定行為，確保一致性
> 3. **模板方法模式**：Abstract class 很適合實現模板方法模式，定義演算法骨架
> 4. **適當抽象層次**：選擇適當的抽象層次，既不要過度抽象也不要過度具體

## 4. try catch

在 C# 程式設計中，適當的例外處理是確保程式穩定性和使用者體驗的關鍵。Try-catch 語句不僅僅是捕獲錯誤，更是一種優雅處理預期和非預期情況的機制。透過合理的例外處理策略，我們可以讓程式在面臨問題時仍能保持運作，並提供有意義的錯誤資訊。

### 🎯 **核心概念**

Try-catch 的設計原則：
- 🎯 **精確捕獲**：只捕獲你能處理的特定例外類型
- 🔄 **適當回復**：提供合理的備用方案或重試機制
- 📝 **詳細記錄**：記錄足夠的上下文資訊用於除錯
- ⚡ **效能考量**：避免在正常流程中依賴例外處理

### 📝 **基本範例**

**基礎 try-catch 結構：**
```csharp
public class FileOperationService
{
    private readonly ILogger<FileOperationService> _logger;
    
    public FileOperationService(ILogger<FileOperationService> logger)
    {
        _logger = logger;
    }
    
    public string ReadFileContent(string filePath)
    {
        try
        {
            // 主要邏輯
            if (string.IsNullOrWhiteSpace(filePath))
                throw new ArgumentException("檔案路徑不能為空", nameof(filePath));
            
            var content = File.ReadAllText(filePath);
            _logger.LogInformation("成功讀取檔案: {FilePath}", filePath);
            
            return content;
        }
        catch (ArgumentException ex)
        {
            // 處理參數錯誤
            _logger.LogWarning("無效的檔案路徑: {Error}", ex.Message);
            throw; // 重新拋出，因為這是程式邏輯錯誤
        }
        catch (FileNotFoundException ex)
        {
            // 處理檔案不存在
            _logger.LogWarning("檔案不存在: {FilePath}", filePath);
            return string.Empty; // 提供預設值
        }
        catch (UnauthorizedAccessException ex)
        {
            // 處理權限問題
            _logger.LogError("無權限存取檔案: {FilePath}, Error: {Error}", 
                filePath, ex.Message);
            throw new InvalidOperationException($"無法存取檔案: {filePath}", ex);
        }
        catch (IOException ex)
        {
            // 處理 I/O 錯誤
            _logger.LogError("檔案讀取 I/O 錯誤: {FilePath}, Error: {Error}", 
                filePath, ex.Message);
            throw; // 讓呼叫者決定如何處理
        }
        catch (Exception ex)
        {
            // 處理其他未預期的錯誤
            _logger.LogError(ex, "讀取檔案時發生未預期錯誤: {FilePath}", filePath);
            throw new InvalidOperationException("檔案讀取失敗", ex);
        }
    }
}
```

### 🔄 **重試機制範例**

**帶重試功能的網路請求：**
```csharp
public class HttpRetryService
{
    private readonly HttpClient _httpClient;
    private readonly ILogger<HttpRetryService> _logger;
    private const int MaxRetries = 3;
    private const int BaseDelayMs = 1000;
    
    public HttpRetryService(HttpClient httpClient, ILogger<HttpRetryService> logger)
    {
        _httpClient = httpClient;
        _logger = logger;
    }
    
    public async Task<T> GetWithRetryAsync<T>(string url, int maxRetries = MaxRetries)
    {
        var attempt = 0;
        Exception lastException = null;
        
        while (attempt < maxRetries)
        {
            try
            {
                attempt++;
                _logger.LogInformation("嘗試請求 {Url}, 第 {Attempt} 次", url, attempt);
                
                var response = await _httpClient.GetAsync(url);
                
                if (response.IsSuccessStatusCode)
                {
                    var content = await response.Content.ReadAsStringAsync();
                    var result = JsonSerializer.Deserialize<T>(content);
                    
                    _logger.LogInformation("請求成功: {Url}", url);
                    return result;
                }
                
                // HTTP 錯誤狀態碼
                throw new HttpRequestException(
                    $"HTTP 錯誤: {response.StatusCode} - {response.ReasonPhrase}");
            }
            catch (HttpRequestException ex) when (IsRetryableHttpError(ex))
            {
                lastException = ex;
                _logger.LogWarning("HTTP 請求失敗 (第 {Attempt}/{MaxRetries} 次): {Error}", 
                    attempt, maxRetries, ex.Message);
            }
            catch (TaskCanceledException ex) when (ex.CancellationToken.IsCancellationRequested)
            {
                lastException = ex;
                _logger.LogWarning("請求逾時 (第 {Attempt}/{MaxRetries} 次): {Url}", 
                    attempt, maxRetries, url);
            }
            catch (JsonException ex)
            {
                // JSON 解析錯誤通常不需要重試
                _logger.LogError("JSON 解析失敗: {Error}", ex.Message);
                throw new InvalidOperationException("回應格式錯誤", ex);
            }
            catch (Exception ex)
            {
                // 其他錯誤記錄但重新拋出
                _logger.LogError(ex, "請求過程中發生未預期錯誤: {Url}", url);
                throw;
            }
            
            // 如果還有重試機會，等待後再試
            if (attempt < maxRetries)
            {
                var delay = CalculateDelay(attempt);
                _logger.LogInformation("等待 {Delay}ms 後重試...", delay);
                await Task.Delay(delay);
            }
        }
        
        // 所有重試都失敗了
        _logger.LogError("請求最終失敗，已達最大重試次數: {Url}", url);
        throw new InvalidOperationException(
            $"請求 {url} 失敗，已重試 {maxRetries} 次", lastException);
    }
    
    private static bool IsRetryableHttpError(HttpRequestException ex)
    {
        // 判斷是否為可重試的 HTTP 錯誤
        var message = ex.Message.ToLower();
        return message.Contains("timeout") || 
               message.Contains("502") || 
               message.Contains("503") || 
               message.Contains("504");
    }
    
    private static int CalculateDelay(int attempt)
    {
        // 指數退避演算法
        return BaseDelayMs * (int)Math.Pow(2, attempt - 1);
    }
}
```

### 🔧 **資料庫操作範例**

**資料庫事務與例外處理：**
```csharp
public class UserRepository
{
    private readonly IDbConnection _connection;
    private readonly ILogger<UserRepository> _logger;
    
    public UserRepository(IDbConnection connection, ILogger<UserRepository> logger)
    {
        _connection = connection;
        _logger = logger;
    }
    
    public async Task<bool> CreateUserWithProfileAsync(User user, UserProfile profile)
    {
        // 確保連線開啟
        if (_connection.State != ConnectionState.Open)
            await _connection.OpenAsync();
        
        using var transaction = await _connection.BeginTransactionAsync();
        
        try
        {
            // 驗證輸入
            ValidateUser(user);
            ValidateUserProfile(profile);
            
            // 插入使用者
            var userId = await InsertUserAsync(user, transaction);
            profile.UserId = userId;
            
            // 插入使用者檔案
            await InsertUserProfileAsync(profile, transaction);
            
            // 提交事務
            await transaction.CommitAsync();
            
            _logger.LogInformation("成功建立使用者和檔案: {UserId}", userId);
            return true;
        }
        catch (ArgumentException ex)
        {
            // 輸入驗證錯誤 - 回滾事務
            await transaction.RollbackAsync();
            _logger.LogWarning("使用者資料驗證失敗: {Error}", ex.Message);
            throw; // 重新拋出，讓呼叫者處理
        }
        catch (SqlException ex) when (ex.Number == 2627) // 主鍵衝突
        {
            await transaction.RollbackAsync();
            _logger.LogWarning("使用者已存在: {Email}", user.Email);
            throw new InvalidOperationException($"使用者 {user.Email} 已存在", ex);
        }
        catch (SqlException ex) when (ex.Number == -2) // 逾時
        {
            await transaction.RollbackAsync();
            _logger.LogError("資料庫操作逾時: {Error}", ex.Message);
            throw new TimeoutException("資料庫操作逾時，請稍後再試", ex);
        }
        catch (Exception ex)
        {
            // 其他錯誤 - 確保回滾事務
            try
            {
                await transaction.RollbackAsync();
                _logger.LogError(ex, "建立使用者時發生錯誤，已回滾事務");
            }
            catch (Exception rollbackEx)
            {
                _logger.LogError(rollbackEx, "回滾事務時發生錯誤");
                // 將原始例外和回滾例外都包含進去
                throw new InvalidOperationException(
                    "資料庫操作失敗且無法回滾事務", 
                    new AggregateException(ex, rollbackEx));
            }
            
            throw new InvalidOperationException("建立使用者失敗", ex);
        }
    }
    
    private void ValidateUser(User user)
    {
        if (user == null)
            throw new ArgumentNullException(nameof(user));
        
        if (string.IsNullOrWhiteSpace(user.Email))
            throw new ArgumentException("電子郵件不能為空", nameof(user.Email));
        
        if (!IsValidEmail(user.Email))
            throw new ArgumentException("電子郵件格式不正確", nameof(user.Email));
    }
    
    private void ValidateUserProfile(UserProfile profile)
    {
        if (profile == null)
            throw new ArgumentNullException(nameof(profile));
        
        if (string.IsNullOrWhiteSpace(profile.DisplayName))
            throw new ArgumentException("顯示名稱不能為空", nameof(profile.DisplayName));
    }
    
    private bool IsValidEmail(string email)
    {
        try
        {
            var addr = new MailAddress(email);
            return addr.Address == email;
        }
        catch
        {
            return false;
        }
    }
}
```

### ⚡ **非同步例外處理**

**Task 和 async/await 的例外處理：**
```csharp
public class AsyncOperationService
{
    private readonly ILogger<AsyncOperationService> _logger;
    
    public AsyncOperationService(ILogger<AsyncOperationService> logger)
    {
        _logger = logger;
    }
    
    public async Task<List<T>> ProcessMultipleAsync<T>(
        IEnumerable<string> urls, 
        Func<string, Task<T>> processor,
        int maxConcurrency = 5)
    {
        var results = new ConcurrentBag<T>();
        var exceptions = new ConcurrentBag<Exception>();
        
        try
        {
            // 使用 SemaphoreSlim 控制並行度
            using var semaphore = new SemaphoreSlim(maxConcurrency);
            
            var tasks = urls.Select(async url =>
            {
                await semaphore.WaitAsync();
                try
                {
                    var result = await processor(url);
                    results.Add(result);
                    _logger.LogDebug("成功處理: {Url}", url);
                }
                catch (Exception ex)
                {
                    exceptions.Add(ex);
                    _logger.LogWarning(ex, "處理失敗: {Url}", url);
                }
                finally
                {
                    semaphore.Release();
                }
            });
            
            await Task.WhenAll(tasks);
            
            // 檢查是否有例外發生
            if (exceptions.Any())
            {
                _logger.LogWarning("批次處理完成，但有 {Count} 個項目失敗", exceptions.Count);
                
                // 如果失敗率太高，拋出聚合例外
                var failureRate = (double)exceptions.Count / urls.Count();
                if (failureRate > 0.5) // 超過 50% 失敗
                {
                    throw new AggregateException(
                        "批次處理失敗率過高", exceptions);
                }
            }
            
            return results.ToList();
        }
        catch (AggregateException)
        {
            throw; // 重新拋出聚合例外
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "批次處理過程中發生未預期錯誤");
            throw new InvalidOperationException("批次處理失敗", ex);
        }
    }
    
    // 使用 CancellationToken 的例外處理
    public async Task<T> ProcessWithTimeoutAsync<T>(
        Func<CancellationToken, Task<T>> operation,
        TimeSpan timeout)
    {
        using var cts = new CancellationTokenSource(timeout);
        
        try
        {
            return await operation(cts.Token);
        }
        catch (OperationCanceledException) when (cts.Token.IsCancellationRequested)
        {
            _logger.LogWarning("操作逾時: {Timeout}", timeout);
            throw new TimeoutException($"操作在 {timeout} 內未完成");
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "操作執行過程中發生錯誤");
            throw;
        }
    }
}
```

### 🎯 **實際應用場景**

**場景一：API 服務的統一例外處理：**
```csharp
public class ApiExceptionMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ApiExceptionMiddleware> _logger;
    
    public ApiExceptionMiddleware(RequestDelegate next, ILogger<ApiExceptionMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            await HandleExceptionAsync(context, ex);
        }
    }
    
    private async Task HandleExceptionAsync(HttpContext context, Exception exception)
    {
        var response = context.Response;
        response.ContentType = "application/json";
        
        var errorResponse = new ErrorResponse();
        
        switch (exception)
        {
            case ArgumentException ex:
                response.StatusCode = 400;
                errorResponse.Message = "請求參數錯誤";
                errorResponse.Details = ex.Message;
                _logger.LogWarning(ex, "參數驗證失敗: {Path}", context.Request.Path);
                break;
                
            case UnauthorizedAccessException ex:
                response.StatusCode = 401;
                errorResponse.Message = "未授權存取";
                _logger.LogWarning(ex, "未授權存取: {Path}", context.Request.Path);
                break;
                
            case TimeoutException ex:
                response.StatusCode = 408;
                errorResponse.Message = "請求逾時";
                errorResponse.Details = ex.Message;
                _logger.LogWarning(ex, "請求逾時: {Path}", context.Request.Path);
                break;
                
            case InvalidOperationException ex:
                response.StatusCode = 422;
                errorResponse.Message = "業務邏輯錯誤";
                errorResponse.Details = ex.Message;
                _logger.LogWarning(ex, "業務邏輯錯誤: {Path}", context.Request.Path);
                break;
                
            default:
                response.StatusCode = 500;
                errorResponse.Message = "內部服務錯誤";
                _logger.LogError(exception, "未處理的例外: {Path}", context.Request.Path);
                break;
        }
        
        errorResponse.Timestamp = DateTime.UtcNow;
        errorResponse.Path = context.Request.Path;
        
        var jsonResponse = JsonSerializer.Serialize(errorResponse);
        await response.WriteAsync(jsonResponse);
    }
}

public class ErrorResponse
{
    public string Message { get; set; }
    public string Details { get; set; }
    public DateTime Timestamp { get; set; }
    public string Path { get; set; }
}
```

### 🔍 **進階技巧：自訂例外類別**

**業務邏輯專用例外類別：**
```csharp
public abstract class BusinessException : Exception
{
    public string ErrorCode { get; }
    public int HttpStatusCode { get; }
    
    protected BusinessException(
        string errorCode, 
        string message, 
        int httpStatusCode = 400) : base(message)
    {
        ErrorCode = errorCode;
        HttpStatusCode = httpStatusCode;
    }
    
    protected BusinessException(
        string errorCode, 
        string message, 
        Exception innerException, 
        int httpStatusCode = 400) : base(message, innerException)
    {
        ErrorCode = errorCode;
        HttpStatusCode = httpStatusCode;
    }
}

public class InsufficientBalanceException : BusinessException
{
    public decimal CurrentBalance { get; }
    public decimal RequiredAmount { get; }
    
    public InsufficientBalanceException(decimal currentBalance, decimal requiredAmount)
        : base("INSUFFICIENT_BALANCE", 
               $"餘額不足。目前餘額: {currentBalance:C}, 需要金額: {requiredAmount:C}", 
               422)
    {
        CurrentBalance = currentBalance;
        RequiredAmount = requiredAmount;
    }
}

public class ProductNotAvailableException : BusinessException
{
    public string ProductId { get; }
    public int RequestedQuantity { get; }
    public int AvailableQuantity { get; }
    
    public ProductNotAvailableException(
        string productId, 
        int requestedQuantity, 
        int availableQuantity)
        : base("PRODUCT_NOT_AVAILABLE", 
               $"商品 {productId} 庫存不足。需要: {requestedQuantity}, 可用: {availableQuantity}", 
               422)
    {
        ProductId = productId;
        RequestedQuantity = requestedQuantity;
        AvailableQuantity = availableQuantity;
    }
}

// 使用自訂例外的服務
public class OrderService
{
    public async Task ProcessOrderAsync(Order order)
    {
        try
        {
            await ValidateOrderAsync(order);
            await ProcessPaymentAsync(order);
            await UpdateInventoryAsync(order);
            await SendConfirmationAsync(order);
        }
        catch (InsufficientBalanceException ex)
        {
            // 餘額不足的特殊處理
            _logger.LogWarning("訂單 {OrderId} 餘額不足: {Details}", 
                order.Id, ex.Message);
            
            await NotifyInsufficientBalanceAsync(order, ex);
            throw; // 重新拋出讓 API 層處理
        }
        catch (ProductNotAvailableException ex)
        {
            // 庫存不足的特殊處理
            _logger.LogWarning("訂單 {OrderId} 庫存不足: {Details}", 
                order.Id, ex.Message);
            
            await SuggestAlternativeProductsAsync(order, ex);
            throw;
        }
        catch (BusinessException ex)
        {
            // 其他業務邏輯例外
            _logger.LogWarning("訂單 {OrderId} 業務邏輯錯誤: {ErrorCode} - {Message}", 
                order.Id, ex.ErrorCode, ex.Message);
            throw;
        }
        catch (Exception ex)
        {
            // 系統層級錯誤
            _logger.LogError(ex, "處理訂單 {OrderId} 時發生系統錯誤", order.Id);
            throw new InvalidOperationException("訂單處理失敗", ex);
        }
    }
}
```

### 🎯 **最佳實踐指南**

| 情況 | 建議做法 | 範例 |
|------|----------|------|
| **預期的業務例外** | 使用 try-catch 處理並提供備用邏輯 | 檔案不存在時返回空字串 |
| **系統層級錯誤** | 記錄詳細資訊後重新拋出或包裝 | 資料庫連線失敗 |
| **輸入驗證錯誤** | 拋出 ArgumentException 系列 | 參數為 null 或格式錯誤 |
| **非同步操作** | 使用 CancellationToken 和逾時控制 | HTTP 請求、資料庫查詢 |
| **資源清理** | 使用 using 語句或 finally 區塊 | 檔案操作、資料庫連線 |

### 🎯 **效能考量**

**避免例外處理的效能陷阱：**
```csharp
// ❌ 錯誤：使用例外控制正常流程
public bool IsValidNumber(string input)
{
    try
    {
        int.Parse(input);
        return true;
    }
    catch
    {
        return false;
    }
}

// ✅ 正確：使用 TryParse 避免例外
public bool IsValidNumber(string input)
{
    return int.TryParse(input, out _);
}

// ❌ 錯誤：頻繁的例外處理
public void ProcessLargeDataSet(IEnumerable<string> data)
{
    foreach (var item in data)
    {
        try
        {
            ProcessItem(item);
        }
        catch (Exception ex)
        {
            // 每個項目都可能拋出例外
            LogError(ex);
        }
    }
}

// ✅ 正確：預先驗證減少例外
public void ProcessLargeDataSet(IEnumerable<string> data)
{
    var validItems = data.Where(IsValidItem).ToList();
    var invalidItems = data.Except(validItems).ToList();
    
    // 批次記錄無效項目
    if (invalidItems.Any())
    {
        LogInvalidItems(invalidItems);
    }
    
    // 處理有效項目，減少例外發生
    foreach (var item in validItems)
    {
        try
        {
            ProcessItem(item);
        }
        catch (Exception ex)
        {
            LogError(ex, item);
        }
    }
}
```

## 5. 方法直接寫在 建立的 entity 裡面

## 6. 關於建構子


### 6.1 string 不要放在建構子注入的位置

#### 🚫 **錯誤做法**

**問題範例：**
```csharp
// ❌ 錯誤：直接注入 string 參數
public class EmailService : IEmailService
{
    private readonly IEmailSender _emailSender;
    private readonly string _smtpServer;
    private readonly string _fromAddress;
    private readonly string _apiKey;
    
    public EmailService(
        IEmailSender emailSender,
        string smtpServer,        // 問題：string 參數難以區分
        string fromAddress,       // 問題：注入時容易搞混
        string apiKey)           // 問題：缺乏型別安全性
    {
        _emailSender = emailSender;
        _smtpServer = smtpServer;
        _fromAddress = fromAddress;
        _apiKey = apiKey;
    }
    
    public async Task SendEmailAsync(string to, string subject, string body)
    {
        // 使用配置傳送郵件
        await _emailSender.SendAsync(_smtpServer, _fromAddress, to, subject, body, _apiKey);
    }
}